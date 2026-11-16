# Requisitos: Docenda

> **Rascunho v0.2 (13/11/2026).** Antes de ser congelado, passa por uma revisão independente (necessário / vale a pena / enchimento).
> Base: [PROPOSITO.md](PROPOSITO.md), notas de estudo das referências (privadas, sem dados pessoais) e um checklist de LGPD e privacidade desde a concepção.

## 0. Como ler
- **Deve** = obrigatório. **Deveria** = desejável; sai primeiro se faltar tempo.
- **Fase:** `v1` = primeira versão publicada. `depois` = próximas versões. A data da v1 está no ponto em aberto 1.
- **Verificação:** `U` teste unitário, `I` teste de integração, `E2E` teste ponta a ponta (cenário BDD), `Eval` avaliação do conteúdo gerado pela IA, `Rev` revisão manual. O sistema todo segue TDD; só o **conteúdo** da IA é medido por evals.
- **Dados pessoais?** indica se o requisito toca dado pessoal. O detalhe está na seção 7.
- **Princípio geral (seção 5.4):** tudo é modular e desacoplado. Nenhuma funcionalidade depende diretamente de um serviço externo específico.

## 1. Atores
| Ator | Quem é | Fase |
|---|---|---|
| Licenciando | Estudante de licenciatura em estágio; cria planos | v1 |
| Professor | Professor da educação básica; cria e avalia os próprios planos | v1 |
| Revisor | Convidado pelo licenciando: professor orientador da faculdade ou supervisor da escola | depois |
| Administrador | Mantém o sistema: convites (na v1, o cadastro é só por convite), limites, teto de gasto, liga e desliga a IA, revisa bloqueios por uso indevido | v1 |

"Autor" = licenciando ou professor, dono do plano.

## 2. Glossário
- **Plano de aula:** documento de uma ou mais aulas, num modelo (roteiro do estágio ou ENADE). Guardado num formato interno único e exportado para o modelo.
- **Etapa:** roteiro simplificado → plano → slides → roteiro padronizado. Cada uma só começa com a anterior aprovada pelo autor.
- **Contexto da turma:** descrição **agregada** da turma e da escola. Nunca de um aluno.
- **Duração nominal:** o tempo oficial da aula (ex.: 50 min ou 1 hora-aula).
- **Tempo efetivo:** o tempo para o qual o plano é feito, descontando atraso, troca de sala, ligar equipamento (ex.: 40 ou 45 min numa aula de 50).
- **Hora-aula:** unidade da escola; padrão de 50 minutos, ajustável pelo autor.
- **Habilidade:** item do complemento da BNCC Computação, citado com código e texto literal.
- **Fonte verificada:** fonte de domínio aceito, com link que responde, aprovada pelo autor.
- **Sugestão da IA:** trecho gerado pela IA que ainda não foi aceito, editado ou recusado pelo autor.
- **Uso indevido:** tentativa de usar a IA fora do propósito ou contra as regras (seção 3.9).
- **Anonimizar:** tirar do dado qualquer possibilidade razoável de ligá-lo a uma pessoa (LGPD, art. 5º, XI). Dado anonimizado deixa de ser dado pessoal (art. 12).

## 3. Requisitos funcionais

### 3.1 Conta e acesso
| ID | Requisito | Critério de aceite | Dados pessoais? | Prioridade | Fase | Verificação |
|---|---|---|---|---|---|---|
| RF-01 | O sistema deve permitir cadastro com nome, e-mail e senha, com aceite dos termos. Na v1, só por convite (RF-78) | Sem convite válido, o cadastro é recusado; abrir o cadastro para todos é uma configuração, sem mudar código. Senha com 8 a 128 caracteres, sem regras de composição e fora da lista de senhas proibidas (decisão 0003); aceite guardado com data e versão dos termos | Sim | Deve | v1 | U, E2E |
| RF-02 | O sistema deve confirmar o e-mail por link que expira em 24 h | Conta só usa a IA depois de confirmada; link vencido não confirma e pode ser reenviado. Conta criada por convite já nasce confirmada, porque o link chegou àquele e-mail | Sim | Deve | v1 | I, E2E |
| RF-03 | O sistema deve permitir entrar com e-mail e senha | Mesma mensagem para qualquer falha ("E-mail ou senha incorretos"), sem dizer se o e-mail existe; no máximo 3 tentativas a cada 10 s por IP (decisão 0006); limite por conta depois da v1 | Sim | Deve | v1 | U, I, E2E |
| RF-04 | O sistema deve manter a sessão (ver RNF-07) e encerrá-la após 30 min sem uso; o usuário pode sair | Depois de 30 min parado, a próxima ação pede login; ao sair, a sessão é apagada no servidor | Sim | Deve | v1 | I |
| RF-05 | O usuário deve poder recuperar a senha por e-mail automático, com link de uso único que expira em 1 h | Link usado ou vencido não funciona; a mensagem não revela se o e-mail existe; trocar a senha encerra as outras sessões | Sim | Deve | v1 | I, E2E |
| RF-06 | O usuário deve poder ver e corrigir nome e e-mail | Troca de e-mail exige confirmação no novo endereço; alteração registrada no log | Sim | Deve | v1 | E2E |
| RF-07 | O usuário deve poder excluir a conta e os dados (ver seção 7.1) | Confirmação em 3 passos: senha, digitar o nome da conta e clicar no link enviado por e-mail. A conta fica desativada por 30 dias; entrar nesse prazo cancela a exclusão; depois, os dados seguem a seção 7.1 | Sim | Deve | v1 | I, E2E |
| RF-08 | O usuário deveria poder baixar todos os seus dados num arquivo (portabilidade) | Arquivo JSON com conta, contextos e planos | Sim | Deveria | depois | I |
| RF-09 | O usuário deveria poder ativar autenticação de dois fatores (código de aplicativo autenticador) | Com 2FA ligado, login pede o código; códigos de recuperação gerados uma vez | Sim | Deveria | depois | I, E2E |
| RF-10 | O usuário deveria poder entrar com Google e outros provedores, e ligar esses acessos à mesma conta | Mesma conta, seja por senha ou por provedor; ligação só com e-mail confirmado | Sim | Deveria | depois | I, E2E |

