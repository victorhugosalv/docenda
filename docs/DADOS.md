# Dados: Docenda

> **v0.1 (17/11/2026). Planejado: ainda não há código.** Traduz o [DOMINIO](DOMINIO.md) em tabelas. A especificação de cada fatia pode ajustar um detalhe; quando ajusta, corrige este arquivo no mesmo PR.
> Base: decisões [0004](decisoes/0004-postgresql.md) (PostgreSQL 18), [0008](decisoes/0008-ai-job-queue-postgresql.md) (fila e reserva), [0010](decisoes/0010-logs-monitoring-and-incident-response.md) (registros) e [0012](decisoes/0012-transactions-repositories-and-module-layout.md) (dono, transação e papéis do banco).
> Irmãos: [CONTRATOS](CONTRATOS.md) (o que entra e sai pela API) e [ACEITE](ACEITE.md) (como saber que cada fatia está pronta).

## 1. Convenções

| Assunto | Regra |
|---|---|
| Nomes | No banco, `snake_case` no singular (`lesson_plan`). No código, `camelCase`, com o mapeamento feito no esquema do Prisma (`@map`). Os nomes vêm do glossário do DOMINIO |
| Identificador | `uuid` gerado pelo banco com `uuidv7()` (0004). Nunca um número em sequência |
| Tempo | `timestamptz`, sempre em UTC. Toda tabela tem `created_at`; as que mudam têm `updated_at` |
| Dinheiro | `bigint` em nano-dólar (INV-12). Nenhuma coluna `numeric`, `float` ou `money` |
| Lista fechada | Tipo `enum` do PostgreSQL, com os valores em inglês do DOMINIO. Valor novo é uma migração |
| Texto | `text`. O tamanho máximo é conferido na borda, pelo esquema de [CONTRATOS](CONTRATOS.md), não no banco |
| `jsonb` | Só para o que muda de forma: conteúdo do plano, cópia do contexto e a versão aprovada. Todo `jsonb` carrega `schemaVersion` |
| Dono | Toda tabela de recurso do autor tem `owner_id`, com chave estrangeira para o usuário |
| Entre módulos | Chave estrangeira só para o usuário e para o catálogo (0012 D6). Qualquer outra ligação entre módulos é um `uuid` sem chave estrangeira, e o módulo dono é chamado pela porta da frente |
| Esquema | Tudo no esquema `public`; o pg-boss fica no esquema `pgboss` |

## 2. Mapa: tabela, módulo dono e fatia

| Tabela | Módulo dono | Criada na fatia |
|---|---|---|
| `curriculum_framework`, `skill` | `modules/bncc` | F01 |
| `ai_usage_record` | `reusable/ai-usage` | F03 |
| `user`, `session`, `account`, `verification` | `reusable/auth` (geradas pelo Better Auth) | F05 |
| `class_context` | `modules/class-context` | F06 |
| `lesson_plan`, `plan_identification`, `lesson_plan_version`, `plan_skill`, `suggestion`, `plan_export` | `modules/plans` | F06 |
| `ai_generation` | `modules/plans` | F07 |
| `credit_entry`, `ai_budget_period` | `reusable/ai-usage` | F07 (reserva e acerto); F11 (crédito por pessoa) |
| `setting` | `reusable/settings` | F07 |
| `misuse_occurrence`, `misuse_block` | `reusable/misuse` | F09 |
| `invite`, `terms_acceptance` | `reusable/auth` | F10 |
| `credit_account` | `reusable/ai-usage` | F11 |
| `audit_event`, `access_record`, `deletion_receipt` | `reusable/audit` | F12 (a auditoria da F05 à F11 entra junto, pelos mesmos eventos) |
| `source` | `modules/sources` | F13 |
| `finding` | `modules/plans` | F15 |

A F00 não cria tabela de produto: cria os três papéis do banco e a primeira migração vazia, para provar o caminho.

## 3. Tabelas

Em todas: `id uuid` como chave primária e `created_at`, salvo quando dito. "→" indica chave estrangeira.

### 3.1 `modules/bncc`

