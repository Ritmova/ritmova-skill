---
name: ritmova
description: >
  A RITMOVA é um estúdio criativo com IA (via MCP): gera imagem, locução/voz e motion/vídeo.
  Use esta skill quando o usuário perguntar o que é a RITMOVA, pedir para GERAR imagem,
  locução/voz, post, carrossel ou motion/vídeo, ou perguntar sobre tokens, saldo, custo,
  planos (Free/Pro) e compra de tokens. Documenta as rotas MCP (gerar_imagem, gerar_locucao,
  gerar_carrossel, gerar_post) e o erro de upsell de saldo.
metadata:
  tags: ritmova, mcp, gerar_imagem, gerar_locucao, gerar_carrossel, gerar_post, carrossel, post, tokens, saldo, motion, geracao
---

# RITMOVA

A **RITMOVA** é um **estúdio criativo com IA**: o usuário pede conteúdo de marketing em
linguagem natural e o Claude **chama as tools da RITMOVA** para gerar.

## Ferramentas

**Principais** (porta de entrada — **entregam a RECEITA** e cobram a peça):

| Tool                  | Como usar (two-phase)                                                                                                       |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **`gerar_carrossel`** | **Chame SEM `slides`** → recebe a receita; autore o HTML de cada slide; **chame de novo com `slides`** → renderiza+cobra 1× |
| **`gerar_post`**      | **Chame SEM `html`** → recebe a receita; autore o HTML; **chame de novo com `html`**                                        |
| **`gerar_motion`**    | motion / vídeo (Fase 2) — mesmo padrão; compõe as sub-ferramentas                                                           |

**Sub-ferramentas** (aprimoram as principais; também funcionam sozinhas):

| Tool                | Para que serve                                                                                               |
| ------------------- | ------------------------------------------------------------------------------------------------------------ |
| **`gerar_imagem`**  | gera 1 imagem (GPT Image). As principais geram por dentro via `{{img:<id>}}`; use sozinha p/ um asset avulso |
| **`gerar_locucao`** | gera 1 locução / voz (Pro)                                                                                   |

**Conta & billing:** `get_account` (plano/saldo/cota) · `listar_pecas` (peças geradas, com links) ·
`assinar_pro` (checkout Pro — upsell de `HIGH_BLOCKED`/`QUOTA_EXCEEDED`) · `comprar_tokens` (checkout
de tokens — upsell `SEM_TOKENS`).

> **A receita vem da PRÓPRIA ferramenta.** Para carrossel/post, chame a tool principal **sem o
> conteúdo** (`gerar_carrossel` sem `slides`, `gerar_post` sem `html`) → ela devolve a **RECEITA**
> (canvas 1080×1350, tipografia, canvas-panorama, placeholder `{{img:<id>}}`). Autore o HTML
> seguindo-a À RISCA e chame de novo com o conteúdo. **Não** rode scripts nem use chaves — as
> imagens saem pela sub-ferramenta, declaradas em `imagens:[{id,prompt}]` e referenciadas por `{{img:<id>}}`.

O resultado volta como **URL assinada** (carrossel/post são renderizados no servidor; o
arquivo fica em storage privado). Contrato em [rules/contrato-tools.md](rules/contrato-tools.md).
Para montar/renderizar o motion, use [`/motion-design-ritmova`](../motion-design-ritmova/SKILL.md).

## Tokens e saldo

- O **Pro** é pago em **tokens**; o **Free** tem **cota** (não tokens) e não libera `qualidade: "high"`.
- **Pro sem saldo** → `SEM_TOKENS`: upsell com **pacotes de tokens** — **explique e ofereça a recarga**.
  Pacotes: **Mini** 30 (R$ 34,90) · **Padrão** 50 (R$ 54,90) · **Turbo** 100 (R$ 109,90).
- **Free pedindo `high`** → `HIGH_BLOCKED`, e **cota estourada** → `QUOTA_EXCEEDED`/`SLIDES_EXCEEDED`
  (ou `PAUSED`): o upsell é **assinar o Pro** (não é recarga de tokens); aguardar o reset também resolve cota.

## Ao atender o usuário

- Para **gerar**, descreva o pedido na tool certa — o cliente MCP faz o `tools/call`.
- Ao receber **`SEM_TOKENS`**: explique o saldo e ofereça os pacotes; não retente sozinho.
- Nunca peça nem exponha chaves de API.
