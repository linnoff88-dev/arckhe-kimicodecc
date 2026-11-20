# Kimi Code — Run Claude Code on Kimi K3 (via OpenRouter)

> **One skill. One terminal. A third of the cost.**

> Drop this skill into Claude Code and say **"open kimmy"** — a new Terminal window opens running the **full Claude Code harness on Kimi K3** through OpenRouter. Your normal Claude sessions stay untouched.

---

# What it does

- Opens a **new Terminal window** running Claude Code, with every API call routed to `moonshotai/kimi-k3` via OpenRouter instead of Anthropic
- **Full harness** — tools, skills, file edits, bash all work exactly the same
- **Session-scoped** — it never touches `~/.claude/settings.json`. Close the window and you're back to normal everywhere
- **Screen-recording safe** — your API key never appears in the terminal scrollback
- Roughly **1/3 the cost** of Fable 5 on real builds (Kimi is 2-3x slower on heavy jobs — perfect for background builds)

---

# Install (2 minutes)

**Step 1** — Create the skill folder:

```bash
mkdir -p ~/.claude/skills/kimi-code
```

**Step 2** — Copy the entire skill from the block below and save it as `~/.claude/skills/kimi-code/SKILL.md`

**Step 3** — Restart Claude Code, then say **"open kimmy"** or **"open kimi"**. On first run it will ask for your OpenRouter API key (create one at https://openrouter.ai/settings/keys and add a few dollars of credit) and save it locally so you never paste it again.

---

# The skill — copy everything in this one block

Save as `~/.claude/skills/kimi-code/SKILL.md` (hover the block and hit the copy icon):

```markdown
---
name: kimi-code
description: Open a new Terminal window running Claude Code powered by Kimi K3 (Moonshot AI) via OpenRouter. Trigger when the user says "open kimi", "open kimmy", "kimi code", "claude code with kimi", "launch kimi terminal", "kimi k3", or "/kimi-code". Runs the full Claude Code harness on the cheap Kimi model without touching the current session or global settings.
---

# Kimi Code — Claude Code Powered by Kimi K3 via OpenRouter

Opens a new Terminal window running Claude Code where every API call is routed to
Moonshot AI's Kimi K3 through OpenRouter instead of Anthropic. You get the full
Claude Code harness (tools, skills, file edits, bash) at roughly a third of the price.

This skill does NOT modify ~/.claude/settings.json. The env overrides live only in
the new terminal window, so the current session and all other Claude Code sessions
stay on Anthropic.

## ARGUMENTS

- No argument — launch a new Kimi-powered Terminal window (default)
- `here` — print the one-liner so the user can run it in their current shell instead
- `model <slug>` — use a specific OpenRouter model slug instead of Kimi K3
- `status` — show which key/model would be used, without launching anything

## EXECUTION

### Step 0: Model

Default is `moonshotai/kimi-k3` — always. Do NOT ask the user to pick a model.
Only deviate if they explicitly passed `model <slug>`, in which case use that verbatim.

### Step 1: Get the OpenRouter API key

Check ~/.config/openrouter.env for OPENROUTER_API_KEY:

    grep "^OPENROUTER_API_KEY=" ~/.config/openrouter.env

If missing, ask the user:

> To run Kimi you need an OpenRouter API key.
> 1. Go to https://openrouter.ai/settings/keys
> 2. Create a key (add a few dollars of credit — Kimi K3 is paid but ~70% cheaper)
> 3. Paste the key here
> I'll save it to ~/.config/openrouter.env so you never have to do this again.

When they paste it, append it to the env file (create the file if needed, chmod 600 it):

    echo 'OPENROUTER_API_KEY=<pasted-key>' >> ~/.config/openrouter.env
    chmod 600 ~/.config/openrouter.env

NEVER echo the full key back in chat. When confirming, show only the last 4 characters.

### Step 2: Verify the slug is live

Quick sanity check that moonshotai/kimi-k3 (or the user's override) exists on OpenRouter:

    curl -s https://openrouter.ai/api/v1/models | grep -o '"moonshotai/kimi-k3"' | head -1

If it's gone (model deprecated), list what grep -o '"moonshotai/kimi[^"]*"' returns and
use the highest-numbered non-:free slug instead, telling the user about the swap. If the
curl fails entirely (offline), proceed with moonshotai/kimi-k3 anyway.

### Step 3: Launch the Kimi terminal

The whole trick is four environment variables in front of `claude`:

| Variable | Value | Why |
|---|---|---|
| ANTHROPIC_BASE_URL | https://openrouter.ai/api | Redirects Claude Code's calls to OpenRouter |
| ANTHROPIC_AUTH_TOKEN | the OpenRouter key | Auth against OpenRouter |
| ANTHROPIC_API_KEY | "" (empty) | Prevents the real Anthropic key from being sent |
| ANTHROPIC_MODEL | moonshotai/kimi-k3 | Which model serves the session |

Also set ANTHROPIC_SMALL_FAST_MODEL to the same Kimi slug so background/haiku-class
calls don't silently fall back to Anthropic.

Write a temp AppleScript file and run it — do NOT try to inline-escape this into
osascript -e (the nested quoting breaks). Substitute the target directory and model
slug, keeping $OPENROUTER_API_KEY as a literal so it expands inside the new window:

    cat > /tmp/kimi-launch.scpt <<'EOF'
    tell application "Terminal"
      activate
      do script "cd ~/Desktop && source ~/.config/openrouter.env && ANTHROPIC_BASE_URL='https://openrouter.ai/api' ANTHROPIC_AUTH_TOKEN=\"$OPENROUTER_API_KEY\" ANTHROPIC_API_KEY='' ANTHROPIC_MODEL='moonshotai/kimi-k3' ANTHROPIC_SMALL_FAST_MODEL='moonshotai/kimi-k3' claude"
    end tell
    EOF
    osascript /tmp/kimi-launch.scpt && rm /tmp/kimi-launch.scpt

Because the key is referenced as $OPENROUTER_API_KEY and only resolved after source
runs inside the new window, the literal key never appears in the command line or the
terminal scrollback — safe for screen recording.

Verify it booted: the osascript call returns something like "tab 1 of window id NNN".
Wait ~6 seconds, then read the window and confirm the banner shows the Kimi slug:

    osascript -e 'tell application "Terminal" to get contents of tab 1 of window id NNN' | head -15

`here` mode — don't launch anything. Print this block for the user to run themselves:

    source ~/.config/openrouter.env && ANTHROPIC_BASE_URL='https://openrouter.ai/api' ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY" ANTHROPIC_API_KEY='' ANTHROPIC_MODEL='moonshotai/kimi-k3' ANTHROPIC_SMALL_FAST_MODEL='moonshotai/kimi-k3' claude

### Step 4: Confirm

Tell the user:

> Kimi terminal is open. New Terminal window running the full Claude Code harness on
> moonshotai/kimi-k3 via OpenRouter (key ending ...XXXX).
> - The banner inside the new window should show the Kimi slug — that's how you know it's live.
> - Your current session and global settings are untouched; close that window and you're
>   back to normal everywhere.
> - Kimi is slower (roughly 2-3x on long builds) but ~1/3 the cost. Best for big design
>   builds you can leave running in the background.

## SAFETY RULES

1. NEVER modify ~/.claude/settings.json — this skill is session-scoped by design.
2. NEVER print the full OpenRouter key in chat or in the confirmation. Last 4 chars only.
3. When showing commands, reference $OPENROUTER_API_KEY via source rather than pasting
   the literal key.
4. If claude isn't on PATH in the new window, tell the user to run the command from a
   shell where claude works, or use `here` mode.

## WHY KIMI K3

- Design output on par with the frontier (edged the slide-deck and dashboard head-to-heads)
- ~30% of the frontier price per token; real-world builds land at ~1/3 the cost even
  though Kimi burns more tokens per task
- 1M-token context — survives huge Claude Code prompts
- Full tool-calling support, so the entire Claude Code harness works unmodified
- Trade-off: speed. Expect 2-3x longer wall-clock on heavy builds. Use it for high-volume
  or background work; keep your Claude subscription for interactive high-leverage sessions.
```

---

# How to use it

| Say this | What happens |
|---|---|
| "open kimmy" / "open kimi" | New Terminal window boots Claude Code on Kimi K3 |
| "/kimi-code here" | Prints the one-liner to run in your current shell |
| "/kimi-code model qwen/qwen3-coder" | Same trick, any OpenRouter model |
| "/kimi-code status" | Shows which key + model would be used |

You'll know it worked when the Claude Code welcome banner shows **`moonshotai/kimi-k3` · API Usage Billing**.

---

# When to use Kimi vs Claude

- **Fable 5 (subscription)** — interactive sessions, high-leverage work, anything where speed matters
- **Kimi K3 (this skill)** — big design builds, batch work, anything you can leave running in the background. Real-world benchmark: a full slide-deck build cost **$10.70 on Kimi vs $35 on Fable 5**, with output quality on par

---

