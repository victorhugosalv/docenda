# Domínio: Docenda

> **v0.2 (17/11/2026).** Nasceu da revisão do planejamento de 16/11; a v0.2 alinha os nomes com o [DADOS](DADOS.md) e o [CONTRATOS](CONTRATOS.md) (mudanças na seção 9). Diz **o que existe** no sistema e **quais regras nunca podem quebrar**. O "como" fica nas [decisões](decisoes/) e nas especificações de cada fatia ([BACKLOG](BACKLOG.md)).
> Base: [REQUISITOS](REQUISITOS.md), decisões 0001 a 0014.

## 1. Linguagem do domínio (português ↔ código)

O código é em inglês e a tela em português (decisão 0001). Esta tabela é a única tradução aceita: nome novo entra aqui antes de entrar no código.

| Termo (tela e documentos) | No código | Observação |
|---|---|---|
| Plano de aula | `LessonPlan` | Documento de uma aula |
| Modelo do plano | `PlanTemplate` | Roteiro do estágio (v1) ou ENADE (depois). É **dado**, não código (seção 8) |
| Identificação do plano | `PlanIdentification` | Escola, professor, turma, data, turno. Fica em tabela separada e **nunca vai para a IA** (INV-09) |
| Contexto da turma | `ClassContext` | Descrição agregada, sem aluno identificável |
| Tamanho da turma | `classSize` | Quantos alunos costumam estar presentes; um número, sem nomes |
| Cópia do contexto | `ContextSnapshot` | O contexto como estava quando o plano foi criado; é o que vai para a IA |
| Etapa de ensino | `EducationStage` | Educação infantil, fundamental (anos iniciais e finais), médio |
| Ano ou série | `grade` | 1 a 9 no fundamental, 1 a 3 no médio |
| Etapa de produção | `ProductionStep` | Roteiro simplificado → plano → slides → roteiro padronizado (depois da v1) |
| Atividade (momento da aula) | `Activity` | Bloco com tempo: introdução, desenvolvimento, sistematização ou fechamento |
| Duração nominal | `nominalMinutes` | O tempo oficial da aula |
| Tempo efetivo | `effectiveMinutes` | O tempo para o qual o plano é feito |
| Hora-aula | `classHourMinutes` | 50 por padrão, ajustável |
| Habilidade | `Skill` | Item do catálogo, com código e texto literal |
| Catálogo | `CurriculumFramework` | BNCC Computação é o primeiro; a tabela aceita outros (seção 8) |
| Fonte | `Source` | Referência com link, conferida e aprovada pelo autor |
| Sugestão da IA | `Suggestion` | Proposta para um campo, ainda não decidida pelo autor |
| Achado | `Finding` | Resultado de uma regra ou da avaliação: bloqueia, avisa ou informa |
| Geração | `AiGeneration` | Um pedido de geração e o seu andamento |
| Autor | `ownerId` | Dono do plano e do contexto |
| Revisão (número de gravação) | `revision` | Sobe a cada gravação do plano ou do contexto; evita que duas abas se atropelem. Não confundir com `version`, que conta aprovações |
| Convite | `Invite` | |
| Crédito de IA | `CreditEntry` | Lançamento no livro-razão (seção 6) |
| Uso de IA | `AiUsageRecord` | Tokens e custo de uma chamada, sem conteúdo |
| Ocorrência de uso indevido | `MisuseOccurrence` | |
| Bloqueio da IA | `MisuseBlock` | Impede a pessoa de usar a IA; não desativa a conta |
| Aceite dos termos | `TermsAcceptance` | Data e versão dos termos aceitos |
| Recibo de exclusão | `DeletionReceipt` | Prova que um pedido de exclusão foi atendido, sem identificar ninguém |
| Auditoria | `AuditEvent` | Quem fez o quê, só com IDs |
| Registro de acesso | `AccessRecord` | Conta, IP e hora do login |

