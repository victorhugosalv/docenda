# IA: pipeline, prompts e avaliação

> **v0.2 (17/11/2026).** Nasceu da revisão do planejamento de 16/11; a v0.2 acrescenta os formatos de resposta (seção 4.1), os rascunhos dos prompts e a ligação com a [AVALIACAO](AVALIACAO.md). Detalha o que a decisão [0005](decisoes/0005-ai-provider.md) deixou para a "área de IA e avaliação". Fatos da API conferidos na documentação da Anthropic em 16/11/2026.
> Termos do domínio: [DOMINIO](DOMINIO.md). Fila e crédito: [0008](decisoes/0008-ai-job-queue-postgresql.md).

## 1. Princípios

1. **A IA sugere, o autor decide.** A IA só cria sugestões; nunca escreve no plano, nunca aprova (INV-03, INV-04).
2. **Impossível é melhor que detectado.** Onde der, o erro é barrado pela forma, não por uma checagem depois: código de habilidade fora do catálogo não cabe no esquema de saída nem no banco.
3. **Texto do usuário é dado, nunca instrução.** Tema, contexto e material colado vão marcados como dados (RNF-08).
4. **Na dúvida, não passa.** Checagem que falha, recusa ou devolve algo inválido conta como "precisa de revisão", nunca como "passou".
5. **Tudo o que custa é medido.** Cada chamada grava tokens, custo, tempo, modelo e versão do prompt (INV-13).
6. **Mudança de prompt ou de modelo passa pela avaliação** antes de entrar (seção 7).

## 2. Operações

| Operação | Modelo (configurável) | Entrada | Saída | Marco |
|---|---|---|---|---|
| `input_check` | Haiku 5.5, esforço `low` | Tema, contexto e material colado, depois da limpeza | `studentData` (`none` ou `suspected`), `misuse` (`none`, `off_topic`, `minor` ou `serious`), categoria, trecho mínimo e motivo curto | M2 (em M1, só a lista de palavras) |
| `generate_plan` | Sonnet 5.5, esforço `medium` | Tema, contexto agregado, material, modelo do plano, habilidades da etapa de ensino | Campos do plano (DOMINIO, seção 3), cada um com a origem (`from_material` ou `added`); ou `off_topic`, quando o pedido não é uma aula | M1 |
| `output_check` | Haiku 5.5, esforço `low` | A resposta gerada | `ok` ou `needs_review`, com motivo | M2 |
| `evaluate_plan` | Sonnet 5.5, esforço `medium` | O plano, a cópia do contexto e a rubrica | Lista de achados `warn` ou `info`, com sugestão opcional. **Nunca `block`:** a IA não trava a aprovação | M2 |

Duas mudanças em relação à 0005, registradas lá em nota de 16/11:
- **As duas checagens de entrada viram uma chamada só** (`input_check`), com saída de vários rótulos. É menos custo e menos espera, e o conjunto de teste é o mesmo.
- **A habilidade é escolhida dentro do `generate_plan`**, não numa chamada à parte. O esquema de saída já limita os códigos (seção 4).

## 3. O caminho de uma geração

Cada passo é uma função `(estado) → estado ou parada`, testada sozinha com o adaptador falso. O serviço que encadeia os passos não sabe se foi chamado pelo worker, por um comando ou por um teste.

| # | Passo | Usa IA? | Se falhar |
|---|---|---|---|
| 1 | **Limpar o texto:** normalização Unicode (NFKC) e remoção de caracteres invisíveis; bloco que parece codificado vira "precisa de revisão" | Não | `input_blocked_misuse` (leve) |
| 2 | **Alerta de privacidade, 1ª camada:** lista de termos (RF-15) | Não | Para antes de qualquer chamada (INV-10) |
| 3 | **`input_check`:** dado de aluno e uso indevido, em qualquer língua | Haiku | Motivo conforme o rótulo; recusa ou saída inválida conta como "precisa de revisão" |
| 4 | **Montar o pedido:** prefixo fixo e parte variável (seção 5). A identificação do plano não existe no tipo de entrada (INV-09) | Não | Erro de programação |
| 5 | **`generate_plan`:** saída presa ao esquema | Sonnet | Recusa: `provider_refusal`. JSON inválido: repete até 2 vezes |
| 6 | **Validar:** esquema Zod completo, habilidades no catálogo, minutos | Não | `invalid_output` |
| 7 | **`output_check`** | Haiku | `output_check_failed`; nada aparece para o autor |
| 8 | **Gravar sugestões** e fechar a geração | Não | Transação: ou grava tudo, ou nada |

