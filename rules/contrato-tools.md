---
name: contrato-tools
description: >
  Contrato completo das tools MCP da RITMOVA — como o Claude chama gerar_imagem, gerar_locucao,
  gerar_carrossel, gerar_post, gerar_motion, obter_trilha (música), obter_efeito (SFX, grátis) e as
  tools do "computador online" (listar_workspace, ler_arquivo, escrever_arquivo, criar_pasta,
  mover_no, excluir_no): argumentos, exemplo de tools/call, retorno (URL assinada + tokens/saldo) e o
  erro estruturado de upsell SEM_TOKENS. Carregue ao gerar qualquer peça, criar/organizar arquivos,
  ou tratar saldo insuficiente.
metadata:
  tags: contrato, tools, gerar_imagem, gerar_locucao, voz, listar_vozes, buscar_vozes, salvar_voz, remover_voz, vozes salvas, gerar_carrossel, gerar_post, gerar_motion, obter_trilha, obter_efeito, listar_workspace, escrever_arquivo, workspace, computador online, arquivos, tools/call, upsell, tokens, mcp
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

| Campo   | Tipo   | Default                | Observação                                                                                            |
| ------- | ------ | ---------------------- | ----------------------------------------------------------------------------------------------------- |
| `texto` | string | —                      | o que será falado (obrigatório)                                                                       |
| `voz`   | string | voz padrão do servidor | opcional; `voiceId` de uma voz SALVA (`listar_vozes`; omita p/ padrão). Não salva → `VOICE_NOT_FOUND` |

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

## 🗣️ Seleção de voz — `listar_vozes` · `buscar_vozes` · `salvar_voz` · `remover_voz`

Cada conta tem a SUA lista de vozes salvas (no NOSSO banco). A conta ElevenLabs é única (a chave fica
no servidor) e **NÃO é tocada** — `salvar_voz` só grava o id no nosso banco. **Cada conta só vê/usa o
que salvou.** Passe o `voiceId` da voz escolhida em `voz` no `gerar_locucao`.

| Tool                               | Args                           | Retorno                                                                                             |
| ---------------------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------- |
| `listar_vozes` (grátis, read-only) | `filtro?` (nome/id)            | as vozes SALVAS desta conta: `{ vozes: [{ voiceId, name }] }`                                       |
| `buscar_vozes` (read-only)         | `busca?`, `idioma?`, `genero?` | catálogo do ElevenLabs (premade + da conta): `{ vozes: [{ voiceId, name, labels?, previewUrl? }] }` |
| `salvar_voz`                       | `voiceId`, `nome`              | salva o id na lista (NOSSO banco): `{ voiceId, name }`                                              |
| `remover_voz`                      | `voiceId`                      | `{ removed }` (tira da lista; não apaga da conta ElevenLabs)                                        |

```jsonc
// 1) ver as vozes salvas e escolher uma
{ "method": "tools/call", "params": { "name": "listar_vozes", "arguments": {} } }
// → { "vozes": [{ "voiceId": "EXAV…", "name": "Bella" }] }

// 2) usar a voz escolhida na locução
{ "method": "tools/call", "params": { "name": "gerar_locucao",
  "arguments": { "texto": "…", "voz": "EXAV…" } } }

// 3) salvar uma voz nova: ache no catálogo e salve (só voiceId + nome — NADA na conta ElevenLabs)
{ "method": "tools/call", "params": { "name": "buscar_vozes",
  "arguments": { "busca": "narrador", "idioma": "pt" } } }
{ "method": "tools/call", "params": { "name": "salvar_voz",
  "arguments": { "voiceId": "EXAV…", "nome": "Narrador PT" } } }
```

- `gerar_locucao` só aceita voz que esteja **salva** pela conta (senão `VOICE_NOT_FOUND`).
- `salvar_voz` NÃO escreve nada na conta ElevenLabs — só no nosso banco. O catálogo (`buscar_vozes`)
  traz só vozes **premade/da conta** (o TTS aceita direto). Vozes da biblioteca da COMUNIDADE não
  entram (exigiriam adicionar à conta). `voiceId` fora do catálogo → `VOICE_NOT_FOUND`.
- **Motion:** antes da narração, chame `listar_vozes`. Lista vazia? **NÃO** use a voz padrão —
  **sugira** opções via `buscar_vozes`, ajude a `salvar_voz`, e pergunte qual usar.

---

## 🎠 `gerar_carrossel` — carrossel (slides 4:5)

