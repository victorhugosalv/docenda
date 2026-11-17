# Backlog: Docenda

> **v0.1 (16/11/2026).** Fatias verticais, em ordem de execução. Cada fatia é uma funcionalidade do Spec Kit (decisão [0014](decisoes/0014-spec-driven-workflow-spec-kit.md)): uma pasta em `specs/`, uma branch e um pull request.
> Marcos: M0 a M2 formam a **v1** do [REQUISITOS](REQUISITOS.md). M3 em diante está no [ROADMAP](ROADMAP.md).
> O que cada fatia precisa provar para estar pronta está no [ACEITE](ACEITE.md); as tabelas, no [DADOS](DADOS.md); as rotas, no [CONTRATOS](CONTRATOS.md).

## Marcos

| Marco | O que entrega | Fatias |
|---|---|---|
| **M0 Fundação** | O repositório sobe com um comando e o CI passa | F00 |
| **M1 Núcleo demonstrável** | Um autor entra, descreve a turma, gera um plano com habilidades reais, decide cada sugestão, aprova e exporta | F01 a F08 |
| **M2 v1 para convidados** | Convites, crédito, proteção contra uso indevido, administração, auditoria e exclusão de conta | F09 a F15 |

## Fatias

| # | Fatia | Entrega | Áreas | Requisitos | Marco | Status |
|---|---|---|---|---|---|---|
| F00 | `foundation` | Workspaces, TypeScript, lint, fronteiras, Compose (PostgreSQL, Mailpit, migração), papéis do banco, `/healthz` e `/readyz`, log, configuração validada, formato de erro, CI, Dockerfile | infra, back | RNF-09, RNF-11, RNF-32, RNF-41 | M0 | a fazer |
| F01 | `bncc-catalog` | Catálogo importado, conferido e versionado; consulta por etapa de ensino e ano; validação de código e de etapa | banco, back | RF-40, RF-41, RF-43 | M1 | a fazer |
| F02 | `plan-core` | Regras puras: conversão de hora-aula, tempo efetivo, soma dos tempos, estados do plano, sugestões, achados e condição de aprovar | back (domínio) | RF-12, RF-13, RF-14, RF-23, RF-25, RF-27, seção 4 | M1 | a fazer |
| F03 | `ai-generation` | Porta de IA, adaptador falso e adaptador da Anthropic; esquema de saída com habilidades em lista fechada; pipeline de geração; registro de uso e custo; comando para gerar a partir de um arquivo | IA, back | RF-20, RF-21, RF-42, RF-73, RF-74, RNF-35, RNF-38 | M1 | a fazer |
| F04 | `evals-min` | Comando de avaliação com a camada determinística, conjunto sintético e tabela de custo e tempo; teste de confirmação da 0005 (D7) | IA | RNF-10, RNF-14 | M1 | a fazer |
| F05 | `auth-sessions` | Better Auth com sessão no servidor, Argon2id, limite por IP, criação de conta por comando, checagem de dono | back, banco | RF-03, RF-04, RF-79, RNF-07, RNF-40 | M1 | a fazer |
| F06 | `plans-api` | Contexto da turma e plano gravados; listar, abrir, editar, duplicar e excluir; decidir sugestões; aprovar; versões | back, banco | RF-11, RF-16, RF-17, RF-24, RF-25, RF-32 | M1 | a fazer |
| F07 | `generation-queue` | pg-boss e worker; geração em transação com reserva contra o teto do sistema (as tabelas do livro-razão nascem aqui); andamento por etapas; clique duplo; encerramento de geração órfã; teto e liga e desliga por configuração | back, banco, infra | RF-70, RF-72, RF-75, RNF-33, RNF-34, RNF-36, RNF-37 | M1 | a fazer |
| F08 | `web-core` | Telas: entrar, lista de planos, novo plano em passos, editor com sugestões e achados, exportar em Markdown | front | RF-60, RF-61, RNF-02, RNF-03, RNF-04, RNF-05; cenários F1 a F4 e F6 | M1 | a fazer |
| F09 | `guardrails` | `input_check` e `output_check`; alerta de privacidade na tela; ocorrências de uso indevido; recusa explicada | IA, back, front | RF-15, RF-90, RF-91, RF-92, RF-95, RNF-08 | M2 | a fazer |
| F10 | `invites-email` | Convite por e-mail, cadastro só por convite, termos com versão, recuperação de senha, troca de e-mail | back, front | RF-01, RF-02, RF-05, RF-06, RF-78 | M2 | a fazer |
| F11 | `ai-credit` | Crédito mensal por pessoa sobre o livro-razão da F07, limites por dia e por minuto, saldo na tela | back, front | RF-71, RNF-42 | M2 | a fazer |
| F12 | `admin` | Painel: convites, uso e custo, ocorrências e revisão de bloqueio, configuração; auditoria, registro de acesso e os três alertas por e-mail | back, front | RF-93, RF-94, RNF-39; decisão 0010 | M2 | a fazer |
| F13 | `sources` | Fonte trazida pelo autor: lista de domínios, checagem segura do link, estado "a revisar", referência em ABNT | back, front | RF-51, RF-52, RF-53, RF-54, RNF-43 | M2 | a fazer |
| F14 | `account-lifecycle` | Exclusão em 3 passos com 30 dias de prazo, contas inativas, retenções e jobs de limpeza | back, front | RF-07; seção 7.1 | M2 | a fazer |
| F15 | `evaluate-plan` | Avaliação do plano: regras e rubrica, com achados e sugestões | IA, back, front | RF-26 | M2 | a fazer |
| F16 | `quality-gate` | Revisão de acessibilidade com leitor de tela, cenários e2e F1 a F8, avaliação completa, README com demonstração | todas | RNF-01, RNF-03, RNF-09, RNF-10 | M2 | a fazer |