### 3.2 Contexto da turma e tempo
| ID | Requisito | Critério de aceite | Dados pessoais? | Prioridade | Fase | Verificação |
|---|---|---|---|---|---|---|
| RF-11 | O autor deve poder descrever a turma de forma agregada: etapa e ano, matéria, conhecimento prévio, dificuldades comuns, contexto social, recursos da escola, acessibilidade em termos gerais, carga da semana (ex.: semana de provas) e a opinião do professor | Só etapa e duração são obrigatórias; cada campo tem ajuda curta dizendo "sem nomes, diagnósticos ou laudos" | Não (se seguido o aviso) | Deve | v1 | E2E |
| RF-12 | A duração da aula deve ser obrigatória e informada em horas e minutos ou em horas-aula | 1 hora-aula = 50 min por padrão, ajustável; o sistema mostra a conversão | Não | Deve | v1 | U, E2E |
| RF-13 | O sistema deve oferecer planejar para um tempo efetivo menor que a duração nominal | Sugere um desconto (ex.: 50 → 40 ou 45 min) e explica o porquê; o autor escolhe o valor ou recusa | Não | Deve | v1 | U, E2E |
| RF-14 | O sistema deve alertar quando o plano tiver atividades que tomam muito tempo (instalar programa, criar conta em plataforma, configurar equipamento) e o tempo efetivo não comportar outros assuntos | Alerta sugere que a aula foque só nessa atividade; o autor decide | Não | Deve | v1 | U, Eval |
| RF-15 | O sistema deve alertar quando um texto parecer ter nome de aluno, diagnóstico ou laudo, antes de salvar ou enviar à IA | Frases do conjunto de teste com termos como "laudo", "CID", "diagnosticado" geram alerta, também em inglês, espanhol, língua misturada e texto disfarçado (decisão 0005, D8); o autor corrige ou confirma que não identifica ninguém | Sim (é a proteção) | Deve | v1 | U, E2E |
| RF-16 | O autor deveria poder reaproveitar o contexto de uma turma em vários planos | Contexto salvo aparece na criação do próximo plano | Não | Deveria | v1 | E2E |

### 3.3 Plano de aula
| ID | Requisito | Critério de aceite | Dados pessoais? | Prioridade | Fase | Verificação |
|---|---|---|---|---|---|---|
| RF-20 | O autor deve poder criar o plano de uma aula a partir de tema e contexto, no modelo do roteiro do estágio | Todos os campos do modelo preenchidos como sugestão da IA, dentro do tempo efetivo | Não | Deve | v1 | E2E, Eval |
| RF-21 | O autor deve poder criar o plano a partir de material que já tem, colado como texto (mini roteiro, texto dos slides) | O plano aproveita o material e marca o que a IA acrescentou | Não | Deve | v1 | E2E, Eval |
| RF-22 | O autor deveria poder enviar arquivos (slides, PDF, DOCX) para criar o plano | Aviso sobre dados de alunos antes do envio; arquivo apagado depois de extrair o texto; só o texto vai à IA | Possível | Deveria | depois | I, E2E |
| RF-23 | Todo trecho gerado pela IA deve ficar marcado como sugestão até o autor aceitar, editar ou recusar | Plano com sugestão pendente não pode ser aprovado | Não | Deve | v1 | U, E2E |
| RF-24 | O autor deve poder editar qualquer campo do plano | Edição salva; o trecho deixa de ser "sugestão" | Não | Deve | v1 | E2E |
| RF-25 | O plano deve seguir os estados rascunho → em revisão → aprovado → exportado. Só o autor aprova | A IA nunca muda o estado para aprovado | Não | Deve | v1 | U, I |
| RF-26 | O autor deveria poder pedir uma avaliação do plano: checagens automáticas (seção 4) + avaliação da IA pela rubrica | Resultado em lista de pontos, cada um com sugestão que o autor aceita ou recusa | Não | Deveria | v1 | I, Eval |
| RF-27 | O plano deve ter um campo de ligação com outras matérias, descrita em texto | A ligação não usa habilidades de outros componentes da BNCC | Não | Deve | v1 | U |
| RF-28 | O autor deveria poder adaptar um plano pronto a um novo contexto (ex.: semana de provas, imprevisto) | Gera nova versão; a anterior fica guardada | Não | Deveria | depois | E2E, Eval |
| RF-29 | O sistema deveria gerar as outras etapas (roteiro simplificado, slides, roteiro padronizado, guia de estudo), cada uma depois da aprovação da anterior | Etapa seguinte bloqueada até a aprovação | Não | Deveria | depois | I, Eval |
| RF-30 | O sistema deveria criar planos de semana, mês, semestre e ano | Sequência de aulas coerente com a carga horária | Não | Deveria | depois | Eval |
| RF-31 | O sistema deveria exportar para os modelos ENADE (EF/EM e EI) | Campos oficiais preenchidos | Não | Deveria | depois | I |

