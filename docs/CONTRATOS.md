# Contratos da API: Docenda

> **v0.1 (17/11/2026). Planejado: ainda não há código.** Diz o que entra e o que sai de cada rota. Vira os esquemas Zod de `packages/contracts`, que valem para a API e para a tela (0006 D5, 0013 D3). A especificação de cada fatia pode ajustar um detalhe; quando ajusta, corrige este arquivo no mesmo PR.
> Irmãos: [INTERFACE](INTERFACE.md) (mapa de telas e rotas), [DADOS](DADOS.md) (tabelas) e [DOMINIO](DOMINIO.md) (nomes e regras).

## 1. Regras gerais

| Assunto | Regra |
|---|---|
| Endereço | Tudo sob `/api/v1`, menos `/healthz`, `/readyz` e `/api/auth/*` |
| Formato | JSON em UTF-8, nomes em `camelCase`. Campo desconhecido no pedido é recusado |
| Identificador | `uuid` em texto |
| Data e hora | ISO 8601 em UTC (`2026-11-17T12:00:00Z`). Data sem hora: `2026-11-17` |
| Dinheiro | Nano-dólar como **texto** (`"1000000000"`), nunca número: o JSON perde precisão em inteiros grandes (INV-12) |
| Texto | Puro. Nenhum campo aceita nem devolve HTML ou Markdown (0006) |
| Sessão | Cookie do Better Auth. Sem sessão: `401`. Todas as rotas exigem sessão, menos as de autenticação e as de saúde |
| Dono | Recurso de outro autor responde `404`, igual a recurso que não existe (RNF-40) |
| Pedido que muda dado | Confere o cabeçalho `Origin` (0006 D5) |
| Resposta | Sempre com `X-Request-Id` e `Cache-Control: no-store` |
| Tamanho | Corpo de até 256 kB; acima disso, `413` (RNF-44) |

## 2. Duas pessoas, duas abas, dois cliques

- **`revision`.** Todo plano e todo contexto tem um número que sobe a cada gravação. Quem altera manda a `revision` que leu. Se ela já mudou, a resposta é `409 revision_conflict` com o estado atual, e a tela decide o que mostrar. Sem isso, o salvamento automático de uma aba apagaria em silêncio o que a outra escreveu.
- **Clique duplo na geração** (RNF-37). O segundo `POST` devolve a geração que já está ativa, com o mesmo `id`. Quem garante é o índice único do banco (INV-05), não a tela.
- **Decisão repetida.** Decidir uma sugestão que já foi decidida responde `409 suggestion_already_decided`.

## 3. Erros

Formato de problema (RFC 9457), com `Content-Type: application/problem+json`:

```json
{
  "type": "https://docenda.example/problems/validation-failed",
  "title": "Dados inválidos",
  "status": 422,
  "code": "validation_failed",
  "requestId": "0193…",
  "errors": [{ "path": "time.effectiveMinutes", "code": "too_small", "message": "Informe um tempo maior que zero." }]
}
```

- `code` é estável e é o que a tela usa para escolher a mensagem. `title` e `message` são em português.
- `errors` só aparece em `validation_failed`. `findings` só em `plan_not_approvable`. `fields` só em `privacy_alert`.
- Nunca há rastro de pilha, nome de tabela ou mensagem interna (0012 D7).

