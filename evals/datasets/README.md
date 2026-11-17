# Evaluation datasets

Inputs for the AI evaluation described in `docs/AVALIACAO.md`. One JSON object per line (JSONL). **Everything here is fictional:** names, classes and situations were made up for testing.

| File | Lines | Used for |
|---|---|---|
| `synthetic.jsonl` | 17 | Class contexts and lesson requests for `generate_plan` |
| `guardrails/privacy.jsonl` | 17 | Student-data alert: word list (layer 1) and `input_check` (layer 2) |
| `guardrails/sensitive-themes.jsonl` | 10 | Legitimate lessons on sensitive topics; none may be blocked |
| `guardrails/injection.jsonl` | 8 | Pasted material that tries to give orders to the model |
| `guardrails/misuse.jsonl` | 7 | Minor misuse, off-topic requests and one lesson that only looks dangerous |

## Line formats

- `synthetic.jsonl`: `id`, `tags`, `context` (the `ContextFields` of `docs/CONTRATOS.md`), `plan` (`theme`, `subtheme`, `effectiveMinutes`, `lesson`, `sourceMaterial`) and `expect` (`outcome`, plus optional `setupHeavy`, `mostlyFromMaterial`, `noUnverifiableFacts`, `language`).
- `guardrails/privacy.jsonl`: `id`, `lang`, `block` (`aula`, `turma` or `material`), `text`, `expect.studentData`, `expect.wordList` and an optional `note`.
- `guardrails/sensitive-themes.jsonl`: `id`, `theme`, `teacherNotes`, `expect`.
- `guardrails/injection.jsonl`: `id`, `block`, `text`, `note`, `expect.behaviourChanged`.
- `guardrails/misuse.jsonl`: `id`, `block`, `text`, `expect.misuse`, `expect.category`.

## What is not here

- **No example of the serious categories** (sexual content involving minors, hate speech against a person or group, attempts to get other users' data). This repository is public, and such a file would itself be the problem. The automatic block is tested with the fake adapter, which returns the `serious` label without the text existing.
- **No real data.** The reference set (contexts of the three reviewed plans, anonymised and authorised) lives in `evals/private/`, which git ignores. Results are written to `evals/results/` with numbers and case IDs only.

## Adding a case

A new case comes from a real failure: add the line first, see it fail, then change the prompt or the code. Keep the IDs stable; results refer to them.
