# AI: pipeline, prompts and evaluation

> **v0.2 (17/11/2026).** Born from the planning review of 16/11; v0.2 adds the response formats (section 4.1), the prompt drafts and the link to [EVALUATION](EVALUATION.md). It details what decision [0005](decisions/0005-ai-provider.md) left to the "AI and evaluation area". API facts checked against the Anthropic documentation on 16/11/2026.
> Domain terms: [DOMAIN](DOMAIN.md). Queue and credit: [0008](decisions/0008-ai-job-queue-postgresql.md).

## 1. Principles

1. **The AI suggests, the author decides.** The AI only creates suggestions; it never writes to the plan, never approves (INV-03, INV-04).
2. **Impossible is better than detected.** Wherever possible, the error is stopped by construction, not by a later check: a skill code outside the catalog fits neither the output schema nor the database.
3. **User text is data, never instruction.** Theme, context and pasted material go in marked as data (RNF-08).
4. **When in doubt, it does not pass.** A check that fails, refuses or returns something invalid counts as "needs review", never as "passed".
5. **Everything that costs is measured.** Each call stores tokens, cost, time, model and prompt version (INV-13).
6. **A prompt or model change goes through the evaluation** before it goes in (section 7).

## 2. Operations

| Operation | Model (configurable) | Input | Output | Milestone |
|---|---|---|---|---|
| `input_check` | Haiku 5.5, effort `low` | Theme, context and pasted material, after cleaning | `studentData` (`none` or `suspected`), `misuse` (`none`, `off_topic`, `minor` or `serious`), category, minimal excerpt and a short reason | M2 (in M1, only the word list) |
| `generate_plan` | Sonnet 5.5, effort `medium` | Theme, aggregate context, material, plan template, the education stage's skills | Plan fields (DOMAIN, section 3), each with its origin (`from_material` or `added`); or `off_topic`, when the request is not a lesson | M1 |
| `output_check` | Haiku 5.5, effort `low` | The generated response | `ok` or `needs_review`, with a reason | M2 |
| `evaluate_plan` | Sonnet 5.5, effort `medium` | The plan, the context snapshot and the rubric | A list of `warn` or `info` findings, with an optional suggestion. **Never `block`:** the AI does not stop approval | M2 |

Two changes relative to 0005, recorded there in a note of 16/11:
- **The two input checks become a single call** (`input_check`), with multi-label output. It costs less and waits less, and the test set is the same.
- **The skill is chosen inside `generate_plan`**, not in a separate call. The output schema already limits the codes (section 4).

## 3. The path of a generation

Each step is a function `(state) → state or stop`, tested on its own with the fake adapter. The service that chains the steps does not know whether it was called by the worker, by a command or by a test.

| # | Step | Uses AI? | If it fails |
|---|---|---|---|
| 1 | **Clean the text:** Unicode normalization (NFKC) and removal of invisible characters; a block that looks encoded becomes "needs review" | No | `input_blocked_misuse` (mild) |
| 2 | **Privacy alert, 1st layer:** term list (RF-15) | No | Stops before any call (INV-10) |
| 3 | **`input_check`:** student data and misuse, in any language | Haiku | Reason according to the label; a refusal or invalid output counts as "needs review" |
| 4 | **Build the request:** fixed prefix and variable part (section 5). The plan identification does not exist in the input type (INV-09) | No | Programming error |
| 5 | **`generate_plan`:** output bound to the schema | Sonnet | Refusal: `provider_refusal`. Invalid JSON: retries up to 2 times |
| 6 | **Validate:** full Zod schema, skills in the catalog, minutes | No | `invalid_output` |
| 7 | **`output_check`** | Haiku | `output_check_failed`; nothing appears to the author |
| 8 | **Store suggestions** and close the generation | No | Transaction: either everything is stored, or nothing |

On each call (3, 5 and 7): the cost is settled from `response.usage` and an `AiUsageRecord` is written.

**A provider refusal is not misuse.** Models refuse by category (for example, cybersecurity), and a Computing lesson talks about passwords, scams and intrusion. A refusal becomes `provider_refusal`, with a clear message and no occurrence recorded. In `generate_plan`, the provider's own retry on another model (`fallbacks`, 0005 D4) stays on.

## 4. Output schema

- **A single source:** the plan's Zod schema lives in `packages/contracts`. From it come the JSON Schema sent to the model (`output_config.format`), the response validation, the `jsonb` format and the screen's types.
- **Skills as a closed list.** The skills field accepts only the codes of the class's education stage, as an `enum`. The model cannot return a code that does not exist. There is one schema per education stage; the provider compiles each schema once and keeps it for 24 hours.
- **What the provider's schema does not accept** (checked on 16/11/2026): minimum and maximum for numbers, text length, recursion and objects with free properties. Those rules live in Zod and run in step 6. Example: "minutes greater than zero" and "sum equal to the effective time".
- **Plain text in every field**, with no Markdown or HTML.

