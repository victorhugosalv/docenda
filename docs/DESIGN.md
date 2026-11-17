# Design: Docenda

> **v0.1 (17/11/2026). Planned: there is no screen yet.** Says how the screens look and behave. The technical structure is in decision [0013](decisions/0013-front-end-stack.md); the screen map, in [INTERFACE](INTERFACE.md); the fields and errors, in [CONTRACTS](CONTRACTS.md).
> Screen text quoted below stays in Brazilian Portuguese: it is what the teacher reads.
> The identity comes from the project cover: green chalkboard, cream chalk and the gold of the full stop in the name.

## 1. Principles

1. **The teacher's text is the centre.** The screen is a notebook, not a dashboard. Little colour, lots of space, large type.
2. **What came from the AI is clear.** A suggestion has its own frame, label and actions, and is never confused with what the author has already decided. Colour helps, but is never the only signal (WCAG 1.4.1).
3. **Errors and warnings say what to do.** "A soma dá 45 de 40 minutos: tire 5 de alguma atividade", not "Tempo inválido".
4. **Nothing disappears without the author knowing.** Autosave with a visible state; conflict, expired session and AI failure explained on the spot.
5. **It works on the phone in the staff room.** The design starts at 360 px and grows.
6. **People with little or no sight use it on their own** (RNF-03). Keyboard, screen reader and 200% zoom without losing anything.

## 2. Colours

Two themes: **paper** (light, default) and **chalkboard** (dark). It follows `prefers-color-scheme`; the author can pin one of them in Minha conta, and the choice stays in the browser (it is not plan text, so it does not break 0013 D7).

The pairs below were computed with the WCAG contrast formula on 17/11/2026. Normal text needs 4.5:1; field borders, icons and the focus ring need 3:1.

| Token | Paper | Chalkboard | Use | Contrast (paper / chalkboard) |
|---|---|---|---|---|
| `--bg` | `#f6f2e9` | `#0b2420` | Page background | |
| `--surface` | `#fffdf8` | `#10302a` | Cards and fields | |
| `--text` | `#1b2925` | `#f3efe6` | Text | 14.9 / 12.4 on `--surface` |
| `--text-muted` | `#4a5a54` | `#b4c6bd` | Help and secondary labels | 7.2 / 8.0 on `--surface` |
| `--primary` | `#16423a` | `#d6b25a` | Main button, links | 10.0 / 8.1 on `--bg` |
| `--on-primary` | `#fffdf8` | `#0b2420` | Text on the main button | 11.0 / 8.1 |
| `--accent` | `#c9a44c` | `#c9a44c` | Decoration only: the dot in the name, rules | Do not use for text on the paper theme |
| `--accent-text` | `#7a5c12` | `#e0c070` | Gold text, when needed | 5.6 / 9.3 on `--bg` |
| `--suggestion-bg` | `#fbf1d6` | `#2b3a1f` | Background of the AI suggestion | Text: 13.4 / 10.6 |
| `--suggestion-border` | `#a8862e` | `#c9a44c` | Frame of the suggestion | 3.1 / 5.2 on its background |
| `--block` | `#a3261b` | `#ff9b8f` | Blocking finding; error | 7.3 / 7.0 |
| `--warn` | `#7f5200` | `#f2c66b` | Warning | 6.6 / 8.8 |
| `--info` | `#25507d` | `#9cc4f0` | Information | 8.2 / 7.8 |
| `--ok` | `#1d6542` | `#8fd6ad` | Saved, approved | 6.9 / 8.4 |
| `--focus` | `#1f4fd1` | `#8fb4ff` | Focus ring | 6.1 / 7.9 on `--bg` |
| `--border` | `#76867f` | `#6f8a80` | Field border | 3.8 / 3.8 on `--surface` |

Rules:
- Blue only appears on focus. That way the ring is never confused with another element.
- Block, warning and information always come with an icon and a word ("Bloqueia a aprovação", "Aviso", "Para saber"), besides the colour.

## 3. Type

| Role | Font | Where |
|---|---|---|
| Interface and plan text | **Atkinson Hyperlegible Next** (400 and 700) | Everything that is read and written |
| Numbers and codes | **Atkinson Hyperlegible Mono** (400) | Skill codes, minutes, balance |
| Brand | **Playwrite BR** (200) | Only the name "Docenda." at the top and on the login screen |