### 3.4 Habilidades (BNCC Computação)
| ID | Requisito | Critério de aceite | Dados pessoais? | Prioridade | Fase | Verificação |
|---|---|---|---|---|---|---|
| RF-40 | O sistema deve ter o catálogo da BNCC Computação importado uma vez, conferido e versionado | 140 códigos; texto de cada um igual ao do documento oficial | Não | Deve | v1 | U, Rev |
| RF-41 | O plano só deve aceitar habilidades do catálogo, no formato `(CÓDIGO) Texto literal. (Competência específica N)` | Código inexistente ou texto alterado é bloqueado com mensagem clara | Não | Deve | v1 | U, E2E |
| RF-42 | A IA deve sugerir habilidades consultando o catálogo (explicação e exemplos oficiais), não a memória do modelo | Toda habilidade sugerida existe no catálogo | Não | Deve | v1 | I, Eval |
| RF-43 | O sistema deve alertar quando a habilidade não combina com a etapa da turma | Ex.: código EM13 numa turma do fundamental gera alerta | Não | Deve | v1 | U |

### 3.5 Fontes
| ID | Requisito | Critério de aceite | Dados pessoais? | Prioridade | Fase | Verificação |
|---|---|---|---|---|---|---|
| RF-50 | Fonte sugerida pela IA só pode vir de domínio da lista de aceitos (governo, instituições, enciclopédias, museus, editoras, periódicos) | Domínio fora da lista é descartado antes de aparecer | Não | Deve | v1 | U, I |
| RF-51 | Toda fonte entra como "a revisar" e só vai para o plano depois que o autor aprova | Plano com fonte "a revisar" não pode ser aprovado | Não | Deve | v1 | U, E2E |
| RF-52 | O sistema deve conferir que o link da fonte responde antes de mostrá-la | Link quebrado é descartado ou marcado | Não | Deve | v1 | I |
| RF-53 | Fonte trazida pelo autor passa pelas mesmas checagens | Mesmo resultado de RF-50 a RF-52; o autor pode manter com justificativa | Não | Deve | v1 | E2E |
| RF-54 | As referências deveriam sair em ABNT, com "Acesso em" | Formato conferido nos testes | Não | Deveria | v1 | U |
| RF-55 | O sistema deveria conferir que o fato citado está escrito na fonte e mostrar o trecho | Trecho mostrado ao autor; divergência registrada | Não | Deveria | depois | Eval |

### 3.6 Exportação
| ID | Requisito | Critério de aceite | Dados pessoais? | Prioridade | Fase | Verificação |
|---|---|---|---|---|---|---|
| RF-60 | O autor deve poder exportar o plano aprovado em Markdown na v1; PDF se der tempo | Arquivo abre sem internet e segue as seções do roteiro do estágio | Sim (nome do autor, se preenchido) | Deve | v1 | I, E2E |
| RF-61 | O documento exportado deveria informar que foi elaborado com apoio de IA | Nota curta no rodapé; o autor pode desligar | Não | Deveria | v1 | I |

### 3.7 IA, limites e custo
| ID | Requisito | Critério de aceite | Dados pessoais? | Prioridade | Fase | Verificação |
|---|---|---|---|---|---|---|
| RF-70 | Só usuário autenticado e com e-mail confirmado deve conseguir chamar a IA | Chamada sem sessão válida é recusada | Não | Deve | v1 | I |
| RF-71 | Cada usuário deve ter um crédito mensal de IA e um limite de uso por dia e por minuto | Na v1, US$ 1 por mês (configurável), renovado no dia 1 no horário de Brasília, sem acumular; o usuário vê o saldo do mês; o custo é reservado no pedido (decisão 0008). Sem saldo ou acima do limite, a IA recusa com mensagem clara e o resto funciona | Sim (uso por pessoa) | Deve | v1 | U, I |
| RF-72 | O sistema deve ter um teto de gasto mensal com IA | Ao atingir o teto, a IA desliga para todos e o resto funciona; o administrador é avisado ao chegar a 80%. Na v1, US$ 10 (10 pessoas × US$ 1) | Não | Deve | v1 | U, I |
| RF-73 | Cada chamada deve ter tamanho máximo de entrada e de saída | Entrada grande demais é recusada antes de chegar ao provedor | Não | Deve | v1 | U |
| RF-74 | O sistema deve registrar o uso da IA por usuário e por operação (tokens e custo estimado), sem o conteúdo | Registro existe para cada chamada; base do estudo de custo | Sim | Deve | v1 | I |
| RF-75 | O administrador deve poder ligar e desligar a IA sem novo deploy | Mudança vale na próxima chamada | Não | Deve | v1 | I |
| RF-76 | O usuário deveria poder usar a própria chave de API | Chave guardada criptografada, nunca em log, com opção de apagar; só provedores testados | Sim (segredo) | Deveria | depois | I, Eval |
| RF-77 | O sistema deveria permitir modelos gratuitos | Só provedores cujos termos não usem os dados para treino | Não | Deveria | depois | Rev |
| RF-78 | O administrador deve poder convidar pessoas pelo e-mail; na v1, o cadastro só existe por convite | Link de uso único que vence em 7 dias; a conta só é criada com o e-mail convidado; o administrador pode cancelar o convite; no máximo 10 contas ativas (configurável); o token do link é guardado só em hash; só recebe convite quem pediu | Sim (e-mail do convidado) | Deve | v1 | I, E2E |

