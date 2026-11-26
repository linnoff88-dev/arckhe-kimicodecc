---
name: kimi-code
description: Abre uma nova janela de terminal rodando o Claude Code com o modelo Kimi K3 (Moonshot AI) via OpenRouter. Acione quando o usuário disser "open kimi", "abre o kimi", "open kimmy", "kimi code", "claude code com kimi", "roda no kimi", "kimi k3" ou "/kimi-code". Roda o harness completo do Claude Code no modelo barato, sem tocar na sessêo atual nem nas configurações globais.
---

# Kimi Code — Claude Code rodando em Kimi K3 via OpenRouter

Abre uma nova janela de terminal rodando o Claude Code onde toda chamada de API é
roteada para o Kimi K3 da Moonshot AI através do OpenRouter, em vez da Anthropic.
Você mantém o harness completo (tools, skills, ediçêo de arquivos, bash) por cerca
de um terço do preço.

Esta skill **NêO** modifica `~/.claude/settings.json`. Os overrides de env vivem só
dentro da nova janela — a sessêo atual e todas as outras sessões continuam na Anthropic.

## ARGUMENTOS

- **sem argumento** — abre uma nova janela de terminal com Kimi (padrêo)
- `here` — só imprime o one-liner pro usuário rodar no shell atual
- `model <slug>` — usa um slug de modelo específico do OpenRouter em vez do Kimi K3
- `status` — mostra qual chave/modelo seria usado, sem lançar nada

## EXECUÇêO

### Passo 0: Modelo

O padrêo é `moonshotai/kimi-k3` — sempre. **Nêo** pergunte ao usuário qual modelo usar.
Só desvie se ele passar `model <slug>` explicitamente, e nesse caso use o slug literal.

### Passo 1: Obter a chave do OpenRouter

Procure `OPENROUTER_API_KEY` nesta ordem (primeiro que existir vence):

    grep -h "^OPENROUTER_API_KEY=" ~/.config/openrouter.env ~/projetos/openpcbot/.env ~/projetos/wifi/.env 2>/dev/null | head -1

Se nêo achar, peça ao usuário:

> Pra rodar o Kimi você precisa de uma chave da OpenRouter.
> 1. Vá em https://openrouter.ai/settings/keys
> 2. Crie uma chave (coloque alguns dólares de crédito — Kimi K3 é pago, mas ~70% mais barato)
> 3. Cole a chave aqui
> Eu salvo em ~/.config/openrouter.env pra você nunca mais precisar colar.

Quando ele colar, grave no arquivo de env (crie se nêo existir, com `chmod 600`):

    mkdir -p ~/.config
    printf 'OPENROUTER_API_KEY=%s\n' '<chave-colada>' >> ~/.config/openrouter.env
    chmod 600 ~/.config/openrouter.env

**NUNCA** repita a chave inteira no chat. Ao confirmar, mostre só os 4 últimos caracteres.

### Passo 2: Verificar se o slug está vivo

Sanity check rápido de que `moonshotai/kimi-k3` (ou o override) existe no OpenRouter:

    curl -s https://openrouter.ai/api/v1/models | grep -o '"moonshotai/kimi-k3"' | head -1

Se sumiu (modelo depreciado), liste o que `grep -o '"moonshotai/kimi[^"]*"'` retorna e
use o slug de maior número que **nêo** seja `:free`, avisando o usuário da troca. Se o
`curl` falhar de vez (offline), siga com `moonshotai/kimi-k3` mesmo assim.

### Passo 3: Lançar o terminal Kimi

O truque inteiro sêo quatro variáveis de ambiente na frente do `claude`:

| Variável | Valor | Por quê |
|---|---|---|
| `ANTHROPIC_BASE_URL` | `https://openrouter.ai/api` | Redireciona as chamadas do Claude Code pro OpenRouter |
| `ANTHROPIC_AUTH_TOKEN` | a chave do OpenRouter | Autentica no OpenRouter |
| `ANTHROPIC_API_KEY` | `""` (vazio) | Impede que a chave real da Anthropic seja enviada |
| `ANTHROPIC_MODEL` | `moonshotai/kimi-k3` | Qual modelo serve a sessêo |

Defina também `ANTHROPIC_SMALL_FAST_MODEL` com o mesmo slug, pra que as chamadas
de fundo (classe haiku) nêo caiam silenciosamente de volta na Anthropic.

**O comando (o mesmo em qualquer plataforma):**

    source ~/.config/openrouter.env && \
      ANTHROPIC_BASE_URL='https://openrouter.ai/api' \
      ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY" \
      ANTHROPIC_API_KEY='' \
      ANTHROPIC_MODEL='moonshotai/kimi-k3' \
      ANTHROPIC_SMALL_FAST_MODEL='moonshotai/kimi-k3' \
      claude