- **Why Atkinson Hyperlegible:** it was designed by the Braille Institute for reading with low vision. Letters that are often confused (`I`, `l`, `1`; `O`, `0`) have clearly distinct shapes. Open licence (OFL); checked on Google Fonts on 17/11/2026.
- **The cover's script and chalk lettering stay out of the interface.** Script and runs of capitals are harder to read, especially with dyslexia and with low vision. The cover is for looking at; the screen is for working.
- **Fonts served by Docenda itself,** in `apps/web/public/fonts/`, as `woff2`. Fetching from Google Fonts would break the content security policy (`default-src 'self'`, 0006) and send the teacher's IP elsewhere.

| Token | Size | Line height | Use |
|---|---|---|---|
| `--text-sm` | 0.875 rem (14 px) | 1.45 | Field help, metadata |
| `--text-md` | 1.0625 rem (17 px) | 1.55 | Text and fields |
| `--text-lg` | 1.25 rem (20 px) | 1.4 | Plan section title |
| `--text-xl` | 1.625 rem (26 px) | 1.25 | Screen title |
| `--text-brand` | 2 rem (32 px) | 1 | Brand |

Sizes in `rem`, to respect the font size the person chose in the browser. Lines of text at most 70 characters long.

## 4. Space, shape and motion

- **Space:** a 4 px scale (`--space-1` = 4 px up to `--space-8` = 64 px).
- **Touch target:** at least 44 × 44 px on buttons and controls. WCAG 2.2 AA asks for 24 px; 44 px fits a thumb.
- **Corners:** 6 px on fields and buttons, 10 px on cards. No heavy shadow: a 1 px border.
- **Focus:** a 3 px outline in `--focus`, 2 px away from the element, on **every** control. Never `outline: none` without a replacement.
- **Motion:** only transitions of up to 150 ms on colour and opacity. With `prefers-reduced-motion`, none.
- **Chalk texture:** stays out of the interface.

## 5. Components

| Component | Behaviour | Accessibility |
|---|---|---|
| Button | Primary (`--primary` background), secondary (border) and subtle (text only). One primary per screen | Native `button`. While sending, shows "Salvando…" and stays focusable; it is not disabled without an explanation |
| Text field | Label on top, short help under the label, error under the field | Linked `label`; help and error via `aria-describedby`; `aria-invalid` on error |
| Long field | Grows with the text, with no inner scroll bar | Same as the text field |
| Editable list (objectives, contents, resources) | One field per item; add, remove and move with buttons | Buttons with the item's name ("Remover objetivo 2"); an announcement on move |
| Option group (education stage, moment) | Radio buttons in a `fieldset` | `legend` with the question |
| Skill picker | Search by code or text, filtered by education stage and grade | React Aria Components combobox (0013 D4) |
| AI suggestion | Section 6.5 | Section 6.5 |
| Finding | Icon, severity label, message and a "Ir para o campo" link | A list with a heading; the link moves focus to the field |
| Dialog | Confirmations, privacy alert and expired session | Native `dialog`; focus trapped inside; Esc closes when no data would be lost |
| Page notice | A strip at the top: AI off, cap reached, revision conflict | `role="status"` or `role="alert"`, depending on urgency |
| Save state | "Salvo às 14:32", "Salvando…", "Não foi possível salvar: tentar de novo" | A polite live region, which does not interrupt reading |

## 6. Screens

On all of them: a header with the brand, "Meus planos" and "Minha conta"; one `h1` per screen, which receives focus when the screen changes; a "Pular para o conteúdo" link as the first item.

### 6.1 Log in (`/entrar`)
Brand, e-mail, password (with a "Mostrar senha" button) and "Entrar". A single error, above the form: "E-mail ou senha incorretos." At the attempt limit: "Muitas tentativas. Espere alguns segundos e tente de novo." A "Esqueci a senha" link from F10.