### 3.8 Revisores, avaliação e consentimento
| ID | Requisito | Critério de aceite | Dados pessoais? | Prioridade | Fase | Verificação |
|---|---|---|---|---|---|---|
| RF-80 | O licenciando deveria poder convidar revisores (orientador da faculdade, supervisor da escola) para um plano | Revisor só vê o que foi compartilhado | Sim | Deveria | depois | I, E2E |
| RF-81 | O revisor deveria poder avaliar o plano com comentários e nota por critério | Avaliação separada por papel | Sim | Deveria | depois | E2E |
| RF-82 | O licenciando deveria poder retirar o acesso de um revisor | Acesso some na hora | Sim | Deveria | depois | I |
| RF-83 | Roteiro e slides só chegam ao revisor se o licenciando enviar | Envio é sempre opcional | Não | Deveria | depois | E2E |
| RF-84 | O professor deveria poder autoavaliar o plano depois da aula | Autoavaliação ligada à versão do plano | Não | Deveria | depois | E2E |
| RF-85 | O revisor deveria poder montar a própria rubrica de avaliação | Métricas definidas pelo revisor | Não | Deveria | depois | E2E |
| RF-86 | Planos e avaliações só podem servir para melhorar o sistema com consentimento separado, opcional e em vigor | Antes da primeira avaliação, uma tela explica o uso e pede o aceite; botão de fácil acesso nas configurações liga e desliga com um clique; ao desligar, nada novo entra e o que não foi anonimizado sai do conjunto | Sim | Deve (quando houver avaliação) | depois | I, E2E, Rev |

### 3.9 Uso indevido da IA
| ID | Requisito | Critério de aceite | Dados pessoais? | Prioridade | Fase | Verificação |
|---|---|---|---|---|---|---|
| RF-90 | O sistema deve checar cada pedido à IA (e o texto colado) antes de enviar, e cada resposta antes de mostrar | Categorias da seção 4.1; conjunto de teste com exemplos de cada categoria, também em inglês, espanhol, língua misturada e texto disfarçado; resposta que falha na checagem não aparece (decisão 0005, D8) | Possível | Deve | v1 | U, Eval |
| RF-91 | Uso indevido **grave** deve bloquear na hora o uso da IA pelo usuário e notificar usuário e administrador | Usuário vê o motivo e como pedir revisão; o resto da conta (ver, exportar, excluir dados) continua acessível | Sim | Deve | v1 | I, E2E |
| RF-92 | Uso indevido **leve** deve gerar aviso dizendo que a próxima ocorrência bloqueia a IA até um administrador revisar | Segunda ocorrência leve em 30 dias bloqueia; administrador é notificado | Sim | Deve | v1 | U, E2E |
| RF-93 | O administrador deve poder ver cada ocorrência, desbloquear ou manter o bloqueio, com justificativa | Decisão registrada no log de auditoria | Sim | Deve | v1 | E2E |
| RF-94 | O usuário deve poder pedir revisão de um bloqueio e saber o critério usado (LGPD, art. 20) | Pedido chega ao administrador; resposta registrada | Sim | Deve | v1 | E2E |
| RF-95 | Pedido apenas fora do propósito (ex.: assunto sem relação com aula) deve ser recusado com explicação, sem contar como uso indevido | Recusa educada; nenhuma ocorrência registrada | Não | Deve | v1 | U, Eval |

## 4. Regras do domínio (condição → ação)
| Condição | Ação | Fase |
|---|---|---|
| Soma dos tempos das atividades ≠ tempo efetivo escolhido | Bloquear aprovação e mostrar a diferença | v1 |
| Duração informada em horas-aula | Converter (50 min por hora-aula, ou o valor do autor) e mostrar | v1 |
| Duração nominal informada | Sugerir tempo efetivo menor (ex.: 50 → 40 ou 45 min); o autor decide | v1 |
| Plano com instalação, criação de conta ou configuração e pouco tempo efetivo | Alertar que a aula deveria focar só nisso | v1 |
| Semana marcada como "de provas" | IA sugere atividades mais leves; o autor decide | v1 |
| Habilidade fora do catálogo ou com texto alterado | Bloquear (RF-41) | v1 |
| Fonte "a revisar" ou sugestão da IA pendente | Bloquear aprovação (RF-23, RF-51) | v1 |
| Texto com indício de nome de aluno, diagnóstico ou laudo | Alertar antes de salvar ou enviar (RF-15) | v1 |
| Modelo ENADE, anos finais ou ensino médio, mais de 3 horas-aula | Bloquear | depois |
| Modelo ENADE, anos iniciais ou educação infantil, mais de 5 horas-aula | Bloquear | depois |
| Envio ao supervisor com menos de 10 dias corridos até a regência | Avisar | depois |

