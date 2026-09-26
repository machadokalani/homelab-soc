# Homelab SOC — Detection engineering

##  Objetivo

Projeto de laboratório pessoal para praticar conceitos de 
cibersegurança na prática, como parte da minha transição de carreira para analista de cibersegurança 
(SOC/Blue Team).

## Arquitetura

Estágio atual: 
- VM Base de estudos (UBUNTU)
- Wazuh Manager (UBUNTU SERVER)
- VM Alvo (WINDOWS 10)

##  Stack utilizada

- VirtualBox (hypervisor)
- Ubuntu 25.04 "Plucky Puffin" (VM base)
- Ubuntu Server 26.04 LTS "Resonant Rhino" (64-bit) (Wazuh)
- Windows 10 (64-bit) (VM alvo)

## 📁 Documentação detalhada

### Setup e Infraestrutura
- [Setup do Ambiente — VM Base](docs/v1/01-setup-ambiente.md)
  - [Setup do Ambiente — VM Alvo](docs/v1/01.01-setup-alvo.md)
- [Configuração de Rede](docs/v1/02-configuracao-rede.md)
- [Configuração da VM Wazuh](docs/v1/03-wazuh-manager.md)
  - [Integração do Sysmon](docs/v1/03.01-integracao-do-sysmon.md)
- [Tuning de Falso Positivo - Rootcheck](docs/v1/04-tuning-de-falso-positivo-rootcheck.md)

### Simulações de Ataque (Atomic Red Team)
Testes práticos de técnicas MITRE ATT&CK, validando a resposta do pipeline de detecção ponta a ponta.
- [T1059.001 — PowerShell Encoded Command](docs/v1/atomic-red-team/01-t1059.001-encoded-command.md)
- [T1136.001 — Criação de Conta e Escalonamento de Privilégio](docs/v1/atomic-red-team/02-t1136.001-privilege-escalation.md)
- [T1685.005 — Simulação de Limpeza de Log de Auditoria](docs/v1/atomic-red-team/03-t1685.005-clear-event-logs.md)

### Playbooks de Triagem
Processos reutilizáveis de decisão para triagem de alertas — diferente dos docs acima, que registram investigações pontuais, estes documentam o raciocínio de análise aplicável a qualquer alerta futuro do mesmo tipo.
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

