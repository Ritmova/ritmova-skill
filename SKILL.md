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

| Tool                  | Para que serve                          |
| --------------------- | --------------------------------------- |
| **`gerar_imagem`**    | gera uma imagem (capa, asset visual)    |
| **`gerar_locucao`**   | gera locução / voz                      |
| **`gerar_carrossel`** | carrossel de slides (4:5) — copy + arte |
| **`gerar_post`**      | post único de feed — copy + arte        |
| **`gerar_motion`**    | motion / vídeo — **compõe** as atômicas |

O resultado volta como **URL** (o vídeo é montado/renderizado na máquina do usuário).
Contrato das rotas (argumentos e retorno) em [rules/contrato-tools.md](rules/contrato-tools.md).
Para montar/renderizar o motion, use [`/motion-design-ritmova`](../motion-design-ritmova/SKILL.md).

## Tokens e saldo

- O uso é pago em **tokens**. **Free** tem cota e não libera imagem em `qualidade: "high"`; **Pro** libera.
- Se o **saldo acabar** (ou pedir `high` no Free), a rota devolve um **erro de upsell**
  (`SEM_TOKENS`) com pacotes — **explique e ofereça a recarga**, não trate como erro técnico.
  Pacotes: **Mini** 30 tokens (R$ 34,90) · **Turbo** 100 tokens (R$ 109,90).

## Ao atender o usuário

- Para **gerar**, descreva o pedido na tool certa — o cliente MCP faz o `tools/call`.
- Ao receber **`SEM_TOKENS`**: explique o saldo e ofereça os pacotes; não retente sozinho.
- Nunca peça nem exponha chaves de API.