### 4.1 Format of each response

TypeScript notation; the real schemas live in `packages/contracts`. `PlanContent` and `Activity` are in [CONTRACTS](CONTRACTS.md), section 4.

```ts
type Origin = 'from_material' | 'added';

type GeneratePlanOutput = {
  outcome: 'plan' | 'off_topic';
  note: string;                                  // short reason on off_topic; empty on plan
  skills: { code: SkillCodeOfStage; reason: string }[];   // enum with the education stage's codes
  fields: {                                      // one key per unit of suggestion
    justification: { value: string; origin: Origin };
    generalObjective: { value: string; origin: Origin };
    specificObjectives: { value: string[]; origin: Origin };
    contents: { value: string[]; origin: Origin };
    priorKnowledge: { value: string; origin: Origin };
    activities: { value: Activity[]; origin: Origin };
    resources: { value: string[]; origin: Origin };
    assessment: { value: { criteria: string[]; instruments: string[] }; origin: Origin };
    adaptations: { value: string; origin: Origin };
    crossSubjectLink: { value: string; origin: Origin };
  };
};

type InputCheckOutput = {
  studentData: 'none' | 'suspected';
  studentDataFields: ('lesson' | 'class' | 'material')[];
  misuse: 'none' | 'off_topic' | 'minor' | 'serious';
  category: 'none' | 'sexual_minors' | 'data_exfiltration' | 'illegal_content'
          | 'hate_speech' | 'rule_bypass' | 'prompt_extraction' | 'offensive';
  excerpt: string;                               // up to 200 characters; becomes the occurrence's minimal excerpt
  reason: string;
};

type OutputCheckOutput = {
  verdict: 'ok' | 'needs_review';
  reasons: ('personal_data' | 'unsafe_for_school' | 'off_purpose'
          | 'instruction_echo' | 'unverifiable_claim' | 'invented_skill')[];
  note: string;
};

type EvaluatePlanOutput = {
  findings: {
    ruleId: `rubric.${string}`;                  // list in EVALUATION, section 4
    severity: 'warn' | 'info';
    fieldPath: keyof GeneratePlanOutput['fields'];
    message: string;
    suggestion: unknown | null;                  // the whole field's value, already fixed
  }[];
};
```

From the format to the system:
- **`generate_plan`** with `outcome: 'plan'`: each key of `fields` becomes a pending `Suggestion`, and each item of `skills`, a skill suggestion. With `outcome: 'off_topic'`, the generation closes as `off_topic` and nothing is stored.
- **`input_check`**: `studentData: 'suspected'` closes as `input_blocked_privacy`, with the blocks in `failureDetails`. A `misuse` other than `none` follows section 4.1 of REQUIREMENTS; only `sexual_minors` and `data_exfiltration` block on their own.
- **`output_check`**: `needs_review` closes as `output_check_failed`.
- **`evaluate_plan`**: each item becomes a stored `Finding`, with the linked `Suggestion` when there is one.

**What the validation step checks without AI**, besides the schema: minutes are integers greater than zero; the sum equals the effective time; 1 to 3 skills, all from the education stage; no website address in any field; no text shaped like a skill code outside `skills`; no field over its maximum size. If it fails, the generation is retried within the limit of RNF-35.

## 5. Prompts

- **In versioned files:** `apps/api/prompts/<operation>/v<N>.md`. A published prompt is not edited; a change is a new file and a PR. The four `v1.md` files already exist as **drafts**: they have never run, and only become the published version 1 after the first evaluation. The file format and the list of variables are in [`apps/api/prompts/README.md`](../apps/api/prompts/README.md).
- **Recorded version:** `generate_plan@3`, for example, goes into `AiGeneration` and `AiUsageRecord`. It is possible to know which prompt produced each plan and to compare versions.
- **Request structure:**
  1. **Fixed prefix:** instructions, rubric, plan template and the education stage's skills. Byte for byte identical across requests for the same stage.
  2. **Variable part:** theme, context and pasted material, each inside a tag that says "this is data". Before going in, `<` and `>` in the author's text are swapped for look-alike characters, so that pasted text cannot close the tag.
- **No secret in the prompt** (0005, LLM07): assume it can leak.
- **Prompt cache:** the cache breakpoint sits at the end of the fixed prefix. With v1's volume (about 60 generations a month) and a 5-minute lifetime, hits between requests from different people will be rare; they happen inside the same generation (retries) and between generating and evaluating right after. Writing to the cache costs 1.25 times the input and reading 0.05 times (Sonnet 5.5). **The budget in 0005 D6 does not count on this saving.** The test checks `cache_read_input_tokens` to know whether the prefix is still stable.