> **Cuidado com "etapa".** A palavra tinha dois sentidos no REQUISITOS v0.2. A partir da v0.3: **etapa de ensino** (`EducationStage`) e **etapa de produção** (`ProductionStep`). "Etapa" sozinha não se usa.

## 2. Entidades e dono de cada uma

"Dono" é o módulo que cria as tabelas e o único que as consulta (decisão 0012). Os outros módulos pedem pela porta da frente (`index`).

| Entidade | Módulo dono | Guarda | Marco |
|---|---|---|---|
| `User`, `Session` | `reusable/auth` (tabelas do Better Auth) | Nome, e-mail, hash da senha, papel, sessões | M1 |
| `Invite`, `TermsAcceptance` | `reusable/auth` | E-mail convidado, hash do token, validade; aceite dos termos | M2 |
| `ClassContext` | `modules/class-context` | Etapa de ensino, ano, matéria, duração, tamanho da turma e os textos agregados. A carga da semana e a opinião do professor valem para uma aula e ficam no plano | M1 |
| `LessonPlan` | `modules/plans` | Dono, cópia do contexto, modelo, estado, versão, tema, tempos, material colado e conteúdo aceito (`jsonb` com versão do esquema) | M1 |
| `PlanIdentification` | `modules/plans` | Escola, professor, turma, data e turno, em tabela separada | M1 |
| `LessonPlanVersion` | `modules/plans` | Cópia imutável do plano a cada aprovação | M1 |
| `PlanExport` | `modules/plans` | Evento: versão exportada, formato, hora | M1 |
| `Suggestion` | `modules/plans` | Campo-alvo, valor proposto, origem, estado | M1 |
| `PlanSkill` | `modules/plans` | Liga o plano a uma `Skill` do catálogo (chave estrangeira) | M1 |
| `Skill`, `CurriculumFramework` | `modules/bncc` | Código, texto literal, etapa de ensino, anos, eixo, competência, explicação, exemplos, versão do catálogo | M1 |
| `Source` | `modules/sources` | Link, domínio, estado, resultado da checagem, referência em ABNT | M2 |
| `AiGeneration` | `modules/plans` | Estado, etapa em andamento, versão do prompt, modelo, motivo da falha | M1 |
| `AiUsageRecord`, `CreditEntry`, `AiBudgetPeriod`, `CreditAccount` | `reusable/ai-usage` | Tokens, custo, lançamentos de crédito, gasto do mês | M1 (uso, reserva e teto do sistema), M2 (crédito por pessoa) |
| `MisuseOccurrence`, `MisuseBlock` | `reusable/misuse` | Categoria, gravidade, trecho mínimo, decisão do administrador; o bloqueio e o pedido de revisão | M2 |
| `AuditEvent`, `AccessRecord`, `DeletionReceipt` | `reusable/audit` | Tipo do evento, IDs e hora; conta, IP e hora; recibo de exclusão | M2 |
| `Setting` | `reusable/settings` | Liga e desliga da IA, limites e tetos mudados sem deploy (RF-75) | M1 (por comando), M2 (pela tela) |

## 3. O plano por dentro

O plano é guardado num **formato interno único** e exportado para o modelo escolhido. O conteúdo aceito fica em `jsonb`, validado por um esquema versionado em `packages/contracts`. O que é consultado ou conferido pelo banco fica em tabelas: habilidades, fontes, sugestões e gerações.

| Campo interno | Tipo | A IA sugere? | No roteiro do estágio |
|---|---|---|---|
| `identification` | tabela à parte | **Nunca** | Identificação |
| `theme`, `subtheme` | texto | Não (vem do autor) | Tema, subtema |
| `skills` | ligação com o catálogo | Sim, por código | Habilidades da BNCC Computação |
| `justification` | texto | Sim | Justificativa |
| `generalObjective` | texto | Sim | Objetivo geral |
| `specificObjectives` | lista de textos | Sim | Objetivos específicos |
| `contents` | lista de textos | Sim | Conteúdos |
| `priorKnowledge` | texto | Sim | Conhecimentos prévios |
| `activities` | lista de `Activity` (momento, título, descrição, minutos, `setupHeavy`) | Sim | Procedimentos |
| `resources` | lista de textos | Sim | Recursos |
| `assessment` (`criteria`, `instruments`) | duas listas de textos, uma unidade só | Sim | Avaliação |
| `adaptations` | texto | Sim | Adaptações |
| `crossSubjectLink` | texto | Sim | Ligação com outras matérias (RF-27) |
| `sources` | ligação com `Source` | Não na v1 | Bibliografia |

