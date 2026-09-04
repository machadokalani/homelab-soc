# Setup do Ambiente — VM Base

## Objetivo

Atualmente estou em processo de transição de carreira, saindo de jogador 
profissional de poker para analista de cybersegurança. Este projeto é um 
laboratório de estudo para aprender e aperfeiçoar conceitos práticos de 
segurança da informação.

O primeiro passo foi configurar uma VM com Ubuntu "cru", sem nenhuma 
ferramenta extra, para construir uma base sólida de entendimento do Linux 
antes de avançar para ambientes mais complexos. A partir dessa base, a 
infraestrutura vai crescer gradualmente — próxima etapa envolve configurar 
rede entre VMs e introduzir uma máquina alvo e um SIEM.

## Ambiente

**Máquina host:**
- Notebook Acer Nitro
- Processador Intel Core i5 (13ª geração)
- GPU RTX 4050
- 16GB RAM DDR5

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

## Processo

### 1. Instalação do VirtualBox

Baixei o instalador oficial do VirtualBox (virtualbox.org) e segui a 
instalação padrão.

### 2. Criação da VM

Criei uma nova máquina virtual chamada "ubuntu-base", selecionando tipo 
Linux e versão Ubuntu (64-bit). Aloquei os seguintes recursos:

- RAM: 4096 MB
- CPUs: 2
- Disco: 40 GB (dinâmico)

Optei por não marcar a opção "Use EFI", mantendo o firmware de boot BIOS 
legado — mais simples, mais compatível, e evita problemas comuns de boot 
que podem ocorrer com EFI em VMs. Como não há necessidade de Secure Boot 
neste ambiente, BIOS legado atende bem.

### 3. Instalação do Ubuntu

Na criação da conta de usuário, deixei "Use Active Directory" desmarcado, 
já que essa VM não faz parte de nenhum domínio corporativo. Essa opção 
será relevante futuramente, quando eu configurar um ambiente com Active 
Directory como parte da rede de estudo (em uma VM de servidor separada).

## Lições Aprendidas

- **BIOS vs EFI**: BIOS legado é o firmware de boot mais simples e 
  compatível para VMs comuns; EFI/UEFI só se justifica quando há 
  necessidade de Secure Boot ou discos maiores que 2TB.

- **Escolha do Ubuntu**: optei por Ubuntu em vez de Debian pela ampla 
  documentação, maior compatibilidade com ferramentas de segurança 
  (SIEMs, scripts de detecção) e por ser a distro mais comum em 
  ambientes corporativos e cloud — mesmo sendo levemente mais pesado.

- **Active Directory**: é um serviço de autenticação centralizada usado 
  em domínios corporativos Windows. Não se aplica a uma VM standalone; 
  será relevante ao montar um Domain Controller como parte da rede de 
  estudo, futuramente.

- **`sudo apt update && sudo apt upgrade -y`**: `sudo` eleva privilégios 
  para root; `apt update` atualiza o índice local de pacotes disponíveis 
  (não instala nada); `&&` executa o segundo comando só se o primeiro 
  tiver sucesso; `apt upgrade` instala as atualizações; `-y` confirma 
  automaticamente, sem perguntar.

Por fim, criei um snapshot chamado "instalação limpa" — um ponto de 
restauração que permite reverter a VM ao estado inicial caso alguma 
configuração futura quebre o sistema.

## Próximos Passos

- [x] Atualizar o sistema (`apt update && apt upgrade`)
- [ ] Praticar navegação básica no terminal (cd, ls, chmod, chown, etc.)
- [ ] Estudar permissões e diferença entre root e usuário comum
- [ ] Configurar modo de rede da VM (NAT vs Rede Interna/Host-Only)
- [ ] Criar snapshot de checkpoint após as atualizações
- [ ] Montar segunda VM (máquina alvo) e instalar um SIEM (Wazuh)
