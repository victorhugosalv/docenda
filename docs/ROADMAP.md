# Roteiro de longo prazo

> **v0.1 (16/11/2026).** O que vem depois da v1 e o que faria cada decisão mudar. A v1 (M0 a M2) está no [BACKLOG](BACKLOG.md). Nada aqui é promessa de data: é a ordem e o motivo.

## Marcos

| Marco | Entrega | Pronto quando |
|---|---|---|
| M0 a M2 | A v1: ver o [BACKLOG](BACKLOG.md) | Os cenários F1 a F8 do REQUISITOS passam como testes |
| **M3 Publicação** | A v1 no ar para os convidados | Lista "antes de publicar" abaixo, completa |
| **M4 Produto** | O que o [PROPÓSITO](PROPOSITO.md) promete além do plano de uma aula | Um licenciando faz o ciclo completo com o orientador dentro do sistema |
| **M5 Escala e sustento** | O sistema se paga e aguenta crescer | Custo por plano conhecido e um modelo de cobrança testado |

### M3 Publicação: lista "antes de publicar"
Reúne o que as decisões já deixaram para este momento.

| Item | Origem |
|---|---|
| Mapa de ameaças (STRIDE): a v0.1 de [AMEACAS](AMEACAS.md) foi escrita em 17/11; revisar contra o sistema construído e resolver cada "Falta" | 0009 D10 |
| 2FA do administrador | 0009 D10 |
| Varredura com OWASP ZAP e uma auditoria de segurança do código | 0009 D10 |
| Checklist do ASVS nível 1 conferido item a item | RNF-06 |
| Termos de uso e aviso de privacidade, com versão; base legal de cada tratamento conferida; canal de contato do titular | REQUISITOS, seção 8 |
| Servidor Lightsail em São Paulo, Caddy, SES fora do sandbox | 0011, 0003 |
| Backup diário, cópia fora do servidor e **um teste de restauração** | 0011 D4 |
| Simulação de incidente, com o tempo de cada passo | 0010 D8 |
| Varredura da imagem Docker por vulnerabilidades | novo |
| Valores finais de crédito, limites e teto, depois de um teste de carga mínimo | REQUISITOS, seção 8 |
| Teste de uso com um professor ou licenciando (meta de 15 minutos) | RNF-01 |

### M4 Produto
| Bloco | Requisitos | Observação |
|---|---|---|
| Revisores: convidar, comentar, dar nota, retirar acesso | RF-80 a RF-85 | Traz o estado "em revisão" e o consentimento de melhoria (RF-86) |
| Outras etapas de produção: roteiro simplificado, slides, roteiro padronizado, guia de estudo | RF-29 | Cada uma só depois da aprovação da anterior |
| Modelos ENADE e as regras de duração e prazo | RF-31; seção 4 | Entra como modelo novo, não como código novo (DOMINIO, seção 8) |
| Adaptar um plano a um imprevisto; regenerar um campo com instrução | RF-28, RF-33 | Usa as versões guardadas |
| Fontes sugeridas pela IA, com busca real, e conferência do fato na fonte | RF-50, RF-55 | IA.md, seção 8 |
| Envio de arquivos e exportação em PDF e DOCX | RF-22, RF-60 | Checagens de arquivo do ASVS V5 |
| Passkeys, código de autenticador, login com Google | RF-09, RF-10 | Plugins do Better Auth (0003 D5) |
| Baixar os próprios dados | RF-08 | |
| Planos de semana, mês e semestre | RF-30 | Depende das versões e do catálogo |

### M5 Escala e sustento
| Bloco | Observação |
|---|---|
| Custo por plano e estudo de assinatura | Os dados de uso já permitem (RNF-14) |
| Pagamento | Porta própria, como os outros serviços externos (RNF-30) |
| Chave de API do próprio usuário; modelos gratuitos | RF-76, RF-77 |
| Escolas e equipes | Coluna de organização nos recursos; o dono já existe em tudo |
| Outros componentes da BNCC | Dados novos no catálogo genérico |
| Busca por significado no catálogo | Só quando o catálogo não couber no pedido; aí, `pgvector` no próprio PostgreSQL |

## Sinais para mudar de arquitetura

Cada troca tem um sinal medido. Sem o sinal, a troca não acontece.

| Hoje | Sinal | Depois | Onde está decidido |
|---|---|---|---|
| Um container da API | Mais de uma cópia é necessária | Contadores de limite no banco ou no Redis | 0006 |
| Fila no PostgreSQL, sem Redis | Cache com ganho medido, contadores pesando no banco ou tela em tempo real | Redis para cache e contadores; a fila continua no PostgreSQL | 0008 D8 |
| A tela pergunta o andamento a cada 2 s | Muitas gerações simultâneas ou bateria no celular | Envio pelo servidor (SSE) | 0008 D3 |
| Trava global no teto do mês | Fila de espera na reserva | Contador atômico por período, sem trava de linha | DOMINIO, seção 6 |
| Worker com 2 tarefas por vez | Fila crescendo com orçamento sobrando | Mais workers; o limite real passa a ser a taxa do provedor | 0008 D7 |
| Lightsail com tudo num servidor | Mais de uma cópia, alta disponibilidade ou o banco pesando | ECS com RDS, mesma imagem | 0011 D6 |
| Log na saída padrão e 3 alertas | Erros que ninguém vê, ou mais de um servidor | Rastreamento e métricas (OpenTelemetry) e um coletor de erros | 0010 D7 |
| Checagem de dono no código | Um erro de autorização em produção, ou muitos módulos novos | Segurança por linha no banco (RLS), como segunda camada | 0012 |
| ASVS nível 1 | Dados mais sensíveis ou cliente que exige | Itens do nível 2, capítulo por capítulo | 0006 |
| Um provedor de IA | Indisponibilidade, preço ou exigência de dados no Brasil | Segundo adaptador (Bedrock ou outro), depois da avaliação | 0005 |
| Pastas reutilizáveis dentro da API | Um segundo projeto vivo precisa do mesmo módulo | Pacote no workspace | 0001 |

## O que não muda

Seja qual for o marco, valem as garantias do [PROPÓSITO](PROPOSITO.md) e os invariantes do [DOMÍNIO](DOMINIO.md): o autor decide, nada inventado, nenhum aluno identificável e uso da IA sob limite.