**`curriculum_framework`**: um catálogo e a sua versão.

| Coluna | Tipo | Observação |
|---|---|---|
| `slug` | `text` | `bncc-computacao` |
| `name`, `version` | `text` | Único por (`slug`, `version`) |
| `source_url`, `source_checksum` | `text` | De onde veio e a conferência do arquivo importado |
| `is_current` | `boolean` | Só um catálogo atual por `slug` (índice único parcial) |
| `imported_at` | `timestamptz` | |

**`skill`**: uma habilidade.

| Coluna | Tipo | Observação |
|---|---|---|
| `framework_id` | `uuid` → `curriculum_framework` | |
| `code` | `text` | `EM13CO21`. Único por (`framework_id`, `code`) |
| `text` | `text` | Texto literal do documento oficial |
| `education_stage` | `enum` | `early_childhood`, `elementary_early`, `elementary_late`, `high_school` |
| `grades` | `smallint[]` | Anos a que se aplica. Vazio na educação infantil; `{1,2,3,4,5}` para um código `EF15`; `{1,2,3}` para `EM13` |
| `axis` | `enum` | `computational_thinking`, `digital_world`, `digital_culture` |
| `competence_number` | `smallint`, opcional | O "N" de "(Competência específica N)", quando o documento liga a habilidade a uma competência |
| `knowledge_object`, `explanation`, `examples` | `text`, opcionais | Texto oficial de apoio, usado no pedido à IA (RF-42) |
| `position` | `integer` | Ordem do documento |

Catálogo novo é uma linha nova em `curriculum_framework` e habilidades novas. As linhas antigas não mudam, então um plano antigo continua apontando para o texto que valia quando foi feito.

### 3.2 `modules/class-context`

**`class_context`**: a turma, descrita em conjunto.

| Coluna | Tipo | Observação |
|---|---|---|
| `owner_id` | `uuid` → `user` | |
| `label` | `text`, opcional | Apelido para o autor achar a turma na lista. **Não vai para a IA** (pode trazer o nome da turma) |
| `education_stage` | `enum` | Obrigatório (RF-11) |
| `grade` | `smallint`, opcional | Vazio na educação infantil |
| `subject` | `text`, opcional | A matéria da aula |
| `nominal_minutes` | `integer` | Obrigatório (RF-12) |
| `class_hour_minutes` | `integer` | 50 por padrão |
| `class_size` | `integer`, opcional | Quantos costumam estar presentes. Dimensiona as dinâmicas |
| `prior_knowledge`, `common_difficulties`, `social_context`, `school_resources`, `accessibility` | `text`, opcionais | Os textos agregados do RF-11 |
| `privacy_confirmations` | `jsonb` | Ver 3.3, "confirmação de privacidade" |
| `revision` | `integer` | Sobe a cada gravação, como no plano |
| `updated_at` | `timestamptz` | |

### 3.3 `modules/plans`

**`lesson_plan`**: o plano em edição.

| Coluna | Tipo | Observação |
|---|---|---|
| `owner_id` | `uuid` → `user` | |
| `class_context_id` | `uuid`, opcional, **sem** chave estrangeira | De qual contexto salvo o plano nasceu. Pode apontar para um contexto já excluído (RF-17) |
| `context_snapshot` | `jsonb` | Cópia do contexto usado. É o que vai para a IA |
| `template_key`, `template_version` | `text`, `integer` | `estagio-roteiro`, 1. O modelo é um arquivo de dados versionado, não uma tabela |
| `state` | `enum` | `draft`, `approved` |
| `version` | `integer` | Número da versão em trabalho; começa em 1 |
| `revision` | `integer` | Sobe a cada gravação. Evita que duas abas se atropelem (CONTRATOS, seção 2) |
| `theme`, `subtheme` | `text` | Vêm do autor. `theme` aparece na lista de planos |
| `nominal_minutes`, `effective_minutes`, `class_hour_minutes` | `integer` | O tempo desta aula |
| `exam_week` | `boolean` | Semana de provas (seção 4 do REQUISITOS) |
| `week_notes`, `teacher_notes` | `text`, opcionais | Carga da semana e a opinião do professor para esta aula |
| `source_material` | `text`, opcional | Material colado (RF-21). O worker lê daqui, porque o job só leva IDs (INV-11) |
| `content` | `jsonb` | Só o que o autor aceitou ou escreveu. Formato em CONTRATOS, seção 4 |
| `privacy_confirmations` | `jsonb` | Lista de `{fieldPath, textHash, confirmedAt}` |
| `approved_at` | `timestamptz`, opcional | |
| `updated_at` | `timestamptz` | |