> **VOCÊ (Claude) autora** o HTML/CSS de cada slide (canvas EXATO 1080×1350, seguindo a skill de
> carrosséis) e os prompts de imagem. O servidor gera as imagens (gpt-image), injeta-as pelo
> placeholder `{{img:<id>}}`, renderiza HTML→PNG (Chrome headless, **rede desligada**) e cobra
> **UMA vez** (1 carrossel — nunca por slide). Referencie imagens SÓ por `{{img:<id>}}` (ex.:
> `<img src="{{img:burger}}">`); **`src` http externo é recusado**.
>
> **Two-phase:** chame SEM `slides` para **receber a RECEITA** (canvas 1080×1350, tipografia,
> canvas-panorama, `{{img:<id>}}`) — essa chamada **NÃO cobra**. Autore o HTML seguindo-a À RISCA e
> chame **de novo** com `slides` para renderizar+cobrar.

| Campo            | Tipo                              | Default    | Observação                                                                                                                  |
| ---------------- | --------------------------------- | ---------- | --------------------------------------------------------------------------------------------------------------------------- |
| `slides`         | `{ html; imagens? }[]` (opcional) | —          | **ausente/vazio ⇒ devolve a RECEITA e NÃO cobra** (two-phase); presente ⇒ 1..N slides (N ≤ `slidesMax`), renderiza+cobra 1× |
| `qualidade`      | `"medium"` \| `"high"`            | `"medium"` | `high` só no Pro                                                                                                            |
| `idempotencyKey` | string                            | —          | opcional; retry devolve a mesma peça                                                                                        |

`Img = { id: string; prompt: string; transparente?: boolean }`.

```jsonc
{
  "jsonrpc": "2.0",
  "id": 14,
  "method": "tools/call",
  "params": {
    "name": "gerar_carrossel",
    "arguments": {
      "slides": [
        {
          "html": "<html>…canvas 1080×1350 com <img src='{{img:burger}}'>…</html>",
          "imagens": [
            {
              "id": "burger",
              "prompt": "hambúrguer suculento, fundo transparente",
              "transparente": true,
            },
          ],
        },
        { "html": "<html>…slide 2…</html>" },
      ],
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
    "content": [{ "type": "text", "text": "Carrossel de 2 slides pronto (custo: 0,11 tokens)." }],
    "structuredContent": {
      "slides": [
        { "index": 1, "url": "https://cdn.ritmova.app/a1b2…/slide-1.png" },
        { "index": 2, "url": "https://cdn.ritmova.app/a1b2…/slide-2.png" },
      ],
      "tokensCobrados": 0.11,
      "saldo": 95.7,
      // Link compartilhável público (string) + o ZIP à parte — ofereça ao usuário.
      "compartilhar": "https://storage.ritmova.app/c/AbC123…",
      "compartilharDownload": "https://storage.ritmova.app/carousel/…/download",
    },
  },
}
```

---

## 🖼️ `gerar_post` — post único (feed)

> Igual ao carrossel, **1 slide**: você autora o `html`; o servidor gera as imagens, injeta por
> `{{img:<id>}}`, renderiza e cobra (kind `post`).
>
> **Two-phase:** chame SEM `html` para **receber a RECEITA** — essa chamada **NÃO cobra**. Autore o
> HTML seguindo-a e chame **de novo** com `html` para renderizar+cobrar.

| Campo            | Tipo                          | Default      | Observação                                                                                                   |
| ---------------- | ----------------------------- | ------------ | ------------------------------------------------------------------------------------------------------------ |
| `html`           | string (opcional)             | —            | **ausente/vazio ⇒ devolve a RECEITA e NÃO cobra** (two-phase); presente ⇒ documento 1080×1080 (ou 1080×1350) |
| `imagens`        | `{id,prompt,transparente?}[]` | —            | opcional; referidas por `{{img:id}}`                                                                         |
| `formato`        | `"quadrada"` \| `"retrato"`   | `"quadrada"` | quadrada=1080×1080, retrato=1080×1350                                                                        |
| `qualidade`      | `"medium"` \| `"high"`        | `"medium"`   | `high` só no Pro                                                                                             |
| `idempotencyKey` | string                        | —            | opcional                                                                                                     |

```jsonc
{
  "jsonrpc": "2.0",
  "id": 15,
  "method": "tools/call",
  "params": {
    "name": "gerar_post",
    "arguments": {
      "html": "<html>…canvas com <img src='{{img:prato}}'>…</html>",
      "imagens": [{ "id": "prato", "prompt": "massa fresca", "transparente": true }],
      "formato": "quadrada",
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
    "content": [{ "type": "text", "text": "Post pronto (custo: 0,05 tokens)." }],
    "structuredContent": {
      "url": "https://cdn.ritmova.app/a1b2…/post.png",
      "tokensCobrados": 0.05,
      "saldo": 95.22,
    },
  },
}
```

---

## 🎬 `gerar_motion` — motion / vídeo narrado (o CLIENTE renderiza)

**Diferente das outras: ENTREGA a receita (grátis) e VOCÊ renderiza o vídeo localmente** (Remotion).
**Não** é two-phase e **não renderiza no servidor**. O **preço-base do motion é cobrado na TRILHA**.

