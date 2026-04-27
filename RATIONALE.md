# Design Rationale

This skill is intentionally minimal. Every design decision was made deliberately during a brainstorming session; this document captures the *why* so future edits don't drift.

## Why a Gemini-delegation skill at all

Two motivations:

1. **Capability gaps Claude has.** Gemini natively understands YouTube video URLs (transcript + frames) and audio files. Claude can't. Anything else (web search, long context, coding) is roughly a wash, but those two are hard gaps.
2. **General sub-agent / token arbitrage.** Gemini CLI authenticated with a personal Google account uses subscription quota, not metered API billing. Bulk work that doesn't need Claude's specific reasoning can run there for free.

## Why CLI only — no browser-driven Deep Research

The Gemini "Deep Research" web product (gemini.google.com) is a separate UX that runs a multi-step planner producing 10–30 page reports. It's not exposed via the CLI or any public API.

Two routes were on the table:
- **CLI-only** — `gemini -p "..."` with prompt engineering to approximate Deep Research.
- **CLI + browser automation** — drive `gemini.google.com` with `chrome-devtools` MCP for the real product.

CLI-only won. Reasons:
- One implementation path, one failure mode.
- No browser fragility, no session-cookie management, no UI changes breaking the skill.
- Quality of CLI agentic web research with a good prompt is good enough for ~95% of cases.
- Real Deep Research is still available manually in the browser when truly needed.

If experience shows CLI depth is insufficient for actual long reports, the browser route can be added as v2 with an opt-in flag.

## Why explicit-only trigger

Skills auto-invoke based on their `description` field. Three options were considered:

- **Eager auto-fire** on any research-shaped request. Burns subscription quota on stuff Claude could do directly, trains over-delegation.
- **Hybrid** — fire on hard triggers, offer on borderline. Less risky but the "offer" surface adds friction and noise.
- **Explicit only** — never auto, only when the user names Gemini. Maximum control, zero surprise.

Explicit-only chosen. The user wants full control over when Gemini runs. The skill's *only* implicit trigger is YouTube / audio analysis, because those are pure capability gaps — Claude literally can't do them, so reaching for Gemini is the only path that works.

## Why pass prompts verbatim (no templates)

Gemini is an agent. It interprets requests and decides what tools to use. Wrapping the user's prompt in a research template would:

- Add latent assumptions the user didn't ask for.
- Bias Gemini toward one shape of output.
- Require maintenance as Gemini's tool list evolves.

The only fixed addition is the citation/reasoning suffix: "Cite sources with inline links for every factual claim. Briefly explain your reasoning." That's a hard requirement (don't accept un-sourced answers), not a templating opinion.

## Why smart-default output routing

Three options:
- **Inline only** — clogs context on long reports.
- **File only** — friction for short Q&A.
- **Smart default by length** — inline ≤ 500 words, file > 500 words. No friction for short asks, no context bloat for long ones.

Smart default chosen. The threshold is arbitrary but defensible: long enough to justify a file, short enough to read inline.

## Why `~/research/<topic>-YYYY-MM-DD.md`

- Predictable path, easy to grep and re-find.
- Date suffix means re-asking the same question doesn't overwrite prior runs (useful for tracking how answers change over time).
- Slug normalization keeps filenames sane.

## Why `-y` (yolo mode) by default

Headless `gemini -p` will hang waiting for tool-approval prompts the moment Gemini decides to use a tool. Without `-y` (or `--approval-mode auto_edit` / `yolo`), every research-shaped task would deadlock. `-y` is the simplest reliable flag.

This is technically less safe than `--approval-mode plan` (read-only), but the use cases (research, video summarization, second opinion) are read-heavy and the user is invoking Gemini deliberately each time. If a future use case needs Gemini to edit code in the user's repo, that's where extra caution would be worth adding — not here.

## Why a pure skill with no helper script

Three packaging options:
- **Skill only** — bash command embedded in SKILL.md.
- **Skill + wrapper script** — `~/.claude/scripts/ask-gemini.sh`.
- **Skill + script + slash command** — `/gemini` in Claude Code.

Skill-only chosen. The entire behavior fits in one ~80-line markdown file you can read in 30 seconds. No second file to maintain, no indirection. If the same boilerplate appears three times in real use, that's the moment to extract a wrapper — not before.