**Confirmação de privacidade.** Quando a regra `privacy-hint` encontra um termo, o texto só é gravado se o autor corrigir ou confirmar que não identifica ninguém (RF-15). A confirmação guarda o resumo criptográfico (hash) do texto confirmado. Mudou o texto, a confirmação deixa de valer.

**`plan_identification`**: escola, professor, turma, data e turno. Uma linha por plano, em **tabela separada**.

| Coluna | Tipo | Observação |
|---|---|---|
| `plan_id` | `uuid` → `lesson_plan`, chave primária | Apagada junto com o plano |
| `education_office`, `school`, `city`, `teacher_name`, `class_label` | `text`, opcionais | |
| `lesson_date` | `date`, opcional | |
| `shift` | `text`, opcional | |

A consulta que monta o pedido à IA lê só `lesson_plan` e `plan_skill`. Não existe método de repositório que devolva a identificação junto com o conteúdo (INV-09).

**`lesson_plan_version`**: cópia imutável a cada aprovação.

| Coluna | Tipo | Observação |
|---|---|---|
| `plan_id` | `uuid` → `lesson_plan` | Único por (`plan_id`, `version`) |
| `version` | `integer` | |
| `snapshot` | `jsonb` | Tema, tempos, conteúdo, habilidades (código, texto e competência), fontes aprovadas (referência e link) e o modelo usado |
| `approved_at` | `timestamptz` | |

A identificação **não** entra na cópia. Ela não é conteúdo: mudar a data ou a turma não reabre um plano aprovado, e a exportação lê a identificação do momento.

**`plan_skill`**: habilidades do plano.

| Coluna | Tipo | Observação |
|---|---|---|
| `plan_id` | `uuid` → `lesson_plan` | Chave primária composta com `skill_id` |
| `skill_id` | `uuid` → `skill` | É esta chave que garante o INV-02 |
| `origin` | `enum` | `author` ou `ai` (sugestão aceita) |
| `position` | `integer` | |

**`suggestion`**: proposta da IA para um campo.

| Coluna | Tipo | Observação |
|---|---|---|
| `plan_id` | `uuid` → `lesson_plan` | |
| `generation_id` | `uuid` → `ai_generation` | |
| `field_path` | `text` | Nome do campo (`justification`, `activities`, …) ou `skills` |
| `value` | `jsonb` | O valor proposto para o campo inteiro. Em `skills`, `{skillId, reason}` |
| `origin` | `enum` | `from_material` ou `added` (RF-21) |
| `status` | `enum` | `pending`, `accepted`, `edited`, `rejected` |
| `decided_at` | `timestamptz`, opcional | |

- No máximo uma sugestão `pending` por (`plan_id`, `field_path`), fora `skills`. Em `skills`, uma por (`plan_id`, habilidade). Dois índices únicos parciais.
- Geração nova **apaga** as `pending` do mesmo campo antes de gravar as novas. As decididas ficam (DOMINIO, 4.3).

**`plan_export`**: evento de exportação.

| Coluna | Tipo | Observação |
|---|---|---|
| `plan_id` | `uuid` → `lesson_plan` | |
| `version_id` | `uuid` → `lesson_plan_version` | |
| `format` | `enum` | `markdown` na v1 |
| `included_ai_note` | `boolean` | Se o arquivo saiu com a nota "elaborado com apoio de IA" (RF-61) |

**`ai_generation`**: um pedido de geração e o seu andamento.