### 4.1 Gravidade do uso indevido (proposta para revisão)
| Gravidade | Exemplos | Ação |
|---|---|---|
| Grave | Pedir conteúdo ilegal ou sexual envolvendo menores; discurso de ódio; tentar extrair chaves ou dados de outros usuários | Bloqueio imediato da IA + aviso ao usuário e ao administrador (RF-91) |
| Leve | Tentar fazer a IA ignorar as regras (ex.: "invente uma habilidade"); tentar obter as instruções internas, que não guardam segredo (decisão 0005, LLM07); conteúdo ofensivo sem alvo; insistir em burlar o limite de uso | Aviso; a próxima em 30 dias bloqueia até revisão (RF-92) |
| Não é uso indevido | Pedido fora do assunto; erro de digitação; texto com dado de aluno | Recusa explicada (RF-95) ou alerta de privacidade (RF-15) |

A checagem automática erra às vezes (falso positivo). Por isso existe revisão do administrador e pedido de revisão pelo usuário.

## 5. Requisitos não funcionais

### 5.1 Produto
| ID | Requisito | Critério de aceite |
|---|---|---|
| RNF-01 | Usabilidade: um professor deve criar uma aula adaptada à turma em menos de 15 minutos | Teste com pelo menos 1 professor ou licenciando |
| RNF-02 | O sistema deve funcionar no celular | Telas principais usáveis numa tela de 360 px |
| RNF-03 | Acessibilidade: uma pessoa cega deve conseguir usar tudo com leitor de tela, sozinha | Segue WCAG 2.2 nível AA; campos com rótulo e descrição claros; ordem de navegação lógica; erros anunciados; teste real com leitor de tela. Detalhes no tópico de front-end |
| RNF-04 | A geração deveria mostrar progresso e terminar em até 60 s | Medido nas chamadas reais |
| RNF-05 | Falha da IA não pode perder o trabalho do autor | Rascunho salvo antes de cada chamada |
| RNF-06 | Segurança da aplicação segue o OWASP ASVS 5.0 como checklist | Itens do nível 1 conferidos antes de publicar; HTTPS em tudo; senha só como hash forte; segredos fora do código |
| RNF-07 | Sessão guardada no servidor, identificada por cookie protegido (HttpOnly, SameSite; Secure em HTTPS); identificador novo a cada login; revogável (sair, trocar senha, sair de todos os aparelhos, suspensão, exclusão) | Teste: sessão revogada é recusada no pedido seguinte. Decisão 0003 |
| RNF-08 | Segurança da IA segue o OWASP Top 10 para aplicações com LLM (2025) | Texto do usuário, material colado e páginas de fontes são tratados como **dados**, nunca como instruções (injeção indireta); limites de consumo (RF-71 a RF-73) |
| RNF-09 | Qualidade: TDD em todo o sistema, com testes unitários, de integração e e2e rodando a cada mudança (CI) | Toda regra da seção 4 tem teste unitário com casos de limite; todo cenário BDD da v1 tem teste e2e |
| RNF-10 | Qualidade da IA: evals com rubrica e conjunto de exemplos | Nenhuma habilidade ou fonte inventada no conjunto de avaliação |

### 5.2 Organizacionais
| ID | Requisito | Critério de aceite |
|---|---|---|
| RNF-11 | Node.js com TypeScript; empacotado em Docker | Sobe localmente com um comando |
| RNF-12 | Deploy na AWS com custo baixo | Meta mensal definida no tópico de infra (AWS) |
| RNF-13 | Na v1 a IA usa a chave do projeto, com limites e teto (RF-71 a RF-73) | Valores definidos e configuráveis |
| RNF-14 | Os dados de uso (RF-74) devem permitir calcular o custo por plano | Relatório de custo por operação; base do estudo futuro de assinatura |

### 5.3 Externos (legais e éticos)
| ID | Requisito | Critério de aceite |
|---|---|---|
| RNF-20 | Cumprir a LGPD (Lei 13.709/2018) | Seção 7 preenchida e revisada |
| RNF-21 | Transparência: o usuário sabe o que veio da IA, qual provedor processa os dados e para quê | Aviso nos termos e na tela de geração |
| RNF-22 | O provedor de IA não pode usar os dados enviados para treino | Termos do provedor conferidos e registrados numa decisão |
| RNF-23 | Respeitar direitos autorais das fontes: citar, sem copiar trechos longos | Revisão nos evals |
| RNF-24 | Backups guardados por tempo limitado; dado excluído some dos backups quando eles expiram | Prazo definido no tópico de infra (proposta: 7 dias) |

