# Tuning de Falso Positivo — Rootcheck (Trojaned md5sum)

> Documento da v1 (VirtualBox, host Windows 11). Mantido como histórico; ver [docs/v2/](../v2/).

## Objetivo

Eliminar um falso positivo recorrente gerado pelo módulo rootcheck do Wazuh
manager, sem enfraquecer a cobertura de detecção da regra original — servindo
também como exercício de tuning de SIEM, uma habilidade tão relevante no
dia a dia de um SOC quanto a criação de novas detecções.

## Ambiente

- **wazuh-manager** (192.168.56.5): Wazuh 4.14, indexer + server + dashboard.
- Módulo afetado: rootcheck, rodando como parte da automonitoração do
  próprio manager (`agent.id: 000`).

## Processo

### 1. Identificação do problema

Logo após a instalação do Wazuh, o próprio manager passou a gerar alertas
recorrentes de severidade média (`rule.level: 7`, `rule.id: 510`) a cada
ciclo de rootcheck, com a descrição "Trojaned version of file detected"
para os binários `/bin/md5sum` e `/usr/bin/md5sum`:

Trojaned version of file '/usr/bin/md5sum' detected. Signature used:
'bash|^/bin/sh|file.h|proc.h|/dev/[^cfhlnrstuw]|^/bin/.*sh' (Generic).


Trata-se de um falso positivo conhecido do rootcheck em distribuições
baseadas em Debian/Ubuntu: a verificação de assinaturas genéricas de
rootkit por vezes coincide com padrões presentes em binários legítimos
do coreutils.

### 2. Decisão de abordagem

Duas formas de resolver foram consideradas:

- **Editar o banco de assinaturas global** (`rootkit_trojans.txt`),
  removendo a entrada que gera o match. Descartada: enfraquece a
  detecção permanentemente para qualquer binário futuro que bata nesse
  padrão, e a mudança seria sobrescrita em atualizações do Wazuh.
- **Criar uma regra de override em `local_rules.xml`**, herdando da
  regra original via `<if_sid>` e rebaixando a severidade apenas para o
  caso específico identificado. Escolhida por preservar a regra pai
  intacta — qualquer outro binário que dispare o mesmo padrão genérico
  continua gerando alerta normalmente.

### 3. Implementação

Regra criada em `/var/ossec/etc/rules/local_rules.xml`, cobrindo os dois
caminhos possíveis do binário (`/bin/md5sum` e `/usr/bin/md5sum`) via
regex, em vez de duas regras separadas:

```xml
<group name="local,rootcheck,">
  <rule id="100510" level="0">
    <if_sid>510</if_sid>
    <field name="file" type="pcre2">^/(usr/)?bin/md5sum$</field>
    <description>Falso positivo conhecido: rootcheck sinaliza md5sum do coreutils via assinatura genérica (Debian/Ubuntu)</description>
  </rule>
</group>
```

IDs de regras locais seguem a convenção do Wazuh (faixa 100000–120000,
reservada para customizações, evitando colisão com o ruleset padrão).
`level="0"` indica que o evento é processado, mas não gera alerta ativo.

### 4. Validação

A tentativa inicial de validar via `wazuh-logtest`, colando o `full_log`
manualmente, não reproduziu o comportamento esperado — o teste processou
a linha pelo decoder genérico `ossec`, não pelo decoder específico do
rootcheck, já que alertas de rootcheck são gerados internamente pelo
módulo, com estrutura diferente de um log de texto simples. O
`wazuh-logtest` confirmou apenas a validade sintática da regra
(`wazuh-analysisd -t`, exit code 0), não seu comportamento funcional.

A validação real exigiu observar o comportamento ao vivo:

1. Reiniciar o `wazuh-manager` (o que também força um novo ciclo de
   rootcheck).
2. Confirmar via log operacional (`ossec.log`) que o ciclo de rootcheck
   completou (`Starting rootcheck scan` / `Ending rootcheck scan`).
3. Verificar em `alerts.json` que nenhum novo alerta de nível 7 para
   `md5sum` foi gerado após o ciclo completo — repetido em dois ciclos
   consecutivos pós-restart, sem novas ocorrências.

## Lições Aprendidas

- **Override via `<if_sid>` preserva a cobertura original.** É a forma
  correta de tratar falso positivo pontual em uma regra genérica,
  diferente de editar o banco de assinaturas ou desabilitar a regra
  inteira — o binário específico é silenciado, mas o padrão continua
  ativo para qualquer outra ocorrência.
- **`wazuh-logtest` não reproduz fielmente todos os módulos.** Ele é
  confiável para validar regras baseadas em decoders de log de texto
  (ex: SSH, PowerShell), mas não simula o pipeline interno de módulos
  como o rootcheck, que geram eventos por mecanismo próprio. Nesses
  casos, a validação sintática (`wazuh-analysisd -t`) e a validação
  funcional (observar o comportamento após um ciclo real do módulo)
  são etapas distintas e ambas necessárias.
- **Alertas de nível 0 são descartados no pipeline, não apenas
  ocultados na interface.** Um evento suprimido por uma regra
  `level="0"` não aparece em `alerts.json` nem no índice de eventos
  padrão do dashboard — ele só seria capturado por um índice de
  archives, caso habilitado. Isso significa que a validação de uma
  supressão precisa ser feita por ausência de novos alertas ao longo
  do tempo, não por busca direta de um evento "silenciado".
- **Timestamps em UTC exigem conversão manual durante troubleshooting.**
  O `alerts.json` registra tudo em UTC, enquanto o ambiente opera em
  horário local (Curitiba, UTC-3) — necessário ter isso em mente ao
  comparar timestamps de log com o horário de execução de uma ação.

## Próximos Passos

- Avaliar, junto com o `soclab` de rede (fase de hardening arquitetural),
  se vale habilitar o índice de archives do Wazuh para reter todo evento
  processado (incluindo nível 0) — útil para auditoria completa de
  tuning aplicado ao longo do tempo.
- Revisar periodicamente se novos falsos positivos de rootcheck surgem
  em outros binários do sistema, aplicando o mesmo padrão de override
  quando necessário.