Regras do formato:
- **Texto puro.** Nenhum campo guarda nem mostra HTML ou Markdown vindo da IA. A tela escreve o texto como texto; a exportação é quem monta o Markdown.
- **`setupHeavy`** marca atividade de instalar programa, criar conta ou configurar equipamento (RF-14). Quem marca é a IA, na geração, ou o autor, na edição.
- **Unidade de sugestão: o campo.** Cada linha da tabela acima é uma unidade que o autor aceita, edita ou recusa por inteiro (RF-23). Habilidades e fontes são decididas item a item, porque cada uma já tem estado próprio. Sugestão por item de lista fica para depois da v1.

## 4. Estados

### 4.1 Plano
```
rascunho ──aprovar──▶ aprovado
    ▲                    │
    └──────editar────────┘   (a versão aprovada fica guardada)
```
- **`draft` (rascunho):** em edição. Pode ter sugestões pendentes e achados.
- **`approved` (aprovado):** só o autor chega aqui, e só com as condições do INV-03. Aprovar grava uma `LessonPlanVersion`.
- **Editar um plano aprovado** pede reabrir: uma ação do autor devolve o plano a rascunho com o número de versão seguinte. A versão aprovada continua guardada e exportável. A identificação não é conteúdo: pode mudar sem reabrir.
- **"Exportado" não é estado, é evento** (`PlanExport`). Um plano pode ser exportado várias vezes; a tela mostra a data da última exportação daquela versão.
- **`in_review` (em revisão)** fica reservado para quando existirem revisores (RF-80 em diante). Na v1 não há quem revise além do autor.

### 4.2 Geração
`queued` → `running` → `succeeded` ou `failed`.

Toda falha tem um motivo de lista fechada, e a tela mostra uma mensagem própria para cada um:

| `failureReason` | Quando | Conta como uso indevido? | Cobra? |
|---|---|---|---|
| `input_blocked_privacy` | Alerta de dado de aluno não resolvido (RF-15) | Não | Só a checagem |
| `input_blocked_misuse` | Checagem de entrada (RF-90) | Sim, conforme a gravidade | Só a checagem |
| `off_topic` | Pedido fora do propósito (RF-95) | Não | Só a checagem |
| `provider_refusal` | O provedor recusou (`stop_reason: refusal`) | **Não** | O que foi usado |
| `invalid_output` | JSON inválido depois de 2 repetições (RNF-35) | Não | O que foi usado |
| `output_check_failed` | A resposta não passou na checagem de saída | Não | O que foi usado |
| `budget_exhausted` | A reserva acabou no meio (0008 D3) | Não | O que foi usado |
| `provider_unavailable` | Disjuntor aberto ou erro do provedor | Não | Nada |
| `worker_lost` | O worker caiu e o job esgotou as repetições | Não | **A reserva inteira** (INV-07) |

### 4.3 Sugestão
`pending` → `accepted`, `edited` ou `rejected`. Aceitar copia o valor para o conteúdo do plano; editar copia a versão do autor; recusar não copia nada. Enquanto um campo tem sugestão pendente, o autor não escreve nele por fora: decide a sugestão primeiro. Uma geração nova substitui as sugestões **pendentes** do mesmo campo e nunca toca no que o autor já decidiu.

### 4.4 Fonte
`to_review` → `approved` ou `rejected`. À parte, o resultado da checagem do link: `unchecked`, `ok` ou `broken`.