| `code` | Status | Quando |
|---|---|---|
| `validation_failed` | 422 | O corpo não passa no esquema |
| `malformed_request` | 400 | JSON quebrado |
| `unauthenticated` | 401 | Sem sessão ou sessão vencida. A tela abre o diálogo de entrar (0013 D7) |
| `forbidden` | 403 | Rota de administrador sem o papel |
| `not_found` | 404 | Não existe ou é de outro autor |
| `payload_too_large` | 413 | Corpo acima do limite |
| `rate_limited` | 429 | Limite geral de pedidos (RNF-44), com `Retry-After` |
| `revision_conflict` | 409 | Seção 2 |
| `privacy_alert` | 409 | Um texto parece ter dado de aluno (RF-15). Traz `fields: [{fieldPath, terms}]` |
| `suggestion_pending` | 409 | Tentou escrever num campo que tem sugestão sem decisão |
| `suggestion_already_decided` | 409 | Seção 2 |
| `plan_approved` | 409 | Tentou editar um plano aprovado sem reabrir |
| `plan_not_approvable` | 409 | Há achado que bloqueia. Traz `findings` |
| `no_approved_version` | 409 | Exportar sem versão aprovada (cenário F6.2) |
| `generation_active` | 409 | Excluir ou reabrir um plano com geração em andamento |
| `skill_not_found` | 422 | Código fora do catálogo (cenário F4.2) |
| `email_not_verified` | 403 | Pedido de IA sem e-mail confirmado (RF-70) |
| `ai_blocked` | 403 | IA bloqueada para a pessoa por uso indevido (RF-91) |
| `ai_disabled` | 503 | IA desligada pelo administrador (RF-75) |
| `ai_budget_reached` | 503 | Teto do mês atingido (RF-72) |
| `ai_credit_exhausted` | 429 | Sem saldo no mês (RF-71) |
| `ai_rate_limited` | 429 | Limite por minuto ou por dia (RF-71), com `Retry-After` |
| `input_too_large` | 422 | Entrada acima do limite da operação (RF-73) |
| `internal_error` | 500 | Defeito. Só o `requestId` aparece |

As falhas **durante** a geração não são erros de HTTP: o pedido já respondeu `202`. Elas aparecem em `failureReason` (seção 4.6).

## 4. Formatos

Notação de TypeScript. `?` é opcional; `| null` é "pode estar vazio".

### 4.1 Conteúdo do plano (`schemaVersion: 1`)

```ts
type PlanContent = {
  schemaVersion: 1;
  justification: string;
  generalObjective: string;
  specificObjectives: string[];
  contents: string[];
  priorKnowledge: string;
  activities: Activity[];
  resources: string[];
  assessment: { criteria: string[]; instruments: string[] };
  adaptations: string;
  crossSubjectLink: string;
};

type Activity = {
  moment: 'introduction' | 'development' | 'systematization' | 'closing';
  title: string;
  description: string;
  minutes: number;        // inteiro, maior que zero
  setupHeavy: boolean;    // instalar, criar conta ou configurar (RF-14)
};
```

- **Rascunho aceita vazio.** Texto vazio e lista vazia são válidos até a aprovação; quem cobra é a regra `empty-required-field`.
- **Unidades de sugestão** (RF-23): cada chave de `PlanContent` é uma unidade, e `assessment` é uma só. Dez unidades, mais as habilidades, que são decididas uma a uma.
- `theme`, `subtheme` e os tempos ficam fora de `PlanContent`: vêm do autor e nunca são sugeridos.

### 4.2 Contexto

```ts
type ContextFields = {
  educationStage: 'early_childhood' | 'elementary_early' | 'elementary_late' | 'high_school';
  grade: number | null;
  subject: string | null;
  nominalMinutes: number;
  classHourMinutes: number;      // 50 por padrão
  classSize: number | null;
  priorKnowledge: string | null;
  commonDifficulties: string | null;
  socialContext: string | null;
  schoolResources: string | null;
  accessibility: string | null;
};

type ClassContext = ContextFields & { id: string; label: string | null; revision: number; updatedAt: string };

type ContextSnapshot = ContextFields & { schemaVersion: 1 };   // sem `label`
```

### 4.3 Plano