| Coluna | Tipo | Observação |
|---|---|---|
| `plan_id` | `uuid` → `lesson_plan` | |
| `owner_id` | `uuid` → `user` | |
| `operation` | `enum` | `generate_plan`; `evaluate_plan` na F15 |
| `status` | `enum` | `queued`, `running`, `succeeded`, `failed` |
| `stage` | `enum`, opcional | `checking_input`, `generating`, `validating`, `checking_output` |
| `failure_reason` | `enum`, opcional | Os nove motivos do DOMINIO, 4.2 |
| `failure_details` | `jsonb`, opcional | Só códigos e nomes de campo, nunca texto do autor |
| `prompt_version`, `model`, `effort` | `text` | Obrigatórios desde a criação (INV-13) |
| `request_id` | `uuid` | Liga a geração ao log (0010 D2) |
| `deadline_at` | `timestamptz` | Passou disso e ainda está ativa: é órfã (INV-08). Calculado na criação: tempo máximo do job vezes as tentativas da fila, mais uma folga (0008 D4) |
| `started_at`, `finished_at` | `timestamptz`, opcionais | |

- Índice único parcial em (`plan_id`) onde `status` é `queued` ou `running` (INV-05).
- Índice em (`status`, `deadline_at`), para a varredura de órfãs.

**`finding`** (F15): só os achados da avaliação por IA, que custam dinheiro. Os das regras são recalculados a cada leitura.

| Coluna | Tipo | Observação |
|---|---|---|
| `plan_id` | `uuid` → `lesson_plan` | |
| `generation_id` | `uuid` → `ai_generation` | A avaliação que o produziu |
| `plan_revision` | `integer` | A `revision` avaliada. Se o plano mudou, a tela mostra o achado como desatualizado |
| `rule_id`, `severity`, `field_path`, `message` | | Formato do DOMINIO, seção 5 |
| `suggestion_id` | `uuid` → `suggestion`, opcional | |

### 3.4 `modules/sources` (F13)

**`source`**: fonte trazida pelo autor.

| Coluna | Tipo | Observação |
|---|---|---|
| `owner_id` | `uuid` → `user` | |
| `plan_id` | `uuid`, **sem** chave estrangeira | Outro módulo. Excluir o plano chama `sources` na mesma transação |
| `url`, `domain` | `text` | Só `https` |
| `title`, `author`, `publisher`, `published_year` | `text`, opcionais | Para a referência em ABNT |
| `accessed_on` | `date` | "Acesso em" |
| `reference_abnt` | `text` | Montada pelo sistema (RF-54) |
| `status` | `enum` | `to_review`, `approved`, `rejected` |
| `link_check` | `enum` | `unchecked`, `ok`, `broken`; com `link_checked_at` |
| `domain_listed` | `boolean` | Está na lista de domínios aceitos? |
| `off_list_justification` | `text`, opcional | Obrigatória para manter fonte fora da lista (RF-53) |

A lista de domínios aceitos é um arquivo de dados versionado no repositório, como o modelo do plano.

### 3.5 `reusable/auth`

**`user`, `session`, `account`, `verification`**: criadas pelo gerador do Better Auth, não à mão. A Docenda acrescenta ao usuário:

| Campo extra | Tipo | Observação |
|---|---|---|
| `role` | `enum` | `author`, `admin` |
| `status` | `enum` | `active`, `deactivated` |
| `deactivated_at` | `timestamptz`, opcional | Início dos 30 dias (RF-07) |
| `last_login_at` | `timestamptz`, opcional | Para achar contas paradas há 6 meses |

**`invite`** (F10)

| Coluna | Tipo | Observação |
|---|---|---|
| `email` | `text`, opcional | **Apagado** no aceite, no cancelamento e no vencimento (REQ-17) |
| `token_hash` | `text` | O token do link nunca é guardado em claro (RF-78) |
| `status` | `enum` | `pending`, `accepted`, `cancelled`, `expired` |
| `invited_by` | `uuid` → `user`, opcional | |
| `expires_at`, `closed_at` | `timestamptz` | |

**`terms_acceptance`** (F10): `user_id` → `user`, `terms_version`, `accepted_at` (RF-01).

### 3.6 `reusable/ai-usage`