**Se o tempo apertar, saem primeiro:** F15, depois F14 (fica a exclusão simples, sem contas inativas) e F13 (o plano sai sem bibliografia conferida pelo sistema). F09 e F11 não saem: sem elas, ninguém de fora deve usar a IA.

## Ordem e motivo

1. **Regras e IA antes das telas (F01 a F04).** O valor da Docenda está em "nada inventado" e "cabe no tempo". Essas partes são funções puras e um pipeline testável por comando: dão para construir e medir sem tela nem login.
2. **Login e gravação (F05, F06)** vêm quando há o que guardar.
3. **Fila (F07) separada do pipeline (F03).** O pipeline não sabe quem o chamou; a fila só o transporta.
4. **Telas (F08)** fecham o M1 com um fluxo completo.
5. **M2** acrescenta o que protege o sistema quando outras pessoas entram.

## Ritmo sugerido (estimativa, não promessa)

| Dia | Fatias |
|---|---|
| 17/11 | Revisão do planejamento em PR; F00 |
| 18/11 | F01, F02 |
| 19/11 | F03, F04 |
| 20/11 | F05, F06 |
| 21/11 | F07, início da F08 |
| 22/11 | F08, F09 |
| 23/11 | F10, F11 |
| 24/11 | F12, F16 e o que couber de F13 a F15 |

Duas fatias por dia só funcionam com revisão no mesmo dia. Se uma fatia estourar, o corte segue a lista acima; o prazo não se estica cortando teste.

## Definição de pronto (toda fatia)

- [ ] Especificação em `specs/NNN-<fatia>/` cita os requisitos pelo ID, sem copiar o texto.
- [ ] Todos os critérios da fatia no [ACEITE](ACEITE.md) têm teste passando.
- [ ] Os testes foram escritos antes do código; cada teste de regra leva o ID do requisito ou do invariante no nome.
- [ ] O CI passa: tipos, lint, fronteiras, testes e build da imagem.
- [ ] Nenhuma chamada real à IA em teste; nenhum segredo no código; nenhum dado pessoal em log.
- [ ] Se a fatia tem tela: usável em 360 px, navegável só por teclado e sem erro do axe.
- [ ] Se a fatia mudou um requisito, uma regra do domínio ou uma decisão, o documento foi atualizado no mesmo PR.
- [ ] Pull request pequeno, com o que foi feito, como testar e o que ficou de fora.

## Entrada de cada fatia para o Spec Kit

O texto abaixo é o ponto de partida do `/speckit-specify`. O plano técnico (`/speckit-plan`) segue as decisões já tomadas e só cria decisão nova por ADR.

- **F00 `foundation`.** Montar o repositório para que `docker compose up` suba banco, e-mail de teste, migração, API e worker, e para que cada PR rode tipos, lint, fronteiras, testes e build. Sem funcionalidade de produto. Decisões: 0001, 0002, 0007, 0009, 0010 (D2, D7), 0012.
- **F01 `bncc-catalog`.** Importar as 140 habilidades do complemento da BNCC Computação uma única vez, a partir do documento oficial, com conferência automática do texto e uma amostra conferida à mão. Consultar por etapa de ensino e ano. Recusar código inexistente e alertar quando a habilidade não combina com a turma.
- **F02 `plan-core`.** Todas as regras da seção 4 do REQUISITOS como funções puras, mais os estados do plano e da sugestão e a condição de aprovar (DOMINIO, seções 4, 5 e 7). Sem banco e sem IA.
- **F03 `ai-generation`.** Gerar as sugestões de um plano a partir de tema, contexto e material colado, pelo caminho da seção 3 do IA.md, com o provedor atrás da porta. Entregar um comando que gera a partir de um arquivo de contexto e mostra o custo.
- **F04 `evals-min`.** Rodar um conjunto de contextos pelo pipeline e produzir a tabela da camada determinística, com custo e tempo. Executar o teste de confirmação da 0005.
- **F05 `auth-sessions`.** Entrar e sair com e-mail e senha, com sessão no servidor e mensagem única de erro. Contas criadas por comando até a F10. Todo recurso checa o dono.
- **F06 `plans-api`.** Guardar contexto e plano; listar, abrir, editar, duplicar e excluir; aceitar, editar ou recusar cada sugestão; aprovar e guardar a versão.
- **F07 `generation-queue`.** Pedir a geração devolve na hora um número; o trabalho roda no worker; a tela acompanha as etapas. Registrar, reservar e enfileirar numa transação só. Nenhuma geração fica presa.
- **F08 `web-core`.** As telas do fluxo principal, no celular e com leitor de tela (INTERFACE, mapa de telas).
- **F09 a F16.** Especificadas quando o M1 fechar, com o que o uso real do M1 ensinar.
