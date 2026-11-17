# Critérios de aceite por fatia: Docenda

> **v0.1 (17/11/2026).** Diz como saber que cada fatia do [BACKLOG](BACKLOG.md) está pronta. Cada linha vira pelo menos um teste, com o ID no começo do nome (0012 D9). Vale junto com a "definição de pronto" do BACKLOG, que é igual para todas.
> As fatias do marco M1 (F00 a F08) estão detalhadas. As do M2 (F09 a F16) têm o mínimo e fecham na especificação, com o que o M1 ensinar.
> Tipos de teste: `U` unitário, `I` integração com o PostgreSQL, `E2E` ponta a ponta, `Eval` avaliação do conteúdo da IA, `Rev` revisão manual.

## Como usar com o Spec Kit

No `/speckit-specify` da fatia, cite a seção daqui em vez de reescrever os critérios. No `/speckit-tasks`, cada critério vira a tarefa de teste que vem **antes** da tarefa de código. Se a especificação descobrir que um critério está errado, ele é corrigido aqui no mesmo PR.

## M0

### F00 `foundation`

| # | Critério | Origem | Teste |
|---|---|---|---|
| 1 | Num clone limpo, `docker compose up` sobe banco, e-mail de teste, migração, API e worker; a API só inicia depois de a migração terminar | RNF-11, 0012 D5 | Rev |
| 2 | `GET /healthz` responde `200`; `GET /readyz` responde `200` com o banco no ar e `503` com o banco parado | 0010 D7 | I |
| 3 | Conectado como `docenda_app`, criar ou alterar tabela falha. Os três papéis existem | 0012 D5 | I |
| 4 | Faltando uma variável obrigatória, o processo não sobe e a mensagem diz o nome da variável, nunca o valor | RNF-32 | U |
| 5 | Rota inexistente em `/api/v1` responde `404` no formato de problema, com `code` e `requestId`. Um erro inesperado responde `500 internal_error`, sem rastro de pilha | 0012 D7 | I |
| 6 | Toda resposta traz `X-Request-Id`; a mesma identificação aparece na linha de log do pedido | 0010 D2 | I |
| 7 | O log não contém corpo de pedido, cookie, cabeçalho de autorização nem IP. Teste com valores-isca | RNF-39, 0010 D3 | I |
| 8 | Respostas com os cabeçalhos do helmet (CSP `default-src 'self'`, `nosniff`, `frame-ancestors 'none'`) e `Cache-Control: no-store` na API | 0006 D5 | I |
| 9 | Corpo acima do limite responde `413`; pedidos em excesso do mesmo IP respondem `429` | RNF-44 | I |
| 10 | O dependency-cruiser falha diante de uma violação de fronteira plantada num arquivo de teste | 0001, 0012 D1 | CI |
| 11 | Em cada PR o CI roda tipos, lint, fronteiras, testes unitários, testes de integração com PostgreSQL e o build da imagem; CodeQL, Dependabot e Gitleaks ligados; `main` protegida | RNF-09, RNF-41, 0009 | Rev |
| 12 | A imagem roda sem root, com sistema de arquivos só de leitura | 0007 D2 | Rev |
| 13 | `apps/web` compila e a API serve a página inicial sem violar a CSP | 0002, 0006 | I |
| 14 | Spec Kit fixado na versão da 0014, com a constituição escrita; a seção "Commands" do `CLAUDE.md` preenchida | 0014 | Rev |

Fora da fatia: tabelas de produto, login e qualquer tela além da página inicial.

## M1

### F01 `bncc-catalog`