Em cada chamada (3, 5 e 7): acerto do custo pelo `response.usage` e um `AiUsageRecord`.

**Recusa do provedor não é uso indevido.** Os modelos recusam por categoria (por exemplo, segurança cibernética), e aula de Computação fala de senha, golpe e invasão. Uma recusa vira `provider_refusal`, com mensagem clara e sem ocorrência registrada. No `generate_plan`, a repetição em outro modelo pelo próprio provedor (`fallbacks`, 0005 D4) continua ligada.

## 4. Esquema de saída

- **Uma fonte só:** o esquema Zod do plano fica em `packages/contracts`. Dele saem o JSON Schema enviado ao modelo (`output_config.format`), a validação da resposta, o formato do `jsonb` e os tipos da tela.
- **Habilidades como lista fechada.** O campo de habilidades aceita só os códigos da etapa de ensino da turma, como `enum`. O modelo não consegue devolver um código que não existe. Há um esquema por etapa de ensino; o provedor compila cada esquema uma vez e guarda por 24 horas.
- **O que o esquema do provedor não aceita** (conferido em 16/11/2026): mínimo e máximo de números, tamanho de texto, recursão e objetos com propriedades livres. Essas regras ficam no Zod e rodam no passo 6. Exemplo: "minutos maior que zero" e "soma igual ao tempo efetivo".
- **Texto puro em todos os campos**, sem Markdown nem HTML.

### 4.1 Formato de cada resposta

Notação de TypeScript; os esquemas de verdade ficam em `packages/contracts`. `PlanContent` e `Activity` estão no [CONTRATOS](CONTRATOS.md), seção 4.

```ts
type Origin = 'from_material' | 'added';

type GeneratePlanOutput = {
  outcome: 'plan' | 'off_topic';
  note: string;                                  // motivo curto no off_topic; vazio no plano
  skills: { code: SkillCodeOfStage; reason: string }[];   // enum com os códigos da etapa de ensino
  fields: {                                      // uma chave por unidade de sugestão
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
  studentDataFields: ('aula' | 'turma' | 'material')[];
  misuse: 'none' | 'off_topic' | 'minor' | 'serious';
  category: 'none' | 'sexual_minors' | 'data_exfiltration' | 'illegal_content'
          | 'hate_speech' | 'rule_bypass' | 'prompt_extraction' | 'offensive';
  excerpt: string;                               // até 200 caracteres; vira o trecho mínimo da ocorrência
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
    ruleId: `rubric.${string}`;                  // lista na AVALIACAO, seção 4
    severity: 'warn' | 'info';
    fieldPath: keyof GeneratePlanOutput['fields'];
    message: string;
    suggestion: unknown | null;                  // o valor inteiro do campo, já corrigido
  }[];
};
```

Do formato para o sistema:
- **`generate_plan`** com `outcome: 'plan'`: cada chave de `fields` vira uma `Suggestion` pendente, e cada item de `skills`, uma sugestão de habilidade. Com `outcome: 'off_topic'`, a geração fecha como `off_topic` e nada é gravado.
- **`input_check`**: `studentData: 'suspected'` fecha como `input_blocked_privacy`, com os blocos em `failureDetails`. `misuse` diferente de `none` segue a seção 4.1 do REQUISITOS; só `sexual_minors` e `data_exfiltration` bloqueiam sozinhas.
- **`output_check`**: `needs_review` fecha como `output_check_failed`.
- **`evaluate_plan`**: cada item vira um `Finding` guardado, com a `Suggestion` ligada quando houver.

