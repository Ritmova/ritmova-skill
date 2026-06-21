---
name: ritmova
description: >
  A RITMOVA é um estúdio criativo com IA via MCP: gera imagem, locução/voz, post, carrossel e
  motion/vídeo de marketing, e guarda tudo num workspace ("computador online") por usuário. Use
  quando o usuário pedir para gerar ou criar imagem, voz/locução, post, carrossel, vídeo ou motion;
  quando perguntar o que é a RITMOVA; quando quiser ver, abrir, organizar, listar ou baixar suas
  peças e arquivos; ou quando falar de tokens, saldo, créditos, custo, planos Free/Pro ou comprar
  tokens.
metadata:
  tags: ritmova, mcp, gerar imagem, gerar locucao, voz, gerar post, gerar carrossel, gerar motion, video, obter trilha, obter efeito, sfx, workspace, computador online, arquivos, pecas, tokens, saldo, creditos, planos, geracao
---

# RITMOVA

A RITMOVA é um estúdio criativo com IA: o usuário pede conteúdo de marketing em linguagem natural,
você (Claude) chama as tools da RITMOVA para gerar, e o resultado fica guardado num workspace que o
usuário acompanha pelo site. As chaves de API e o render ficam no servidor — você descreve o pedido
e o cliente MCP faz o `tools/call`.

## Pedido do usuário → tool

| O usuário quer…                       | Use                                                                                                 |
| ------------------------------------- | --------------------------------------------------------------------------------------------------- |
| uma imagem avulsa                     | `gerar_imagem`                                                                                      |
| uma locução / voz                     | `gerar_locucao` (Pro)                                                                               |
| um post de feed                       | `gerar_post` (fluxo de 2 chamadas)                                                                  |
| um carrossel                          | `gerar_carrossel` (fluxo de 2 chamadas)                                                             |
| um vídeo narrado / motion             | `gerar_motion` (receita; render no cliente)                                                         |
| ver/abrir/organizar pastas e arquivos | `listar_workspace` · `ler_arquivo` · `escrever_arquivo` · `criar_pasta` · `mover_no` · `excluir_no` |
| ver as peças já criadas               | `listar_pecas`                                                                                      |
| saber plano, saldo ou cota            | `get_account`                                                                                       |
| assinar Pro / comprar tokens          | `assinar_pro` · `comprar_tokens`                                                                    |

Contrato completo das tools (argumentos, exemplos, retorno): [rules/contrato-tools.md](rules/contrato-tools.md).

## Etapa 0 — Planeje antes de construir

Para qualquer entrega de produção (vídeo, arte, post, campanha), comece escrevendo para si mesmo um
plano curto antes de executar — isso reduz erro de forma medida (Plan-and-Solve, ACL 2023; ferramenta
`think` da Anthropic). Na ordem:

1. Defina o resultado e os critérios de sucesso em 1–2 frases (o que é, para quem, o que é "pronto").
2. Confira o que o MCP da RITMOVA oferece de fato — liste as tools antes de planejar, sem presumir.
3. Reúna o que precisa para um resultado profissional (referências, restrições, boas práticas do formato).
4. Escreva o plano: papel e objetivo, contexto e motivação, critérios de sucesso, passos numerados, quais
   tools usar e quando, formato da saída, e uma autoverificação no fim. Use exemplos (few-shot) e tags XML
   quando o conteúdo misturar instrução, contexto e dados.
5. Construa seguindo o plano, mantenha o escopo, e revise no final.

Se faltar o essencial para algo profissional — paleta/cores, estética, tom de voz, público-alvo, formato e
proporção, marca/logo, referências e duração (motion) — pergunte ao usuário antes de construir. Prefira
poucas perguntas específicas, com opções quando ajudar.

## Ferramentas

**Geração (porta de entrada):**

| Tool                  | Como funciona                                                                                                                                               |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`gerar_carrossel`** | 2 chamadas: chame sem `slides` para receber a receita; autore o HTML de cada slide seguindo-a; chame de novo com `slides` para renderizar (cobra 1× a peça) |
| **`gerar_post`**      | 2 chamadas: chame sem `html` para receber a receita; autore o HTML; chame de novo com `html` para renderizar (cobra 1×)                                     |
| **`gerar_motion`**    | 1 chamada: devolve a receita do vídeo (grátis). Você monta e renderiza o MP4 (Remotion) — ver "Como funciona"                                               |

**Sub-ferramentas** (compõem as principais; também funcionam sozinhas):