## 6. Records and price

`AiUsageRecord`, one row per call, with no content (RF-74):

| Column | What for |
|---|---|
| operation, prompt version, model, effort | Compare cost and quality per version (RNF-39) |
| input, output, cache-read and cache-write tokens | Cost and cache health |
| cost in nano-dollars and price table version | Recompute if the price changes |
| duration in ms, `stop_reason`, number of retries | RNF-04 and diagnosis |
| generation ID and user ID | Credit and reporting; on account deletion, the link to the user is cut (section 7.1 of REQUIREMENTS) |

**Price table** in configuration, per model and with a start date. Prices change; the history stays explainable.

**Maximum input size (RF-73):** counted by the provider's token-counting endpoint before the paid call, with a character limit as the first barrier.

**Keys:** one per environment (local and production), in separate workspaces at the provider, each with a spending limit. **CI never has a key:** tests use the fake adapter.

## 7. Evaluation (evals)

An eval is the "test" of the AI's content: a set of inputs, a way to grade and a record of the result. The code around the AI stays under TDD (RNF-09); the **content** is measured here (RNF-10). This section is the summary; the detail (cases, checks, judge rubric, protocol for the human review and for the confirmation test) is in [EVALUATION](EVALUATION.md).

### 7.1 Datasets
| Set | What it holds | Where it lives |
|---|---|---|
| Synthetic | Invented class contexts, covering education stages, times and situations (no lab, exam week, large class) | `evals/datasets/`, in the repository |
| Reference | The contexts of the 3 reviewed plans, anonymized and with authorization | `evals/private/`, outside git |
| Guardrails | Sentences for RF-15 and RF-90 in Portuguese, English, Spanish, mixed language and disguised text | `evals/datasets/guardrails/` |
| Legitimate sensitive themes | Lessons on cyberbullying, digital safety, scams, misinformation and hate speech as a **theme** | `evals/datasets/guardrails/` |
| Injection | Pasted material that tries to give orders to the model | `evals/datasets/guardrails/` |

### 7.2 How grading works, in three layers
1. **Deterministic (no AI, free):** the schema is valid; every skill exists and matches the education stage; the sum of minutes adds up; no required field is empty; no decoy value in the text.
2. **Rubric judge (AI):** the "AI + teacher" items of the cohesion checklist (each objective has an activity that works on it; objectives, contents and assessment talk about the same lesson; the activities fit the class). The judge grades each item, with a short justification.
3. **Human:** the supervising teacher compares against the reference plans. It is the success criterion of [PURPOSE](PURPOSE.md).

### 7.3 Targets for accepting a prompt or a model
Summary. The full table, with the check that measures each target, is in [EVALUATION](EVALUATION.md), section 6.

| Measure | Target |
|---|---|
| Valid schema on the first attempt | ≥ 95% (100% after retries) |
| Skill outside the catalog | 0 (by construction) |
| Correct sum of minutes | ≥ 90% on the first attempt |
| Student data not flagged, in the guardrails set | 0 |
| Legitimate sensitive theme blocked | 0 |
| Injection that changed the behaviour | 0 |
| Cost and time per plan | Within the budget (0005 D6) and 60 s (RNF-04) |

### 7.4 When it runs
- **By command**, with a spending cap per run: `npm run eval -- --suite <name>`. Never in CI.
- **Whenever** a prompt, a model or the effort **changes**. The PR carries the before and after table.
- **The confirmation test of 0005 (D7)** is the first run: Haiku, Sonnet and Opus on the reference contexts.
- **Result:** `evals/results/<date>-<version>.md`, with the summary table. Nothing from the private set goes into the repository.

## 8. Sources

- **v1 (M2): the author brings the source.** The system checks the domain, checks that the link responds and formats it in ABNT (RF-51 to RF-54). Fetching the link follows the SSRF protections of decision [0006](decisions/0006-web-security-asvs-level-1.md) (note of 16/11).
- **Why the AI does not suggest sources in v1:** a link taken from the model's memory is the kind of answer that fails most, and "the link responds" does not prove the page says that. It is the weakest point of the "nothing invented" promise.
- **Later (M4):** the provider's own web search, restricted to the list of accepted domains (`allowed_domains`). The link then comes from a real search. The check "the fact is written in the source" (RF-55) uses citations in a separate call, because citations and schema-bound output do not work in the same call. When page content enters the model, indirect injection becomes a risk again (LLM01) and gets its own test set.

## 9. Switching models

Models get retired. The path, without touching code:
1. run the evaluation with the new model (section 7.3);
2. compare quality, cost and time with the current one;
3. change the configuration and the price table;
4. record it in a dated note on 0005.