```ts
type Plan = {
  id: string;
  state: 'draft' | 'approved';
  version: number;
  revision: number;
  theme: string;
  subtheme: string | null;
  time: { nominalMinutes: number; effectiveMinutes: number; classHourMinutes: number };
  lesson: { examWeek: boolean; weekNotes: string | null; teacherNotes: string | null };
  sourceMaterial: string | null;
  classContextId: string | null;
  context: ContextSnapshot;
  content: PlanContent;
  skills: PlanSkill[];
  sources: Source[];                       // vazio até a F13
  identification: PlanIdentification;
  suggestions: Suggestion[];               // só as pendentes
  findings: Finding[];
  activeGeneration: Generation | null;
  lastGeneration: Generation | null;       // para mostrar a mensagem da última falha
  approvedVersions: { version: number; approvedAt: string; lastExportedAt: string | null }[];
  createdAt: string;
  updatedAt: string;
};

type PlanSummary = Pick<Plan, 'id' | 'state' | 'version' | 'theme' | 'updatedAt'> & {
  educationStage: ContextFields['educationStage'];
  grade: number | null;
  hasActiveGeneration: boolean;
};

type PlanIdentification = {
  educationOffice: string | null; school: string | null; city: string | null;
  teacherName: string | null; classLabel: string | null;
  lessonDate: string | null; shift: string | null;
};
```

A identificação aparece para o autor, que é o dono dela. Ela só não existe no tipo que a porta de IA recebe (INV-09).

### 4.4 Habilidade

```ts
type Skill = {
  code: string;                 // "EM13CO21"
  text: string;                 // literal
  formatted: string;            // "(EM13CO21) Texto literal. (Competência específica 6)" (RF-41)
  educationStage: ContextFields['educationStage'];
  grades: number[];
  axis: 'computational_thinking' | 'digital_world' | 'digital_culture';
  competenceNumber: number | null;
};

type PlanSkill = Skill & { origin: 'author' | 'ai' };
```

`formatted` é montado pelo servidor a partir do catálogo. A tela nunca monta esse texto, então não tem como alterá-lo.

### 4.5 Sugestão e achado

```ts
type Suggestion = {
  id: string;
  fieldPath: keyof Omit<PlanContent, 'schemaVersion'> | 'skills';
  value: unknown;               // o valor do campo inteiro; em 'skills', { skill: Skill, reason: string }
  origin: 'from_material' | 'added';
  status: 'pending';
  generationId: string;
};

type Finding = {
  ruleId: string;               // lista no DOMINIO, seção 5
  severity: 'block' | 'warn' | 'info';
  fieldPath: string | null;
  message: string;
  suggestionId: string | null;
  stale: boolean;               // achado de avaliação feito sobre uma revisão antiga
};
```

### 4.6 Geração

```ts
type Generation = {
  id: string;
  operation: 'generate_plan' | 'evaluate_plan';
  status: 'queued' | 'running' | 'succeeded' | 'failed';
  stage: 'checking_input' | 'generating' | 'validating' | 'checking_output' | null;
  failureReason:
    | 'input_blocked_privacy' | 'input_blocked_misuse' | 'off_topic' | 'provider_refusal'
    | 'invalid_output' | 'output_check_failed' | 'budget_exhausted'
    | 'provider_unavailable' | 'worker_lost' | null;
  failureDetails: { fields?: string[] } | null;   // só nomes de campo
  createdAt: string;
  finishedAt: string | null;
};
```

### 4.7 Limites de tamanho (valores iniciais, em configuração)

| Campo | Máximo |
|---|---|
| `theme`, `subtheme`, `label` | 200 caracteres |
| Cada texto do contexto; `weekNotes`, `teacherNotes` | 1.500 |
| `sourceMaterial` | 12.000 |
| Cada texto de `PlanContent`; `description` da atividade | 3.000 |
| Item de lista; `title` da atividade | 500 |
| Itens por lista | 12 |
| Atividades | 10 |
| Habilidades por plano | 6 |
| Cada campo da identificação | 150 |

Esses limites são a primeira barreira do RF-73. A segunda é a contagem de tokens antes da chamada paga ([IA](IA.md), seção 6). A F04 mede os valores com os contextos de teste e os ajusta.

## 5. Rotas

### 5.1 Saúde (F00)