| Tool                | Para que serve                                                                                                 |
| ------------------- | -------------------------------------------------------------------------------------------------------------- |
| **`gerar_imagem`**  | gera 1 imagem (GPT Image). As principais geram por dentro via `{{img:<id>}}`; use sozinha para um asset avulso |
| **`gerar_locucao`** | gera 1 locução/voz (Pro)                                                                                       |
| **`obter_trilha`**  | trilha sonora oficial do motion (pack licenciado; URL temporária) — é onde o motion é cobrado                  |
| **`obter_efeito`**  | efeito sonoro (SFX) do pack curado por `tags` (whoosh, transição…) — grátis, não cobra                         |

**Computador online (workspace)** — espaço de arquivos por usuário, visível no site. Útil para quem não
tem Claude Code: você organiza o trabalho em pastas e arquivos que o usuário acessa pela web. As peças
geradas já aparecem na pasta `/pecas`.

| Tool                                                  | Para que serve                                                                 |
| ----------------------------------------------------- | ------------------------------------------------------------------------------ |
| **`listar_workspace`**                                | lista pastas/arquivos (sem `caminho` = raiz; `recursivo:true` = tudo)          |
| **`ler_arquivo`**                                     | lê um arquivo (texto/HTML volta como conteúdo; peça volta como URL)            |
| **`escrever_arquivo`**                                | cria/sobrescreve um arquivo de texto/HTML (cria as pastas do caminho)          |
| **`criar_pasta`** · **`mover_no`** · **`excluir_no`** | cria pasta · move/renomeia · exclui (`recursivo:true` para pasta com conteúdo) |

As tools de geração aceitam um `caminho` opcional para salvar a peça onde você quiser (ex.:
`gerar_carrossel(..., caminho:"/campanha-x")`).

**Conta & billing:** `get_account` (plano/saldo/cota) · `listar_pecas` (peças geradas, com links) ·
`assinar_pro` · `comprar_tokens`.

## Como funciona

**Carrossel e post (2 chamadas).** A receita vem da própria tool: chame sem o conteúdo e ela devolve o
guia (canvas 1080×1350, tipografia, canvas-panorama, o placeholder `{{img:<id>}}`). Autore o HTML
seguindo a receita e chame de novo com o conteúdo — o servidor gera as imagens, renderiza (HTML→PNG) e
cobra a peça 1×. Declare as imagens em `imagens:[{id,prompt}]` e referencie por `{{img:<id>}}` (o
servidor as gera; não use `src` externo nem rode scripts). O resultado volta como URL assinada.

**Fluxo do motion.** `gerar_motion` devolve a receita (grátis) e você monta o vídeo no cliente
(Remotion): gere a narração com `gerar_locucao` (o áudio é a fonte do tempo), pegue a trilha com
`obter_trilha` (faz parte da receita e é onde o motion é cobrado — pack licenciado, URL temporária),
some os assets com `gerar_imagem` e efeitos com `obter_efeito`, e renderize o MP4. No claude.ai web dá
para montar o roteiro e os blocos e escrever o projeto no workspace; o render final do vídeo roda no PC
do usuário (Claude Code/desktop).

**Link compartilhável (storage).** Toda peça gerada é guardada no storage da RITMOVA, e o resultado
traz um campo `compartilhar`: no carrossel, `compartilhar.shareUrl` é um **link único** com visualizador
de setas + download ZIP; em imagem/post/locução, `compartilhar.mediaUrl` aponta a mídia. Ofereça esse
link ao usuário — é o jeito mais fácil de ele ver, baixar e compartilhar a peça.

## Tokens e saldo

- O **Pro** é pago em **tokens**; o **Free** tem **cota** (não tokens) e não libera `qualidade: "high"`.
- **Pro sem saldo** → erro `SEM_TOKENS`: ele já traz os pacotes de recarga atuais e os preços — explique o
  saldo e ofereça o que vier no erro (não invente valores). Não retente sozinho.
- **Free pedindo `high`** → `HIGH_BLOCKED`; **cota estourada** → `QUOTA_EXCEEDED`/`SLIDES_EXCEEDED` (ou
  `PAUSED`): o caminho é assinar o Pro; aguardar o reset mensal também resolve cota.

## Ao atender o usuário

- Para gerar, descreva o pedido na tool certa — o cliente MCP faz o `tools/call`.
- Trate os erros de upsell (`SEM_TOKENS`, `HIGH_BLOCKED`, `QUOTA_EXCEEDED`) explicando o saldo/plano e
  oferecendo o caminho que o erro indica.
- Não peça nem exponha chaves de API.