### 6.2 My plans (`/`)
- Primary button "Novo plano".
- List, newest first. Each item: theme, education stage and grade, state ("Rascunho", "Aprovado, versão 2"), date of the last change and "Gerando…" when there is an active generation.
- Actions per item, in a menu: open, duplicate, delete. Deleting asks for confirmation saying what disappears: "O plano, as versões aprovadas e as sugestões serão apagados."
- Empty: one sentence and the "Criar o primeiro plano" button.
- Desktop: a table. Phone: a list of cards, with the theme as a link.

### 6.3 New plan (`/planos/novo`), in three steps
Indicator "Passo 1 de 3". Going back does not lose what was written.

1. **Class.** Pick a saved context or describe a new class. Only education stage and length are required. Above the text fields, a fixed notice: "Descreva a turma em conjunto. Não escreva nomes, diagnósticos ou laudos." Each field has help with an example ("Ex.: parte da turma ainda confunde arquivo e pasta").
2. **Time.** Length in minutes or in class hours, with the conversion shown ("2 horas-aula = 100 minutos"). Right below: "Quanto tempo sobra de verdade? Chamada, troca de sala e ligar equipamento costumam tomar de 5 a 10 minutos." Suggested options (90, 95) and "outro valor". Mark exam week and note the week's workload.
3. **Theme and material.** Theme, subtheme and the pasted material, optional. Next to the material: "O texto colado é enviado a um provedor de IA fora do Brasil para gerar o plano."

The final button is "Criar plano e gerar sugestões". The plan is created as a draft and the generation starts; the author goes to the editor.

**Privacy alert.** If saving returns `privacy_alert`, a dialog opens: "Este trecho parece falar de um estudante específico: '…laudo…'. Descreva a turma em conjunto, sem nomes ou diagnósticos." Actions: "Corrigir o texto" (back to the field, with focus on it) and "O texto não identifica ninguém" (confirms and saves). Nothing goes to the AI before that.

### 6.4 Plan editor (`/planos/:id`)
The heart of Docenda.

**Top, fixed while scrolling:**
- Theme, state ("Rascunho, versão 1") and the save state.
- **Pending bar:** "3 sugestões para decidir · 1 bloqueio · 2 avisos", each number a link to the first item of that kind.
- **Time:** "35 de 40 minutos", with a bar that fills. Over the time, the bar turns `--block` and the text says "45 de 40 minutos: 5 a mais".
- Actions: "Gerar sugestões" (or "Gerar de novo"), "Aprovar" and, on an approved plan, "Exportar" and "Reabrir para editar".

**Body:** the fields in the order of the internship script. Each field is a section with an `h2` title, the author's text and, when there is one, the pending suggestion and the findings for that field.

- **Desktop (from 1024 px):** the plan column on the left and, on the right, the list of findings and the index of fields.
- **Phone:** one column; findings appear right below the field they refer to.

**Identification:** a collapsed section at the end, "Identificação do documento (escola, turma, data)", with the note "Estes dados aparecem só no arquivo exportado e nunca são enviados à IA."

**Activities:** each one is a card with moment, title, description and minutes. The minutes field sits beside the title, in the mono font. An activity marked "Toma a aula toda (instalação ou configuração)" shows the badge.

**Approve:** the button is always active. If there is a block, a dialog opens listing the blocks, each with a link to the field. With no block, it confirms: "Aprovar a versão 1? Ela fica guardada e pode ser exportada. Para mudar depois, reabra o plano." Warnings do not prevent approval, but they appear in the dialog.

**Approved plan:** the fields appear as text, with no editing. A notice at the top: "Versão 2 aprovada em 17/11. Para mudar, reabra: a versão 2 continua guardada."

### 6.5 AI suggestion, inside the field

```
┌ Sugestão da IA ─ acrescentado pela IA ──────────────┐
│ Texto proposto para o campo, inteiro.                │
│                                                      │
│ [Aceitar]   [Editar a partir desta]   [Recusar]      │
└──────────────────────────────────────────────────────┘
```