Como a chave é referenciada como `$OPENROUTER_API_KEY` e só é resolvida depois do
`source` rodar **dentro** da nova janela, o valor literal nunca aparece na linha de
comando nem no scrollback — seguro pra gravaçêo de tela.

**Linux — descubra o emulador de terminal disponível e use o primeiro que existir:**

    for t in gnome-terminal konsole xfce4-terminal kitty alacritty x-terminal-emulator xterm; do
      command -v "$t" && break
    done

Depois lance com a sintaxe do emulador encontrado (substitua `<DIR>` pelo diretório
de trabalho desejado — o projeto atual, por padrêo):

    # gnome-terminal
    gnome-terminal --working-directory=<DIR> -- bash -lc '<COMANDO>; exec bash'
    # konsole
    konsole --workdir <DIR> -e bash -lc '<COMANDO>; exec bash'
    # xfce4-terminal
    xfce4-terminal --working-directory=<DIR> -e "bash -lc '<COMANDO>; exec bash'"
    # kitty / alacritty / xterm
    kitty --directory <DIR> bash -lc '<COMANDO>; exec bash'
    alacritty --working-directory <DIR> -e bash -lc '<COMANDO>; exec bash'
    xterm -e bash -lc 'cd <DIR>; <COMANDO>; exec bash'

O `exec bash` no final mantém a janela aberta quando o Claude Code sair.

Se **nenhum** emulador for encontrado (servidor headless, WSL sem X, container),
nêo insista: caia direto no modo `here` e explique o motivo.

**macOS** — escreva um AppleScript temporário e rode; **nêo** tente escapar isso
inline em `osascript -e` (as aspas aninhadas quebram):

    cat > /tmp/kimi-launch.scpt <<'EOF'
    tell application "Terminal"
      activate
      do script "cd <DIR> && source ~/.config/openrouter.env && ANTHROPIC_BASE_URL='https://openrouter.ai/api' ANTHROPIC_AUTH_TOKEN=\"$OPENROUTER_API_KEY\" ANTHROPIC_API_KEY='' ANTHROPIC_MODEL='moonshotai/kimi-k3' ANTHROPIC_SMALL_FAST_MODEL='moonshotai/kimi-k3' claude"
    end tell
    EOF
    osascript /tmp/kimi-launch.scpt && rm /tmp/kimi-launch.scpt

**Modo `here`** — nêo lance nada. Só imprima o bloco do comando acima pro usuário
rodar ele mesmo.

**Modo `status`** — nêo lance nada. Mostre: de qual arquivo a chave veio, os 4
últimos caracteres dela, o slug do modelo que seria usado, e se o slug respondeu
na verificaçêo do Passo 2.

### Passo 4: Confirmar

Diga ao usuário:

> Terminal Kimi aberto. Nova janela rodando o harness completo do Claude Code em
> `moonshotai/kimi-k3` via OpenRouter (chave terminada em ...XXXX).
> - O banner dentro da nova janela deve mostrar o slug do Kimi — é assim que você sabe que pegou.
> - Sua sessêo atual e as configurações globais estêo intactas; feche aquela janela e tudo volta ao normal.
> - O Kimi é mais lento (~2-3x em builds longos), mas custa ~1/3. Ideal pra builds grandes que você deixa rodando em segundo plano.

## REGRAS DE SEGURANÇA

1. **NUNCA** modifique `~/.claude/settings.json` — esta skill é escopada à sessêo por design.
2. **NUNCA** imprima a chave completa do OpenRouter no chat nem na confirmaçêo. Só os 4 últimos caracteres.
3. Ao mostrar comandos, referencie `$OPENROUTER_API_KEY` via `source`, nunca cole a chave literal.
4. Se o `claude` nêo estiver no PATH da nova janela, oriente o usuário a rodar o comando
   de um shell onde o `claude` funciona, ou usar o modo `here`.

## POR QUE KIMI K3

- Qualidade de design no nível da fronteira (venceu os head-to-heads de slide deck e dashboard)
- ~30% do preço por token da fronteira; builds reais saem por ~1/3 do custo, mesmo o Kimi queimando mais tokens por tarefa
- Contexto de 1M tokens — aguenta os prompts gigantes do Claude Code
- Suporte completo a tool-calling, entêo o harness inteiro funciona sem modificaçêo
- Trade-off: velocidade. Espere 2-3x mais tempo de relógio em builds pesados. Use pra
  volume alto ou trabalho de fundo; guarde a assinatura Claude pras sessões interativas.

## QUANDO USAR KIMI VS CLAUDE

- **Claude (assinatura)** — sessões interativas, trabalho de alta alavancagem, tudo onde velocidade importa
- **Kimi K3 (esta skill)** — builds grandes de design, trabalho em lote, qualquer coisa que dá pra deixar rodando. Benchmark real: um build completo de slide deck custou **$10,70 no Kimi vs $35 no Claude**, com qualidade equivalente
