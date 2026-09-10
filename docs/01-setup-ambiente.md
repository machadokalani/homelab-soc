# Setup do Ambiente — VM Base

## Objetivo

O primeiro passo foi configurar uma VM com Ubuntu "cru", sem nenhuma 
ferramenta extra, para construir uma base sólida de entendimento do Linux 
antes de avançar para ambientes mais complexos. A partir dessa base, a 
infraestrutura vai crescer gradualmente — próxima etapa envolve configurar 
rede entre VMs e introduzir uma máquina alvo e um SIEM.

## Ambiente

**Hypervisor:**
- VirtualBox

**Sistema operacional da VM:**
- Ubuntu 25.04 "Plucky Puffin" (64-bit)
- Escolhido pela documentação/comunidade ampla, compatibilidade com 
  ferramentas de segurança (SIEM, scripts de detecção) e por ser a 
  distro mais usada em ambientes corporativos e cloud

**Recursos alocados à VM:**
- RAM: 4096 MB
- CPUs: 2
- Disco: 40 GB (dinâmico)

## Lições Aprendidas

- **Escolha do Ubuntu**: optei por Ubuntu em vez de Debian pela ampla 
  documentação, maior compatibilidade com ferramentas de segurança 
  (SIEMs, scripts de detecção) e por ser a distro mais comum em 
  ambientes corporativos e cloud — mesmo sendo levemente mais pesado.


Por fim, criei um snapshot chamado "instalação limpa" — um ponto de 
restauração que permite reverter a VM ao estado inicial caso alguma 
configuração futura quebre o sistema.