**`ai_usage_record`**: uma linha por chamada ao provedor, sem conteúdo (RF-74).

| Coluna | Tipo | Observação |
|---|---|---|
| `user_id` | `uuid` → `user`, opcional | Na exclusão da conta vira `NULL`: o número fica, a pessoa some (seção 6) |
| `reference_id` | `uuid`, opcional, sem chave estrangeira | A Docenda passa o ID da geração. O módulo não sabe o que é uma geração |
| `operation`, `prompt_version`, `model`, `effort` | `text` | INV-13 |
| `input_tokens`, `output_tokens`, `cache_read_tokens`, `cache_write_tokens` | `bigint` | |
| `cost_nano` | `bigint` | |
| `price_table_version` | `text` | Qual tabela de preços fez a conta |
| `duration_ms`, `retries` | `integer` | |
| `stop_reason` | `text` | |

**`credit_entry`**: o livro-razão (DOMINIO, seção 6). Só recebe linhas novas.

| Coluna | Tipo | Observação |
|---|---|---|
| `user_id` | `uuid` → `user` | |
| `period` | `char(7)` | `2026-11`, no horário de Brasília |
| `kind` | `enum` | `grant`, `reserve`, `settle` |
| `amount_nano` | `bigint` | Com sinal: `grant` e `settle` somam, `reserve` subtrai |
| `reference_id` | `uuid`, opcional | O ID da geração |

- Único em (`user_id`, `period`) onde `kind = 'grant'`: o crédito do mês não entra duas vezes.
- Único em (`reference_id`) onde `kind = 'reserve'`, e outro onde `kind = 'settle'`: um worker que repete não reserva nem acerta em dobro.

**`credit_account`** (F11): `user_id` como chave primária. É a linha travada na reserva para que dois pedidos da mesma pessoa entrem um atrás do outro (INV-06).

**`ai_budget_period`**: o gasto do sistema no mês.

| Coluna | Tipo | Observação |
|---|---|---|
| `period` | `char(7)`, chave primária | |
| `committed_nano` | `bigint` | Reservado mais gasto. É a linha travada para conferir o teto (RF-72) |
| `alert_80_sent_at` | `timestamptz`, opcional | Para o aviso de 80% sair uma vez só |

O valor do teto não fica aqui: é lido de `setting` na hora da reserva, então mudar o teto vale no pedido seguinte.

### 3.7 `reusable/settings`

**`setting`**: `key text` como chave primária, `value jsonb`, `updated_at`, `updated_by`. Chaves da v1: `ai.enabled`, `ai.monthly_ceiling_nano`, `ai.monthly_credit_nano`, `ai.daily_generation_limit`, `ai.per_minute_limit`, `ai.models`, `signup.open`, `signup.max_active_accounts`. Cada chave tem um valor padrão no código; a tabela guarda só o que foi mudado (RF-75).

### 3.8 `reusable/misuse` (F09)

**`misuse_occurrence`**

| Coluna | Tipo | Observação |
|---|---|---|
| `user_id` | `uuid` → `user` | |
| `category` | `enum` | A lista do `input_check` ([IA](IA.md), seção 4.1), que detalha a seção 4.1 do REQUISITOS |
| `severity` | `enum` | `minor`, `serious_review`, `serious_auto` |
| `detected_by` | `enum` | `input_check`, `output_check` |
| `excerpt` | `text` | O trecho mínimo que motivou (REQ-05) |
| `reference_id` | `uuid`, opcional | ID da geração |
| `decision` | `enum`, opcional | `block`, `unblock`, `dismiss`; com `decided_by`, `decision_reason`, `decided_at` (RF-93) |

**`misuse_block`**: o bloqueio da IA para uma pessoa. Não desativa a conta (DOMINIO, 4.5).

| Coluna | Tipo | Observação |
|---|---|---|
| `user_id` | `uuid` → `user` | No máximo um bloqueio ativo por pessoa (índice único parcial onde `lifted_at` é nulo) |
| `occurrence_id` | `uuid` → `misuse_occurrence` | |
| `review_requested_at`, `review_request` | | O pedido de revisão do usuário (RF-94) |
| `lifted_at`, `lifted_by` | | |