| # | Critério | Origem | Teste |
|---|---|---|---|
| 1 | O comando de importação grava 140 habilidades: 11 da educação infantil e 26 do ensino médio | RF-40 | I |
| 2 | O texto de cada habilidade é igual ao do documento oficial: conferência automática de todas e uma amostra de 14 conferida à mão, registrada | RF-40 | U, Rev |
| 3 | Nenhum texto tem palavras coladas ou espaço duplo (defeito conhecido da extração do PDF) | RF-40 | U |
| 4 | Importar a mesma versão duas vezes não muda nada. Uma versão nova cria linhas novas e não altera as antigas | RF-40 | I |
| 5 | `GET /skills?stage=high_school` devolve só habilidades do ensino médio; `grade` e `q` filtram | RF-42 | I |
| 6 | `formatted` segue `(CÓDIGO) Texto literal. (Competência específica N)`; sem competência, sai sem o parêntese final | RF-41 | U |
| 7 | Código inexistente é recusado com `skill_not_found` | RF-41 | U |
| 8 | `skill-stage-mismatch`: habilidade `EM13` numa turma do fundamental gera aviso; `EF15` numa turma do 7º ano gera aviso; `EF69` no 7º ano não gera | RF-43 | U |
| 9 | Como `docenda_app`, escrever em `skill` falha | DADOS, seção 5 | I |

### F02 `plan-core`

Tudo aqui é função pura: sem banco, sem IA, sem relógio.

| # | Critério | Origem | Teste |
|---|---|---|---|
| 1 | 1 hora-aula vale 50 minutos; com hora-aula de 45, 2 horas-aula valem 90; horas e minutos convertem para minutos | RF-12 | U |
| 2 | Para 50 minutos, o sistema sugere 40 e 45 como tempo efetivo; o autor pode escolher outro valor ou manter 50 | RF-13 | U |
| 3 | `time-sum`: soma igual não gera achado; soma maior ou menor gera `block` com a diferença em minutos; plano sem atividade gera `block` | Seção 4 | U |
| 4 | `pending-suggestion`, `skill-required`, `empty-required-field`: cada regra com o caso que passa, o que bloqueia e o limite | RF-23, RF-41, RF-20 | U |
| 5 | `setup-heavy`: avisa quando há atividade `setupHeavy` junto com outra atividade de desenvolvimento e o tempo efetivo é de até 60 minutos; nunca bloqueia | RF-14 | U |
| 6 | `privacy-hint`: "laudo", "CID" e "diagnosticado" geram alerta, sem diferença de maiúscula ou acento; "cidade" não gera; texto confirmado não gera; texto alterado depois da confirmação volta a gerar | RF-15 | U |
| 7 | INV-03: aprovar só é possível sem achado `block`; aprovar um plano aprovado é erro | RF-25 | U |
| 8 | Reabrir um plano aprovado devolve rascunho com `version + 1` | RF-25 | U |
| 9 | Sugestão: aceitar copia o valor; editar copia o texto do autor; recusar não copia; decidir duas vezes é erro | RF-23 | U |
| 10 | Geração nova substitui só as sugestões pendentes do mesmo campo | RF-23 | U |
| 11 | A ligação com outras matérias é texto e não aceita código de habilidade | RF-27 | U |
| 12 | Nenhum arquivo de `domain/` importa algo fora de `shared/` | 0012 D1 | CI |

Duas regras ficaram com valor proposto e podem mudar na especificação: a sugestão de tempo efetivo (nominal menos 10 e menos 5 minutos) e o limite de 60 minutos do `setup-heavy`.

### F03 `ai-generation`