### 5.4 Desacoplamento, reuso e escala
| ID | Requisito | Critério de aceite |
|---|---|---|
| RNF-30 | Todo serviço externo (IA, e-mail, login social, armazenamento de arquivos, banco, pagamento futuro) fica atrás de uma interface própria, com adaptadores trocáveis | Cada interface tem pelo menos 2 implementações (uma real e uma falsa para testes); trocar o provedor é mudar configuração e adaptador, sem mexer nas regras do sistema |
| RNF-31 | Autenticação, limites e custo de IA, checagem de uso indevido, auditoria e exportação são módulos independentes, reutilizáveis em outros projetos | Nenhum desses módulos depende do módulo de planos; cada um tem testes próprios e documentação de uso |
| RNF-32 | Configuração por ambiente (variáveis), sem segredo nem endereço fixo no código | Mesmo código roda em local, teste e produção só trocando configuração |
| RNF-33 | O servidor não guarda estado em memória entre pedidos | Duas cópias rodando ao mesmo tempo atendem o mesmo usuário sem erro |
| RNF-34 | Tarefas longas (geração por IA, exportação) podem rodar em fila, separadas do atendimento das telas | A tela não trava durante a geração; uma falha na fila não derruba o site |
| RNF-35 | A resposta da IA vem em formato estruturado (JSON com esquema) e é validada antes de usar | Resposta inválida é repetida no máximo 2 vezes e depois vira erro claro |
| RNF-36 | Chamadas a serviços externos (IA, e-mail) repetem com espera crescente e aleatória, e param por um tempo quando o serviço está fora (circuit breaker) | Teste com provedor falso fora do ar: nenhuma tempestade de chamadas |
| RNF-37 | Pedido igual repetido em sequência (ex.: clique duplo) não gera nem cobra de novo | Teste de idempotência |
| RNF-38 | Cada tarefa de IA usa o modelo adequado: barato para checagens e classificação, melhor para geração | Modelo por tarefa definido em configuração |
| RNF-39 | Observabilidade: custo, tempo de resposta e qualidade por modelo e por operação; logs sem senha, token nem conteúdo sensível | Painel ou relatório simples; teste que procura segredos nos logs |
| RNF-40 | IDs não sequenciais nas URLs e checagem de dono em todo recurso | Teste: usuário A não acessa recurso de B nem trocando o ID |
| RNF-41 | Dependências monitoradas (alertas de vulnerabilidade) e análise estática de segurança no CI | CI falha com vulnerabilidade grave conhecida |
| RNF-42 | Valores de dinheiro e custo guardados em inteiro (ex.: milésimos de centavo), nunca em ponto flutuante | Teste de soma sem erro de arredondamento |

## 6. Estórias e cenários (v1)
Cada cenário vira um teste de aceitação (e2e).

**F1. Entrar no sistema**
Como autor, quero entrar com e-mail e senha, para acessar meus planos com segurança.
- *Cenário 1:* Dado que tenho conta confirmada, quando informo e-mail e senha corretos, então entro na lista dos meus planos.
- *Cenário 2:* Dado que já tentei entrar 3 vezes em 10 segundos, quando tento de novo, então o sistema pede para esperar, e a mensagem de erro não diz se o e-mail existe.
- *Cenário 3:* Dado que esqueci a senha, quando peço a recuperação, então recebo um link por e-mail que vale por 1 hora e só uma vez.

**F2. Descrever a turma e o tempo**
Como autor, quero descrever minha turma e o tempo da aula, para que o plano caiba na realidade.
- *Cenário 1:* Dado que informei 1 hora-aula, quando sigo para o plano, então o sistema mostra 50 minutos e sugere planejar para 40 ou 45, e eu escolho.
- *Cenário 2:* Dado que escrevi "aluno com laudo de TDAH", quando tento salvar, então vejo um alerta pedindo para descrever a turma sem diagnóstico, e nada é enviado à IA.

**F3. Criar o plano**
Como licenciando, quero gerar um plano a partir do tema e do contexto, para começar de algo pronto e revisar.
- *Cenário 1:* Dado que escolhi tema e contexto, quando peço o plano, então recebo todos os campos do roteiro do estágio marcados como sugestão da IA, somando o tempo efetivo.
- *Cenário 2:* Dado que colei meu mini roteiro, quando peço o plano, então o plano aproveita o meu texto e marca só o que a IA acrescentou.
- *Cenário 3:* Dado que o plano inclui instalar um programa e tenho 40 minutos efetivos, quando o plano é gerado, então vejo um alerta sugerindo dedicar a aula só à instalação.

**F4. Habilidades**
Como autor, quero que só entrem habilidades reais da BNCC Computação, para não errar no documento oficial.
- *Cenário 1:* Dado que a IA sugeriu habilidades, quando vejo a lista, então todas existem no catálogo, com texto literal e número da competência.
- *Cenário 2:* Dado que digitei um código que não existe, quando salvo, então o código é recusado com mensagem clara.

**F5. Fontes**
Como autor, quero revisar as fontes antes de usá-las, para não citar algo falso.
- *Cenário 1:* Dado que a IA sugeriu uma fonte de domínio aceito com link funcionando, quando abro o plano, então ela aparece como "a revisar" até eu aprovar.
- *Cenário 2:* Dado que a IA sugeriu um blog, quando o plano é gerado, então essa fonte não aparece.

**F6. Exportar**
Como autor, quero exportar o plano aprovado, para entregar ou levar para a aula.
- *Cenário 1:* Dado que o plano está aprovado, quando exporto, então recebo o documento no modelo do roteiro do estágio.
- *Cenário 2:* Dado que o plano ainda é rascunho, quando tento exportar, então a exportação final é bloqueada.

**F7. Limites e uso indevido da IA**
Como administrador, quero limitar e proteger o uso da IA, para que o gasto e o abuso não fujam do controle.
- *Cenário 1:* Dado que o teto mensal foi atingido, quando qualquer usuário pede uma geração, então a IA está desligada e o resto do sistema funciona.
- *Cenário 2:* Dado que um usuário já recebeu um aviso leve, quando comete outro uso indevido leve, então a IA fica bloqueada para ele até eu revisar, e eu sou notificado.
- *Cenário 3:* Dado que um usuário pediu algo fora do assunto, quando a IA responde, então ele recebe uma recusa explicada e nenhuma ocorrência é registrada.

