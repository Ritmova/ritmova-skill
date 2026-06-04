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

## Rotas (o que cada uma serve)

| Tool                  | Para que serve                                                                       |
| --------------------- | ------------------------------------------------------------------------------------ |
| **`gerar_imagem`**    | gera uma imagem (capa, asset visual)                                                 |
| **`gerar_locucao`**   | gera locução / voz                                                                   |
| **`gerar_carrossel`** | carrossel (4:5) — VOCÊ autora o HTML/CSS+prompts; servidor renderiza+cobra 1×        |
| **`gerar_post`**      | post de feed — VOCÊ autora o HTML/CSS+prompts; servidor renderiza+cobra 1×           |
| **`gerar_motion`**    | motion / vídeo — **compõe** as atômicas (Fase 2)                                     |
| **`listar_pecas`**    | lista as peças já geradas (read-only, URL assinada)                                  |
| **`assinar_pro`**     | link de checkout p/ assinar o Pro (use no upsell de `HIGH_BLOCKED`/`QUOTA_EXCEEDED`) |
| **`comprar_tokens`**  | link de checkout p/ comprar tokens avulsos (Pro) — use no upsell `SEM_TOKENS`        |

> **Carrossel/post — leia a RECEITA primeiro.** Antes de autorar o HTML, **leia o resource MCP
> `ritmova://skills/carrossel`** (servido pelo servidor): é a receita completa de layout, fontes,
> técnica canvas-panorama e o placeholder `{{img:<id>}}`. Sem ela, o resultado não tem a qualidade
> da RITMOVA.

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