- Frame in `--suggestion-border`, background `--suggestion-bg` and a written label: "Sugestão da IA". The label also states the origin: "veio do seu material" or "acrescentado pela IA" (RF-21).
- **Aceitar** copies the text into the field. **Editar a partir desta** opens the text in a field, with "Salvar minha versão" and "Cancelar". **Recusar** removes the suggestion; the field stays as it was.
- In a list (objectives, activities), the suggestion shows the whole list, because the unit is the field (RF-23).
- Suggested skill: code and official text, plus the reason in one sentence ("A aula trabalha a construção de algoritmos na atividade 2"). Actions: "Incluir" and "Recusar". There is no edit: the text is literal.
- After the decision, focus goes to the next pending suggestion, and an announcement says "Sugestão aceita. Faltam 2."
- For the screen reader, each suggestion is a named region: "Sugestão da IA para Justificativa".

### 6.6 Generation progress
A panel at the top of the editor while the generation is active:

```
Gerando sugestões
✓ Conferindo os dados
● Escrevendo o plano            ← etapa atual
○ Validando
○ Conferindo a resposta
```

- A polite live region: it announces the change of step, not every second.
- The editor stays usable during the generation. The fields that will receive a suggestion still belong to the author.
- On finishing: "Pronto: 10 sugestões para decidir." and focus goes to the pending bar.

**Message for each failure** (`failureReason`, DOMAIN 4.2):

| Reason | Message | Action offered |
|---|---|---|
| `input_blocked_privacy` | "Encontramos o que parece ser dado de um estudante em Turma. Corrija ou confirme antes de gerar." | Go to the field |
| `input_blocked_misuse` | "Este pedido não pode ser atendido pela IA. Se acha que é um engano, peça uma revisão." | See the reason; if the AI was blocked, ask for a review (F09) |
| `off_topic` | "O pedido não parece ser de uma aula. Revise o tema e o material." | Go to the theme |
| `provider_refusal` | "O provedor de IA recusou este pedido. Isso acontece às vezes com temas de segurança; não conta como problema seu. Tente reformular o tema." | Generate again |
| `invalid_output` | "A IA devolveu um plano incompleto. Nada foi cobrado além do que foi usado." | Generate again |
| `output_check_failed` | "A resposta da IA não passou na nossa conferência e não foi mostrada." | Generate again |
| `budget_exhausted` | "O crédito acabou no meio da geração." | See balance |
| `provider_unavailable` | "A IA está fora do ar agora. Nada foi cobrado. Tente em alguns minutos." | Generate again |
| `worker_lost` | "A geração foi interrompida. Tente de novo." | Generate again |

### 6.7 Export
On the approved plan: choose the version (the latest comes selected) and download the Markdown. Beside it, "Incluir a nota 'elaborado com apoio de IA'", on by default (RF-61). The list of versions shows the date of each one's last export.

### 6.8 My account (`/conta`)
Name, e-mail, theme (paper or chalkboard), the month's AI balance from F11 ("US$ 0,62 de US$ 1,00; renova em 01/12"), log out, log out of all devices and, from F14, delete the account.

### 6.9 Session expired in the middle of editing
Dialog: "Sua sessão terminou. Entre de novo para continuar; o que você escreveu está guardado nesta tela." After logging in, the pending save is repeated and the dialog closes (0013 D7).

### 6.10 Revision conflict
A strip at the top: "Este plano foi alterado em outra aba ou aparelho." Actions: "Ver a versão mais recente" and "Copiar o meu texto". Never overwrite silently.

## 7. Interface text

- Brazilian Portuguese, addressing the user as "você". Short sentences.
- A button says the action: "Aprovar plano", not "Confirmar".
- The same word for the same thing, always the one in the glossary of [DOMAIN](DOMAIN.md): "sugestão"; "achado" only in code (on screen: "bloqueio", "aviso", "para saber"); "etapa de ensino"; "tempo efetivo".
- Dates as `17/11/2026`; times as `14:32`; money as `US$ 0,62`.
- All text comes from the single text module (0013 D6).

## 8. Checks

- **Automatic, in every end-to-end test:** axe with no violations (0013 D8), 360 px without horizontal scrolling, no plan text in browser storage.
- **Manual, before closing v1 (F16), on each screen:** keyboard only; NVDA on Windows and TalkBack on Android; 200% zoom and increased text spacing (WCAG 1.4.12).
- **Contrast:** the pairs in section 2 already pass. A new colour goes into the table with its computed contrast.
