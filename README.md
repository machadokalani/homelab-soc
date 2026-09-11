# Homelab SOC — Da instalação ao detection engineering

##  Objetivo

Projeto de laboratório pessoal para aprender e praticar conceitos de 
cibersegurança na prática, como parte da minha transição de carreira 
de jogador profissional de poker para analista de cibersegurança 
(SOC/Blue Team).

A ideia é documentar cada etapa — desde a configuração inicial do 
ambiente até, eventualmente, um ambiente completo com SIEM, máquina 
alvo e geração/análise de logs reais.

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
| 09/09/2026 | Setup da VM alvo | Em progresso |

## 📚 Documentação detalhada

- [Setup do Ambiente — VM Base](docs/01-setup-ambiente.md)
    - [Setup do Ambiente — VM Alvo](docs/01.01-setup-alvo.md)
- [Configuração de Rede](docs/02-configuracao-rede.md)
- [Configuração da VM Wazuh](docs/03-wazuh-manager.md)
