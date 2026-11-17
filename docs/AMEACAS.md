# Mapa de ameaças: Docenda

> **v0.1 (17/11/2026). Planejado: o sistema ainda não existe.** Pedido pela decisão [0009](decisoes/0009-ci-cd-and-security-checks.md) (D10) para antes de publicar; escrito agora para guiar as fatias. É revisado no marco M3, contra o sistema de verdade.
> Método: **STRIDE**, seis perguntas para cada parte do sistema: alguém pode se passar por outro (*Spoofing*)? Alterar algo (*Tampering*)? Negar o que fez (*Repudiation*)? Ver o que não devia (*Information disclosure*)? Derrubar o serviço (*Denial of service*)? Ganhar poder que não tem (*Elevation of privilege*)?
> Riscos em aberto são descritos em termos gerais, sem o caminho para explorá-los.

## 1. Fluxo de dados

```
                     ┌──────────── fronteira 1: internet ────────────┐
  Navegador do autor ─HTTPS─▶ Proxy (TLS) ─▶ API (Express) ─▶ PostgreSQL
  Navegador do admin ─HTTPS─┘                    │   ▲          ▲  (dados, sessões,
                                                 │   │          │   fila pg-boss)
                                     job só com IDs  │          │
                                                 ▼   │          │
                                               Worker ──────────┘
                                                 │
           fronteira 2: fora do Brasil ──────────┼──────────▶ API da Anthropic
           fronteira 3: internet aberta ─────────┼──────────▶ Link da fonte (F13)
           fronteira 4: e-mail ──────────────────┴──────────▶ Mailpit / Amazon SES
  
  GitHub (código, CI) ─▶ GHCR (imagem) ─▶ Servidor          Administrador ─SSH─▶ Servidor
```

| Fluxo | O que passa | Dado pessoal? |
|---|---|---|
| Navegador → API | Login, contexto da turma, plano, identificação do plano | Sim: conta e identificação |
| API → PostgreSQL | Tudo o que é gravado | Sim |
| API → fila → worker | Só IDs (0008 D6) | Não |
| Worker → Anthropic | Tema, contexto agregado, material colado, campos do plano, habilidades | **Não deveria:** nunca identificação nem dados da conta (INV-09); alerta de privacidade antes (INV-10) |
| Worker → link da fonte | O endereço que o autor informou | Não |
| API → e-mail | Endereço e link de convite, de confirmação ou de senha | Sim: e-mail |
| Administrador → servidor | Acesso por SSH, só do IP dele | Não |

## 2. Ameaças, defesas e o que falta

Defesas citadas pela decisão ou pelo requisito que as cria. "Falta" é o que ainda não está coberto.