### 4.5 Convite e conta
- Convite: `pending` → `accepted`, `cancelled` ou `expired` (7 dias).
- Conta: `active` → `deactivated` (30 dias, cancelável ao entrar) → apagada ou anonimizada (REQUISITOS, seção 7.1). O bloqueio da IA por uso indevido é uma marca separada: não desativa a conta.

## 5. Achados: um formato só para regras e avaliação

Toda regra da seção 4 do REQUISITOS e toda crítica da avaliação por IA (RF-26) devolvem a mesma coisa:

| Campo | Significado |
|---|---|
| `ruleId` | Qual regra (lista abaixo) ou qual item da rubrica |
| `severity` | `block` (impede aprovar), `warn` (o autor decide) ou `info` |
| `fieldPath` | O campo do plano a que se refere, quando houver |
| `message` | Texto para o autor, em português |
| `suggestionId` | Sugestão ligada ao achado, quando a avaliação propõe uma correção |

Cada regra determinística é uma **função pura** `(plano, contexto) → achados`, com teste unitário e casos de limite (RNF-09). Elas são recalculadas a cada leitura; só os achados da avaliação por IA são guardados, porque custam dinheiro.

**Achado vindo da IA nunca é `block`.** A avaliação por IA devolve só `warn` ou `info`, com `ruleId` começando por `rubric.` ([AVALIACAO](AVALIACAO.md), seção 4). Quem trava a aprovação são as regras por código; a IA opina, não decide (INV-03).

| `ruleId` | Regra | Gravidade | Requisito |
|---|---|---|---|
| `time-sum` | A soma dos minutos das atividades é igual ao tempo efetivo | `block` | Seção 4 |
| `pending-suggestion` | Há sugestão da IA pendente | `block` | RF-23 |
| `source-to-review` | Há fonte "a revisar" | `block` | RF-51 |
| `skill-required` | O plano tem pelo menos uma habilidade | `block` | RF-41 |
| `skill-stage-mismatch` | A habilidade não combina com a etapa de ensino ou o ano da turma | `warn` | RF-43 |
| `setup-heavy` | Há atividade `setupHeavy` e o tempo efetivo não comporta o resto | `warn` | RF-14 |
| `privacy-hint` | O texto parece ter nome de aluno, diagnóstico ou laudo | `block` até o autor corrigir ou confirmar. Na gravação, o texto com alerta nem é guardado (DADOS, D6); o achado cobre texto antigo quando a lista de termos cresce | RF-15 |
| `empty-required-field` | Campo obrigatório do modelo vazio | `block` | RF-20 |

Habilidade fora do catálogo não é um achado: ela **não consegue ser gravada** (INV-02).

## 6. Crédito de IA: livro-razão

Um livro-razão é uma tabela em que só se acrescentam lançamentos; o saldo é a soma deles. Nada é reescrito, então todo centavo tem explicação.

| Lançamento | Valor | Quando |
|---|---|---|
| `grant` | + crédito do mês (US$ 1 na v1) | No primeiro uso do mês, ou pelo job do dia 1 |
| `reserve` | − custo máximo da operação | Ao enfileirar a geração (0008 D3) |
| `settle` | + (reserva − custo real) | Ao terminar, com o `response.usage` |

- **Período:** o mês no horário de Brasília. O crédito não acumula porque cada período tem a sua soma.
- **Trava:** a reserva roda numa transação que trava a linha da conta de crédito do usuário e a linha do teto do mês antes de somar. Dois pedidos simultâneos entram em fila um atrás do outro.
- **Resultado desconhecido:** se o worker cai e não se sabe se a chamada foi cobrada, não há `settle`: a reserva inteira fica como custo. O sistema pode cobrar a mais do usuário por engano, nunca a menos do teto.
- **Unidade:** nano-dólar (US$ 1 = 1.000.000.000), em `bigint`. Com milésimo de centavo, o preço por token do modelo mais barato já daria fração (RNF-42).

## 7. Invariantes

Regras que valem sempre. Cada uma tem pelo menos um teste com o seu número no nome.

