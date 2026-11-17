# Trilha de fundamentos da arquitetura

Começou em 13/11/2026, antes do Spec Kit. É um tópico por vez: explicação, boa prática com fonte atual, opções para a Docenda e decisão do Victor. Cada decisão fica em `docs/decisoes/` no formato curto de ADR (registro de decisão de arquitetura).

| # | Tópico | Status | Decisão |
|---|---|---|---|
| 1 | Arquitetura desacoplada: monólito modular, portas e adaptadores, estrutura de pastas e módulos reutilizáveis (nada preso a um provedor) | decidido | [0001](decisoes/0001-modular-monolith-ports-and-adapters.md), [0002](decisoes/0002-web-stack.md), [0004](decisoes/0004-postgresql.md) |
| 2 | Autenticação: sessão × JWT, token de renovação, hash de senha, e-mail transacional; 2FA e login social depois | decidido | [0003](decisoes/0003-authentication.md) |
| 3 | Segurança web: OWASP ASVS 5.0 nível 1 aplicado à Docenda | decidido | [0006](decisoes/0006-web-security-asvs-level-1.md) |
| 4 | Segurança e custo de IA: OWASP Top 10 para LLM 2025, orçamento de tokens, provedor e modelo da v1 (sem treino com nossos dados) | decidido | [0005](decisoes/0005-ai-provider.md) (modelos e limites confirmados pelo teste com os 3 planos de referência) |
| 5 | Docker e ambiente isolado para o Claude: container no WSL2 (preferido), sandbox como alternativa, sessão na nuvem; container da aplicação sem root e só com o necessário | decidido | [0007](decisoes/0007-docker-and-isolated-environment.md) |
| 6 | CI/CD: testes, análise de segurança (SAST, dependências e busca de segredos commitados) e branch protegida; mapa de ameaças de uma página (STRIDE); 2FA só para o administrador na v1; varredura com OWASP ZAP (DAST) contra o container | decidido | [0009](decisoes/0009-ci-cd-and-security-checks.md) |
| 7 | AWS com custo baixo (se faltar tempo, a demonstração local basta) | decidido | [0011](decisoes/0011-low-cost-aws.md) (v1 local; depois Lightsail em São Paulo, abaixo de US$ 20 por mês) |
| 8 | Performance e escala: fila da geração por IA no próprio PostgreSQL (pg-boss, `SKIP LOCKED`), repetições em camadas, clique duplo, concorrência do worker e Redis só quando houver motivo medido | decidido | [0008](decisoes/0008-ai-job-queue-postgresql.md) |
| 9 | Logs e monitoramento; plano de incidente (trocar segredos, encerrar sessões, avisar a ANPD e os titulares, LGPD art. 48, prazo a conferir) | decidido | [0010](decisoes/0010-logs-monitoring-and-incident-response.md); plano em [INCIDENTES](INCIDENTES.md) (3 dias úteis, Resolução CD/ANPD 15/2024) |

## Revisão do planejamento (16/11)
Com a trilha concluída, uma revisão independente de requisitos e decisões apontou o que faltava antes do código. Ela gerou três decisões e notas datadas nas 0001, 0002, 0005, 0006 e 0008.

| # | Tópico | Decisão |
|---|---|---|
| 10 | Molde de módulo, transação entre módulos, repositórios sem banco falso, papéis do banco, formato de erro | [0012](decisoes/0012-transactions-repositories-and-module-layout.md) |
| 11 | Front-end: rotas, dados do servidor, formulários, acessibilidade | [0013](decisoes/0013-front-end-stack.md) |
| 12 | Método de desenvolvimento por especificação (Spec Kit) | [0014](decisoes/0014-spec-driven-workflow-spec-kit.md) |

Da mesma revisão saíram o [DOMINIO](DOMINIO.md), o [IA](IA.md), o [BACKLOG](BACKLOG.md), o [ROADMAP](ROADMAP.md), a [INTERFACE](INTERFACE.md) e o REQUISITOS v0.3.

Em 17/11, cinco pacotes detalharam o que o código precisa para começar sem decidir no escuro: [DADOS](DADOS.md) (tabelas e papéis), [CONTRATOS](CONTRATOS.md) (API), [ACEITE](ACEITE.md) (critérios por fatia), [AVALIACAO](AVALIACAO.md) com os rascunhos dos prompts e os conjuntos de teste, [DESIGN](DESIGN.md) e o [mapa de ameaças](AMEACAS.md), adiantado do marco M3.

## Ordem de execução (15/11)
A numeração acima continua sendo o total da trilha. A ordem de trabalho, enxuta para a primeira versão, é **5 → 8 → 6 → 9 → 7**:
1. **5. Docker:** `docker compose` com PostgreSQL e Mailpit, para o sistema subir com um comando; container sem root. Ambiente isolado para o Claude junto.
2. **8. Fila:** escolher a fila da geração por IA (RNF-34), de preferência sobre o próprio PostgreSQL; o resto já está nos RNF-33 a 37.
3. **6. CI:** desde o primeiro código, tipos, testes e alerta de dependências, com o trabalho em branches e pull request para a main protegida. Antes de publicar: ZAP, mapa de ameaças e 2FA do admin.
4. **9. Logs:** logger simples desde o começo; monitoramento, alertas e plano de incidente antes de publicar.
5. **7. AWS:** nota curta: demonstração local na v1, AWS depois.

## Pontos em aberto do REQUISITOS (seção 8) ligados à trilha
| Ponto | Tópico que fecha |
|---|---|
| 1. Data da v1 | Reavaliar ao fim da trilha |
| 2. Limites de uso e teto mensal | 4: valores iniciais na [0005](decisoes/0005-ai-provider.md); finais depois do teste |
| 3. Provedor e modelo de IA; provedor de e-mail | Fechado: IA na [0005](decisoes/0005-ai-provider.md); e-mail na [0003](decisoes/0003-authentication.md) |
| 4. Registros de acesso por 6 meses | 9 |
| 5. Prazo de arrependimento na exclusão | Fora da trilha (requisitos), salvo se a arquitetura pedir |
| 6. Formato de exportação (PDF ou DOCX) | Fora da trilha (requisitos) |
| 7. Retenção de contas inativas | Fora da trilha (requisitos) |
