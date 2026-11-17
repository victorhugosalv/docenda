# Design: Docenda

> **v0.1 (17/11/2026). Planejado: ainda não há tela.** Diz como as telas se parecem e se comportam. A estrutura técnica está na decisão [0013](decisoes/0013-front-end-stack.md); o mapa de telas, no [INTERFACE](INTERFACE.md); os campos e os erros, no [CONTRATOS](CONTRATOS.md).
> A identidade vem da capa do projeto: quadro verde, giz creme e o dourado do ponto final do nome.

## 1. Princípios

1. **O texto do professor é o centro.** A tela é um caderno, não um painel. Pouca cor, muito espaço, letra grande.
2. **Fica claro o que veio da IA.** Sugestão tem moldura, rótulo e ações próprias, e nunca se confunde com o que o autor já decidiu. Cor ajuda, mas nunca é o único sinal (WCAG 1.4.1).
3. **Erro e aviso dizem o que fazer.** "A soma dá 45 de 40 minutos: tire 5 de alguma atividade", não "Tempo inválido".
4. **Nada some sem o autor saber.** Salvamento automático com estado visível; conflito, sessão vencida e falha da IA explicados na hora.
5. **Funciona no celular da sala dos professores.** O desenho começa em 360 px e cresce.
6. **Quem enxerga pouco ou nada usa sozinho** (RNF-03). Teclado, leitor de tela e zoom de 200% sem perder nada.

## 2. Cores

Dois temas: **papel** (claro, padrão) e **quadro** (escuro). Segue `prefers-color-scheme`; o autor pode fixar um deles em Minha conta, e a escolha fica no navegador (não é texto do plano, então não fere a 0013 D7).

Os pares abaixo foram calculados pela fórmula de contraste da WCAG em 17/11/2026. Texto normal pede 4,5:1; borda de campo, ícone e anel de foco pedem 3:1.

| Token | Papel | Quadro | Uso | Contraste (papel / quadro) |
|---|---|---|---|---|
| `--bg` | `#f6f2e9` | `#0b2420` | Fundo da página | |
| `--surface` | `#fffdf8` | `#10302a` | Cartões e campos | |
| `--text` | `#1b2925` | `#f3efe6` | Texto | 14,9 / 12,4 sobre `--surface` |
| `--text-muted` | `#4a5a54` | `#b4c6bd` | Ajuda e rótulos secundários | 7,2 / 8,0 sobre `--surface` |
| `--primary` | `#16423a` | `#d6b25a` | Botão principal, links | 10,0 / 8,1 sobre `--bg` |
| `--on-primary` | `#fffdf8` | `#0b2420` | Texto no botão principal | 11,0 / 8,1 |
| `--accent` | `#c9a44c` | `#c9a44c` | Só decoração: o ponto do nome, fios | Não usar em texto no tema papel |
| `--accent-text` | `#7a5c12` | `#e0c070` | Texto em dourado, quando precisar | 5,6 / 9,3 sobre `--bg` |
| `--suggestion-bg` | `#fbf1d6` | `#2b3a1f` | Fundo da sugestão da IA | Texto: 13,4 / 10,6 |
| `--suggestion-border` | `#a8862e` | `#c9a44c` | Moldura da sugestão | 3,1 / 5,2 sobre o fundo dela |
| `--block` | `#a3261b` | `#ff9b8f` | Achado que bloqueia; erro | 7,3 / 7,0 |
| `--warn` | `#7f5200` | `#f2c66b` | Aviso | 6,6 / 8,8 |
| `--info` | `#25507d` | `#9cc4f0` | Informação | 8,2 / 7,8 |
| `--ok` | `#1d6542` | `#8fd6ad` | Salvo, aprovado | 6,9 / 8,4 |
| `--focus` | `#1f4fd1` | `#8fb4ff` | Anel de foco | 6,1 / 7,9 sobre `--bg` |
| `--border` | `#76867f` | `#6f8a80` | Borda de campo | 3,8 / 3,8 sobre `--surface` |

Regras:
- O azul só aparece no foco. Assim o anel nunca se confunde com outro elemento.
- Bloqueio, aviso e informação sempre vêm com ícone e palavra ("Bloqueia a aprovação", "Aviso", "Para saber"), além da cor.

## 3. Letras

| Papel | Fonte | Onde |
|---|---|---|
| Interface e texto do plano | **Atkinson Hyperlegible Next** (400 e 700) | Tudo o que se lê e se escreve |
| Números e códigos | **Atkinson Hyperlegible Mono** (400) | Códigos de habilidade, minutos, saldo |
| Marca | **Playwrite BR** (200) | Só o nome "Docenda." no topo e na tela de entrar |