### 3.9 `reusable/audit` (F12)

**`audit_event`**: quem fez o quê. Só IDs, nenhuma chave estrangeira.

| Coluna | Tipo | Observação |
|---|---|---|
| `type` | `text` | Lista fechada no código, por exemplo `auth.login_succeeded`, `plan.approved`, `misuse.block_applied`, `settings.changed` (REQ-22) |
| `actor_id`, `target_id` | `uuid`, opcionais | Em falha de login não há ator: o e-mail tentado **não** é guardado (0006 D4) |
| `target_type` | `text`, opcional | |
| `request_id` | `uuid` | |

**`access_record`**: `user_id uuid` (sem chave estrangeira), `ip inet`, `created_at`. Uma linha por login com sucesso (0010 D5).

**`deletion_receipt`**: `requested_at`, `completed_at`, `outcome`. Sem ID de usuário, nome nem e-mail: prova que o pedido foi atendido (REQUISITOS, 7.1).

## 4. O que no banco garante cada invariante

| Invariante | Mecanismo no banco |
|---|---|
| INV-01 dono | `owner_id` obrigatório em toda tabela do autor; índice começando por `owner_id` |
| INV-02 só habilidade do catálogo | Chave estrangeira `plan_skill.skill_id` → `skill` |
| INV-04 a IA não escreve no plano | O worker usa um repositório que só tem `suggestion` e `ai_generation`; `lesson_plan.content` só muda nas ações do autor |
| INV-05 uma geração ativa por plano | Índice único parcial em `ai_generation` |
| INV-06 saldo e teto | `SELECT … FOR UPDATE` em `credit_account` e em `ai_budget_period`, dentro da transação da reserva |
| INV-07 custo nunca menor que o real | Sem `settle`, a reserva inteira fica; `settle` é único por geração |
| INV-08 nada roda para sempre | `deadline_at` e o índice da varredura |
| INV-09 identificação fora da IA | `plan_identification` é outra tabela, lida por outro método |
| INV-12 dinheiro inteiro | Só `bigint` |
| INV-13 prompt, modelo e esforço | `NOT NULL` nas três colunas |
| INV-14 auditoria só cresce | Permissões do papel `docenda_app` (seção 5) |
| Versão aprovada imutável | `docenda_app` sem `UPDATE` em `lesson_plan_version` |

INV-03, INV-10 e INV-11 são garantidos no código (função pura de aprovação, passo do pipeline e tipo do job), não no banco.

## 5. Papéis e permissões (0012 D5)

| Tabela | `docenda_app` (API e worker) | `docenda_retention` (jobs de limpeza) |
|---|---|---|
| `audit_event`, `access_record` | Inserir e ler | Apagar o que venceu |
| `lesson_plan_version`, `credit_entry`, `deletion_receipt` | Inserir e ler | — |
| `curriculum_framework`, `skill` | Ler | — |
| Todas as outras | Ler, inserir, alterar e apagar | — |

- O catálogo só é escrito pelo comando de importação, que roda como `docenda_owner` junto com as migrações.
- Apagar um plano ou uma conta remove as linhas filhas por `ON DELETE CASCADE`. No PostgreSQL essa ação roda com a permissão do dono da tabela, então o `docenda_app` não precisa de `DELETE` em `lesson_plan_version` nem em `credit_entry`. **A conferir com um teste na F06.**

## 6. Exclusão e retenção

O que acontece com cada tabela quando a conta é apagada, lado a lado com a seção 7.1 do REQUISITOS.

| Tabela | Na exclusão da conta | Como |
|---|---|---|
| `user`, `session`, `account`, `verification`, `terms_acceptance` | Apagado | Linha do usuário apagada; as outras em cascata |
| `class_context`, `lesson_plan` e tudo o que pende dele, `source` | Apagado | `ON DELETE CASCADE` a partir de `owner_id` |
| `credit_entry`, `credit_account`, `misuse_occurrence`, `misuse_block` | Apagado | Cascata |
| `ai_usage_record` | **Anonimizado** | `ON DELETE SET NULL` em `user_id` |
| `ai_budget_period` | Fica | Não tem ligação com pessoa |
| `access_record`, `audit_event` | Ficam até 6 meses | Sem chave estrangeira; o job de retenção apaga |
| `deletion_receipt` | Fica | Não identifica ninguém |
| Jobs do pg-boss | Nada a apagar | Só levam IDs e somem em 1 dia (0008 D6) |

