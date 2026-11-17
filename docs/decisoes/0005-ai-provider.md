# 0005: AI provider, models, LLM security and token budget

- **Date and status:** 15/11/2026, approved by Victor. Models, effort levels and limits are confirmed by the test below; until then they are starting values in configuration.
- **Context:** Docenda uses AI to generate and evaluate lesson plans (RF-20, RF-21, RF-26), suggest BNCC skills (RF-42) and check misuse and student data (RF-15, RF-90). The requirements ask for: a provider that does not train on our data (RNF-22), a cheap model for checks and a better one for generation (RNF-38), validated JSON output (RNF-35), the OWASP Top 10 for LLM applications 2025 (RNF-08), and limits and a monthly ceiling (RF-71 to RF-75). The provider stays behind the AI port (0001, RNF-30). Proposal from 14/11 (REQUISITOS, section 8, point 3); terms, OWASP list and prices checked on 15/11/2026.
- **Decision:**
  - **D1. Provider: Anthropic API**, called directly through the official SDK `@anthropic-ai/sdk` (pinned version), in the adapter `providers/ai`. What the terms say:
    - by default, inputs and outputs of the commercial API are **not used for training**. Exceptions are explicit feedback or opting in, and Docenda does neither;
    - inputs and outputs are deleted **within 30 days**. If the automated safety systems flag a request, it may be kept for **up to 2 years**;
    - processing happens outside Brazil (international transfer, LGPD art. 33). The privacy notice and the generation screen say so (RNF-21);
    - default global routing. US-only inference (`inference_geo`) costs 1.1x and brings nothing here.
  - **D2. What goes into a call:** only the topic, the aggregated class context, the pasted material, the plan text and the BNCC skills of the class stage. **Never** the user's name, e-mail or ID. Nothing is sent while an RF-15 alert (student name, diagnosis or medical report) is unresolved.
  - **D3. Model per task**, set in configuration (RNF-38):

    | Task | Model | Starting effort | Price per million tokens (input / output) |
    |---|---|---|---|
    | Generate and evaluate the plan | Claude Sonnet 5.5 (`claude-sonnet-5-5`) | `medium` | US$ 2 / US$ 10 |
    | Checks: misuse, student data, skill suggestion | Claude Haiku 5.5 (`claude-haiku-5-5`) | `low` | US$ 0.10 / US$ 0.50 (prompts up to 100k tokens) |

    Claude Opus 5.5 is used only in the comparison test. On Sonnet 5.5 the model's reasoning ("thinking") cannot be turned off; effort is the lever for cost and depth.
  - **D4. How calls are made:**
    - **structured output** (`output_config.format` with a JSON schema), validated again with Zod. An invalid answer is retried at most twice (RNF-35);
    - **prompt caching** on the fixed prefix (instructions, rubric, plan template, BNCC skills of the stage). Repeated reads of it cost 5% of the input price;
    - **streaming** in generation, to show progress (RNF-04), and `max_tokens` per operation (RF-73). *Update 16/11:* streaming stays between the worker and Anthropic; the screen shows stages, not the streamed text (0008 D3);
    - **refusal** (`stop_reason: "refusal"`) becomes a clear message. On Sonnet, the server-side fallback (`fallbacks: "default"`, beta) retries on another model. In checks (Haiku, no server-side fallback), a refusal counts as "needs review", never as "passed".
  - **D5. OWASP Top 10 for LLM 2025 applied to Docenda:**

    | Risk | How Docenda handles it |
    |---|---|
    | LLM01 Prompt injection | User text and pasted material are sent marked as **data**, never as instructions. Output bound to a schema limits the damage. Haiku checks the request and the answer (RF-90, D8). In v1 the server only checks that a source link responds; the page content never reaches the model |
    | LLM02 Sensitive information disclosure | RF-15 alert before sending; each call carries only the author's own data; usage records keep no content (RF-74) |
    | LLM03 Supply chain | Official SDK, pinned, with lockfile; model IDs pinned in configuration; vulnerability alerts (RNF-41) |
    | LLM04 Data and model poisoning | No training or fine-tuning; the BNCC catalog is imported once and checked (RF-40) |
    | LLM05 Improper output handling | Zod validation; Markdown sanitized (0006); skills checked against the catalog (RF-41) and sources against the domain list (RF-50) |
    | LLM06 Excessive agency | No tools that act on the system; everything is a suggestion and only the author approves (RF-23, RF-25) |
    | LLM07 System prompt leakage | No secret in the prompt; it is assumed it can leak. So trying to get the instructions is a **minor** misuse (REQUISITOS 4.1); trying to get keys or other users' data stays **serious** |
    | LLM08 Vector and embedding weaknesses | Not in v1: the catalog is filtered by stage, with no vector search |
    | LLM09 Misinformation | Skills only from the catalog; sources from the domain list, with the link checked and reviewed by the author; evals (RNF-10) |
    | LLM10 Unbounded consumption | Limits per user, monthly ceiling, maximum input and output size, on/off switch (RF-71 to RF-75); no second charge on a double click (RNF-37); circuit breaker (RNF-36) |

  - **D6. Token budget (estimate, confirmed by the test):**

    | Operation | Input | Output (with thinking) | Approximate cost |
    |---|---|---|---|
    | Generate a plan (Sonnet) | ~11k tokens | ~6k | US$ 0.07 to 0.08 |
    | Evaluate a plan (Sonnet) | ~10k | ~3k | US$ 0.05 |
    | 4 checks (Haiku) | ~2k each | ~300 each | under US$ 0.002 |

    With room for retries, **US$ 0.15 to 0.20 per plan**; output is most of the cost. Starting values in configuration: **monthly ceiling of US$ 10** (about 50 to 65 full generations), and per user **5 generations a day and 2 a minute**. Three layers protect the ceiling: the app ceiling (RF-72, warning at 80%), a monthly spend limit on the Anthropic workspace (Console), and prepaid credits with auto-reload off. Tokens and cost come from `response.usage` and are stored as integers (RF-74, RNF-42).
    - **Update (16/11/2026, approved by Victor):** v1 is invite-only, for up to 10 people (RF-78). Each person gets **US$ 1 of AI credit per month**, renewed on the 1st at Brasília time; unused credit does not carry over. US$ 1 covers about 5 or 6 full generations. The monthly ceiling stays at US$ 10 (10 × US$ 1) as the system-wide layer, and the limits of 2 a minute and 5 a day stay as protection against bursts. The maximum cost of an operation is reserved from the credit when the job is queued and settled from `response.usage` (0008 D3).
  - **D7. Confirmation test:** generate with Haiku, Sonnet and Opus from the contexts of the 3 reference plans and compare with the version reviewed by the internship teacher. Measure quality against the rubric, tokens, cost and time. The reference plans go only to this provider, which does not train on them (RNF-22). The result can change the models, effort levels and limits in D3 and D6, through configuration only.
  - **D8. Guardrails, in any language:**
    - **Full path:** input → text cleanup → student data alert (RF-15) → misuse check (RF-90) → generation → output validation (schema, catalog, domains) → **Haiku checks the answer** → screen. An answer that fails the check is not shown, and the author sees a clear message.
    - **Text cleanup before the checks:** Unicode normalization (NFKC, which turns look-alike characters into the common ones) and removal of invisible characters. Blocks that look encoded (for example base64) count as "needs review".
    - **Checks in any language:** the Portuguese word list of RF-15 ("laudo", "CID", "diagnosticado") is only the first, fast and cheap layer. The second layer is Haiku, which reads the meaning in several languages. A different language is never a reason to block: computing material in English is normal.
    - **Tests:** the RF-15 and RF-90 test sets include examples in English and Spanish, mixed languages and disguised text. The prompts for the checks and the test set are written in the AI and evaluation area.