| # | Critério | Origem | Teste |
|---|---|---|---|
| 1 | INV-09: o tipo que a porta de IA recebe não tem identificação, nome, e-mail nem ID do usuário. Com valores-isca nesses campos, nenhum aparece no pedido | RNF-45 | U |
| 2 | INV-10: com alerta de privacidade não resolvido, o adaptador falso não é chamado nenhuma vez | RF-15 | U |
| 3 | A limpeza aplica NFKC e remove caracteres invisíveis; bloco que parece codificado para a geração | 0005 D8 | U |
| 4 | O esquema de saída lista como `enum` só os códigos da etapa de ensino da turma; resposta com código de fora é recusada na validação | RF-42, INV-02 | U |
| 5 | JSON inválido é repetido no máximo 2 vezes; na terceira falha, `invalid_output`. O adaptador falso é chamado exatamente 3 vezes | RNF-35 | U |
| 6 | Recusa do provedor vira `provider_refusal` e não registra uso indevido | IA, seção 3 | U |
| 7 | Minutos menores que 1 ou soma diferente do tempo efetivo: a validação aponta e a geração repete dentro do limite do critério 5 | IA, seção 4 | U |
| 8 | Cada chamada grava um `ai_usage_record` com tokens, custo, modelo, esforço e versão do prompt, e nenhum texto | RF-74, INV-13 | I |
| 9 | INV-12: o custo é calculado em inteiro. 1 milhão de tokens de entrada a US$ 2 dá exatamente 2.000.000.000 nano-dólares | RNF-42 | U |
| 10 | Entrada acima do limite é recusada antes de qualquer chamada paga | RF-73 | U |
| 11 | O prompt vem de `prompts/<operação>/v<N>.md`; dois pedidos da mesma etapa de ensino têm o prefixo fixo idêntico, byte a byte | IA, seção 5 | U |
| 12 | Cada campo gerado traz a origem (`from_material` ou `added`) | RF-21 | U |
| 13 | O adaptador da Anthropic é testado contra respostas gravadas, sem rede; nenhum teste nem o CI usa chave real | IA, seção 6 | U |
| 14 | `npm run generate -- --context <arquivo>` mostra as sugestões e o custo, usando a chave do ambiente local | RNF-38 | Rev |
| 15 | O log da geração não contém texto de prompt nem de resposta | RNF-39 | U |

O passo "gravar sugestões" fica atrás de uma interface; a gravação no banco entra na F07.

### F04 `evals-min`

| # | Critério | Origem | Teste |
|---|---|---|---|
| 1 | `npm run eval -- --suite <nome> --max-cost <valor>` roda o conjunto e para antes de passar do teto de gasto | IA, seção 7.4 | U |
| 2 | A camada determinística (checagens C01 a C16 e G01 a G06) produz a tabela de metas da [AVALIACAO](AVALIACAO.md), seção 6 | RNF-10 | U |
| 3 | O resultado sai em `evals/results/<data>-<versão>.md`, com custo e tempo por plano | RNF-14 | Rev |
| 4 | O conjunto sintético cobre as quatro etapas de ensino, aulas de 25 a 90 minutos efetivos e as situações da [AVALIACAO](AVALIACAO.md), seção 2 | RNF-10 | Rev |
| 5 | O teste de confirmação da 0005 (D7) foi executado e o resultado virou nota datada na 0005 | 0005 D7 | Rev |
| 6 | `evals/private/` está fora do git; nenhum arquivo do conjunto de referência foi commitado | IA, seção 7.1 | CI |
| 7 | O próprio comando de avaliação tem testes com o adaptador falso | RNF-09 | U |

### F05 `auth-sessions`

| # | Critério | Origem | Teste |
|---|---|---|---|
| 1 | Entrar com dados certos cria sessão no servidor e cookie `HttpOnly`, `SameSite=Lax`; o identificador muda a cada login | RNF-07 | I |
| 2 | Senha errada e e-mail inexistente recebem a mesma resposta: "E-mail ou senha incorretos" | RF-03 | I |
| 3 | A quarta tentativa em 10 segundos do mesmo IP é recusada | RF-03, cenário F1.2 | I |
| 4 | Depois de 30 minutos sem uso, o pedido seguinte responde `401` | RF-04 | I |
| 5 | Sair apaga a sessão no servidor; o cookie antigo responde `401` | RNF-07 | I |
| 6 | A senha é guardada com Argon2id; senha de 7 caracteres, de 129 ou da lista proibida é recusada | 0003 D3 | U, I |
| 7 | Conta e administrador são criados só por comando. Nenhuma rota cria nem promove administrador | RF-79 | I |
| 8 | Toda rota de `/api/v1` responde `401` sem sessão. O teste percorre a lista de rotas, então rota nova entra sozinha | RF-70, 0006 D5 | I |
| 9 | Pedido que muda dado com `Origin` de outro site é recusado | 0006 D5 | I |
| 10 | `X-Forwarded-For` enviado pelo cliente não muda o IP usado no limite | 0006 D3 | I |
| 11 | `GET /me` devolve os dados da conta | CONTRATOS, 5.3 | I |
| 12 | Os dois pontos "a confirmar" do DADOS (identificador `uuid`; IP e navegador na sessão) têm resposta registrada | DADOS, seção 8 | Rev |

