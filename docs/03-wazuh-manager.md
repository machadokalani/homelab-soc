# Wazuh Manager — Instalação e Primeiro Acesso

## Objetivo
Com a VM base já validada, o próximo passo foi montar uma VM dedicada
exclusivamente à infraestrutura de monitoramento: o **Wazuh manager**
(indexer + server + dashboard, em instalação "all-in-one"). Diferente
da VM de estudo, essa VM tem propósito de produção do próprio lab —
vai receber e correlacionar os logs da futura máquina alvo (Windows).
Optei por mantê-la separada da VM de estudo para isolar
responsabilidades e reproduzir a arquitetura real de um SOC (agente
no endpoint, manager centralizado).

## Ambiente

Sistema operacional da VM:
- Ubuntu 26.04 LTS "Resonant Rhino" (64-bit)
- Optei pela versão LTS (em vez de uma intermediária como a 25.04 usada
  na VM base) por ser uma peça de infraestrutura do lab, não de estudo
  exploratório — LTS prioriza estabilidade e ciclo de suporte mais
  longo (5 anos)

Recursos alocados à VM:

| Recurso | Valor |
|---|---|
| RAM | 6144 MB |
| CPUs | 4 |
| Disco | 50 GB (dinâmico, LVM) |

> Requisito oficial do Wazuh (quickstart, 1–25 agentes): 4 vCPU / 8 GiB
> RAM. Aloquei 6GB por restrição de RAM total do host (16GB),
> compensando com atenção ao consumo do OpenSearch (indexer) caso
> necessário.

## Processo

**Instalação do Wazuh (modo all-in-one)**

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

A flag `-a` instala indexer, server e dashboard juntos, na mesma
máquina — adequado para o volume de agentes deste lab (1,
inicialmente). Ao final, o instalador gera as credenciais de acesso
(usuário `admin` + senha aleatória).

**6. Acesso ao dashboard**

Tentativa inicial de acessar `https://10.0.2.15` diretamente do
navegador do host falhou. Causa: o modo NAT padrão do VirtualBox
isola a VM — ela consegue sair para a internet, mas o host não
consegue "entrar" nela usando esse IP interno, sem uma regra
explícita.

**Solução — Port Forwarding:** em Configurações da VM → Rede →
Adaptador 1 → Avançado → Encaminhamento de Portas, criei a regra:

| Nome | Protocolo | Porta Host | IP Convidado | Porta Convidado |
|---|---|---|---|---|
| wazuh-dashboard | TCP | 8443 | 10.0.2.15 | 443 |

Com isso, `https://localhost:8443` no host passou a rotear
corretamente para o dashboard na VM.

##  Falso positivo identificado (rootcheck)

Logo após a instalação, o próprio Wazuh manager (que se automonitora
como agente implícito, `agent.id: 000`) gerou dezenas de alertas de
severidade média (`rule.level: 7`) com a descrição:

Trojaned version of file detected

para binários como `/bin/md5sum` e `/usr/bin/md5sum`.

Trata-se de um falso positivo conhecido do módulo **rootcheck** em
distribuições baseadas em Debian/Ubuntu — a verificação de
assinaturas genéricas de rootkit por vezes coincide com padrões de
binários legítimos do sistema.

##  Próximos Passos
- Ajustar a regra do rootcheck para suprimir o falso positivo
  identificado (ou documentar a supressão como exercício)
- Adicionar um segundo adaptador de Rede Interna (`soclab`) na
  `wazuh-manager`, com IP estático seguinte na faixa `192.168.56.x`,
  replicando a configuração da VM base
- Montar a VM Windows 10 (máquina alvo)
- Instalar e enrolar o agente Wazuh na máquina alvo
- Validar o fluxo completo: evento gerado no Windows → visível no
  dashboard do Wazuh
