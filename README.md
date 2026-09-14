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

## 📋 Progresso

| Data | Etapa | Status |
|------|-------|--------|
| 04/09/2026 | Setup da VM base de estudo (Ubuntu) | ✅ Concluído |
| 05/09/2026 | Configuração inicial da rede interna do lab | ✅ Concluído |
| 06/09/2026 | Configuração da VM Wazuh (Ubuntu) | ✅ Concluído |
| 09/09/2026 | Configuração da VM alvo | ✅ Concluído |
| 11/09/2026 | Integração do Sysmon e validação do pipeline de telemetria | ✅ Concluído |
| 14/09/2026 | Simulação de mitre att&ck T1059.001 | ✅ Concluído |

## 📚 Documentação detalhada

- [Setup do Ambiente — VM Base](docs/01-setup-ambiente.md)
    - [Setup do Ambiente — VM Alvo](docs/01.01-setup-alvo.md)
- [Configuração de Rede](docs/02-configuracao-rede.md)
- [Configuração da VM Wazuh](docs/03-wazuh-manager.md)
    - [Integração do Sysmon](docs/03.01-integracao-do-sysmon.md)
- [Simulação de mitre att&ck T1059.001](docs/04.simulacao-de-ataque-T1059.001.md)