### F06 `plans-api`

| # | Critério | Origem | Teste |
|---|---|---|---|
| 1 | INV-01: para cada rota com ID, o autor B recebe `404` ao usar o ID de um recurso do autor A; nas listas, não vê nada de A. O teste percorre a lista de rotas | RNF-40 | I |
| 2 | Criar, listar, abrir, alterar e excluir contexto; só etapa de ensino e duração são obrigatórias | RF-11, RF-17 | I |
| 3 | Excluir um contexto não apaga os planos feitos com ele; o plano mantém a cópia | RF-17 | I |
| 4 | Texto com "laudo" não é gravado e responde `privacy_alert`; com a confirmação, grava; o mesmo no plano | RF-15, cenário F2.2 | I |
| 5 | A lista de planos mostra só os do autor, do mais recente para o mais antigo, com paginação | RF-32 | I |
| 6 | `PATCH` com `revision` antiga responde `revision_conflict` e não grava nada | CONTRATOS, seção 2 | I |
| 7 | Campo com sugestão pendente recusa `PATCH`; depois da decisão, aceita | RF-23, RF-24 | I |
| 8 | Aceitar, editar e recusar sugestão mudam o plano como no domínio; decidir de novo responde `409` | RF-23 | I |
| 9 | INV-02: código fora do catálogo responde `skill_not_found`; inserir direto no banco uma habilidade inexistente falha pela chave estrangeira | RF-41, cenário F4.2 | I |
| 10 | Aprovar com achado `block` responde `plan_not_approvable` com os achados; sem bloqueio, cria a versão | RF-25, INV-03 | I |
| 11 | Plano aprovado recusa edição; `reopen` devolve rascunho com a versão seguinte; a versão aprovada continua igual | RF-25 | I |
| 12 | Como `docenda_app`, alterar uma linha de `lesson_plan_version` falha | DADOS, seção 4 | I |
| 13 | INV-09: a função que carrega a entrada da geração não devolve nenhum campo de `plan_identification` | RNF-45 | I |
| 14 | Alterar a identificação de um plano aprovado não muda o estado nem a versão | DADOS, D3 | I |
| 15 | Duplicar copia conteúdo aceito e habilidades; não copia identificação nem sugestões pendentes | RF-32 | I |
| 16 | Excluir o plano apaga versões, sugestões, gerações e identificação | RF-32 | I |
| 17 | Duas cópias da API, apontando para o mesmo banco, atendem a mesma sessão | RNF-33 | I |

Nesta fatia as sugestões entram por dados de teste; a geração de verdade chega na F07.

### F07 `generation-queue`

| # | Critério | Origem | Teste |
|---|---|---|---|
| 1 | Pedir a geração responde `202` e deixa, na mesma transação, a geração `queued`, a reserva e o job. Se o envio à fila falhar, nada disso existe | 0008 D3 | I |
| 2 | INV-05: dois pedidos simultâneos para o mesmo plano devolvem o mesmo `id` e criam uma geração só | RNF-37 | I |
| 3 | INV-11: o job contém só IDs | 0008 D6 | U |
| 4 | INV-04: o worker, com o adaptador falso, termina em `succeeded` com sugestões pendentes; `lesson_plan.content` não muda | RF-23 | I |
| 5 | O campo `stage` passa pelas quatro etapas, na ordem | RNF-04 | I |
| 6 | Ao terminar, o acerto devolve a diferença entre a reserva e o custo real; repetir o acerto não muda o saldo | 0008 D3, DADOS D10 | I |
| 7 | INV-06: com saldo do sistema para uma geração, dois pedidos simultâneos de planos diferentes deixam passar exatamente um | RF-72 | I |
| 8 | Teto do mês atingido responde `ai_budget_reached`; listar, editar, aprovar e exportar continuam funcionando | RF-72, cenário F7.1 | I |
| 9 | Desligar a IA por comando vale no pedido seguinte, sem reiniciar: `ai_disabled` | RF-75 | I |
| 10 | INV-08 e INV-07: geração ativa além do prazo é fechada como `worker_lost`; a reserva inteira fica como custo; o plano aceita nova geração | 0008, nota de 16/11 | I |
| 11 | Com o provedor falso fora do ar, o disjuntor abre e os pedidos seguintes falham na hora com `provider_unavailable`, sem cobrança e sem tempestade de chamadas | RNF-36 | I |
| 12 | Erro conhecido fecha a geração como `failed` sem repetir o job; job que esgota as repetições vai para a fila de falhas, só com IDs | 0008 D4, D6 | I |
| 13 | Conta sem e-mail confirmado recebe `email_not_verified` | RF-70 | I |
| 14 | Parar o worker não derruba a API; pedidos de geração ficam `queued` | RNF-34 | I |