**F8. Excluir a conta**
Como usuário, quero excluir minha conta e meus dados, para exercer meu direito previsto na LGPD.
- *Cenário 1:* Dado que confirmei a senha, digitei o nome da conta e cliquei no link do e-mail, quando a exclusão termina, então não consigo mais entrar e meus planos não existem mais.
- *Cenário 2:* Dado que digitei o nome da conta errado, quando confirmo, então nada é excluído.

## 7. Privacidade (checklist LGPD, Privacy by Design)
**Inventário de dados (v1):**
- **Conta:** nome, e-mail, hash da senha, aceite dos termos (data e versão).
- **Convite:** e-mail do convidado e data, só até o aceite, o cancelamento ou o vencimento (RF-78).
- **Uso:** registros de acesso (data, hora, IP) e de uso da IA (sem conteúdo).
- **Uso indevido:** ocorrências com o trecho mínimo que motivou, para revisão.
- **Plano e contexto:** dados da turma de forma agregada. Identificação do plano (escola, autor, turma) é opcional e preenchida pelo autor.
- **Não coletamos:** nenhum dado de aluno, dado sensível ou dado de criança.

### 7.1 Exclusão: o que some, o que é anonimizado e o que fica
Base: LGPD, arts. 15, 16 e 18, VI (eliminação, salvo as exceções do art. 16); art. 12 (anonimizado não é dado pessoal); Marco Civil da Internet, art. 15.

| Dado | Na exclusão | Por quê |
|---|---|---|
| Nome, e-mail, senha, contextos, planos, sessões, chave de API | **Apagado** | Fim do tratamento (art. 15, III, e art. 16) |
| Comentários de revisores sobre os planos excluídos | **Apagado** | Perdem a finalidade junto com o plano |
| Registros de uso da IA (tokens, custo) | **Anonimizado**: a ligação com o usuário é cortada; fica só o número agregado | Uso exclusivo do controlador, sem acesso de terceiros, com dados anonimizados (art. 16, IV) |
| Planos e avaliações já incluídos no conjunto de avaliação, com consentimento | **Anonimizados** antes de entrar no conjunto; o que não estiver anonimizado é apagado | Art. 12 e art. 16, IV; revisão manual do texto livre, porque tirar o nome não basta |
| Registros de acesso (data, hora, IP) | **Guardados por 6 meses**, sob sigilo, e depois apagados | Obrigação legal (art. 16, I) para pessoa jurídica com fins econômicos (Marco Civil, art. 15). Seguida desde a v1, para o sistema estar pronto para virar produto (ponto 4 da seção 8) |
| Registro mínimo do pedido de exclusão (data e resultado, sem nome nem e-mail) | **Guardado** | Comprovar que o pedido foi atendido |
| Backups | Expiram no prazo de RNF-24 | O dado some quando o backup vence |

**Prazo de arrependimento:** a exclusão não é imediata. A conta fica desativada por 30 dias (o mesmo prazo do X e do Instagram); entrar nesse período cancela o pedido. Depois, vale a tabela acima.

**Contas inativas:** depois de 6 meses sem uso, o sistema avisa por e-mail e a conta entra no mesmo fluxo: 30 dias desativada e, depois, apagada ou anonimizada conforme a tabela. Desativar (soft delete) só esconde os dados; para a LGPD, o que conta é apagar ou anonimizar no fim do prazo.

**Sobre as confirmações:** a lei pede que a revogação do consentimento seja por "procedimento gratuito e facilitado" (art. 8º, §5º) e que os direitos sejam atendidos sem custo (art. 18, §5º). Por isso:
- **Excluir a conta:** os 3 passos (senha, nome da conta e link por e-mail) servem para **segurança**, contra exclusão por engano ou por quem invadiu a conta. Não podem virar um obstáculo para desanimar o usuário: o botão é fácil de achar e o processo é curto.
- **Retirar o consentimento de melhoria (RF-86):** um clique, sem confirmação extra.