- **Chame** `gerar_motion` (sem argumentos) → devolve a **RECEITA** de motion (grátis).
- Monte o vídeo seguindo-a: **narração** com `gerar_locucao` (o **áudio é a FONTE DO TEMPO**),
  **trilha OBRIGATÓRIA** com `obter_trilha`, **assets** com `gerar_imagem`, e **renderize o MP4
  na sua máquina** (Remotion).
- **`obter_trilha` é a âncora de cobrança do motion**: entrega a música do pack licenciado da
  RITMOVA por URL temporária e cobra o preço-base 1× por trabalho — **Pro: 3 tokens · Free: a
  amostra única** — além de contar o motion no painel. Re-pedir no MESMO trabalho (janela ~30 min)
  não cobra de novo; **cada vídeo NOVO cobra** (a URL expira — chame de novo). Free sem amostra /
  Pro sem saldo → erro de upsell.
- **`obter_efeito` (SFX) — GRÁTIS:** efeitos sonoros (whoosh, transição, pop, impacto…) do pack
  curado da RITMOVA, por URL temporária, escolhidos por `tags`. **Não cobra token nem cota.** Ótimo
  para transições/impactos no motion. As tags disponíveis estão na description da tool.
- Voz e imagens cobram **à parte**, pelo custo real de cada uma.
- Em clientes sem execução local (ex.: claude.ai web) dá p/ montar roteiro + blocos, mas o render
  final precisa de ambiente local (Claude Code/desktop).

```jsonc
// 1) receita (grátis)
{ "method": "tools/call", "params": { "name": "gerar_motion", "arguments": {} } }

// 2) trilha (cobra o preço-base do motion). As TAGS DISPONÍVEIS estão listadas na própria
//    description da tool (cardápio do pack) — use-as no briefing em vez de inventar.
//    SEMPRE mande idempotencyKey = um id ESTÁVEL do vídeo (mesmo valor p/ re-baixar a trilha do
//    MESMO vídeo; valor NOVO p/ cada vídeo NOVO). É isso que separa "retry" de "vídeo novo" —
//    sem ela, cada chamada cobra/conta de novo (e no Free a 2ª é bloqueada).
{
  "method": "tools/call",
  "params": {
    "name": "obter_trilha",
    "arguments": { "tags": ["epicas"], "duracaoMinSegundos": 40, "idempotencyKey": "promo-cafe-v1" }
  }
}
// → devolve { trilha: { id, tags, duracaoMs?, bpm?, url }, tokensCobrados, replayed, saldo? }
//   Baixe o áudio da `url` (temporária!) e use no render. `replayed: true` = mesmo vídeo (mesma
//   idempotencyKey), não cobrou de novo.

// 3) efeito sonoro (SFX) — GRÁTIS, sem idempotencyKey (não cobra). Opcional.
{ "method": "tools/call", "params": { "name": "obter_efeito", "arguments": { "tags": ["whoosh"] } } }
// → devolve { efeito: { id, tags, duracaoMs?, url }, tokensCobrados: 0 }. Baixe a `url` e use no render.
```

---

## 🗂️ Computador online (workspace) — pastas e arquivos por usuário

Espaço de arquivos por conta, persistente, que o usuário vê/baixa no painel do site. Útil para quem
**não tem Claude Code**: você organiza o trabalho em pastas/arquivos/HTML que ele acessa pela web.
Tudo é **tenant-scoped** (cada conta só enxerga o seu) e **não cobra tokens**. Caminhos são lógicos
(`/projeto/x.html`); `..`, `//` e caminhos codificados são recusados.

| Tool               | Args                                            | Retorno                                                            |
| ------------------ | ----------------------------------------------- | ------------------------------------------------------------------ |
| `listar_workspace` | `caminho?`, `recursivo?`                        | `{ caminho, nos: [{ path, name, type, mime?, bytes, isPeca }] }`   |
| `ler_arquivo`      | `caminho`                                       | texto → `{ conteudo, mime, bytes }`; peça → `{ url, mime, bytes }` |
| `escrever_arquivo` | `caminho`, `conteudo`, `mime?`, `sobrescrever?` | `{ caminho, bytes, version, criado }`                              |
| `criar_pasta`      | `caminho`                                       | `{ caminho, criado }`                                              |
| `mover_no`         | `origem`, `destino`                             | `{ origem, destino }`                                              |
| `excluir_no`       | `caminho`, `recursivo?`                         | `{ caminho, removidos }`                                           |

```jsonc
// criar um arquivo HTML (cria as pastas do caminho)
{
  "method": "tools/call",
  "params": {
    "name": "escrever_arquivo",
    "arguments": { "caminho": "/campanha-x/landing.html", "conteudo": "<!doctype html>…" },
  },
}
// → { "caminho": "/campanha-x/landing.html", "bytes": 1234, "version": 1, "criado": true }
```

