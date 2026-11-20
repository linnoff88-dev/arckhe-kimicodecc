# Kimi Code

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="48" />
  <p><strong>Arckhé AI</strong></p>
</div>


> **Uma skill. Um terminal. Um terÃ§o do custo.**

Skill do Claude Code que abre uma **nova janela de terminal** rodando o **harness completo do Claude Code** com o modelo **Kimi K3 (Moonshot AI)** via **OpenRouter** â€” sem tocar na sua sessÃ£o atual nem nas configuraÃ§Ãµes globais.

Diga **"abre o kimi"** e pronto.

## ðŸ“– Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/kimicodecc/guia/**

---

## O que ela faz

- Abre uma nova janela de terminal com o Claude Code, com todas as chamadas de API roteadas para `moonshotai/kimi-k3` no OpenRouter em vez da Anthropic
- **Harness completo** â€” tools, skills, ediÃ§Ã£o de arquivos e bash funcionam exatamente igual
- **Escopada Ã  sessÃ£o** â€” nunca escreve em `~/.claude/settings.json`. Fechou a janela, voltou tudo ao normal
- **Segura pra gravaÃ§Ã£o de tela** â€” sua API key nunca aparece no scrollback do terminal
- Cerca de **1/3 do custo** do modelo de fronteira em builds reais (o Kimi Ã© 2-3x mais lento em trabalho pesado â€” perfeito pra build em segundo plano)

---

## InstalaÃ§Ã£o (2 minutos)

```bash
mkdir -p ~/.claude/skills
cp -r skills/kimi-code ~/.claude/skills/
```

Reinicie o Claude Code e diga **"abre o kimi"**.

Na primeira execuÃ§Ã£o ela pede sua chave da OpenRouter (crie em <https://openrouter.ai/settings/keys> e coloque alguns dÃ³lares de crÃ©dito) e salva em `~/.config/openrouter.env` com `chmod 600`, pra vocÃª nunca mais precisar colar.

---

## Como usar

| VocÃª diz | O que acontece |
|---|---|
| "abre o kimi" / "open kimmy" | Nova janela de terminal com o Claude Code em Kimi K3 |
| `/kimi-code here` | SÃ³ imprime o one-liner pra vocÃª rodar no shell atual |
| `/kimi-code model qwen/qwen3-coder` | Mesmo truque, com qualquer modelo do OpenRouter |
| `/kimi-code status` | Mostra qual chave + modelo seriam usados, sem lanÃ§ar nada |

VocÃª sabe que funcionou quando o banner de boas-vindas do Claude Code mostrar **`moonshotai/kimi-k3` Â· API Usage Billing**.

---

## Como funciona

Quatro variÃ¡veis de ambiente na frente do `claude`:

| VariÃ¡vel | Valor | Por quÃª |
|---|---|---|
| `ANTHROPIC_BASE_URL` | `https://openrouter.ai/api` | Redireciona as chamadas pro OpenRouter |
| `ANTHROPIC_AUTH_TOKEN` | a chave do OpenRouter | Autentica no OpenRouter |
| `ANTHROPIC_API_KEY` | `""` (vazio) | Impede que a chave real da Anthropic seja enviada |
| `ANTHROPIC_MODEL` | `moonshotai/kimi-k3` | Qual modelo serve a sessÃ£o |

Mais `ANTHROPIC_SMALL_FAST_MODEL` com o mesmo slug, pra que as chamadas de fundo (classe haiku) nÃ£o voltem silenciosamente pra Anthropic.

O one-liner equivalente, se preferir rodar na mÃ£o:

```bash
source ~/.config/openrouter.env && \
  ANTHROPIC_BASE_URL='https://openrouter.ai/api' \
  ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY" \
  ANTHROPIC_API_KEY='' \
  ANTHROPIC_MODEL='moonshotai/kimi-k3' \
  ANTHROPIC_SMALL_FAST_MODEL='moonshotai/kimi-k3' \
  claude
```

A chave sÃ³ Ã© resolvida **depois** do `source`, jÃ¡ dentro da nova janela â€” por isso o valor literal nunca entra na linha de comando.

---

## Plataformas

- **Linux** â€” a skill detecta o emulador disponÃ­vel (`gnome-terminal`, `konsole`, `xfce4-terminal`, `kitty`, `alacritty`, `x-terminal-emulator`, `xterm`) e lanÃ§a com a sintaxe certa. Sem emulador (headless/WSL sem X/container), ela cai no modo `here`.
- **macOS** â€” lanÃ§a via AppleScript no Terminal.app.

---

## Quando usar Kimi vs Claude

- **Claude (assinatura)** â€” sessÃµes interativas, trabalho de alta alavancagem, tudo onde velocidade importa.
- **Kimi K3 (esta skill)** â€” builds grandes de design, trabalho em lote, qualquer coisa que dÃ¡ pra deixar rodando em segundo plano. Benchmark real: um build completo de slide deck custou **$10,70 no Kimi vs $35 no modelo de fronteira**, com qualidade equivalente.

---

## SeguranÃ§a

- A skill **nunca** modifica `~/.claude/settings.json`.
- A chave completa **nunca** Ã© impressa no chat â€” sÃ³ os 4 Ãºltimos caracteres.
- `~/.config/openrouter.env` Ã© criado com `chmod 600`.

---

## Estrutura

```
kimicodecc/
â”œâ”€â”€ README.md
â”œâ”€â”€ doc/
â”‚   â””â”€â”€ kimi-code-openrouter.md   # material de origem
â””â”€â”€ skills/
    â””â”€â”€ kimi-code/
        â””â”€â”€ SKILL.md              # a skill
```

---

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="32" />
  <p><strong>Arckhé AI</strong></p>
</div>