A exclusão é **pela forma**: apagada a linha do usuário, o banco não deixa sobrar conteúdo dele. Um teste confere que nenhuma tabela do autor tem `owner_id` sem cascata.

Limpezas agendadas (jobs do pg-boss):

| Job | Frequência | Apaga ou muda | Papel |
|---|---|---|---|
| Gerações órfãs | A cada minuto | Fecha como `failed` e `worker_lost` | `docenda_app` |
| Convites vencidos | Diária | `status = expired` e e-mail apagado | `docenda_app` |
| Contas desativadas há mais de 30 dias | Diária | Apaga a conta (tabela acima) | `docenda_app` |
| Contas paradas há 6 meses | Diária | Avisa por e-mail e desativa | `docenda_app` |
| Auditoria e registros de acesso com mais de 6 meses | Diária | Apaga | `docenda_retention` |
| Ocorrências de uso indevido com mais de 90 dias (proposta) | Diária | Apaga | `docenda_app` |

## 7. Decisões de modelagem

Tomadas ao escrever este documento, em 17/11/2026, dentro do que as decisões 0001 a 0014 já permitiam.

| # | Decisão | Motivo |
|---|---|---|
| D1 | Exclusão de conta por cascata do banco, não por uma lista de `DELETE` no código | Código esquece tabela nova; a chave estrangeira não esquece |
| D2 | Identificação do plano em tabela separada | A consulta da IA não tem como trazê-la por engano |
| D3 | A identificação fica fora da versão aprovada | Trocar a data da aula não deveria reabrir o plano |
| D4 | Carga da semana, semana de provas e opinião do professor ficam no plano, não no contexto salvo | O contexto é reaproveitado por meses; "semana de provas" vale para uma aula |
| D5 | `class_size` no contexto | A rubrica confere se as dinâmicas cabem na turma, e isso pede um número |
| D6 | Texto com alerta de privacidade não é gravado sem correção ou confirmação | "Nenhum dado de aluno gravado" passa a ser regra da gravação, não só um aviso |
| D7 | Sugestão pendente substituída é apagada, não marcada | Ela nunca foi decidida; guardar não serve a ninguém |
| D8 | Modelo do plano e lista de domínios como arquivos de dados versionados | Na v1 só o desenvolvedor muda; tabela pediria tela de administração |
| D9 | Teto do mês lido de `setting` na reserva | Mudar o teto vale no pedido seguinte (RF-75) |
| D10 | Reserva e acerto únicos por geração, por índice | Repetição do worker não cobra duas vezes |
| D11 | `revision` no plano | Salvamento automático em duas abas não perde texto em silêncio |
| D12 | Excluir plano com geração ativa é recusado | Evita reserva sem dono e worker lendo plano que sumiu |

## 8. A confirmar na fatia

| Ponto | Fatia |
|---|---|
| Prisma 7 com um arquivo de esquema por módulo e `uuidv7()` como padrão do banco | F00 |
| De qual fonte oficial sai o catálogo e se toda habilidade tem competência, objeto de conhecimento, explicação e exemplos | F01 |
| Better Auth com `uuid` como identificador, para as chaves estrangeiras de `owner_id` | F05 |
| Better Auth grava IP e navegador na sessão por padrão. A 0010 quer o IP só em `access_record`: desligar, se a opção existir sem quebrar o limite por IP; senão, registrar as duas colunas no inventário de dados | F05 |
| A cascata apaga `lesson_plan_version` e `credit_entry` com o `docenda_app` sem `DELETE` | F06 |
| Retenção de 90 dias das ocorrências de uso indevido (REQUISITOS, ponto em aberto 10) | F09 |
