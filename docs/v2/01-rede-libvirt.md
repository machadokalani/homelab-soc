# Rede do Lab na v2 — Rede libvirt `soclab`

## Objetivo

Definir a rede onde as VMs da v2 conversam entre si: Kali, Wazuh manager e Windows 10 alvo. Nesta versão, a rede não é configurada com cliques numa interface gráfica. Ela é **descrita num arquivo XML versionado** ([`infra/libvirt/soclab.xml`](../../infra/libvirt/soclab.xml)) e aplicada com `virsh`.

O porquê: na v1, a rede era uma soma de ajustes manuais (dois adaptadores por VM, IP estático via Netplan dentro de cada guest, port forwarding para chegar ao dashboard do Wazuh). Reconstruir aquilo exigia refazer cada passo de memória. Com a rede em XML, ela pode ser recriada de forma idêntica com um comando, e qualquer mudança fica registrada no histórico do git.

## Ambiente

- **Host:** Ubuntu 26.04.1 LTS
- **Hypervisor:** KVM/QEMU, gerenciado por libvirt 12.0.0 (`libvirtd`) e virt-manager
- **Arquivo de definição:** `infra/libvirt/soclab.xml`

Resumo da rede, conforme o XML:

| Item | Valor |
|---|---|
| Nome da rede | `soclab` |
| Modo | `nat` |
| Bridge no host | `virbr10` (STP ligado, delay 0) |
| Domínio DNS | `soclab` (`localOnly='yes'`) |
| IP do host na rede (gateway) | `10.10.10.1/24` (máscara `255.255.255.0`) |
| Faixa DHCP dinâmica | `10.10.10.100` – `10.10.10.200` |

Reservas de DHCP (IP fixo por MAC):

| VM | MAC | IP |
|---|---|---|
| `kali` | `52:54:00:10:00:10` | `10.10.10.10` |
| `wazuh` | `52:54:00:10:00:20` | `10.10.10.20` |
| `win10` | `52:54:00:10:00:30` | `10.10.10.30` |

## Processo

### 1. Entendendo cada decisão do XML

**Modo NAT (`<forward mode='nat'/>`)**
O libvirt cria a bridge `virbr10` no host, com o IP `10.10.10.1`, e faz NAT do tráfego das VMs para a interface de saída do host. Na prática:
- as VMs acessam a internet (updates, download de ferramentas) através do host;
- o **host alcança as VMs diretamente** pelos IPs `10.10.10.x`, porque tem uma interface na mesma rede;
- dispositivos da rede doméstica **não** alcançam as VMs, porque elas ficam atrás do NAT.

Comparação com a v1: lá foram necessários dois adaptadores por VM (NAT para internet + Host-Only para o lab) e uma regra de port forwarding para abrir o dashboard do Wazuh no host. Aqui, uma única rede cobre os dois papéis.

**Bridge `virbr10` com STP ligado e delay 0**
`virbr10` é o nome da interface de bridge criada no host. Um nome explícito evita conflito com a rede `default` do libvirt, que costuma usar `virbr0`. STP (Spanning Tree Protocol) protege contra loops na bridge, e `delay='0'` faz a porta de uma VM encaminhar tráfego assim que ela liga, sem esperar a fase de aprendizado do STP.

**Domínio `soclab` com `localOnly='yes'`**
O servidor DNS/DHCP da rede (dnsmasq, gerenciado pelo libvirt) responde por nomes do domínio `soclab` e **não repassa** consultas desse domínio para servidores DNS externos. Isso evita que nomes internos do lab vazem para fora.

**DHCP com reservas por MAC, fora da faixa dinâmica**
Em vez de configurar IP estático dentro de cada VM (como na v1, com Netplan), o IP fixo é definido **na própria rede**: quando uma VM com o MAC `52:54:00:10:00:20` pede um endereço, recebe sempre `10.10.10.20`. As reservas (`.10`, `.20`, `.30`) ficam fora da faixa dinâmica (`.100`–`.200`), então uma VM qualquer que entre na rede nunca pega o IP reservado de outra.