- **Por que Atkinson Hyperlegible:** foi desenhada pelo Braille Institute para leitura com baixa visão. Letras que costumam se confundir (`I`, `l`, `1`; `O`, `0`) têm formas bem distintas. Licença aberta (OFL); conferida no Google Fonts em 17/11/2026.
- **A cursiva e a letra de giz da capa ficam fora da interface.** Cursiva e maiúsculas em sequência são mais difíceis de ler, principalmente com dislexia e com baixa visão. A capa é para ser vista; a tela é para trabalhar.
- **Fontes servidas pela própria Docenda,** em `apps/web/public/fonts/`, em `woff2`. Buscar no Google Fonts violaria a política de segurança de conteúdo (`default-src 'self'`, 0006) e mandaria o IP do professor para fora.

| Token | Tamanho | Altura da linha | Uso |
|---|---|---|---|
| `--text-sm` | 0,875 rem (14 px) | 1,45 | Ajuda dos campos, metadados |
| `--text-md` | 1,0625 rem (17 px) | 1,55 | Texto e campos |
| `--text-lg` | 1,25 rem (20 px) | 1,4 | Título de seção do plano |
| `--text-xl` | 1,625 rem (26 px) | 1,25 | Título da tela |
| `--text-brand` | 2 rem (32 px) | 1 | Marca |

Tamanhos em `rem`, para respeitar a letra que a pessoa escolheu no navegador. Linhas de texto com no máximo 70 caracteres.

## 4. Espaço, forma e movimento

- **Espaço:** escala de 4 px (`--space-1` = 4 px até `--space-8` = 64 px).
- **Alvo de toque:** pelo menos 44 × 44 px em botões e controles. A WCAG 2.2 AA pede 24 px; 44 px cabe no polegar.
- **Cantos:** 6 px em campos e botões, 10 px em cartões. Sem sombra forte: borda de 1 px.
- **Foco:** contorno de 3 px em `--focus`, afastado 2 px do elemento, em **todo** controle. Nunca `outline: none` sem substituto.
- **Movimento:** só transições de até 150 ms em cor e opacidade. Com `prefers-reduced-motion`, nenhuma.
- **Textura de giz:** não entra na interface.

## 5. Componentes

| Componente | Comportamento | Acessibilidade |
|---|---|---|
| Botão | Principal (fundo `--primary`), secundário (borda) e discreto (só texto). Um principal por tela | `button` nativo. Enquanto envia, mostra "Salvando…" e continua focável; não fica desabilitado sem explicação |
| Campo de texto | Rótulo em cima, ajuda curta embaixo do rótulo, erro embaixo do campo | `label` ligado; ajuda e erro por `aria-describedby`; `aria-invalid` no erro |
| Campo longo | Cresce com o texto, sem barra de rolagem interna | Mesmo do campo de texto |
| Lista editável (objetivos, conteúdos, recursos) | Um campo por item; adicionar, remover e mover com botões | Botões com o nome do item ("Remover objetivo 2"); anúncio ao mover |
| Grupo de opções (etapa de ensino, momento) | Botões de rádio em `fieldset` | `legend` com a pergunta |
| Escolha de habilidade | Busca por código ou texto, filtrada pela etapa de ensino e pelo ano | Combobox do React Aria Components (0013 D4) |
| Sugestão da IA | Seção 6.5 | Seção 6.5 |
| Achado | Ícone, rótulo da gravidade, mensagem e link "Ir para o campo" | Lista com título; o link move o foco para o campo |
| Diálogo | Confirmações, alerta de privacidade e sessão vencida | `dialog` nativo; foco preso dentro; Esc fecha quando não há perda de dados |
| Aviso de página | Faixa no topo: IA desligada, teto atingido, conflito de revisão | `role="status"` ou `role="alert"`, conforme a urgência |
| Estado do salvamento | "Salvo às 14:32", "Salvando…", "Não foi possível salvar: tentar de novo" | Região viva educada, que não interrompe a leitura |

## 6. Telas

Em todas: cabeçalho com a marca, "Meus planos" e "Minha conta"; um título `h1` por tela, que recebe o foco ao trocar de tela; link "Pular para o conteúdo" como primeiro item.

### 6.1 Entrar (`/entrar`)
Marca, e-mail, senha (com botão "Mostrar senha") e "Entrar". Erro único, acima do formulário: "E-mail ou senha incorretos." No limite de tentativas: "Muitas tentativas. Espere alguns segundos e tente de novo." Link "Esqueci a senha" a partir da F10.