Nesta fatia a reserva é conferida contra o teto do sistema. O crédito por pessoa e os limites por dia e por minuto entram na F11, sobre as mesmas tabelas.

### F08 `web-core`

| # | Critério | Origem | Teste |
|---|---|---|---|
| 1 | Cenários F1.1 e F1.2 | RF-03 | E2E |
| 2 | Cenários F2.1 e F2.2: conversão de hora-aula com a escolha do tempo efetivo; alerta de privacidade sem envio à IA | RF-12, RF-13, RF-15 | E2E |
| 3 | Cenários F3.1, F3.2 e F3.3, com o adaptador falso: campos como sugestão, origem marcada e alerta de instalação | RF-20, RF-21, RF-14 | E2E |
| 4 | Cenários F4.1 e F4.2: habilidades com texto literal; código inexistente recusado | RF-41 | E2E |
| 5 | Cenários F6.1 e F6.2: exporta o aprovado; rascunho sem versão aprovada não exporta | RF-60 | E2E |
| 6 | O arquivo exportado segue as seções do roteiro do estágio, abre sem internet e traz a nota de IA, que o autor pode desligar. Caracteres especiais do texto não viram formatação | RF-60, RF-61 | I |
| 7 | Todas as telas do fluxo cabem em 360 px sem rolagem horizontal | RNF-02 | E2E |
| 8 | O fluxo inteiro (entrar, criar, gerar, decidir, aprovar, exportar) é feito só com o teclado | RNF-03 | E2E |
| 9 | Nenhuma tela tem violação do axe; erro de formulário fica ligado ao campo e o foco vai para o primeiro erro | RNF-03, 0013 D4 | E2E |
| 10 | O andamento da geração é anunciado ao leitor de tela; cada um dos nove motivos de falha tem mensagem própria | RNF-04, DOMINIO 4.2 | E2E, U |
| 11 | Sessão vencida no meio da edição abre o diálogo de entrar; depois do login, o salvamento é repetido e o texto digitado continua na tela | RF-04, RNF-05 | E2E |
| 12 | Depois de usar o editor, `localStorage` e `sessionStorage` não têm texto do plano | 0013 D7 | E2E |
| 13 | As páginas carregam sem violação da CSP, sem estilo nem script em linha | 0006, 0013 D5 | E2E |
| 14 | Todo texto de tela vem do módulo único de textos | 0013 D6 | CI |
| 15 | A tela de geração avisa que o texto é processado por um provedor de IA fora do Brasil | RNF-21 | E2E |

O visual segue o [DESIGN](DESIGN.md).

## M2 (mínimo; fecha na especificação)

### F09 `guardrails`
- O conjunto de guardrails atinge as metas da [AVALIACAO](AVALIACAO.md), seção 6: nenhum dado de aluno sem alerta, nenhum tema sensível legítimo bloqueado, nenhuma injeção que mude o comportamento (`Eval`).
- Bloqueio automático só nas duas categorias inequívocas; nas outras graves, pedido recusado, ocorrência registrada e decisão do administrador (RF-91; `I`).
- Segunda ocorrência leve em 30 dias bloqueia; a primeira só avisa (RF-92, cenário F7.2; `I`).
- Pedido fora do assunto e recusa do provedor não criam ocorrência (RF-95, cenário F7.3; `I`).
- Resposta que falha no `output_check` não aparece para o autor (RF-90; `I`).
- Com a IA bloqueada, ver, editar, exportar e excluir a conta continuam funcionando; o usuário vê o motivo e pede revisão (RF-91, RF-94; `E2E`).