**O que o passo de validação confere sem IA**, além do esquema: minutos inteiros e maiores que zero; soma igual ao tempo efetivo; de 1 a 3 habilidades, todas da etapa de ensino; nenhum endereço de site em campo nenhum; nenhum texto no formato de código de habilidade fora de `skills`; nenhum campo acima do tamanho máximo. Se falhar, a geração é repetida dentro do limite do RNF-35.

## 5. Prompts

- **Em arquivos versionados:** `apps/api/prompts/<operação>/v<N>.md`. Um prompt publicado não é editado; mudança é um arquivo novo e um PR. Os quatro `v1.md` já existem como **rascunho**: nunca rodaram, e só viram a versão 1 publicada depois da primeira avaliação. O formato do arquivo e a lista de variáveis estão em [`apps/api/prompts/README.md`](../apps/api/prompts/README.md).
- **Versão gravada:** `generate_plan@3`, por exemplo, vai para `AiGeneration` e `AiUsageRecord`. Dá para saber qual prompt produziu cada plano e comparar versões.
- **Estrutura do pedido:**
  1. **Prefixo fixo:** instruções, rubrica, modelo do plano e habilidades da etapa de ensino. Byte a byte igual entre pedidos da mesma etapa.
  2. **Parte variável:** tema, contexto e material colado, cada um dentro de uma marcação que diz "isto é dado". Antes de entrar, `<` e `>` do texto do autor são trocados por caracteres parecidos, para que um texto colado não consiga fechar a marcação.
- **Sem segredo no prompt** (0005, LLM07): presume-se que ele pode vazar.
- **Cache de prompt:** a marca de cache fica no fim do prefixo fixo. Com o volume da v1 (cerca de 60 gerações por mês) e validade de 5 minutos, o acerto entre pedidos de pessoas diferentes será raro; ele acontece dentro da mesma geração (repetições) e entre gerar e avaliar em seguida. A escrita no cache custa 1,25 vez a entrada e a leitura 0,05 vez (Sonnet 5.5). **O orçamento da 0005 D6 não conta com essa economia.** O teste confere `cache_read_input_tokens` para saber se o prefixo continua estável.

## 6. Registro e preço

`AiUsageRecord`, uma linha por chamada, sem conteúdo (RF-74):

| Coluna | Para quê |
|---|---|
| operação, versão do prompt, modelo, esforço | Comparar custo e qualidade por versão (RNF-39) |
| tokens de entrada, de saída, lidos do cache e escritos no cache | Custo e saúde do cache |
| custo em nano-dólar e versão da tabela de preços | Recalcular se o preço mudar |
| duração em ms, `stop_reason`, número de repetições | RNF-04 e diagnóstico |
| ID da geração e do usuário | Crédito e relatório; na exclusão da conta, a ligação com o usuário é cortada (seção 7.1 do REQUISITOS) |

**Tabela de preços** em configuração, por modelo e com data de início. Preço muda; o histórico continua explicável.

**Tamanho máximo da entrada (RF-73):** contado pelo endpoint de contagem de tokens do provedor antes da chamada paga, com um limite de caracteres como primeira barreira.

**Chaves:** uma por ambiente (local e produção), em áreas de trabalho separadas no provedor, cada uma com limite de gasto. **O CI nunca tem chave:** testes usam o adaptador falso.

## 7. Avaliação (evals)

Eval é o "teste" do conteúdo da IA: um conjunto de entradas, um jeito de dar nota e um registro do resultado. O código em volta da IA continua com TDD (RNF-09); o **conteúdo** é medido aqui (RNF-10). Esta seção é o resumo; o detalhe (casos, checagens, rubrica do juiz, protocolo da revisão humana e do teste de confirmação) está na [AVALIACAO](AVALIACAO.md).