### 6.2 Meus planos (`/`)
- Botão principal "Novo plano".
- Lista, do mais recente para o mais antigo. Cada item: tema, etapa de ensino e ano, estado ("Rascunho", "Aprovado, versão 2"), data da última mudança e "Gerando…" quando houver geração ativa.
- Ações por item, num menu: abrir, duplicar, excluir. Excluir pede confirmação dizendo o que some: "O plano, as versões aprovadas e as sugestões serão apagados."
- Vazio: uma frase e o botão "Criar o primeiro plano".
- Desktop: tabela. Celular: lista de cartões, com o tema como link.

### 6.3 Novo plano (`/planos/novo`), em três passos
Indicador "Passo 1 de 3". Voltar não perde o que foi escrito.

1. **Turma.** Escolher um contexto salvo ou descrever uma turma nova. Só etapa de ensino e duração são obrigatórias. Acima dos campos de texto, um aviso fixo: "Descreva a turma em conjunto. Não escreva nomes, diagnósticos ou laudos." Cada campo tem uma ajuda com exemplo ("Ex.: parte da turma ainda confunde arquivo e pasta").
2. **Tempo.** Duração em minutos ou em horas-aula, com a conversão mostrada ("2 horas-aula = 100 minutos"). Logo abaixo: "Quanto tempo sobra de verdade? Chamada, troca de sala e ligar equipamento costumam tomar de 5 a 10 minutos." Opções sugeridas (90, 95) e "outro valor". Marcar semana de provas e anotar a carga da semana.
3. **Tema e material.** Tema, subtema e o material colado, opcional. Junto do material: "O texto colado é enviado a um provedor de IA fora do Brasil para gerar o plano."

O botão final é "Criar plano e gerar sugestões". O plano é criado em rascunho e a geração começa; o autor vai para o editor.

**Alerta de privacidade.** Se a gravação voltar `privacy_alert`, abre um diálogo: "Este trecho parece falar de um estudante específico: '…laudo…'. Descreva a turma em conjunto, sem nomes ou diagnósticos." Ações: "Corrigir o texto" (volta ao campo, com o foco nele) e "O texto não identifica ninguém" (confirma e grava). Nada vai para a IA antes disso.

### 6.4 Editor do plano (`/planos/:id`)
O coração da Docenda.

**Topo, fixo ao rolar:**
- Tema, estado ("Rascunho, versão 1") e o estado do salvamento.
- **Barra de pendências:** "3 sugestões para decidir · 1 bloqueio · 2 avisos", cada número sendo um link para o primeiro item daquele tipo.
- **Tempo:** "35 de 40 minutos", com um traço que enche. Passou do tempo, o traço fica em `--block` e o texto diz "45 de 40 minutos: 5 a mais".
- Ações: "Gerar sugestões" (ou "Gerar de novo"), "Aprovar" e, no aprovado, "Exportar" e "Reabrir para editar".

**Corpo:** os campos na ordem do roteiro do estágio. Cada campo é uma seção com título `h2`, o texto do autor e, quando houver, a sugestão pendente e os achados daquele campo.

- **Desktop (a partir de 1024 px):** coluna do plano à esquerda e, à direita, a lista de achados e o índice dos campos.
- **Celular:** uma coluna; os achados aparecem logo abaixo do campo a que se referem.

**Identificação:** seção recolhida no fim, "Identificação do documento (escola, turma, data)", com a nota "Estes dados aparecem só no arquivo exportado e nunca são enviados à IA."

**Atividades:** cada uma é um cartão com momento, título, descrição e minutos. O campo de minutos fica ao lado do título, em fonte mono. Atividade marcada como "Toma a aula toda (instalação ou configuração)" mostra o selo.

**Aprovar:** o botão fica sempre ativo. Se houver bloqueio, abre um diálogo listando os bloqueios, cada um com link para o campo. Sem bloqueio, confirma: "Aprovar a versão 1? Ela fica guardada e pode ser exportada. Para mudar depois, reabra o plano." Avisos não impedem a aprovação, mas aparecem no diálogo.

**Plano aprovado:** os campos aparecem como texto, sem edição. Um aviso no topo: "Versão 2 aprovada em 17/11. Para mudar, reabra: a versão 2 continua guardada."

### 6.5 Sugestão da IA, dentro do campo

```
┌ Sugestão da IA ─ acrescentado pela IA ──────────────┐
│ Texto proposto para o campo, inteiro.                │
│                                                      │
│ [Aceitar]   [Editar a partir desta]   [Recusar]      │
└──────────────────────────────────────────────────────┘
```

