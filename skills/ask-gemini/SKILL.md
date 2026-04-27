---
name: ask-gemini
description: "Use ONLY when the user explicitly asks to delegate to Gemini, ask Gemini, run Gemini, or use the Gemini CLI. Also triggers on requests to analyze YouTube videos or audio files (capabilities Claude lacks). Never auto-invoke for general research, web searches, or coding tasks — those stay with Claude unless the user names Gemini."
---

# ask-gemini

Delegate a task to Google's Gemini CLI as a sub-agent. Use only when the user explicitly asks for it, or when the request requires a capability Claude lacks (YouTube video analysis, native audio file processing).

## Invocation

Always use this exact command shape:

```bash
GOOGLE_GENAI_USE_GCA=true gemini -y -p "<USER_PROMPT>

Cite sources with inline links for every factual claim. Briefly explain your reasoning."
```

- `GOOGLE_GENAI_USE_GCA=true` is required — Gemini CLI refuses to start without an auth method env var. This one tells it to use the user's personal Google account (subscription quota, not metered API billing).
- `-y` auto-approves tool use so the headless run doesn't hang on a confirmation prompt.
- The citation/reasoning suffix is appended to every prompt — the user explicitly required it. Without sources, don't accept the answer.

## Pass prompts verbatim

Do not rewrite, summarize, or "improve" the user's request before sending it. Gemini is an agent — trust it to interpret. The only addition is the source-citation suffix above.

## Output routing

After the call returns, strip the leading banner lines (`Loaded cached credentials.` and `Hook registry initialized...`) and count words in the remaining body:

- **≤ 500 words** — paste inline in the chat reply.
- **> 500 words** — save to `~/research/<topic-slug>-YYYY-MM-DD.md`, then reply with a one-paragraph summary plus the absolute file path. Create `~/research/` if it doesn't exist. Slug the topic from the user's request (lowercase, hyphens, drop punctuation).

If the user explicitly asks for a file path or inline output, honor that instead.

## When NOT to use Gemini

- General coding, refactoring, debugging in the current conversation — use your own tools.
- Web search the user didn't name Gemini for — use `WebSearch` / `WebFetch`.
- Anything where you'd otherwise reach for a Claude-native tool. The trigger is *explicit*, not *eager*.

YouTube-link analysis and audio-file analysis are the only implicit triggers, because Claude literally cannot do them.

## Failure modes

- **`Please set an Auth method...`** — the env var was dropped. Re-run with `GOOGLE_GENAI_USE_GCA=true` prefixed. Do not fall back to `GEMINI_API_KEY`; that would charge metered API billing instead of using the subscription.
- **Hang with no output** — the `-y` flag was likely missed; the CLI is waiting for tool approval. Kill it and re-run with `-y`.
- **Quota exceeded / rate limit** — surface the raw error to the user. Do not retry silently.
- **Truncated or garbled output** — show what came back and flag the truncation. Do not pretend it succeeded.

## Verification

A quick smoke test the skill is wired correctly:

```bash
GOOGLE_GENAI_USE_GCA=true gemini -y -p "Reply with the single word OK." 2>&1 | tail -1
```

Should print `OK`.