| Rota | Resposta |
|---|---|
| `GET /healthz` | `200 {"status":"ok"}` se o processo está vivo |
| `GET /readyz` | `200` se o banco responde; `503` se não |

### 5.2 Catálogo (F01)

| Rota | Entrada | Resposta |
|---|---|---|
| `GET /skills` | `stage` (obrigatório), `grade?`, `q?` (busca no código e no texto) | `200 { frameworkVersion, items: Skill[] }` |
| `GET /skills/:code` | — | `200 Skill & { explanation, examples, knowledgeObject }`; `404` |

### 5.3 Conta e sessão (F05)

| Rota | Entrada | Resposta |
|---|---|---|
| `/api/auth/*` | Rotas do Better Auth: entrar e sair | Definidas pela biblioteca; a confirmar na F05 |
| `GET /me` | — | `200 { id, name, email, role, emailVerified, aiBlocked }` |

Entrar com dados errados devolve sempre a mesma mensagem, "E-mail ou senha incorretos" (RF-03).

### 5.4 Contexto da turma (F06)

| Rota | Entrada | Resposta | Erros próprios |
|---|---|---|---|
| `GET /class-contexts` | — | `200 { items: ClassContext[] }`, do mais recente para o mais antigo | |
| `POST /class-contexts` | `ContextFields & { label?, privacyConfirmations? }` | `201 ClassContext` | `privacy_alert` |
| `GET /class-contexts/:id` | — | `200 ClassContext` | |
| `PATCH /class-contexts/:id` | `{ revision, …campos, privacyConfirmations? }` | `200 ClassContext` | `revision_conflict`, `privacy_alert` |
| `DELETE /class-contexts/:id` | — | `204`. Os planos feitos com ele ficam (RF-17) | |

**Alerta de privacidade.** Se um texto tiver termo da lista (por exemplo "laudo", "CID", "diagnosticado"), nada é gravado e a resposta é `409 privacy_alert` com os campos e os termos. O autor corrige o texto ou reenvia com `privacyConfirmations: ["commonDifficulties"]`, declarando que o trecho não identifica ninguém. Vale o mesmo no plano.

### 5.5 Plano (F06)

| Rota | Entrada | Resposta | Erros próprios |
|---|---|---|---|
| `GET /plans` | `cursor?`, `limit?` (20 por padrão, 50 no máximo) | `200 { items: PlanSummary[], nextCursor }` | |
| `POST /plans` | `{ classContextId, theme, subtheme?, time: { effectiveMinutes, nominalMinutes?, classHourMinutes? }, lesson?, sourceMaterial?, privacyConfirmations? }` | `201 Plan`, em rascunho, com a cópia do contexto | `privacy_alert` |
| `GET /plans/:id` | — | `200 Plan` | |
| `PATCH /plans/:id` | `{ revision, theme?, subtheme?, time?, lesson?, sourceMaterial?, context?, content?, privacyConfirmations? }` | `200 Plan` | `revision_conflict`, `privacy_alert`, `suggestion_pending`, `plan_approved` |
| `PUT /plans/:id/identification` | `PlanIdentification` | `200 Plan`. Vale em qualquer estado e não muda a `version` | |
| `POST /plans/:id/skills` | `{ code }` | `201 Plan` | `skill_not_found`, `plan_approved` |
| `DELETE /plans/:id/skills/:code` | — | `200 Plan` | `plan_approved` |
| `POST /plans/:id/suggestions/:sid/decision` | `{ decision: 'accept' }`, `{ decision: 'reject' }` ou `{ decision: 'edit', value }` | `200 Plan` | `suggestion_already_decided`, `privacy_alert` |
| `POST /plans/:id/approve` | `{ revision }` | `200 Plan`, agora `approved`, com a versão nova em `approvedVersions` | `plan_not_approvable`, `revision_conflict` |
| `POST /plans/:id/reopen` | — | `200 Plan`, em rascunho, com `version + 1` | `generation_active` |
| `POST /plans/:id/duplicate` | — | `201 Plan` novo, em rascunho, versão 1 | |
| `DELETE /plans/:id` | — | `204` | `generation_active` |

