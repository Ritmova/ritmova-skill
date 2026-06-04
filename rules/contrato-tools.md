---
name: contrato-tools
description: >
  Contrato completo das tools MCP de geração da RITMOVA — como o Claude chama gerar_imagem,
  gerar_locucao, gerar_carrossel e gerar_post: argumentos, exemplo de tools/call, retorno
  (URL assinada + tokens/saldo) e o erro estruturado de upsell SEM_TOKENS. Carregue ao gerar
  imagem/locução/carrossel/post ou ao tratar saldo insuficiente.
metadata:
  tags: contrato, tools, gerar_imagem, gerar_locucao, gerar_carrossel, gerar_post, tools/call, upsell, tokens, mcp
---

# Contrato das tools MCP (geração)

> **Escopo:** apenas **como o Claude chama** as tools de geração — a tool + os argumentos
> que ele envia e o que recebe de volta. Servidor, chaves de API, câmbio e débito de tokens
> são detalhe interno do servidor, nunca expostos aqui.
>
> Na prática o Claude não escreve o JSON na mão: o cliente MCP traduz o pedido em linguagem
> natural num `tools/call`. As chaves de API ficam **só no servidor**.

---

## 🖼️ `gerar_imagem` — gera imagem

Argumentos que o Claude manda:

| Campo        | Tipo                        | Default      | Observação                                     |
| ------------ | --------------------------- | ------------ | ---------------------------------------------- |
| `prompt`     | string                      | —            | descrição da imagem (obrigatório)              |
| `qualidade`  | `"medium"` \| `"high"`      | `"medium"`   | `high` só no Pro; no Free volta erro de upsell |
| `formato`    | `"quadrada"` \| `"retrato"` | `"quadrada"` | retrato = 9:16 (vertical livre)                |
| `quantidade` | int 1–4                     | `1`          | variações                                      |

Chamada (o que o Claude emite):

```jsonc
{
  "jsonrpc": "2.0",
  "id": 12,
  "method": "tools/call",
  "params": {
    "name": "gerar_imagem",
    "arguments": {
      "prompt": "Capa de carrossel: prato de massa fumegante, fundo escuro, luz quente lateral",
      "qualidade": "high",
      "formato": "retrato",
      "quantidade": 1,
    },
  },
}
```

Retorno (o que o Claude recebe):

```jsonc
{
  "jsonrpc": "2.0",
  "id": 12,
  "result": {
    "content": [{ "type": "text", "text": "Imagem pronta. Custo: 1,91 tokens. Saldo: 98,09." }],
    "structuredContent": {
      "imagens": [
        {
          "url": "https://cdn.ritmova.app/a1b2…/img.png",
          "formato": "retrato",
          "qualidade": "high",
        },
      ],
      "tokensCobrados": 1.91,
      "saldo": 98.09,
    },
  },
}
```

---

## 🎙️ `gerar_locucao` — gera locução

> **Exclusiva do Pro** (voz = pipeline de motion, cobrada por tokens). No Free volta
> `QUOTA_EXCEEDED` (assine o Pro). É bloco atômico de `gerar_motion`.

Argumentos que o Claude manda:

| Campo   | Tipo   | Default                | Observação                                       |
| ------- | ------ | ---------------------- | ------------------------------------------------ |
| `texto` | string | —                      | o que será falado (obrigatório)                  |
| `voz`   | string | voz padrão do servidor | opcional; id de voz ElevenLabs (omita p/ padrão) |

Chamada (o que o Claude emite):

```jsonc
{
  "jsonrpc": "2.0",
  "id": 13,
  "method": "tools/call",
  "params": {
    "name": "gerar_locucao",
    "arguments": {
      "texto": "Massa fresca, no ponto certo. Só na sua casa, só no seu ritmo.",
    },
  },
}
```

Retorno (o que o Claude recebe):

```jsonc
{
  "jsonrpc": "2.0",
  "id": 13,
  "result": {
    "content": [{ "type": "text", "text": "Locução pronta (custo: 0,02 tokens)." }],
    "structuredContent": {
      "audioUrl": "https://cdn.ritmova.app/a1b2…/voz.mp3",
      "caracteres": 62,
      "tokensCobrados": 0.02,
      "saldo": 98.07,
    },
  },
}
```

---

## 🎠 `gerar_carrossel` — carrossel (slides 4:5)

Peça de alto nível: o servidor escreve a copy e monta os slides (1080×1350). Argumentos:

| Campo       | Tipo                   | Default    | Observação                                  |
| ----------- | ---------------------- | ---------- | ------------------------------------------- |
| `tema`      | string                 | —          | assunto/briefing do carrossel (obrigatório) |
| `slides`    | int                    | `6`        | nº de slides (limite varia por plano)       |
| `qualidade` | `"medium"` \| `"high"` | `"medium"` | `high` só no Pro                            |

```jsonc
{
  "jsonrpc": "2.0",
  "id": 14,
  "method": "tools/call",
  "params": {
    "name": "gerar_carrossel",
    "arguments": {
      "tema": "5 erros ao cozinhar massa em casa",
      "slides": 6,
      "qualidade": "medium",
    },
  },
}
```

Retorno:

```jsonc
{
  "jsonrpc": "2.0",
  "id": 14,
  "result": {
    "content": [
      { "type": "text", "text": "Carrossel de 6 slides pronto. Custo: 4,30 tokens. Saldo: 95,70." },
    ],
    "structuredContent": {
      "slides": [
        { "index": 1, "url": "https://cdn.ritmova.app/a1b2…/slide-1.png" },
        { "index": 2, "url": "https://cdn.ritmova.app/a1b2…/slide-2.png" },
      ],
      "legenda": "Massa al dente sem mistério 🍝 …",
      "tokensCobrados": 4.3,
      "saldo": 95.7,
    },
  },
}
```

---

## 🖼️ `gerar_post` — post único (feed)

Peça de alto nível: o servidor escreve a legenda e gera a arte. Argumentos:

| Campo       | Tipo                        | Default      | Observação                             |
| ----------- | --------------------------- | ------------ | -------------------------------------- |
| `tema`      | string                      | —            | assunto/briefing do post (obrigatório) |
| `formato`   | `"quadrada"` \| `"retrato"` | `"quadrada"` | retrato = 4:5                          |
| `qualidade` | `"medium"` \| `"high"`      | `"medium"`   | `high` só no Pro                       |

```jsonc
{
  "jsonrpc": "2.0",
  "id": 15,
  "method": "tools/call",
  "params": {
    "name": "gerar_post",
    "arguments": {
      "tema": "Promoção de massa fresca artesanal",
      "formato": "quadrada",
      "qualidade": "medium",
    },
  },
}
```

Retorno:

```jsonc
{
  "jsonrpc": "2.0",
  "id": 15,
  "result": {
    "content": [{ "type": "text", "text": "Post pronto. Custo: 0,48 tokens. Saldo: 95,22." }],
    "structuredContent": {
      "url": "https://cdn.ritmova.app/a1b2…/post.png",
      "legenda": "Hoje tem massa fresca 🍝 …",
      "tokensCobrados": 0.48,
      "saldo": 95.22,
    },
  },
}
```

---

## Erros de upsell (estruturados — nunca 500)

A rota devolve `isError: true` + `structuredContent.code` em vez do resultado. **Explique e
ofereça o caminho certo; nunca retente sozinho nem trate como falha técnica.**

**Pro sem saldo de tokens** — `SEM_TOKENS` (gate de saldo), com pacotes de recarga:

```jsonc
{
  "content": [
    { "type": "text", "text": "Saldo zerado. Compre tokens ou aguarde o reset do ciclo." },
  ],
  "isError": true,
  "structuredContent": {
    "code": "SEM_TOKENS",
    "upsell": {
      "resource": "tokens",
      "packages": [
        { "name": "Mini", "tokens": 30, "priceBrl": 34.9 },
        { "name": "Padrão", "tokens": 50, "priceBrl": 54.9 },
        { "name": "Turbo", "tokens": 100, "priceBrl": 109.9 },
      ],
    },
  },
}
```

**Free** — cota/qualidade: o upsell é **assinar o Pro** (`upsell.suggestedPlan: "pro"`, **sem**
`packages`); aguardar o reset mensal também resolve cota.

| code              | quando                                                                       |
| ----------------- | ---------------------------------------------------------------------------- |
| `HIGH_BLOCKED`    | pediu `qualidade: "high"` (exclusivo do Pro)                                 |
| `QUOTA_EXCEEDED`  | estourou posts/carrosséis/motion do mês, ou pediu locução no Free (Pro-only) |
| `SLIDES_EXCEEDED` | slides acima do máximo do plano (ou não informados)                          |
| `PAUSED`          | geração do Free pausada (kill switch)                                        |

---

## Notas

- `gerar_imagem` e `gerar_locucao` são os **blocos atômicos**; `gerar_post`, `gerar_carrossel`
  e `gerar_motion` **compõem** essas chamadas por baixo.
- O resultado volta como **URL** (o cliente baixa o arquivo); as chaves de API ficam no servidor.