### 7.1 Conjuntos de dados
| Conjunto | O que tem | Onde fica |
|---|---|---|
| Sintético | Contextos de turma inventados, cobrindo etapas de ensino, tempos e situações (sem laboratório, semana de provas, turma grande) | `evals/datasets/`, no repositório |
| Referência | Os contextos dos 3 planos revisados, anonimizados e com autorização | `evals/private/`, fora do git |
| Guardrails | Frases para RF-15 e RF-90 em português, inglês, espanhol, língua misturada e texto disfarçado | `evals/datasets/guardrails/` |
| Temas sensíveis legítimos | Aulas sobre cyberbullying, segurança digital, golpes, desinformação e discurso de ódio como **tema** | `evals/datasets/guardrails/` |
| Injeção | Material colado que tenta dar ordens ao modelo | `evals/datasets/guardrails/` |

### 7.2 Como se dá nota, em três camadas
1. **Determinística (sem IA, grátis):** o esquema é válido; toda habilidade existe e combina com a etapa de ensino; a soma dos minutos fecha; nenhum campo obrigatório vazio; nenhum valor-isca no texto.
2. **Juiz por rubrica (IA):** os itens "IA + professor" do checklist de coesão (cada objetivo tem uma atividade que o trabalha; objetivos, conteúdos e avaliação falam da mesma aula; as dinâmicas cabem na turma). O juiz dá nota por item, com justificativa curta.
3. **Humana:** a professora orientadora compara com os planos de referência. É o critério de sucesso do [PROPÓSITO](PROPOSITO.md).

### 7.3 Metas para aceitar um prompt ou um modelo
Resumo. A tabela completa, com a checagem que mede cada meta, está na [AVALIACAO](AVALIACAO.md), seção 6.

| Medida | Meta |
|---|---|
| Esquema válido na primeira tentativa | ≥ 95% (100% depois das repetições) |
| Habilidade fora do catálogo | 0 (pela forma) |
| Soma dos minutos correta | ≥ 90% na primeira tentativa |
| Dado de aluno não alertado, no conjunto de guardrails | 0 |
| Tema sensível legítimo bloqueado | 0 |
| Injeção que mudou o comportamento | 0 |
| Custo e tempo por plano | Dentro do orçamento (0005 D6) e de 60 s (RNF-04) |

### 7.4 Quando roda
- **Por comando**, com teto de gasto por execução: `npm run eval -- --suite <nome>`. Nunca no CI.
- **Sempre que mudar** um prompt, um modelo ou o esforço. O PR traz a tabela antes e depois.
- **O teste de confirmação da 0005 (D7)** é a primeira execução: Haiku, Sonnet e Opus nos contextos de referência.
- **Resultado:** `evals/results/<data>-<versão>.md`, com a tabela resumida. Nada do conjunto privado entra no repositório.

## 8. Fontes

- **v1 (M2): o autor traz a fonte.** O sistema confere o domínio, confere se o link responde e formata em ABNT (RF-51 a RF-54). A busca do link segue as proteções contra SSRF da decisão [0006](decisoes/0006-web-security-asvs-level-1.md) (nota de 16/11).
- **Por que a IA não sugere fonte na v1:** link tirado da memória do modelo é o tipo de resposta que mais erra, e "o link responde" não prova que a página diz aquilo. É o ponto mais frágil da promessa "nada inventado".
- **Depois (M4):** a busca na web do próprio provedor, restrita à lista de domínios aceitos (`allowed_domains`). O link passa a vir de uma busca real. A conferência "o fato está escrito na fonte" (RF-55) usa citações numa chamada separada, porque citações e saída presa a esquema não funcionam na mesma chamada. Quando o conteúdo de páginas entrar no modelo, a injeção indireta volta a ser risco (LLM01) e ganha conjunto de teste próprio.

## 9. Trocar de modelo

Modelo sai de linha. O caminho, sem mexer em código:
1. rodar a avaliação com o modelo novo (seção 7.3);
2. comparar qualidade, custo e tempo com o atual;
3. mudar a configuração e a tabela de preços;
4. registrar numa nota datada da 0005.
