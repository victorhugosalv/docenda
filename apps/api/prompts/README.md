# Prompts

Versioned prompt files for the four AI operations. The design is in `docs/IA.md` (pipeline, output schema, caching) and `docs/AVALIACAO.md` (how a prompt is accepted). The prompts are written in Portuguese because the plans are.

**Status: drafts.** The `v1.md` files were written in the planning phase and have never been run. Each one becomes the published `v1` only after the first evaluation run (slice F04 for `generate_plan`, F09 for the checks, F15 for `evaluate_plan`). Until then they may be edited in place.

## Rules

- **A published prompt is never edited.** A change is a new file (`v2.md`) and a pull request with the evaluation table before and after.
- **The version is recorded** on every generation and usage record as `<operation>@<N>` (INV-13).
- **No secret in a prompt.** Assume it can leak (ADR 0005, LLM07).
- **No user data in the `SYSTEM` part.** Everything written by the author goes in the `USER` part, inside the data tags.

## File format

One file per version, with two parts:

- `## SYSTEM`: the fixed prefix. It is byte-for-byte equal between requests of the same education stage, so the provider can cache it. The cache mark goes at its end.
- `## USER`: the variable part, with `{{placeholders}}`.

The loader splits the file on those two headings and fills the placeholders. It fails if a placeholder has no value or if a value is given for a placeholder that does not exist.

## Placeholders

| Placeholder | Filled with |
|---|---|
| `{{skills_catalog}}` | The catalog skills of the class's education stage, one per line: code, literal text, explanation and examples. Always in catalog order, so the prefix stays stable |
| `{{theme}}`, `{{subtheme}}`, `{{effective_minutes}}`, `{{nominal_minutes}}`, `{{exam_week}}`, `{{week_notes}}`, `{{teacher_notes}}` | The lesson fields of the plan |
| `{{education_stage}}`, `{{grade}}`, `{{subject}}`, `{{class_size}}`, `{{prior_knowledge}}`, `{{common_difficulties}}`, `{{social_context}}`, `{{school_resources}}`, `{{accessibility}}` | The context snapshot of the plan |
| `{{source_material}}` | The pasted material |
| `{{lesson_block}}`, `{{class_block}}` | The same lesson and class fields, already formatted as in `generate_plan` |
| `{{generated_plan}}`, `{{plan_content}}`, `{{plan_skills}}` | The plan fields as labelled plain text |

What never has a placeholder: the plan identification (school, teacher, class, date), the context label, and the user's name, e-mail and ID (INV-09).

## Filling rules

- An empty value is written as `(não informado)`, so the model does not guess.
- Education stage and yes/no values are written in Portuguese (`ensino médio`, `sim`).
- In author text, `<` and `>` are replaced by `‹` and `›` before filling, so pasted text cannot close a data tag.
- Text goes through the cleanup step first (Unicode NFKC, invisible characters removed).

## Output

Each operation has a response schema in `packages/contracts`, described in `docs/IA.md`, section 4. The prompt explains what to write in each field; the schema enforces the shape.
