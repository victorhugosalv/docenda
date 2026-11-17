# Docenda

Plataforma web para professores de Computação da educação básica e licenciandos em estágio **criarem, avaliarem e roteirizarem planos de aula** alinhados à BNCC Computação. A estrutura do plano é fixa; a IA adapta só o conteúdo ao contexto da turma (conhecimento prévio, dificuldades, contexto social e acessibilidade).

> **Status (17/11/2026):** requisitos, domínio, arquitetura, dados, contratos da API, critérios de aceite, design, avaliação da IA e mapa de ameaças planejados; o código começa pela fundação ([backlog](docs/BACKLOG.md)). **Ainda não há código.** As tecnologias abaixo são planejadas e cada escolha está registrada numa decisão de arquitetura (ADR).

## O problema
Montar um plano de aula alinhado à BNCC e adaptado à turma real leva tempo. Modelos prontos são genéricos, e as IAs comuns inventam habilidades e fontes.

## O que a Docenda garante
- **O professor decide:** a IA sugere, o autor aceita, edita ou recusa, e o sistema mostra o que veio da IA.
- **Nada inventado:** habilidades só do catálogo oficial da BNCC Computação; fontes só reais e aprovadas pelo professor.
- **Nenhum aluno identificável:** a turma é descrita de forma agregada, sem nome, diagnóstico ou laudo (LGPD).
- **Uso controlado da IA:** só usuário autenticado, com limite por pessoa e teto de gasto.

## Arquitetura planejada
| Tema | Decisão |
|---|---|
| Estrutura | Monólito modular com portas e adaptadores; nada preso a um fornecedor ([0001](docs/decisoes/0001-modular-monolith-ports-and-adapters.md)) |
| Stack | Node.js 26 LTS com TypeScript, Express 5, React + Vite, Prisma 7 ([0002](docs/decisoes/0002-web-stack.md)) |
| Autenticação | Sessão no servidor com Better Auth ([0003](docs/decisoes/0003-authentication.md)) |
| Banco | PostgreSQL 18, IDs UUID v7 ([0004](docs/decisoes/0004-postgresql.md)) |
| IA | API da Anthropic, sem treino com os dados; guardrails do OWASP Top 10 para LLM 2025 e orçamento de tokens ([0005](docs/decisoes/0005-ai-provider.md)) |
| Segurança web | OWASP ASVS 5.0 nível 1 ([0006](docs/decisoes/0006-web-security-asvs-level-1.md)) |
| Ambiente | Docker Compose, container sem root e ambiente isolado para o agente de código ([0007](docs/decisoes/0007-docker-and-isolated-environment.md)) |
| Fila | Geração por IA em fila no próprio PostgreSQL (pg-boss), separada das telas ([0008](docs/decisoes/0008-ai-job-queue-postgresql.md)) |
| CI/CD | Tipos, testes, CodeQL, Dependabot e busca de segredos em cada pull request; imagem publicada pelo hash do commit; mapa de ameaças (STRIDE), 2FA do administrador e OWASP ZAP antes de publicar ([0009](docs/decisoes/0009-ci-cd-and-security-checks.md)) |
| Logs e incidentes | Log em JSON sem dados pessoais, registro de acesso e auditoria separados, alertas ao administrador e plano de resposta a incidentes ([0010](docs/decisoes/0010-logs-monitoring-and-incident-response.md)) |
| Transações e módulos | Molde de pastas de um módulo, transação compartilhada entre módulos, banco testado de verdade e papéis separados no PostgreSQL ([0012](docs/decisoes/0012-transactions-repositories-and-module-layout.md)) |
| Front-end | React Router, TanStack Query, React Hook Form e componentes acessíveis; WCAG 2.2 AA conferido em cada tela ([0013](docs/decisoes/0013-front-end-stack.md)) |
| Método | Uma especificação por fatia com o Spec Kit ([0014](docs/decisoes/0014-spec-driven-workflow-spec-kit.md)) |
| Implantação | v1 em demonstração local; depois, um servidor Lightsail em São Paulo com o mesmo Compose, HTTPS automático e backups de 7 dias, abaixo de US$ 20 por mês ([0011](docs/decisoes/0011-low-cost-aws.md)) |

Trilha de arquitetura concluída em 16/11/2026 ([trilha](docs/TRILHA-ARQUITETURA.md)) e revisada no mesmo dia. O próximo passo é o código, uma fatia por vez.

## Como o projeto é feito
Um tópico de arquitetura por vez: conceito, boa prática com fonte oficial conferida na data, duas ou três opções com custo e uma decisão registrada em ADR. O desenvolvimento usa o Claude Code como apoio, com revisão humana de cada decisão.

## Documentos
- [Propósito](docs/PROPOSITO.md): problema, público, garantias e o que o produto não faz.
- [Requisitos](docs/REQUISITOS.md): requisitos funcionais e não funcionais, LGPD e pontos em aberto.
- [Domínio](docs/DOMINIO.md): entidades, estados, invariantes e o glossário português–código.
- [IA](docs/IA.md): caminho de uma geração, esquema de saída e prompts; [avaliação](docs/AVALIACAO.md): casos de teste, rubrica e metas.
- [Dados](docs/DADOS.md): tabelas, papéis do banco e o que acontece na exclusão.
- [Contratos da API](docs/CONTRATOS.md): formatos, erros e limites de cada rota.
- [Backlog](docs/BACKLOG.md), [critérios de aceite](docs/ACEITE.md) e [roteiro de longo prazo](docs/ROADMAP.md): marcos, fatias, como saber que cada uma está pronta e os sinais para mudar de arquitetura.
- [Interface](docs/INTERFACE.md) e [design](docs/DESIGN.md): mapa de telas e rotas; cores, letras e comportamento das telas.
- [Mapa de ameaças](docs/AMEACAS.md): fluxo de dados e defesas, pelo método STRIDE.
- [Trilha de arquitetura](docs/TRILHA-ARQUITETURA.md) e [decisões (ADRs)](docs/decisoes/).
- [Plano de resposta a incidentes](docs/INCIDENTES.md).

## Autor
Victor Hugo Salviano, licenciando em Ciência da Computação (UFPB). [LinkedIn](https://www.linkedin.com/in/victor-hugo-salviano).

© 2026 Victor Hugo Salviano. Todos os direitos reservados.
