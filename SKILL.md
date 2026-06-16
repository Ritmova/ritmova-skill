---
name: ritmova
description: >
  A RITMOVA é um estúdio criativo com IA (via MCP): gera imagem, locução/voz e motion/vídeo.
  Use esta skill quando o usuário perguntar o que é a RITMOVA, pedir para GERAR imagem,
  locução/voz, post, carrossel ou motion/vídeo, ou perguntar sobre tokens, saldo, custo,
  planos (Free/Pro) e compra de tokens. Documenta as rotas MCP (gerar_imagem, gerar_locucao,
  gerar_carrossel, gerar_post, gerar_motion, obter_trilha, obter_efeito) e o erro de upsell de saldo.
metadata:
  tags: ritmova, mcp, gerar_imagem, gerar_locucao, gerar_carrossel, gerar_post, obter_trilha, obter_efeito, sfx, trilha, carrossel, post, tokens, saldo, motion, geracao
---

# RITMOVA

A **RITMOVA** é um **estúdio criativo com IA**: o usuário pede conteúdo de marketing em
linguagem natural e o Claude **chama as tools da RITMOVA** para gerar.

## Etapa 0 — Auto-prompt obrigatório antes de qualquer construção

Antes de produzir qualquer coisa (código, vídeo, arte, documento, campanha, etc.), você não começa a construir direto. Primeiro você escreve, para você mesmo, um prompt profissional com o plano completo do trabalho. Só depois de escrito e revisado esse auto-prompt você inicia a execução, seguindo-o.

Faça nesta ordem:

1. Entenda a tarefa e defina os critérios de sucesso. Diga, em uma ou duas frases, qual é o resultado, para quem é, e o que significa "pronto" e "profissional" neste caso específico.
2. Inspecione o MCP da Ritmova antes de planejar. Liste e leia as ferramentas e capacidades disponíveis no MCP da Ritmova para saber o que realmente pode usar. Não presuma o que existe — verifique.
3. Pesquise como fazer o melhor trabalho possível. Reúna o que for necessário (requisitos, referências, restrições, boas práticas do formato em questão) para entregar algo de nível profissional na tarefa designada.
4. Escreva o auto-prompt (o plano completo) aplicando boas práticas de criação de prompt:
   - Papel e objetivo claros: defina o papel que você assume e o que deve ser entregue.
   - Contexto e motivação: explique o porquê e para quem — isso melhora o foco do resultado.
   - Critérios de sucesso explícitos e definição de "pronto".
   - Passos sequenciais numerados, na ordem de execução.
   - Quais ferramentas do MCP da Ritmova serão usadas e em que momento.
   - Formato e estrutura da saída desejada — descreva o que fazer (e não apenas o que evitar).
   - Exemplos quando ajudarem (few-shot), delimitados em tags <exemplo>…</exemplo>.
   - Use tags XML para separar instruções, contexto e dados quando o prompt misturar esses elementos.
   - Termine com uma etapa de autoverificação: revise a saída contra os critérios de sucesso antes de considerar o trabalho concluído.
5. Investigue antes de afirmar. Não faça suposições sobre arquivos, dados ou ferramentas que você não abriu/verificou; baseie o plano no que confirmou.
6. Só então construa, seguindo o auto-prompt. Mantenha o escopo no que foi pedido, sem complexidade desnecessária, e revise no final.

**Se faltar informação, pergunte antes de construir.** Quando você não tiver o essencial para entregar algo profissional — cores e paleta, estética e estilo visual, tom de voz, público-alvo, formato e proporção, marca/logo, referências e duração (no caso de motion) — faça perguntas objetivas ao usuário em vez de presumir. Prefira poucas perguntas e específicas, oferecendo opções quando ajudar, e só avance para a construção quando o essencial estiver definido.

---

### Referência — boas práticas de prompt

Base: fontes primárias (Anthropic / OpenAI / Google) + o repositório **[prompt-blueprint](https://github.com/thibaultyou/prompt-blueprint)**. Pontos que reforçam esta Etapa 0:

- **Planejar antes de executar tem efeito medido** — separar o plano da execução reduz erro (Plan-and-Solve, ACL 2023; ferramenta `think` da Anthropic: +54% rel. em tau-bench). É a razão de esta etapa existir.
- **Abra o auto-prompt com papel + objetivo + critérios de sucesso mensuráveis** e delimite as partes com tags (`<plano>`, `<contexto>`, `<tarefa>`, `<exemplo>`).
- **Enquadre no positivo e calibre a intensidade** — diga o que fazer; evite linguagem agressiva ("CRÍTICO / VOCÊ DEVE"), que nos modelos recentes tende a causar overtriggering.

## Ferramentas

**Principais** (porta de entrada — **entregam a RECEITA**):

| Tool                  | Como usar                                                                                                                                                                                                                                                                                                         |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`gerar_carrossel`** | **Chame SEM `slides`** → recebe a receita; autore o HTML de cada slide; **chame de novo com `slides`** → renderiza+cobra 1×                                                                                                                                                                                       |
| **`gerar_post`**      | **Chame SEM `html`** → recebe a receita; autore o HTML; **chame de novo com `html`** → renderiza+cobra 1×                                                                                                                                                                                                         |
| **`gerar_motion`**    | **Chame** → recebe a receita do vídeo (grátis). Você **monta e RENDERIZA o MP4 localmente** (Remotion), usando `obter_trilha` (trilha OBRIGATÓRIA — **cobra o preço-base do motion: Pro 3 tokens / Free a amostra única**, 1× por trabalho) + `gerar_locucao` (voz) + `gerar_imagem` (assets), que cobram à parte |

**Sub-ferramentas** (aprimoram as principais; também funcionam sozinhas):

| Tool                | Para que serve                                                                                               |
| ------------------- | ------------------------------------------------------------------------------------------------------------ |
| **`gerar_imagem`**  | gera 1 imagem (GPT Image). As principais geram por dentro via `{{img:<id>}}`; use sozinha p/ um asset avulso |
| **`gerar_locucao`** | gera 1 locução / voz (Pro)                                                                                   |
| **`obter_trilha`**  | trilha sonora oficial do motion (pack licenciado; URL temporária) — cobra o preço-base do motion             |
| **`obter_efeito`**  | efeito sonoro (SFX) do pack curado por `tags` (whoosh, transição…) — **grátis**, não cobra token nem cota    |

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

> **Motion (vídeo):** chame **`gerar_motion`** → ele devolve a **RECEITA** de motion (grátis).
> Diferente do carrossel/post, **você** monta o projeto e **renderiza o MP4 na sua máquina**
> (Remotion): gere a narração com `gerar_locucao` (o **áudio é a fonte do tempo**), obtenha a
> **trilha OBRIGATÓRIA** com `obter_trilha` (é onde o motion é cobrado; as tags disponíveis estão
> na própria descrição da tool) e os assets com `gerar_imagem`. Em clientes sem execução local
> (ex.: claude.ai web) dá p/ montar o roteiro e os blocos, mas o render final precisa de um
> ambiente local (Claude Code/desktop).

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
