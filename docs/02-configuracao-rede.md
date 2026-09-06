# Configuração de Rede da VM Base

## Objetivo
Configurar a VM base do laboratório com **duas interfaces de rede** para
separar dois contextos distintos: acesso à internet (para instalar
pacotes e ferramentas) e uma rede isolada, exclusiva do lab, onde o
tráfego entre as futuras VMs (máquina alvo, SIEM, etc.) vai trafegar
sem tocar a internet ou a rede doméstica real.

Esse isolamento é um princípio básico de qualquer lab de segurança:
nunca deixar tráfego de teste/ataque com rota direta para fora do
ambiente controlado.

##  IP estático via Netplan

A interface NAT recebe IP automaticamente (DHCP). A Rede Interna não
tem servidor DHCP — por ser uma rede isolada, sem roteador — então o
IP precisa ser configurado manualmente.

Arquivo editado:
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Configuração aplicada:
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      addresses: [192.168.56.10/24]
```

Aplicação:
```bash
sudo netplan apply
```

##  Troubleshooting

Ao aplicar, apareceram avisos:
```
WARNING: Permissions for /etc/netplan/50-cloud-init.yaml are too open.
Failed to reload network settings: Unit dbus-org.freedesktop.network1.service not found.
Falling back to a hard restart of systemd-networkd.
```

- O aviso de permissões é resolvido restringindo o acesso ao arquivo:
```bash
sudo chmod 600 /etc/netplan/50-cloud-init.yaml
```
- O segundo aviso indica que o Netplan não conseguiu recarregar a
  rede de forma "suave" e caiu para um restart completo do serviço —
  não impediu a aplicação da configuração, apenas usou um caminho
  alternativo internamente.

##  Verificação final

```bash
ip a
```

Resultado confirmando as duas interfaces ativas:
```
2: enp0s3: ... state UP
    inet 10.0.2.15/24 ... dynamic enp0s3

3: enp0s8: ... state UP
    inet 192.168.56.10/24 ... noprefixroute enp0s8
```

| Interface | Rede | IP | Origem |
|-----------|------|----|--------|
| enp0s3 | NAT | 10.0.2.15 | DHCP automático |
| enp0s8 | Interna (`soclab`) | 192.168.56.10 | Estático (Netplan) |


