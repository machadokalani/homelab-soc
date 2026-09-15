# Playbook de Triagem — Criação de Conta e Escalonamento de Privilégio Local

## Objetivo

Padronizar a triagem de alertas relacionados à criação de contas locais seguida de adição ao grupo de administradores (rule.id 60110, 60154, entre outros relacionados), permitindo classificação consistente entre atividade legítima de TI e possível comprometimento.

## 1. Identificação do Alerta

Ao receber o alerta, coletar:

- `rule.id`, `rule.description`, `rule.level`
- `data.win.system.eventID`
- `rule.mitre.tactic` / `rule.mitre.technique`
- `agent.name` e `data.win.system.computer`
- `data.win.eventdata.subjectUserName` (quem executou a ação)
- `data.win.system.message` (apenas a primeira linha, para contexto rápido)
- `@timestamp`

## 2. Triagem Inicial — Contexto

Perguntas a responder antes de investigar tecnicamente:

- **O horário é compatível com atividade normal?** Fora de horário aumenta suspeita, mas dentro do horário comercial não descarta o caso — um atacante ativo pode agir durante o expediente para se camuflar.
- **Qual o perfil/departamento do usuário requerente?** Verificar OU no Active Directory (ou CMDB, se disponível). Usuários de TI têm maior probabilidade de legitimidade nesse tipo de ação; usuários de outros setores (RH, Financeiro, Marketing) tornam a ação intrinsecamente mais suspeita, já que escalonamento de privilégio local normalmente não faz parte do escopo de trabalho deles.
- **Existe chamado de mudança (change management) associado?** Se a empresa possuir esse controle, verificar silenciosamente se há um ticket de manutenção que justifique a ação — essa consulta não alerta o usuário nem investiga-o diretamente.

## 3. Investigação — Confirmar ou Descartar

- **Identificar o `LogonType` da sessão do requerente**, cruzando `subjectLogonId` com o evento de logon (Event ID 4624) correspondente:
  - `2` (Interactive) — acesso físico/local à máquina.
  - `3` (Network) — acesso via rede (ex: compartilhamento).
  - `10` (RemoteInteractive) — acesso via RDP.
  
  RDP ou Network isoladamente não são maliciosos por si só (uso legítimo de administração remota é comum), mas aumentam o peso da suspeita quando combinados com outros sinais.

- **Correlacionar com outros logs da mesma máquina no mesmo período** — buscar evidências adicionais como download de executáveis via linha de comando, processos incomuns, ou outros alertas próximos no tempo que possam indicar uma cadeia de ataque maior.

- **Importante:** nenhum indicador isolado (horário, tipo de logon, ou departamento do usuário) deve ser tratado como veredito sozinho — a classificação deve considerar a combinação de sinais.

## 4. Critérios de Decisão

| Cenário | Classificação | Ação |
|---|---|---|
| Conta criada e promovida a admin; LogonType 2 (interativo local); usuário pertence à OU de TI; horário comercial | Verdadeiro positivo esperado (atividade administrativa legítima) | Documentar como comportamento normal, sem escalonamento; considerar allowlist se a rotina for recorrente |
| Conta criada e promovida a admin; LogonType 10 (RDP); usuário fora do perfil técnico (ex: RH); múltiplos logs de download de executáveis via cmd na mesma janela de tempo | Máquina possivelmente comprometida — verdadeiro positivo crítico | Isolar a máquina da rede imediatamente (ação dentro da autoridade do N1); escalar para N2 com prioridade crítica, anexando todas as evidências coletadas |
| LogonType 3 (network); usuário pertence à OU de TI; fora do horário comercial; sem outros logs suspeitos corroborantes | Suspeita de comprometimento sem certeza — cenário ambíguo | Não tomar nenhuma ação que possa alertar o usuário ou investigação (evitar "tipping off"); verificar silenciosamente change management, se disponível; reunir evidências e escalar para N2 como prioridade relevante para análise mais profunda |

## 5. Princípios Gerais de Ação

- Ações de contenção reversíveis e rápidas (isolamento de rede) estão dentro da autoridade do N1 e devem ser tomadas imediatamente quando o cenário justificar, em paralelo ao escalonamento — não é necessário aguardar aprovação do N2 para isso.
- Ações irreversíveis ou que possam destruir evidência forense (formatação de disco, reinstalação de sistema) ou que exponham a investigação ao usuário (reset de senha, confronto direto) **não são decisão do N1** — cabe ao N2/time de resposta a incidentes avaliar e decidir, com base nas evidências entregues.
- Em cenários ambíguos, priorizar a coleta silenciosa de evidências sobre ações que possam alertar um possível atacante ou colaborador comprometido.

## 6. Fechamento e Documentação

Todo ticket fechado deve registrar:
- Alerta original (rule.id, timestamp, agente, usuário envolvido)
- LogonType identificado e evidências de correlação (ou ausência delas)
- Contexto de departamento/perfil do usuário, quando disponível
- Classificação final e justificativa
- Ação tomada (contenção, escalonamento, ou nenhuma) com motivo