- Moldura em `--suggestion-border`, fundo `--suggestion-bg` e um rótulo escrito: "Sugestão da IA". O rótulo também diz a origem: "veio do seu material" ou "acrescentado pela IA" (RF-21).
- **Aceitar** copia o texto para o campo. **Editar a partir desta** abre o texto num campo, com "Salvar minha versão" e "Cancelar". **Recusar** some com a sugestão; o campo fica como estava.
- Em lista (objetivos, atividades), a sugestão mostra a lista inteira, porque a unidade é o campo (RF-23).
- Habilidade sugerida: código e texto oficial, mais o motivo em uma frase ("A aula trabalha a construção de algoritmos na atividade 2"). Ações: "Incluir" e "Recusar". Não há editar: o texto é literal.
- Depois da decisão, o foco vai para a próxima sugestão pendente, e um anúncio diz "Sugestão aceita. Faltam 2."
- Para o leitor de tela, cada sugestão é uma região com nome: "Sugestão da IA para Justificativa".

### 6.6 Andamento da geração
Painel no topo do editor enquanto a geração está ativa:

```
Gerando sugestões
✓ Conferindo os dados
● Escrevendo o plano            ← etapa atual
○ Validando
○ Conferindo a resposta
```

- Região viva educada: anuncia a troca de etapa, não cada segundo.
- O editor continua usável durante a geração. Os campos que vão receber sugestão continuam do autor.
- Ao terminar: "Pronto: 10 sugestões para decidir." e o foco vai para a barra de pendências.

**Mensagem de cada falha** (`failureReason`, DOMINIO 4.2):

| Motivo | Mensagem | Ação oferecida |
|---|---|---|
| `input_blocked_privacy` | "Encontramos o que parece ser dado de um estudante em Turma. Corrija ou confirme antes de gerar." | Ir para o campo |
| `input_blocked_misuse` | "Este pedido não pode ser atendido pela IA. Se acha que é um engano, peça uma revisão." | Ver o motivo; se a IA foi bloqueada, pedir revisão (F09) |
| `off_topic` | "O pedido não parece ser de uma aula. Revise o tema e o material." | Ir para o tema |
| `provider_refusal` | "O provedor de IA recusou este pedido. Isso acontece às vezes com temas de segurança; não conta como problema seu. Tente reformular o tema." | Gerar de novo |
| `invalid_output` | "A IA devolveu um plano incompleto. Nada foi cobrado além do que foi usado." | Gerar de novo |
| `output_check_failed` | "A resposta da IA não passou na nossa conferência e não foi mostrada." | Gerar de novo |
| `budget_exhausted` | "O crédito acabou no meio da geração." | Ver saldo |
| `provider_unavailable` | "A IA está fora do ar agora. Nada foi cobrado. Tente em alguns minutos." | Gerar de novo |
| `worker_lost` | "A geração foi interrompida. Tente de novo." | Gerar de novo |

### 6.7 Exportar
No plano aprovado: escolher a versão (a última vem marcada) e baixar o Markdown. Ao lado, "Incluir a nota 'elaborado com apoio de IA'", ligada por padrão (RF-61). A lista de versões mostra a data da última exportação de cada uma.

### 6.8 Minha conta (`/conta`)
Nome, e-mail, tema (papel ou quadro), saldo de IA do mês a partir da F11 ("US$ 0,62 de US$ 1,00; renova em 01/12"), sair, sair de todos os aparelhos e, a partir da F14, excluir a conta.

### 6.9 Sessão vencida no meio da edição
Diálogo: "Sua sessão terminou. Entre de novo para continuar; o que você escreveu está guardado nesta tela." Depois de entrar, a gravação pendente é repetida e o diálogo fecha (0013 D7).

### 6.10 Conflito de revisão
Faixa no topo: "Este plano foi alterado em outra aba ou aparelho." Ações: "Ver a versão mais recente" e "Copiar o meu texto". Nunca sobrescrever em silêncio.

## 7. Texto da interface

- Português do Brasil, tratando por "você". Frases curtas.
- Botão diz a ação: "Aprovar plano", não "Confirmar".
- Mesma palavra para a mesma coisa, sempre a do glossário do [DOMINIO](DOMINIO.md): "sugestão", "achado" só no código (na tela: "bloqueio", "aviso", "para saber"), "etapa de ensino", "tempo efetivo".
- Datas em `17/11/2026`; horas em `14:32`; dinheiro em `US$ 0,62`.
- Todo texto vem do módulo único de textos (0013 D6).

## 8. Conferência

- **Automática, em todo teste ponta a ponta:** axe sem violação (0013 D8), 360 px sem rolagem horizontal, nenhum texto do plano no armazenamento do navegador.
- **Manual, antes de fechar a v1 (F16), em cada tela:** só teclado; NVDA no Windows e TalkBack no Android; zoom de 200% e espaçamento de texto aumentado (WCAG 1.4.12).
- **Contraste:** os pares da seção 2 já passam. Cor nova entra na tabela com o contraste calculado.