### 7.2 Itens do checklist
| Item | Aplica-se? | Como atende | Evidência |
|---|---|---|---|
| REQ-01 Dados comuns e finalidade | Sim | Nome e e-mail para a conta; e-mail do convidado só para enviar o convite; identificação do plano opcional, para o documento exportado | Inventário; RF-01, RF-60, RF-78 |
| REQ-02 Dados sensíveis | Sim, como risco | Não coletamos. O risco é o autor escrever diagnóstico no texto livre: alerta antes de salvar ou enviar | RF-15; seção 4 |
| REQ-03 Finalidade de cada uso | Sim | Conta: acesso. Contexto: adaptar o plano. Uso da IA: limite, teto e custo. Ocorrências: segurança | RF-11, RF-74, RF-90 |
| REQ-04 Necessidade e opcionais | Sim | Só etapa e duração são obrigatórias no contexto; identificação do plano é opcional | RF-11, RF-12 |
| REQ-05 Minimização | Sim | Turma agregada; registro de uso sem conteúdo; ocorrência guarda só o trecho mínimo; arquivo apagado após extração | RF-11, RF-22, RF-74, RF-90 |
| REQ-07 Crianças e adolescentes | Sim, indiretamente | Os planos são para turmas de crianças e adolescentes, mas nenhum dado deles é coletado | RF-11, RF-15 |
| REQ-08 Proteções extras | Sim | Alerta de nome, diagnóstico e laudo; ajuda nos campos; nada vai à IA com alerta não resolvido | RF-15; F2 cenário 2 |
| REQ-09 Funcionalidades com IA | Sim | Gerar plano (RF-20, RF-21), sugerir habilidades (RF-42) e fontes (RF-50), avaliar (RF-26), checar uso indevido (RF-90). Entrada: tema, contexto agregado, material colado. Saída: sugestões | Seção 3 |
| REQ-10 Intervenção humana | Sim | Tudo da IA é sugestão; só o autor aprova; bloqueio por uso indevido revisado por administrador | RF-23, RF-25, RF-93 |
| REQ-11 Informação sobre a IA | Sim | Aviso nos termos e na tela de geração; marcação de sugestão; nota no documento exportado | RNF-21, RF-61 |
| REQ-12 Acesso, correção, exclusão | Sim | Ver e corrigir dados; excluir a conta (seção 7.1) | RF-06, RF-07 |
| REQ-13 Decisões automáticas | Sim | Bloqueios de regra (seção 4), limite de uso e bloqueio por uso indevido. O usuário sabe o critério e pode pedir revisão (art. 20) | RF-91 a RF-94 |
| REQ-14 Perfil deduzido | Sim, limitado | O sistema não deduz nada sobre alunos. Sobre o usuário, só conta ocorrências de uso indevido, para a regra de reincidência | RF-92 |
| REQ-15 Autenticação e recuperação | Sim | Cadastro só por convite na v1 (link de uso único, 7 dias); confirmação de e-mail, senha forte, limite de tentativas por IP (decisão 0006), sessão de 30 min revogável no servidor, recuperação por e-mail de 1 h; 2FA e login social depois | RF-01 a RF-05, RF-09, RF-10, RF-78, RNF-07 |
| REQ-16 Quem vê e altera | Sim | Cada autor vê só os próprios planos. Revisor vê só o compartilhado (depois). Administrador vê uso, limites e ocorrências, não o conteúdo dos planos | RF-70, RF-80 a RF-82, RF-93 |
| REQ-17 Retenção | Sim | Dados enquanto a conta existir; registros de acesso 6 meses; ocorrências de uso indevido: proposta de 90 dias; contas inativas: aviso aos 6 meses sem uso, 30 dias desativadas e depois apagadas ou anonimizadas; convites: e-mail apagado no aceite (vira a conta), no cancelamento ou ao vencer (7 dias) | Seção 7.1; RF-07, RF-78 |
| REQ-18 Descarte | Sim | Seção 7.1, incluindo backups | RF-07, RNF-24 |
| REQ-19 Anonimização | Sim | Uso de IA: corte da ligação com o usuário. Conjunto de avaliação: remover autor, escola e turma e revisar o texto livre | Seção 7.1; RF-86 |
| REQ-20 Criptografia | Sim | HTTPS no trânsito; banco e backups criptografados em repouso; chave do usuário criptografada (depois) | RNF-06, RF-76 |
| REQ-21 Consentimento | Sim | Termos aceitos no cadastro (data e versão). Melhoria: consentimento separado, pedido antes da primeira avaliação, retirado com um clique | RF-01, RF-86 |
| REQ-22 Logs de auditoria | Sim | Login, falhas, limite de tentativas atingido, mudança de dados da conta, exclusão, mudança de estado do plano, ocorrências de uso indevido e decisões do administrador. Só o administrador consulta | RF-03, RF-07, RF-25, RF-93 |
| REQ-23 Portabilidade | Sim (depois) | Baixar todos os dados em JSON | RF-08 |
| REQ-23b Backup | Sim | Backups automáticos do banco, criptografados, só o administrador acessa, expiram no prazo de RNF-24 | RNF-24 |

## 8. Pontos em aberto
1. ~~Data da v1.~~ **Fechado (14/11):** primeira versão rodando no local; AWS depois.
2. Valores: limite por dia e por minuto, teto mensal, prazo de reincidência (proposta: 30 dias). **Valores iniciais (15/11, decisão 0005):** teto de US$ 10 por mês; por usuário, 5 gerações por dia e 2 por minuto; custo estimado de US$ 0,15 a 0,20 por plano. **Atualização (16/11, decisões 0005 e 0008):** até 10 pessoas convidadas, cada uma com US$ 1 de crédito por mês, renovado no dia 1 e sem acumular; o teto de US$ 10 continua para o sistema todo. **Valores finais depois do teste com os 3 planos de referência e de um teste de carga mínimo.**
3. ~~Provedor e modelo de IA.~~ **Fechado (15/11, decisão 0005):** API da Anthropic na v1, atrás da porta de IA (RNF-30); sem treino com os dados (RNF-22). Claude Sonnet 5.5 para gerar e avaliar; Claude Haiku 5.5 para checagens. Modelos, esforço e limites confirmados pelo teste com Haiku, Sonnet e Opus nos 3 planos de referência. Nível gratuito de provedor que treina com os dados só com dados fictícios. **E-mail fechado na decisão 0003:** Mailpit no local, Amazon SES em produção.
4. ~~Registros de acesso.~~ **Fechado (14/11):** guardados por 6 meses desde a v1 (seção 7.1).
5. ~~Prazo de arrependimento.~~ **Fechado (14/11):** conta desativada por 30 dias, cancelável; depois, seção 7.1 (RF-07).
6. ~~Formato de exportação.~~ **Fechado (14/11):** Markdown na v1; PDF se der tempo (RF-60).
7. ~~Contas inativas.~~ **Fechado (14/11):** aviso aos 6 meses sem uso e o mesmo fluxo do ponto 5 (seção 7.1).