- **Auto-save:** `gerar_imagem`/`gerar_locucao`/`gerar_post`/`gerar_carrossel` já salvam a peça no
  workspace (pasta `/pecas/…`) automaticamente. Passe um `caminho` opcional na chamada de geração
  para salvar onde preferir (ex.: `gerar_carrossel(..., "caminho": "/campanha-x")`).
- **Sobrescrever:** `escrever_arquivo` recusa um arquivo que já existe a não ser que você mande
  `sobrescrever: true`. Erros estruturados: `WORKSPACE_EXISTS`, `WORKSPACE_NOT_FOUND`,
  `WORKSPACE_QUOTA_EXCEEDED`, `WORKSPACE_PATH_INVALID`, `WORKSPACE_FOLDER_NOT_EMPTY`.
- **Motion no claude.ai web:** monte o roteiro e os blocos e **escreva o projeto no workspace**; o
  usuário baixa pelo painel e faz o render do MP4 no PC dele (o render de vídeo não roda no servidor).

---

## 🔗 `gerar_link_unico` — recuperar/renovar o link compartilhável

Gera (ou renova) o **link público** de uma peça **já criada** —
`storage.ritmova.app/c/:token` (visualizador + ZIP) — re-espelhando os bytes que já estão no
storage. **NÃO cobra** (não re-gera nada). Vale p/ **carrossel, imagem e post** (peça única vira
"carrossel de 1 item" e também fica pública). **Voz/motion não se aplicam.** Sem `pecaId`, usa a
peça elegível mais recente (ache os ids em `listar_pecas`).

| Campo     | Tipo    | Default | Observação                                                      |
| --------- | ------- | ------- | --------------------------------------------------------------- |
| `pecaId`  | string  | —       | id da peça (de `listar_pecas`); ausente ⇒ mais recente elegível |
| `renovar` | boolean | `false` | força um link novo (ex.: o token de 30 dias expirou)            |

```jsonc
{
  "jsonrpc": "2.0",
  "id": 20,
  "method": "tools/call",
  "params": { "name": "gerar_link_unico", "arguments": { "pecaId": "a1b2…" } },
}
// Retorno:
// result.content[0].text = "Link pronto: https://storage.ritmova.app/c/abc123"
// result.structuredContent = { compartilhar: "https://storage.ritmova.app/c/abc123",
//                              compartilharDownload: "https://storage.ritmova.app/carousel/…/download" }
```

Erros (`structuredContent.code`): `PECA_NAO_ENCONTRADA`, `PECA_INCOMPLETA`, `TIPO_NAO_SUPORTADO`
(voz/motion), `STORAGE_DESABILITADO`, `LINK_FALHOU` (falha ao enviar — tente de novo).

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

| code                | quando                                                                       |
| ------------------- | ---------------------------------------------------------------------------- |
| `HIGH_BLOCKED`      | pediu `qualidade: "high"` (exclusivo do Pro)                                 |
| `QUOTA_EXCEEDED`    | estourou posts/carrosséis/motion do mês, ou pediu locução no Free (Pro-only) |
| `SLIDES_EXCEEDED`   | slides acima do máximo do plano (`slides.length > slidesMax`)                |
| `PAUSED`            | geração do Free pausada (kill switch)                                        |
| `VOICE_NOT_FOUND`   | `voz` não está nas vozes salvas da conta — use `listar_vozes`/`salvar_voz`   |
| `VOICE_UNAVAILABLE` | a voz sumiu da conta entre a escolha e a geração — escolha outra             |

---

## Notas

- `gerar_imagem` e `gerar_locucao` são tools **atômicas** (1 imagem / 1 locução).
- `gerar_post` e `gerar_carrossel` **NÃO** reusam `gerar_imagem`: VOCÊ autora o HTML/CSS + os
  prompts; o servidor gera as imagens, injeta por `{{img:<id>}}`, renderiza e cobra **1× por
  peça** (nunca por slide). Já `gerar_motion` ENTREGA a receita e é VOCÊ (cliente) que compõe os
  blocos atômicos (`gerar_locucao`/`gerar_imagem`) e renderiza o vídeo localmente.
- O resultado volta como **URL** (o cliente baixa o arquivo); as chaves de API ficam no servidor.
- Toda peça gerada também é guardada no **storage da RITMOVA**, e o retorno traz `compartilhar` —
  uma **string** com o link público `storage.ritmova.app/c/:token` (carrossel, imagem e post); o ZIP
  vem em `compartilharDownload`. Para recuperar/renovar o link de uma peça já criada, use
  `gerar_link_unico`. É best-effort: se vier ausente, entregue as URLs normais mesmo assim.
