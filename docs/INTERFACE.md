# Interface: Docenda

> **v0.2 (17/11/2026). Tudo aqui é planejado: ainda não há código.** Cada fatia do [BACKLOG](BACKLOG.md) atualiza este arquivo quando a tela ou a rota passa a existir, e troca "planejado" por "feito".
> Decisões: [0012](decisoes/0012-transactions-repositories-and-module-layout.md) (formato da API e dos erros) e [0013](decisoes/0013-front-end-stack.md) (front-end).
> Este arquivo é o **mapa**. O detalhe de cada rota (campos, respostas e erros) está no [CONTRATOS](CONTRATOS.md); o visual das telas, no [DESIGN](DESIGN.md).

## Mapa de telas

| Tela | Caminho | O que o autor faz | Fatia | Estado |
|---|---|---|---|---|
| Entrar | `/entrar` | E-mail e senha; mensagem única de erro | F08 | planejado |
| Meus planos | `/` | Lista do mais recente para o mais antigo; novo, abrir, duplicar, excluir | F08 | planejado |
| Novo plano, passo 1: turma | `/planos/novo` | Escolhe um contexto salvo ou descreve a turma; ajuda curta em cada campo; alerta de privacidade | F08 | planejado |
| Novo plano, passo 2: tempo | `/planos/novo` | Duração em minutos ou horas-aula; escolhe o tempo efetivo sugerido ou outro | F08 | planejado |
| Novo plano, passo 3: tema e material | `/planos/novo` | Tema, subtema e material colado (opcional); aviso de que a IA processa o texto | F08 | planejado |
| Geração | `/planos/:id` | Acompanha as etapas ("conferindo", "gerando", "validando"); mensagem própria para cada falha | F08 | planejado |
| Editor do plano | `/planos/:id` | Cada campo mostra a sugestão da IA com aceitar, editar e recusar; habilidades escolhidas do catálogo; achados ao lado do campo; identificação à parte; aprovar. Plano aprovado abre só para leitura, com a ação de reabrir | F08 | planejado |
| Exportar | `/planos/:id` | Baixa o Markdown da versão aprovada | F08 | planejado |
| Minha conta | `/conta` | Nome, e-mail, saldo de IA do mês, sair, excluir a conta | F10, F11, F14 | planejado |
| Aceitar convite | `/convite/:token` | Cria a conta com o e-mail convidado e aceita os termos | F10 | planejado |
| Recuperar senha | `/senha` | Pede o link e define a senha nova | F10 | planejado |
| Administração | `/admin` | Convites, uso e custo, ocorrências e pedidos de revisão, configuração da IA | F12 | planejado |

Regras de todas as telas (decisão 0013): usáveis em 360 px, só com teclado e com leitor de tela; erro ligado ao campo e anunciado; nenhum texto do plano guardado no navegador.

## Rotas da API (`/api/v1`)

Todas exigem sessão, menos as de autenticação. Recurso de outro autor responde `404`. Erros seguem o formato de problema (RFC 9457), com `code` e o número do pedido.

| Método | Rota | Entrada | Saída | Fatia | Estado |
|---|---|---|---|---|---|
| `GET` | `/healthz`, `/readyz` (fora de `/api/v1`) | — | Vivo; pronto | F00 | planejado |
| — | `/api/auth/*` | Rotas do Better Auth: entrar, sair, sessão | — | F05 | planejado |
| `GET` | `/me` | — | Dados da conta e se a IA está bloqueada | F05 | planejado |
| `GET` | `/skills?stage=&grade=` | Etapa de ensino e ano | Habilidades do catálogo | F01 | planejado |
| `GET`, `POST` | `/class-contexts` | Contexto da turma | Lista; contexto criado | F06 | planejado |
| `GET`, `PATCH`, `DELETE` | `/class-contexts/:id` | Campos alterados | Contexto | F06 | planejado |
| `GET`, `POST` | `/plans` | Contexto, tempo, tema, material | Lista; plano em rascunho | F06 | planejado |
| `GET`, `PATCH`, `DELETE` | `/plans/:id` | Campos editados pelo autor, com o número da revisão | Plano, sugestões pendentes e achados | F06 | planejado |
| `PUT` | `/plans/:id/identification` | Escola, professor, turma, data e turno | Plano | F06 | planejado |
| `POST`, `DELETE` | `/plans/:id/skills`, `/plans/:id/skills/:code` | Código da habilidade | Plano; código fora do catálogo é recusado | F06 | planejado |
| `POST` | `/plans/:id/reopen` | — | Plano de volta a rascunho, na versão seguinte | F06 | planejado |
| `POST` | `/plans/:id/duplicate` | — | Plano novo em rascunho | F06 | planejado |
| `POST` | `/plans/:id/suggestions/:sid/decision` | `accept`, `edit` (com o texto) ou `reject` | Plano atualizado | F06 | planejado |
| `POST` | `/plans/:id/approve` | — | Plano aprovado e a versão; `409` com os achados que bloqueiam | F06 | planejado |
| `POST` | `/plans/:id/generations` | — | `202` com o número da geração; a mesma geração num segundo clique | F07 | planejado |
| `GET` | `/plans/:id/generations/:gid` | — | Estado, etapa e motivo da falha | F07 | planejado |
| `POST` | `/plans/:id/exports` | Versão aprovada e formato | Arquivo para baixar; registra o evento de exportação | F08 | planejado |
| `GET` | `/me/ai-credit` | — | Saldo do mês | F11 | planejado |
| vários | `/admin/*` | Papel de administrador (o 2FA entra antes de publicar, 0009 D10) | Convites, uso, ocorrências, configuração | F12 | planejado |

## Dados compartilhados

Os esquemas de pedido e resposta ficam em `packages/contracts` (Zod) e valem para a API e para a tela. Os campos do plano estão no [DOMINIO](DOMINIO.md), seção 3, e o formato exato no [CONTRATOS](CONTRATOS.md), seção 4.