| # | Invariante | Como se garante |
|---|---|---|
| INV-01 | Um autor só alcança o que é dele | Todo repositório de recurso do autor exige `ownerId`; recurso de outro responde 404; teste genérico em todas as rotas |
| INV-02 | Só entra no plano habilidade do catálogo, com o texto do catálogo | Chave estrangeira de `PlanSkill` para `Skill`; o plano guarda o código, e o texto é lido do catálogo |
| INV-03 | Plano só é aprovado sem achado `block`, e só pelo autor | Função pura de aprovação; a operação existe só na rota do dono, não no worker |
| INV-04 | A IA nunca escreve no conteúdo do plano | O worker só cria `Suggestion`; não há operação de escrita no conteúdo fora das ações do autor |
| INV-05 | No máximo uma geração ativa por plano | Índice único parcial (0008 D5) |
| INV-06 | O saldo do mês nunca fica negativo, e o teto nunca é ultrapassado | Trava de linha na reserva (seção 6) |
| INV-07 | O custo registrado nunca é menor que o custo real | Resultado desconhecido fica com a reserva inteira |
| INV-08 | Nenhuma geração fica "rodando" para sempre | Job que encerra gerações órfãs (0008, nota de 16/11) |
| INV-09 | Identificação do plano, nome, e-mail e ID do usuário nunca entram num prompt | A porta de IA recebe um tipo que não tem esses campos; teste com valores-isca |
| INV-10 | Nada vai à IA com alerta de privacidade não resolvido | Passo do pipeline, antes de qualquer chamada |
| INV-11 | O job da fila carrega só IDs | Tipo do job; teste (0008 D6) |
| INV-12 | Dinheiro é inteiro | Tipo próprio; nenhum `number` em custo |
| INV-13 | Toda geração sabe qual prompt, modelo e esforço a produziu | Colunas obrigatórias em `AiGeneration` e `AiUsageRecord` |
| INV-14 | A auditoria só cresce | Papel do banco sem `UPDATE` nem `DELETE` (0010 D6, 0012 D5) |

## 8. Decisões baratas agora, caras depois

1. **Catálogo genérico.** `Skill` pertence a um `CurriculumFramework` (nome e versão). A BNCC Computação é a primeira linha. Outros componentes da BNCC entram depois como dados, sem mudar tabela.
2. **Modelo do plano como dado.** O `PlanTemplate` diz quais campos internos aparecem, em que ordem e com que título. O exportador lê o modelo; acrescentar o ENADE é escrever um modelo novo e, no máximo, um exportador novo.
3. **Dono em tudo.** Todo recurso do autor tem `ownerId`. Quando existirem escolas ou equipes, entra uma coluna opcional de organização, sem reescrever as consultas.
4. **Versão do esquema no conteúdo.** O `jsonb` do plano carrega `schemaVersion`. Mudar o formato é escrever uma função de migração de uma versão para a seguinte.

## 9. Mudanças da v0.2 (17/11/2026)

| O que mudou | Por quê |
|---|---|
| Identificação do plano em tabela separada, fora da versão aprovada | A consulta da IA não a alcança; trocar a data não reabre o plano |
| Avaliação virou um campo só, `assessment`, com critérios e instrumentos | É uma unidade de sugestão; dois nomes soltos confundiam |
| Reabrir é uma ação própria do autor | Uma tecla por engano não deve desfazer uma aprovação |
| Carga da semana e opinião do professor no plano; tamanho da turma no contexto | O contexto dura meses; a semana de provas, uma aula |
| Nomes novos no glossário: `revision`, `ContextSnapshot`, `classSize`, `MisuseBlock`, `TermsAcceptance`, `DeletionReceipt` | Nome novo entra aqui antes do código |
| Achado vindo da IA nunca bloqueia a aprovação | A IA sugere e opina; travar o plano seria decidir |
| `Setting` passa a existir no M1, mudada por comando | O liga e desliga da IA (RF-75) está na fatia F07 |

