# Homelab SOC — Detection engineering

##  Objetivo

Projeto de laboratório pessoal para praticar conceitos de 
cibersegurança na prática, como parte da minha transição de carreira para analista de cibersegurança 
(SOC/Blue Team).

## Arquitetura

**Estado atual: v2** — lab sendo reconstruído do zero em KVM/libvirt sobre host Linux.

- **Host:** Ubuntu, com KVM/QEMU gerenciado por libvirt e virt-manager
- **Rede do lab:** rede libvirt `soclab` (NAT, `10.10.10.0/24`), definida em XML versionado ([`infra/libvirt/soclab.xml`](infra/libvirt/soclab.xml))
- **VM Kali Linux** — criada
- **Wazuh Manager** — em reconstrução
- **VM Alvo (Windows 10)** — em reconstrução

Por que a troca de hypervisor: ver o [ADR 00 — Migração VirtualBox → KVM](docs/v2/00-adr-migracao-virtualbox-kvm.md).

**Versões anteriores:** a v1 (VirtualBox sobre host Windows 11, com VM base Ubuntu, Wazuh Manager e Windows 10 alvo) está preservada na tag [`v1.0`](https://github.com/machadokalani/homelab-soc/tree/v1.0) e documentada em [`docs/v1/`](docs/v1/).

##  Stack utilizada

**v2 (atual)**
- KVM/QEMU + libvirt 12.0.0 + virt-manager (hypervisor)
- Ubuntu 26.04.1 LTS (host)
- Kali Linux (VM)
- Wazuh (SIEM) + Sysmon — em reconstrução
- Windows 10 (64-bit) (VM alvo) — em reconstrução

**v1 (histórico)**
- VirtualBox (hypervisor), host Windows 11
- Ubuntu 25.04 "Plucky Puffin" (VM base)
- Ubuntu Server 26.04 LTS "Resolute Raccoon" (64-bit) (Wazuh)
- Windows 10 (64-bit) (VM alvo)

## 📁 Documentação detalhada

### v2 — KVM/libvirt (atual)
- [ADR 00 — Migração do hypervisor: VirtualBox → KVM/libvirt](docs/v2/00-adr-migracao-virtualbox-kvm.md)
- [Rede do lab — rede libvirt `soclab`](docs/v2/01-rede-libvirt.md)

### v1 — VirtualBox (histórico)
Documentação preservada como registro da primeira versão do lab. Os passos específicos de VirtualBox não se aplicam à v2.

#### Setup e Infraestrutura
- [Setup do Ambiente — VM Base](docs/v1/01-setup-ambiente.md)
  - [Setup do Ambiente — VM Alvo](docs/v1/01.01-setup-alvo.md)
- [Configuração de Rede](docs/v1/02-configuracao-rede.md)
- [Configuração da VM Wazuh](docs/v1/03-wazuh-manager.md)
  - [Integração do Sysmon](docs/v1/03.01-integracao-do-sysmon.md)
- [Tuning de Falso Positivo - Rootcheck](docs/v1/04-tuning-de-falso-positivo-rootcheck.md)

#### Simulações de Ataque (Atomic Red Team)
Testes práticos de técnicas MITRE ATT&CK, validando a resposta do pipeline de detecção ponta a ponta.
- [T1059.001 — PowerShell Encoded Command](docs/v1/atomic-red-team/01-t1059.001-encoded-command.md)
- [T1136.001 — Criação de Conta e Escalonamento de Privilégio](docs/v1/atomic-red-team/02-t1136.001-privilege-escalation.md)
- [T1685.005 — Simulação de Limpeza de Log de Auditoria](docs/v1/atomic-red-team/03-t1685.005-clear-event-logs.md)

### Playbooks de Triagem
Independentes de hypervisor, valem para as duas versões. Processos reutilizáveis de decisão para triagem de alertas — diferente dos docs acima, que registram investigações pontuais, estes documentam o raciocínio de análise aplicável a qualquer alerta futuro do mesmo tipo.
- [Execução suspeita de windows e powershell](docs/playbooks/01-execucao-suspeita-windows-powershell.md)
- [Criação de Conta e Escalonamento de Privilégio](docs/playbooks/02-criacao-conta-escalonamento.md)


## 📋 Progresso

| Data | Etapa | Status |
|------|-------|--------|
| 04/09/2026 | Setup da VM base de estudo (Ubuntu) | ✅ Concluído |
| 05/09/2026 | Configuração inicial da rede interna do lab | ✅ Concluído |
| 06/09/2026 | Configuração da VM Wazuh (Ubuntu) | ✅ Concluído |
| 09/09/2026 | Configuração da VM alvo | ✅ Concluído |
| 11/09/2026 | Integração do Sysmon e validação do pipeline de telemetria | ✅ Concluído |
| 14/09/2026 | Simulação de mitre att&ck T1059.001 | ✅ Concluído |
| 14/09/2026 | Simulação de mitre att&ck T1136.001 | ✅ Concluído |
| 14/09/2026 | Playbook do mitre att&ck T1136.001 | ✅ Concluído |
| 15/09/2026 | Playbook do mitre att&ck T1059.001 | ✅ Concluído |
| 15/09/2026 | Tuning de Falso Positivo - Rootcheck| ✅ Concluído |
| 17/09/2026 | Simulação de mitre att&ck T1685.005 (Clear Event Logs) | ✅ Concluído |
| 25/09/2026 | Migração para KVM/libvirt (host Ubuntu) e início da v2: rede `soclab` e VM Kali | 🔄 Em andamento |