Dois detalhes de convenção:
- `52:54:00` é o prefixo de MAC usado por padrão pelo QEMU/KVM.
- o último byte do MAC acompanha o último octeto do IP (`:10` → `.10`, `:20` → `.20`, `:30` → `.30`), o que facilita reconhecer uma VM olhando só para um dos dois.

**Consequência importante:** a reserva só funciona se a VM for criada **com o MAC correspondente** na interface ligada à rede `soclab`. Uma VM criada com MAC aleatório recebe um IP da faixa dinâmica.

### 2. Por que o arquivo versionado não tem `<uuid>` nem `<mac address>`

Quando uma rede é definida, o libvirt gera um UUID para ela e um MAC para a bridge. Esses valores identificam **aquela** instância da rede **naquele** host. Se ficassem no arquivo versionado, reaplicar o XML em outra máquina (ou depois de apagar a rede) reaproveitaria identificadores que deveriam ser únicos. Sem eles, o `virsh net-define` gera valores novos a cada definição. O XML exportado de `virsh net-dumpxml` vai mostrar esses campos; isso é esperado.

### 3. Aplicando a rede

Atenção à conexão do `virsh`: o virt-manager usa `qemu:///system`, mas o `virsh` rodado como usuário comum pode se conectar a `qemu:///session`, onde a rede não apareceria para as VMs do virt-manager. Por isso, os comandos abaixo explicitam a conexão (alternativa: rodar com `sudo`).

A partir da raiz do repositório:

```bash
# 1. Registra a rede no libvirt (persistente, mas ainda desligada)
virsh -c qemu:///system net-define infra/libvirt/soclab.xml

# 2. Liga a rede: cria a bridge virbr10, sobe o dnsmasq e as regras de NAT
virsh -c qemu:///system net-start soclab

# 3. Faz a rede subir sozinha quando o serviço do libvirt iniciar (ex.: após reboot)
virsh -c qemu:///system net-autostart soclab
```

A ordem importa: `net-define` apenas registra a definição, `net-start` a coloca em funcionamento naquele momento e `net-autostart` garante que ela volte depois de um reboot. Sem o terceiro passo, a rede fica definida mas desligada depois de reiniciar o host, e as VMs não sobem com a interface ligada.

### 4. Verificação

```bash
# Rede ativa e com autostart
virsh -c qemu:///system net-list --all

# Bridge criada no host com o IP do gateway
ip addr show virbr10

# Concessões DHCP entregues às VMs
virsh -c qemu:///system net-dhcp-leases soclab
```

Resultado esperado: `soclab` com estado `active` e autostart `yes`; `virbr10` com `10.10.10.1/24`; e cada VM criada com o MAC reservado aparecendo com o IP correspondente.

[PREENCHER: saída real da verificação, especialmente a VM `kali` recebendo `10.10.10.10`]

### 5. Alterando a rede depois

Mudanças devem ser feitas **no arquivo versionado** e reaplicadas, para o repositório continuar sendo a fonte da verdade:

```bash
virsh -c qemu:///system net-destroy soclab     # desliga
virsh -c qemu:///system net-undefine soclab    # remove a definição
virsh -c qemu:///system net-define infra/libvirt/soclab.xml
virsh -c qemu:///system net-start soclab
virsh -c qemu:///system net-autostart soclab
```

Editar direto com `virsh net-edit` funciona, mas deixa o XML do repositório desatualizado.

## Lições Aprendidas

[PREENCHER: o que aconteceu de fato ao criar e aplicar a rede (erros, surpresas, o que precisou ajustar)]

## Próximos Passos

- Registrar a saída real da verificação (seção 4).
- Recriar o Wazuh manager e o Windows 10 alvo com os MACs reservados (`:20` e `:30`).
- Validar se o host alcança o dashboard do Wazuh direto em `10.10.10.20`, sem port forwarding.
