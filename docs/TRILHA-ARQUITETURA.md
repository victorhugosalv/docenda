# Trilha de fundamentos da arquitetura

Começou em 13/11/2026, antes do Spec Kit. É um tópico por vez: explicação, boa prática com fonte atual, opções para a Docenda e decisão do Victor. Cada decisão fica em `docs/decisoes/` no formato curto de ADR (registro de decisão de arquitetura).

| # | Tópico | Status | Decisão |
|---|---|---|---|
| 1 | Arquitetura desacoplada: monólito modular, portas e adaptadores, estrutura de pastas e módulos reutilizáveis (nada preso a um provedor) | decidido | [0001](decisoes/0001-modular-monolith-ports-and-adapters.md), [0002](decisoes/0002-web-stack.md), [0004](decisoes/0004-postgresql.md) |
| 2 | Autenticação: sessão × JWT, token de renovação, hash de senha, e-mail transacional; 2FA e login social depois | decidido | [0003](decisoes/0003-authentication.md) |
| 3 | Segurança web: OWASP ASVS 5.0 nível 1 aplicado à Docenda | decidido | [0006](decisoes/0006-web-security-asvs-level-1.md) |
| 4 | Segurança e custo de IA: OWASP Top 10 para LLM 2025, orçamento de tokens, provedor e modelo da v1 (sem treino com nossos dados) | decidido | [0005](decisoes/0005-ai-provider.md) (modelos e limites confirmados pelo teste com os 3 planos de referência) |
| 5 | Docker e ambiente isolado para o Claude: container no WSL2 (preferido), sandbox como alternativa, sessão na nuvem; container da aplicação sem root e só com o necessário | decidido | [0007](decisoes/0007-docker-and-isolated-environment.md) |
| 6 | CI/CD: testes, análise de segurança (SAST, dependências e busca de segredos commitados) e branch protegida; mapa de ameaças de uma página (STRIDE); 2FA só para o administrador na v1; varredura com OWASP ZAP (DAST) contra o container | pendente | — |
| 7 | AWS com custo baixo (se faltar tempo, a demonstração local basta) | pendente | — |
| 8 | Performance e escala | pendente | — |
| 9 | Logs e monitoramento; plano de incidente (trocar segredos, encerrar sessões, avisar a ANPD e os titulares, LGPD art. 48, prazo a conferir) | pendente | — |

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