- **Scaling path:** other adapters behind the same port: Amazon Bedrock or Claude Platform on AWS (when deploying on AWS, topic 7), and paid Gemini. Each new provider needs its terms checked (RNF-22). Users' own API keys (RF-76) after v1.
- **Alternatives considered:**
  - One model for everything (Sonnet also for checks): simpler, but each check costs 20 times more and takes longer.
  - Opus 5.5 for generation: twice Sonnet's price; used only if the test shows a clear quality gain.
  - Gemini free tier: its terms allow training on the data, so it is used only with fictitious data, never with the reference plans (RNF-22).
  - Bedrock from day one: the data stays in our AWS account, but it needs AWS set up before the local demo; left for topic 7.
- **Consequences:**
  - Pros: no training on our data; low cost per plan; checks almost free; switching provider means switching an adapter.
  - Cons: data processed abroad and kept up to 30 days (2 years if flagged); dependence on one vendor until a second adapter exists; prices and models change, so the estimates must be measured again.
  - Changes in REQUISITOS: section 4.1 (getting the instructions moves to minor), RF-15 and RF-90 (checks in any language, answer checked by Haiku) and section 8, points 2 and 3.
- **Update (16/11/2026, planning review).** Details in [IA](../IA.md):
  - **D2:** the plan's identification (school, teacher, class) is kept apart from the plan content and never enters a call. The AI port receives a type that does not have those fields, and a test with decoy values checks it.
  - **D3:** the two input checks (student data and misuse) become one Haiku call with several labels. Skills are chosen inside the generation call, not in a separate one.
  - **D4:** the skill field of the output schema is a closed list (`enum`) with the codes of the class stage, so a code outside the catalog cannot be returned. Number ranges and text lengths are not accepted by the provider's schema and are validated with Zod.
  - **D4, caching:** with about 60 generations a month and a 5-minute lifetime, hits across requests will be rare. The D6 budget does not count on the saving. A cache write costs 1.25 times the input price.
  - **D8:** a refusal from the provider (`stop_reason: "refusal"`) is never recorded as misuse by the user. The models refuse by category, security among them, and lessons about passwords, scams and attacks are part of the subject.
  - **Prompts** are versioned files; each generation records the prompt version, the model and the effort.
  - **Keys:** one per environment, each with its own spend limit. CI never holds a key; tests use the fake adapter.
- **Sources:** [Anthropic: is my data used for model training?](https://privacy.claude.com/en/articles/7996868-is-my-data-used-for-model-training), [Anthropic: how long do you store my organization's data?](https://privacy.claude.com/en/articles/7996866-how-long-do-you-store-my-organization-s-data) (dated 01/07/2026), [Claude API pricing](https://platform.claude.com/docs/en/about-claude/pricing), [spend limits](https://platform.claude.com/docs/en/api/beta/organization/spend_limits.md), [paying for API usage](https://support.claude.com/en/articles/8977456-how-do-i-pay-for-my-claude-api-usage), [OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/llm-top-10/). All checked on 15/11/2026.