Regras:
- **`PATCH` troca o campo inteiro.** `content` leva só as chaves alteradas, e cada chave substitui o valor antigo. A tela salva um campo por vez, com um pequeno atraso (0013 D7).
- **Campo com sugestão pendente não aceita `PATCH`.** O autor decide a sugestão primeiro: aceita, edita a partir dela ou recusa. Assim nenhuma sugestão some sem decisão.
- **Plano aprovado é só leitura.** Para mudar, o autor reabre (`reopen`). A versão aprovada fica guardada e continua exportável (RF-25).
- **Duplicar** copia tema, tempos, contexto, conteúdo aceito, habilidades e fontes aprovadas. Não copia a identificação, as sugestões pendentes nem as versões.
- **Decidir uma habilidade sugerida:** `accept` cria a ligação com o catálogo; `edit` não existe para habilidades, porque o texto é literal.
- `findings` vem em toda resposta com `Plan`, já recalculado.

### 5.6 Geração (F07)

| Rota | Entrada | Resposta | Erros próprios |
|---|---|---|---|
| `POST /plans/:id/generations` | — | `202 Generation`. Se já há uma ativa, a mesma | `plan_approved`, `email_not_verified`, `ai_blocked`, `ai_disabled`, `ai_budget_reached`, `ai_credit_exhausted`, `ai_rate_limited`, `input_too_large` |
| `GET /plans/:id/generations/:gid` | — | `200 Generation` | |

A tela pergunta o andamento a cada 2 s enquanto `status` for `queued` ou `running` (0008 D3). Ao terminar, lê o plano de novo: as sugestões vêm em `Plan.suggestions`.

### 5.7 Exportação (F08)

| Rota | Entrada | Resposta | Erros próprios |
|---|---|---|---|
| `POST /plans/:id/exports` | `{ version?, format: 'md', includeAiNote? }`. Sem `version`, a última aprovada; `includeAiNote` vale `true` se não vier (RF-61) | `200` com `Content-Type: text/markdown; charset=utf-8` e `Content-Disposition: attachment` | `no_approved_version`, `not_found` |

É `POST` porque exportar cria um evento (`plan_export`). O arquivo é montado pelo servidor a partir da versão aprovada e da identificação do momento.

### 5.8 Marco M2 (esboço; fecha na especificação de cada fatia)

| Rota | Fatia | Para quê |
|---|---|---|
| `GET /me/ai-credit` | F11 | `{ period, monthlyCreditNano, availableNano, generationsLeftToday, aiEnabled }` |
| `POST /invites/:token/accept` | F10 | Cria a conta com o e-mail convidado, a senha e o aceite dos termos |
| `PATCH /me`; troca de e-mail e de senha pelas rotas do Better Auth | F10 | RF-05, RF-06 |
| `POST /me/deletion`, `POST /me/deletion/confirm` | F14 | Os três passos do RF-07 |
| `GET, POST /plans/:id/sources`; `PATCH, DELETE /plans/:id/sources/:sid` | F13 | Fonte trazida pelo autor |
| `POST /plans/:id/evaluations` | F15 | Avaliação por IA; devolve uma `Generation` com `operation: 'evaluate_plan'` |
| `POST /me/ai-block/review-request` | F09 | Pedido de revisão do bloqueio (RF-94) |
| `GET, POST /admin/invites`; `DELETE /admin/invites/:id` | F12 | RF-78 |
| `GET /admin/usage` | F12 | Custo por operação, por modelo e por dia (RNF-14, RNF-39) |
| `GET /admin/occurrences`; `POST /admin/occurrences/:id/decision` | F12 | RF-93 |
| `GET, PUT /admin/settings/:key` | F12 | RF-75 |

Nenhuma rota de administrador devolve conteúdo de plano ou de contexto (REQ-16).