| # | STRIDE | Ameaça | Defesa planejada | Falta |
|---|---|---|---|---|
| T01 | S | Adivinhar a senha de alguém ou testar senhas vazadas de outros sites | Argon2id, lista de senhas proibidas, 3 tentativas a cada 10 s por IP, mensagem única (0003, 0006 D4) | Limite por conta e 2FA para todos, depois da v1 |
| T02 | S | Roubar a sessão | Cookie `HttpOnly`, `SameSite`, `Secure`; sessão no servidor, revogável; 30 min parada (0003, 0006 D1, D2) | — |
| T03 | S | Tomar a conta do administrador | Criado por comando (RF-79); 2FA obrigatório para rotas de administrador (0009 D10) | 2FA entra antes de publicar |
| T04 | S | Fingir ser o IP de outro para escapar do limite | IP lido só do cabeçalho do próprio proxy (0006 D3) | — |
| T05 | T | Usar a sessão do autor a partir de outro site (CSRF) | `SameSite=Lax` e conferência do `Origin` (0006 D5) | — |
| T06 | T | Alterar o plano de outro autor trocando o ID | `ownerId` em todo repositório; 404 para recurso alheio; teste que percorre todas as rotas (0012 D4, INV-01) | Segurança por linha no banco, como segunda camada (ROADMAP) |
| T07 | T | Texto colado que faz o modelo mudar de comportamento (injeção de prompt) | Texto marcado como dado; marcação escapada; saída presa a esquema; checagem de entrada e de saída; a IA só cria sugestões (IA.md, 0005 D5) | Não existe defesa completa contra injeção; o dano fica limitado porque a IA não age no sistema |
| T08 | T | IA escrever conteúdo direto no plano ou aprovar | O worker só cria sugestões; aprovar só pela rota do dono (INV-03, INV-04) | — |
| T09 | T | Habilidade ou fonte inventada entrar no plano | Lista fechada no esquema; chave estrangeira; fonte só aprovada pelo autor (INV-02, RF-51) | — |
| T10 | T | Dependência ou ação do CI comprometida | Lockfile, Dependabot com espera de 7 dias, CodeQL, ações fixadas, Gitleaks (0009) | Varredura da imagem antes de publicar (ROADMAP, M3) |
| T11 | R | Autor ou administrador negar o que fez | Auditoria só com IDs, que a aplicação não altera (INV-14, 0010 D6); registro de acesso por 6 meses | — |
| T12 | I | Dado de aluno chegar à IA ou ao banco | Turma descrita em conjunto; lista de termos e checagem por modelo; texto com alerta não é gravado (RF-15, DADOS D6) | Checagem automática erra; a revisão humana dos dados gravados é um critério do PROPÓSITO |
| T13 | I | Identificação ou dados da conta irem para a IA | Tabela separada; tipo da porta de IA sem esses campos; teste com valores-isca (INV-09, RNF-45) | — |
| T14 | I | Dado pessoal no log técnico | Corpo do pedido nunca registrado; `redact`; IP fora do log; teste com valores-isca (0010 D3) | — |
| T15 | I | Erro revelar detalhes internos | Formato de problema com `code` e número do pedido, sem rastro de pilha (0012 D7) | — |
| T16 | I | Usar a checagem de link para alcançar endereços internos (SSRF) | Só `https`; endereço conferido depois de resolver o nome; redirecionamento conferido; tempo e tamanho máximos (RNF-43) | Entra com a F13 |
| T17 | I | Vazamento do banco ou de um backup | Senha em hash; cookie assinado com segredo fora do banco; backup criptografado e com prazo (0006 D1, 0011) | Criptografia em repouso e teste de restauração antes de publicar |
| T18 | I | Segredo no repositório | Gitleaks no CI; `.env` ignorado; CI sem chave da IA (0009) | — |
| T19 | I | Texto da IA ou do autor virar código na tela (XSS) | Texto puro em todos os campos; React escapa; CSP `default-src 'self'` (0006) | — |
| T20 | D | Esgotar o dinheiro da IA | Crédito por pessoa, teto do sistema, limite por minuto e por dia, reserva antes da chamada, limite de gasto no provedor (0005 D6, 0008, INV-06) | — |
| T21 | D | Derrubar a API com muitos pedidos ou pedidos enormes | Limite geral por IP e por sessão; corpo de até 256 kB (RNF-44) | Proteção de rede contra ataque distribuído fica com a hospedagem |
| T22 | D | Provedor de IA fora do ar travar o sistema | Disjuntor; geração na fila; o resto funciona sem IA (RNF-36, RNF-34) | — |
| T23 | D | Geração presa para sempre bloquear o plano | Prazo por geração e varredura de órfãs (INV-08) | — |
| T24 | E | Autor acessar rota de administrador | Papel conferido na rota; negado por padrão; teste (0006 D5) | — |
| T25 | E | A aplicação alterar o próprio esquema ou a auditoria | Três papéis no banco; a aplicação não migra nem apaga auditoria (0012 D5) | — |
| T26 | E | Fuga do container da API | Sem root, sem capacidades, sistema de arquivos só de leitura, sem o socket do Docker (0007 D2) | — |
| T27 | E | Agente de código com acesso além do projeto | Dev container com firewall de saída; sessões na nuvem sem os arquivos privados (0007 D3) | Montar o dev container na F00 |

## 3. Riscos aceitos na v1

| Risco | Por que é aceito |
|---|---|
| Dados enviados à IA são processados fora do Brasil e guardados por até 30 dias pelo provedor | É o custo de usar o provedor; o aviso de privacidade e a tela de geração dizem isso (RNF-21). Nada identificável deveria ir |
| Banco e segredo vazados juntos permitem usar sessões ativas | Exigiria dois vazamentos; trocar o segredo encerra todas as sessões (0006 D1) |
| Sem coletor de erros externo | Até 10 pessoas na v1; os três alertas por e-mail cobrem o que custa dinheiro ou perde trabalho (0010 D7) |

## 4. Antes de publicar (M3)

- Revisar este mapa contra o sistema construído: cada "Falta" vira item do backlog ou risco aceito com motivo.
- Rodar a varredura do OWASP ZAP e uma auditoria de segurança do código (0009 D10).
- Conferir o ASVS nível 1 item a item (RNF-06).
- Fazer a simulação de incidente do [INCIDENTES](INCIDENTES.md).
