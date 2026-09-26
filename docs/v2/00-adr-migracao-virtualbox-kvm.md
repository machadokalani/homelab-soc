# ADR 00 — Migração do Hypervisor: VirtualBox → KVM/QEMU com libvirt

- **Status:** Aceito
- **Data do registro:** 25/09/2026
- **Escopo:** infraestrutura de virtualização do lab (v2) e organização do repositório

## Contexto

A v1 do laboratório rodava em **VirtualBox sobre um host Windows 11**. Nela foram montadas a VM base de estudos (Ubuntu), o Wazuh manager (Ubuntu Server) e a VM alvo (Windows 10), com o pipeline Sysmon → Wazuh validado e três simulações de ataque documentadas ([docs/v1/](../v1/)).

O PC foi formatado e **o host passou a ser Ubuntu**. Com isso, o lab precisa ser reconstruído do zero: não existem mais VMs a migrar, só a experiência acumulada e a documentação da v1. Essa reconstrução abriu a pergunta que não fazia sentido antes: **qual hypervisor usar num host Linux?**

Duas restrições do host pesam na decisão:

- **CPU híbrida** (Intel i5 de 13ª geração, com P-cores e E-cores). Na v1 isso causou travamentos da VM Windows durante a instalação, contornados trocando o provider de paravirtualização do VirtualBox para KVM (`--paravirtprovider kvm`); ver [Setup da VM Alvo (v1)](../v1/01.01-setup-alvo.md).
- **RAM limitada** (16 GB para o host e todas as VMs). Na v1 o Wazuh manager ficou com 6 GB, abaixo dos 8 GB recomendados, justamente por isso; ver [Wazuh Manager (v1)](../v1/03-wazuh-manager.md).

Havia também uma segunda decisão, sobre o repositório: criar um repositório novo para a v2 ou manter a v2 no mesmo repositório. O objetivo do repositório é servir de portfólio, e **a evolução do projeto (o que mudou, por que mudou) é parte do que ele demonstra**.

## Decisão

1. **Adotar KVM/QEMU, gerenciado por libvirt (virsh) e virt-manager**, como hypervisor da v2.
2. **Manter a v2 no mesmo repositório**, preservando a v1:
   - tag anotada `v1.0` marcando o último estado da v1;
   - documentação da v1 movida com `git mv` para `docs/v1/`, mantendo o histórico de cada arquivo;
   - documentação da v2 em `docs/v2/`;
   - playbooks de triagem em `docs/playbooks/`, fora das versões, por não dependerem de hypervisor;
   - definições de infraestrutura versionadas em `infra/` (ex.: a rede libvirt em `infra/libvirt/soclab.xml`).

## Motivos

**1. KVM é nativo do kernel Linux.**
Os módulos `kvm` e `kvm_intel` fazem parte do kernel e chegam assinados pela própria distribuição. O VirtualBox depende de módulos externos (`vboxdrv` e afins), compilados via DKMS a cada atualização de kernel e que, com **Secure Boot** ativo, precisam ser assinados com uma chave própria (MOK) registrada no firmware. Com KVM essa manutenção não existe: menos uma peça que pode quebrar depois de um `apt upgrade`.

**2. Melhor tratamento da CPU híbrida.**
Na v1, o workaround para os travamentos foi justamente fazer o VirtualBox **imitar a interface de paravirtualização do KVM**. Com KVM, essa interface é o modo nativo, sem camada de tradução. Além disso, cada vCPU vira uma thread comum do host, agendada pelo scheduler do Linux, e o libvirt permite fixar vCPUs em núcleos específicos (`<cputune>` / `vcpupin`) caso o problema de agendamento entre P-cores e E-cores reapareça. *Hipótese a validar quando a VM Windows 10 for recriada.*

**3. KSM e balloon driver ajudam com a RAM limitada.**
- **KSM (Kernel Samepage Merging):** o kernel identifica páginas de memória idênticas entre VMs e as mantém uma única vez. Útil com várias VMs rodando juntas.
- **Balloon driver (virtio-balloon):** permite que o host recupere memória não usada de uma VM sem desligá-la.

Nenhum dos dois cria RAM do nada, mas ambos dão margem numa máquina de 16 GB.

**4. Relevância no mercado.**
libvirt/KVM é a base do **Proxmox VE** e do **OpenStack**, além de ser o padrão de virtualização em servidores Linux. Aprender a operar essa stack (virsh, redes libvirt, discos qcow2) é conhecimento transferível para ambientes reais, diferente de um hypervisor de desktop.

**5. VirtualBox e KVM não devem coexistir no mesmo host.**
Os dois disputam a mesma extensão de virtualização por hardware (VT-x). É o mesmo tipo de conflito já visto na v1 entre VirtualBox e Hyper-V/VBS no Windows. Manter os dois exigiria descarregar módulos alternadamente. Como o lab está sendo reconstruído, escolher um só é a opção limpa.

## Alternativas consideradas

**Manter o VirtualBox (agora no host Linux).**
A principal vantagem seria continuidade: mesma ferramenta, mesmos comandos, mesma documentação. Com a **reconstrução do zero**, essa vantagem perdeu o sentido, porque não há VMs a aproveitar e a infraestrutura seria refeita de qualquer forma. Continuariam os custos do motivo 1 (módulos externos + Secure Boot) e a dependência do workaround da CPU híbrida. **Rejeitada.**

**Criar um repositório novo para a v2.**
Deixaria a v2 "limpa", mas separaria a história do projeto em dois lugares e esconderia justamente a evolução que o portfólio quer mostrar. **Rejeitada.** Com a tag `v1.0` e a pasta `docs/v1/`, a v1 continua acessível sem se misturar com o estado atual.

## Consequências

**Positivas**
- Hypervisor integrado ao kernel, sem módulos externos para manter e assinar.
- Mais ferramentas para lidar com CPU híbrida e RAM limitada (pinning de vCPU, KSM, balloon).
- Infraestrutura descrita em XML e versionada no repositório (`infra/libvirt/`): a rede do lab pode ser recriada com `virsh net-define`, não só com cliques em interface gráfica.
- O histórico do repositório mostra a evolução v1 → v2 e o motivo da mudança.

**Negativas**
- **Curva de aprendizado maior:** virsh, XML de domínio e de rede, pools de armazenamento e qcow2 são mais verbosos que a interface do VirtualBox.
- **Drivers virtio na VM Windows:** para usar disco e rede paravirtualizados (virtio), a instalação do Windows 10 precisa dos drivers virtio-win (não vêm no Windows), carregados durante a instalação para o disco ser reconhecido.
- **Tutoriais assumem VirtualBox:** a maior parte do material de homelab SOC mostra VirtualBox. Cada passo vai exigir tradução de conceito (ex.: rede Host-Only → rede libvirt; `VBoxManage` → `virsh`; snapshots do VirtualBox → snapshots do libvirt/qcow2).
- **KSM tem custo:** a varredura de páginas consome CPU do host e, por compartilhar memória entre VMs, abre espaço para ataques de canal lateral entre VMs. Num lab isolado é aceitável, mas é uma troca a registrar, não uma vantagem gratuita.
- **Os docs da v1 com passos específicos de VirtualBox** (paravirtprovider, port forwarding, Host-Only) passam a ser histórico e não servem como passo a passo da v2.

## Referências internas

- [Rede libvirt da v2](01-rede-libvirt.md)
- [Documentação da v1](../v1/)
- Tag `v1.0`: último estado do lab em VirtualBox
