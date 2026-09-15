# Playbook de Triagem — Execução Suspeita de Shell/PowerShell (Comandos Ofuscados)

## Objetivo

Padronizar a triagem de alertas relacionados à execução suspeita de shell/PowerShell (rule.id 92032 e correlatos), com foco especial em comandos ofuscados via Base64 (`-EncodedCommand`), garantindo consistência na decisão entre falso positivo, verdadeiro positivo de risco variável e escalonamento.

## 1. Identificação do Alerta

Ao receber o alerta, coletar:

- `rule.id`, `rule.description`, `rule.level`
- `data.win.system.eventID`
- `rule.mitre.id` / `rule.mitre.tactic` / `rule.mitre.technique`
- `agent.name` e `data.win.system.computer`
- `data.win.eventdata.subjectUserName` (ou campo de usuário equivalente)
- `data.win.eventdata.parentCommandLine` (ou `commandLine`) — **essencial** quando o comando estiver ofuscado em Base64, pois sem esse campo não é possível decodificar e avaliar o conteúdo real
- `@timestamp`

## 2. Triagem Inicial — Contexto

Perguntas a responder antes de decodificar tecnicamente o comando:

- **Qual o nível de privilégio do usuário requerente?** Influencia diretamente o escopo de dano possível caso o comando seja malicioso.
- **O horário é compatível com atividade normal?** Fora de horário aumenta suspeita, mas não é veredito isolado.
- **O processo pai é uma ferramenta de administração conhecida (ex: agente RMM, script de automação documentado), ou é uma cadeia pai-filho não identificada (ex: `cmd.exe → powershell.exe` com `-EncodedCommand`)?** Importante lembrar que `-EncodedCommand` sozinho **não é veredito de malícia** — é usado legitimamente por ferramentas de administração remota para evitar problemas de escaping de caracteres. A suspeita aumenta quando combinado a uma cadeia de processo incomum.

*Nota de escopo: fatores como geolocalização e detecção de movimento lateral são comumente usados em triagens reais, mas não se aplicam ao ambiente atual do lab (rede isolada, host único) — mantidos aqui apenas como registro de conhecimento, não como critério aplicável.*

## 3. Investigação — Decodificação e Análise

1. Localizar o comando ofuscado no campo `data.win.eventdata.parentCommandLine` (ou `commandLine`, dependendo de onde o comando foi executado).
2. Decodificar em duas etapas, na ordem correta: primeiro reverter o Base64, depois interpretar o resultado como UTF-16LE — encoding que o PowerShell aplica ao comando original **antes** de codificá-lo em Base64 via `-EncodedCommand`:
```bash
   echo "<base64>" | base64 -d | iconv -f UTF-16LE -t UTF-8
```
3. Analisar o comando revelado, atentando para:
   - Ações sensíveis (download de arquivo externo, acesso a credenciais, criação de tarefa agendada, modificação de registro) vs. comportamento inofensivo.
   - Técnicas de evasão adicionais dentro do próprio comando (concatenação/fragmentação de strings, resolução dinâmica de nomes de comando via `Get-Command` ou similar) — presença dessas técnicas indica intenção deliberada de evasão, mesmo que o payload final seja inofensivo.
4. Confirmar se o processo pai identificado corresponde a uma ferramenta ou processo legítimo conhecido no ambiente.

## 4. Critérios de Decisão

| Cenário | Classificação | Ação |
|---|---|---|
| Comando decodificado revela instrução para baixar e executar arquivo de origem externa/desconhecida; horário de execução incomum (madrugada) | Verdadeiro positivo crítico | Isolar a máquina da rede imediatamente; escalar para N2 com prioridade alta, anexando o comando decodificado e todas as evidências |
| Horário comercial; comando decodificado revela processo pai identificado como ferramenta nativa/legítima do Windows ou automação documentada de TI | Falso positivo / verdadeiro positivo de risco mínimo | Documentar como comportamento esperado; nenhuma ação de contenção necessária |
| Horário comercial; comando decodificado revela processo pai não identificado, ou comportamento além do esperado (mesmo sem indício de download externo) | Verdadeiro positivo — severidade elevada | Escalar para N2 com evidências (comando decodificado, processo pai); considerar isolamento se houver qualquer indício adicional de comprometimento |
| Usuário pertence à OU de TI (setor com rotina legítima de conexão remota para suporte); processo pai é `cmd.exe` tentando cadeia de escalonamento | Suspeita de comprometimento sem certeza — cenário ambíguo | Reunir todas as evidências e escalar para N2 para veredito; nenhuma ação isolada de contenção nesta etapa, dado o risco de falso positivo em rotina legítima |

## 5. Princípios Gerais de Ação

- Evitar qualquer ação que possa alertar um possível atacante ou colaborador comprometido sobre a investigação em andamento (evitar "tipping off") — este princípio se aplica a **todas** as análises de triagem, não apenas a cenários ambíguos.
- `-EncodedCommand` isoladamente não é indicador suficiente de malícia — a decisão deve sempre considerar a combinação entre conteúdo decodificado, processo pai e contexto do usuário.
- Ações de contenção reversíveis e rápidas (isolamento de rede) estão dentro da autoridade do N1 e devem ser tomadas quando o cenário justificar, em paralelo ao escalonamento.
- Ações irreversíveis, que destruam evidência forense, ou que exponham a investigação ao usuário não são decisão do N1 — cabe ao N2/time de resposta a incidentes avaliar.

## 6. Fechamento e Documentação

Todo ticket fechado deve registrar:
- Alerta original (rule.id, timestamp, agente, usuário envolvido)
- Comando decodificado e técnicas de ofuscação identificadas
- Processo pai e se foi identificado como legítimo ou não
- Classificação final e justificativa
- Ação tomada (contenção, escalonamento, ou nenhuma) com motivo