### F10 `invites-email`
- Convite de uso único, válido por 7 dias, preso ao e-mail convidado; o token fica só em hash (RF-78; `I`).
- No máximo 10 contas ativas, valor em configuração (RF-78; `I`).
- O e-mail do convite é apagado no aceite, no cancelamento e no vencimento (REQ-17; `I`).
- Aceite dos termos guardado com data e versão (RF-01; `I`).
- Recuperação de senha: link de 1 hora e uso único; a mensagem não revela se o e-mail existe; trocar a senha encerra as outras sessões (RF-05, cenário F1.3; `I`, `E2E`).
- Troca de e-mail só vale depois da confirmação no endereço novo (RF-06; `I`).

### F11 `ai-credit`
- O crédito do mês entra uma vez só, e a virada segue o horário de Brasília: teste em 31/10 às 23h59 e em 01/11 à 0h00 de Brasília (RF-71; `I`).
- INV-06: o saldo nunca fica negativo, com pedidos simultâneos da mesma pessoa (`I`).
- Sexta geração do dia e terceira no mesmo minuto respondem `ai_rate_limited` (RF-71; `I`).
- Crédito não usado não passa para o mês seguinte (RF-71; `I`).
- A tela mostra o saldo do mês (RF-71; `E2E`).

### F12 `admin`
- Rotas de administrador respondem `403` para autor e nunca devolvem conteúdo de plano ou de contexto (REQ-16; `I`).
- INV-14: como `docenda_app`, alterar ou apagar `audit_event` falha (`I`).
- Cada evento do REQ-22 gera uma linha de auditoria só com IDs (`I`).
- Um registro de acesso por login com sucesso; o job de retenção apaga o que passou de 6 meses, com o papel próprio (0010 D5, D6; `I`).
- Os três alertas por e-mail saem uma vez cada: 80% do teto, disjuntor aberto e job na fila de falhas (0010 D7; `I`).
- Mudar uma configuração fica na auditoria e vale no pedido seguinte (RF-75; `I`).

### F13 `sources`
- A checagem do link recusa endereço privado, local e de metadados, também depois de redirecionamento e quando o nome resolve para endereço interno (RNF-43; `I`).
- Só `https`; tempo e tamanho máximos; só o status é lido (RNF-43; `I`).
- `source-to-review`: fonte entra como "a revisar" e bloqueia a aprovação até o autor decidir (RF-51, cenário F5.1; `I`).
- Domínio fora da lista pede justificativa (RF-53, cenário F5.2; `E2E`).
- Referência em ABNT com "Acesso em" (RF-54; `U`).

### F14 `account-lifecycle`
- Os três passos da exclusão; nome da conta errado não exclui nada (RF-07, cenário F8.2; `E2E`).
- Entrar dentro dos 30 dias cancela a exclusão (RF-07; `I`).
- Passado o prazo, nenhuma tabela do autor guarda linha dele, os registros de uso ficam sem a ligação com a pessoa e existe um recibo sem identificação (seção 7.1; cenário F8.1; `I`).
- Conta parada há 6 meses recebe o aviso e entra no mesmo fluxo (seção 7.1; `I`).

### F15 `evaluate-plan`
- A avaliação devolve achados no formato único, cada um com sugestão opcional que o autor aceita ou recusa (RF-26; `I`).
- Achado feito sobre uma revisão antiga aparece como desatualizado (`I`).
- A avaliação reserva e acerta o custo como a geração (`I`).

### F16 `quality-gate`
- Os cenários F1 a F8 passam como testes ponta a ponta (RNF-09).
- Cada tela passou por uma revisão manual com teclado e leitor de tela, registrada (RNF-03; `Rev`).
- A avaliação completa foi executada e o resultado está em `evals/results/` (RNF-10; `Eval`).
- Um professor ou licenciando criou uma aula em menos de 15 minutos (RNF-01; `Rev`).
