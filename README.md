# claude-skill-ask-gemini

A [Claude Code](https://claude.com/claude-code) skill that lets Claude delegate a task to Google's [Gemini CLI](https://github.com/google-gemini/gemini-cli) as a sub-agent.

## What it does

When you say "ask Gemini to..." or "use Gemini for...", Claude shells out to:

```bash
GOOGLE_GENAI_USE_GCA=true gemini -y -p "<your prompt>

Cite sources with inline links for every factual claim. Briefly explain your reasoning."
```

Claude pastes the answer back inline (≤500 words) or saves it to `~/research/<topic>-YYYY-MM-DD.md` (longer) and reports the path.

Also fires automatically when you ask Claude to analyze a **YouTube video URL** or **audio file** — capabilities Gemini has that Claude doesn't.

## Why

Gemini's CLI uses your personal Google account (subscription quota, not metered API billing) and brings two hard capability gaps that Claude can't fill: native YouTube video understanding and native audio file processing. It also offloads bulk work to a different model and a different rate-limit pool.

Full design rationale: [RATIONALE.md](RATIONALE.md).

## Install

This is a Claude Code **plugin** (standard distribution format). Install one of three ways:

**Option A — as a plugin (recommended).** Clone into Claude Code's plugin directory:

```bash
git clone https://github.com/w1ne/claude-skill-ask-gemini.git ~/.claude/plugins/ask-gemini
```

The `.claude-plugin/plugin.json` manifest registers it; Claude Code picks up everything under `skills/`.

**Option B — as a standalone skill.** If you only want the skill (no plugin wrapper):

```bash
git clone https://github.com/w1ne/claude-skill-ask-gemini.git /tmp/ag && \
  cp -r /tmp/ag/skills/ask-gemini ~/.claude/skills/ask-gemini && rm -rf /tmp/ag
```

**Option C — symlink for development.** Edit in place:

```bash
git clone https://github.com/w1ne/claude-skill-ask-gemini.git ~/Projects/claude-skill-ask-gemini
ln -s ~/Projects/claude-skill-ask-gemini/skills/ask-gemini ~/.claude/skills/ask-gemini
```

## Prereqs

1. **Gemini CLI** installed: `npm install -g @google/gemini-cli` (or via your package manager).
2. **Authenticated** with your Google account. First run:
   ```bash
   GOOGLE_GENAI_USE_GCA=true gemini
   ```
   Pick "Login with Google" in the auth picker, complete browser OAuth, then `/quit`. Credentials cache under `~/.gemini/`.

## Usage

Just say it in Claude Code:

> "Ask Gemini what the current state of Mali-G78 Vulkan compute is."

> "Use Gemini to summarize this YouTube video: https://youtu.be/..."

> "Run Gemini for a deep dive on the EU CRA hardware certification timeline. Cite sources."

Claude will only fire the skill on those explicit triggers (or for YouTube/audio links). Plain "research X" requests stay with Claude.

## License

MIT — see [LICENSE](LICENSE).
