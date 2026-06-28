---
name: ritmova
description: >
  A RITMOVA é um estúdio criativo com IA via MCP: gera imagem, locução/voz, post, carrossel e
  motion/vídeo de marketing, e guarda tudo num workspace ("computador online") por usuário. Use
  quando o usuário pedir para gerar ou criar imagem, voz/locução, post, carrossel, vídeo ou motion;
  quando perguntar o que é a RITMOVA; quando quiser ver, abrir, organizar, listar ou baixar suas
  peças e arquivos; quando falar de tokens, saldo, créditos, custo, planos Free/Pro ou comprar
  tokens; ou quando pedir o LINK compartilhável de uma peça já criada (gerar_link_unico).
metadata:
  tags: ritmova, mcp, gerar imagem, gerar locucao, voz, escolher voz, listar vozes, salvar voz, vozes salvas, gerar post, gerar carrossel, gerar motion, video, obter trilha, obter efeito, sfx, workspace, computador online, arquivos, pecas, tokens, saldo, creditos, planos, geracao, gerar link unico, link compartilhavel, compartilhar
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
| uma locução / voz                     | `gerar_locucao` (Pro) — passe `voz` (uma das vozes salvas)                                          |
| ver as vozes salvas da conta          | `listar_vozes` (grátis)                                                                             |
| salvar / remover uma voz da lista     | `buscar_vozes_biblioteca` → `salvar_voz` · `remover_voz`                                            |
| um post de feed                       | `gerar_post` (fluxo de 2 chamadas)                                                                  |
| um carrossel                          | `gerar_carrossel` (fluxo de 2 chamadas)                                                             |
| um vídeo narrado / motion             | `gerar_motion` (receita; render no cliente)                                                         |
| ver/abrir/organizar pastas e arquivos | `listar_workspace` · `ler_arquivo` · `escrever_arquivo` · `criar_pasta` · `mover_no` · `excluir_no` |
| ver as peças já criadas               | `listar_pecas`                                                                                      |
| o link compartilhável de uma peça     | `gerar_link_unico` (grátis; recupera/renova o link público `/c/:token`)                             |
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

| Tool                          | Para que serve                                                                                                 |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **`gerar_imagem`**            | gera 1 imagem (GPT Image). As principais geram por dentro via `{{img:<id>}}`; use sozinha para um asset avulso |
| **`gerar_locucao`**           | gera 1 locução/voz (Pro). Passe `voz` = id de uma voz SALVA (via `listar_vozes`); omita p/ a padrão            |
| **`listar_vozes`**            | lista as vozes SALVAS desta conta (nome + id) — grátis, read-only. É o que você mostra ao usuário              |
| **`buscar_vozes_biblioteca`** | busca vozes na biblioteca pública do ElevenLabs (`busca`/`idioma`/`genero`) — read-only                        |
| **`salvar_voz`**              | salva uma voz na lista DESTA conta; se vier da biblioteca, passe `publicOwnerId` (eu trago pra conta)          |
| **`remover_voz`**             | tira uma voz da lista desta conta (não apaga da conta ElevenLabs)                                              |
| **`obter_trilha`**            | trilha sonora oficial do motion (pack licenciado; URL temporária) — é onde o motion é cobrado                  |
| **`obter_efeito`**            | efeito sonoro (SFX) do pack curado por `tags` (whoosh, transição…) — grátis, não cobra                         |

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
`gerar_link_unico` (recupera/renova o link público de uma peça já criada — grátis) · `assinar_pro` ·
`comprar_tokens`.

## Como funciona

**Carrossel e post (2 chamadas).** A receita vem da própria tool: chame sem o conteúdo e ela devolve o
guia (canvas 1080×1350, tipografia, canvas-panorama, o placeholder `{{img:<id>}}`). Autore o HTML
seguindo a receita e chame de novo com o conteúdo — o servidor gera as imagens, renderiza (HTML→PNG) e
cobra a peça 1×. Declare as imagens em `imagens:[{id,prompt}]` e referencie por `{{img:<id>}}` (o
servidor as gera; não use `src` externo nem rode scripts). O resultado volta como URL assinada.

**Fluxo do motion.** `gerar_motion` devolve a receita (grátis) e você monta o vídeo no cliente
(Remotion): **escolha a voz** (veja "Seleção de voz" — não use voz genérica) e gere a narração com
`gerar_locucao` (o áudio é a fonte do tempo), pegue a trilha com
`obter_trilha` (faz parte da receita e é onde o motion é cobrado — pack licenciado, URL temporária),
some os assets com `gerar_imagem` e efeitos com `obter_efeito`, e renderize o MP4. No claude.ai web dá
para montar o roteiro e os blocos e escrever o projeto no workspace; o render final do vídeo roda no PC
do usuário (Claude Code/desktop).

**Seleção de voz (por conta).** Cada conta tem a SUA lista de vozes salvas — `listar_vozes` mostra
só as desta conta. Para escolher, chame `listar_vozes` e passe o `voiceId` em `voz` no
`gerar_locucao`. Para adicionar uma voz nova, ache em `buscar_vozes_biblioteca` e salve com
`salvar_voz` (passando o `publicOwnerId`); `remover_voz` tira da lista. Voz não salva → `VOICE_NOT_FOUND`.

**No motion, a voz importa — NÃO use uma voz genérica.** Antes de gerar a narração, chame
`listar_vozes`. Se houver vozes salvas, mostre (nome + id) e **pergunte qual usar**. Se a lista
estiver **vazia**, **NÃO** caia na voz padrão: **sugira opções** — use `buscar_vozes_biblioteca`
(apresente 2–3 candidatas com nome/idioma/gênero), ajude o usuário a **salvar** a escolhida com
`salvar_voz`, e só então gere a narração com aquela `voz`. A voz é parte da identidade do vídeo.

**Link compartilhável (storage).** Toda peça gerada é guardada no storage da RITMOVA, e o resultado
traz `compartilhar` — uma **string** com o **link público** `storage.ritmova.app/c/:token`
(visualizador de setas + download; o ZIP vem em `compartilharDownload`). Vale para **carrossel,
imagem e post**. Ofereça esse link ao usuário — é o jeito mais fácil de ver, baixar e compartilhar.
Se ele perdeu o link de uma peça já criada (ou ele expirou), use **`gerar_link_unico`** (passe o
`pecaId` de `listar_pecas`, ou nada para a peça mais recente) — recupera/renova o link **sem custo**
(não re-gera nada). Voz e motion não têm esse link.

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
