ï»¿# Kimi Code

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="48" />
  <p><strong>Arckhé AI</strong></p>
</div>


> **Uma skill. Um terminal. Um terêÂ§o do custo.**

Skill do Claude Code que abre uma **nova janela de terminal** rodando o **harness completo do Claude Code** com o modelo **Kimi K3 (Moonshot AI)** via **OpenRouter** ê¢â¬â sem tocar na sua sessêÂ£o atual nem nas configuraêÂ§êÂµes globais.

Diga **"abre o kimi"** e pronto.

## ê°Å¸ââ Guia de uso

Guia completo (landing + passo a passo): **https://linnoff88-dev.github.io/arckhe-/guia/**

---

## O que ela faz

- Abre uma nova janela de terminal com o Claude Code, com todas as chamadas de API roteadas para `moonshotai/kimi-k3` no OpenRouter em vez da Anthropic
- **Harness completo** ê¢â¬â tools, skills, ediêÂ§êÂ£o de arquivos e bash funcionam exatamente igual
- **Escopada êÂ  sessêÂ£o** ê¢â¬â nunca escreve em `~/.claude/settings.json`. Fechou a janela, voltou tudo ao normal
- **Segura pra gravaêÂ§êÂ£o de tela** ê¢â¬â sua API key nunca aparece no scrollback do terminal
- Cerca de **1/3 do custo** do modelo de fronteira em builds reais (o Kimi êÂ© 2-3x mais lento em trabalho pesado ê¢â¬â perfeito pra build em segundo plano)

---

## InstalaêÂ§êÂ£o (2 minutos)

```bash
mkdir -p ~/.claude/skills
cp -r skills/kimi-code ~/.claude/skills/
```

Reinicie o Claude Code e diga **"abre o kimi"**.

Na primeira execuêÂ§êÂ£o ela pede sua chave da OpenRouter (crie em <https://openrouter.ai/settings/keys> e coloque alguns dêÂ³lares de crêÂ©dito) e salva em `~/.config/openrouter.env` com `chmod 600`, pra vocêÂª nunca mais precisar colar.

---

## Como usar

| VocêÂª diz | O que acontece |
|---|---|
| "abre o kimi" / "open kimmy" | Nova janela de terminal com o Claude Code em Kimi K3 |
| `/kimi-code here` | SêÂ³ imprime o one-liner pra vocêÂª rodar no shell atual |
| `/kimi-code model qwen/qwen3-coder` | Mesmo truque, com qualquer modelo do OpenRouter |
| `/kimi-code status` | Mostra qual chave + modelo seriam usados, sem lanêÂ§ar nada |

VocêÂª sabe que funcionou quando o banner de boas-vindas do Claude Code mostrar **`moonshotai/kimi-k3` êÂ· API Usage Billing**.

---

## Como funciona

Quatro variêÂ¡veis de ambiente na frente do `claude`:

| VariêÂ¡vel | Valor | Por quêÂª |
|---|---|---|
| `ANTHROPIC_BASE_URL` | `https://openrouter.ai/api` | Redireciona as chamadas pro OpenRouter |
| `ANTHROPIC_AUTH_TOKEN` | a chave do OpenRouter | Autentica no OpenRouter |
| `ANTHROPIC_API_KEY` | `""` (vazio) | Impede que a chave real da Anthropic seja enviada |
| `ANTHROPIC_MODEL` | `moonshotai/kimi-k3` | Qual modelo serve a sessêÂ£o |

Mais `ANTHROPIC_SMALL_FAST_MODEL` com o mesmo slug, pra que as chamadas de fundo (classe haiku) nêÂ£o voltem silenciosamente pra Anthropic.

O one-liner equivalente, se preferir rodar na mêÂ£o:

```bash
source ~/.config/openrouter.env && \
  ANTHROPIC_BASE_URL='https://openrouter.ai/api' \
  ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY" \
  ANTHROPIC_API_KEY='' \
  ANTHROPIC_MODEL='moonshotai/kimi-k3' \
  ANTHROPIC_SMALL_FAST_MODEL='moonshotai/kimi-k3' \
  claude
```

A chave sêÂ³ êÂ© resolvida **depois** do `source`, jêÂ¡ dentro da nova janela ê¢â¬â por isso o valor literal nunca entra na linha de comando.

---

## Plataformas

- **Linux** ê¢â¬â a skill detecta o emulador disponêÂ­vel (`gnome-terminal`, `konsole`, `xfce4-terminal`, `kitty`, `alacritty`, `x-terminal-emulator`, `xterm`) e lanêÂ§a com a sintaxe certa. Sem emulador (headless/WSL sem X/container), ela cai no modo `here`.
- **macOS** ê¢â¬â lanêÂ§a via AppleScript no Terminal.app.

---

## Quando usar Kimi vs Claude

- **Claude (assinatura)** ê¢â¬â sessêÂµes interativas, trabalho de alta alavancagem, tudo onde velocidade importa.
- **Kimi K3 (esta skill)** ê¢â¬â builds grandes de design, trabalho em lote, qualquer coisa que dêÂ¡ pra deixar rodando em segundo plano. Benchmark real: um build completo de slide deck custou **$10,70 no Kimi vs $35 no modelo de fronteira**, com qualidade equivalente.

---

## SeguranêÂ§a

- A skill **nunca** modifica `~/.claude/settings.json`.
- A chave completa **nunca** êÂ© impressa no chat ê¢â¬â sêÂ³ os 4 êÂºltimos caracteres.
- `~/.config/openrouter.env` êÂ© criado com `chmod 600`.

---

## Estrutura

```
kimicodecc/
ê¢âÅê¢ââ¬ê¢ââ¬ README.md
ê¢âÅê¢ââ¬ê¢ââ¬ doc/
ê¢ââ   ê¢ââê¢ââ¬ê¢ââ¬ kimi-code-openrouter.md   # material de origem
ê¢ââê¢ââ¬ê¢ââ¬ skills/
    ê¢ââê¢ââ¬ê¢ââ¬ kimi-code/
        ê¢ââê¢ââ¬ê¢ââ¬ SKILL.md              # a skill
```

---

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="32" />
  <p><strong>Arckhé AI</strong></p>
</div>