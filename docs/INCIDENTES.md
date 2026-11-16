# Plano de resposta a incidentes

> Decisão: [0010](decisoes/0010-logs-monitoring-and-incident-response.md). Base: LGPD, art. 48, e Resolução CD/ANPD 15/2024 (conferida em 16/11/2026). Ensaiado uma vez antes de publicar.

**Incidente de segurança** é qualquer evento que comprometa o sigilo, a integridade ou a disponibilidade de dados pessoais. Exemplos: uma chave vazada num commit, alguém entrando numa conta que não é sua ou o banco exposto.

O que já limita o estrago: a Docenda não guarda dados de alunos identificáveis (seção 7 dos requisitos). Um vazamento atinge, no máximo, os dados das contas.

## Os 7 passos

| # | Passo | O que fazer | Quando |
|---|---|---|---|
| 1 | **Conter** | Desligar a IA (teto em zero) e suspender as contas envolvidas; tirar a aplicação do ar se o ataque continuar | Na hora |
| 2 | **Trocar os segredos** | `BETTER_AUTH_SECRET` (derruba todos os cookies), senha do banco, chave da Anthropic e credenciais de e-mail; segredo que entrou num commit é trocado, não só apagado do histórico | Na hora |
| 3 | **Encerrar as sessões** | Apagar todas as sessões no banco; todos entram de novo | Na hora |
| 4 | **Avaliar o risco** | Que dados, de quantas pessoas, desde quando. O incidente pode afetar direitos dos titulares e envolve algum critério do art. 5º (credenciais de autenticação, dados sensíveis, larga escala, dados de crianças, sigilo)? | Até 1 dia útil |
| 5 | **Comunicar** | Se houver risco ou dano relevante: à ANPD (formulário no site) e a cada titular por e-mail, em linguagem simples, com o conteúdo mínimo do art. 9º (abaixo) | **Até 3 dias úteis** depois de saber que dados pessoais foram afetados |
| 6 | **Registrar** | Todo incidente, comunicado ou não, no registro privado: data, o que aconteceu, dados afetados, avaliação de risco, medidas e comunicações | Sempre; guardado por **5 anos** |
| 7 | **Aprender** | Virar um teste, um alerta ou uma ADR, para não se repetir | Depois de encerrado |

## O que o aviso aos titulares precisa ter (art. 9º)
- O que aconteceu e quais dados foram afetados.
- Os riscos e os possíveis impactos.
- O que já foi feito e o que a pessoa pode fazer (por exemplo, trocar a senha se ela for repetida em outro site).
- A data em que o incidente foi descoberto e, se houver, o motivo do atraso.
- Um contato para dúvidas.

## Onde fica cada coisa
- **Este plano:** público, no repositório.
- **Registro de incidentes:** privado, fora do repositório, guardado por 5 anos.
- **Logs e registros usados na investigação:** log técnico (sem dados pessoais), registro de acesso (6 meses) e auditoria (6 meses), conforme a [0010](decisoes/0010-logs-monitoring-and-incident-response.md).
