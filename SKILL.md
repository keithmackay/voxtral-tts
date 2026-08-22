---
name: voxtral-tts
description: >
  Mistral Voxtral TTS — convert text to speech using Mistral's Voxtral model
  ($0.016/1,000 chars). Use when asked to speak a response aloud, send a voice
  note to Discord or Telegram, or when the --voice flag is set on any coding
  agent response. Also use when Keith says "read that to me", "voice note",
  or "send as audio".
metadata:
  {
    "openclaw":
      {
        "emoji": "🔊",
        "requires": { "env": ["MISTRAL_API_KEY"] },
      },
  }
---

# voxtral-tts

Mistral Voxtral TTS via `scripts/voxtral`.

**Pricing:** $0.016 / 1,000 characters (~$0.016 for a 1,000-char response)
**Model:** `voxtral` · **Latency:** ~70ms for 500 chars
**Requires:** `MISTRAL_API_KEY`

## Flags

### `--help`

If the user invokes this skill with a `--help` flag (e.g. `/voxtral-tts --help`), do not run the TTS flow. Instead, read and display the contents of `help.md` (in this skill's folder) verbatim, then stop.

### `--version`

If the user invokes this skill with a `--version` flag (e.g. `/voxtral-tts --version`), do not run the workflow. Instead:

1. Read the installed version from this skill's own manifest: `.claude-plugin/plugin.json` if present, else `.codex-plugin/plugin.json`, else `gemini-extension.json` — whichever exists for this platform install. If none exist (a bare Claude Code skill with only SKILL.md), read the topmost version heading in `CHANGELOG.md` instead.
2. Print: `voxtral-tts v<installed-version>`
3. Best-effort update check — determine this skill's GitHub source repo:
   a. If `.git` exists here and `git remote get-url origin` resolves to a `github.com` URL, use that `owner/repo`.
   b. Otherwise, search this skill's own `README.md` for the first `https://github.com/<owner>/<repo>` URL and use that.
   c. If neither yields a repo, or the `gh` CLI isn't installed/authenticated: stop here. Print nothing further — no status line, no error.
4. If a repo was found: run `gh api repos/<owner>/<repo>/releases/latest -q .tag_name` (strip a leading `v`). Compare to the installed version:
   - Equal → append: `Status: up to date`
   - Installed is older → append: `Status: newer version available (v<latest>). To update: if you installed this via a Claude Code marketplace, run /plugin marketplace update <marketplace-name> then reinstall; otherwise, git pull in your install directory if it's a git checkout, or re-copy from https://github.com/<owner>/<repo> per this README's Installation section.`
   - Installed is newer → append: `Status: ahead of latest release (development checkout)`
   - If the API call fails for any reason (network, auth, rate limit, malformed tag): print nothing further — no status line, no error shown to the user.
5. Stop — do not proceed to run the skill's actual workflow.

### `--dry-run`

If the user invokes this skill with a `--dry-run` flag (e.g. `/voxtral-tts --dry-run "text"`), or asks to "preview" the cost/output before speaking something: run the normal analysis — validate the text/voice, compute the character count and estimated cost per the Cost Estimate table below, and determine the output file path — but **stop before calling the Mistral API, writing the audio file, or sending anything to Discord/Telegram**. Pass `--dry-run` straight through to `scripts/voxtral`, which implements this itself:

```bash
uv run scripts/voxtral --dry-run -v nick "Here's your summary."
```

This prints a report (voice, model, format, character count, estimated cost, and the output path — noting if it would be served from cache) and exits — no API call, no file write.

When `--dry-run` is combined with a request to send to Discord/Telegram (or with `--voice`), also report which channel/destination *would* receive the audio (channel ID / chat ID) without calling the `message` tool. Take no other action.

## Quick Start

```bash
# Generate and get file path (prints path to stdout)
uv run scripts/voxtral "Hello, Keith."

# Choose a voice
uv run scripts/voxtral -v nick "Here's your summary."

# Save to specific path
uv run scripts/voxtral -v margaret -o /tmp/reply.mp3 "Your briefing is ready."

# Strip markdown from AI agent output before speaking
uv run scripts/voxtral --strip-md -v nick "**Summary:** task complete."

# Pipe from stdin, truncate if too long
echo "$LONG_OUTPUT" | uv run scripts/voxtral --strip-md --truncate -v nick

# Preview cost, output path, and voice without generating, writing, or sending anything
uv run scripts/voxtral --dry-run -v nick "Here's your summary."
```

## Voices

| Name | Style |
|------|-------|
| margaret | British female |
| nick | American male |
| angele | French female |
| sanchit | Hindi-accented English male |
| gustavo | Portuguese/Spanish male |
| khyathi | Indian English female |
| yassir | Arabic-accented English male |
| patrick | American male (casual) |

Default: `margaret`. For Keith, prefer `nick` or `margaret`.

## Send Voice Note to Discord

```bash
# 1. Generate audio
FILE=$(uv run scripts/voxtral -v nick "Response text here")

# 2. Send as voice note via message tool
message channel:discord action:send to:"channel:<channelId>" media:"file://$FILE"
```

## Send Voice Note to Telegram

```bash
FILE=$(uv run scripts/voxtral -v nick "Response text here")
message channel:telegram action:send to:"<chatId>" media:"file://$FILE"
```

## --voice Flag Pattern (Coding Agents)

`--voice` is handled by **Mac (nanobot)**. When the user appends `--voice` to a request, or says "send as voice note":

1. Get the text response (from agent output or your own reply)
2. Use `--strip-md` to clean markdown; use `--truncate` for long content
3. Generate audio: `FILE=$(uv run scripts/voxtral --strip-md --truncate -v nick "...")`
4. Send `media:"file://$FILE"` to the originating channel

If `--dry-run` is also requested (e.g. `--voice --dry-run`), run step 3 with `--dry-run` appended, report the cost/output preview and the destination channel that would receive it, and stop — skip steps 3's real generation and step 4 entirely.

## Cost Estimate

| Length | ~Chars | Cost |
|--------|--------|------|
| Short reply | 200 | $0.003 |
| Briefing | 2,000 | $0.032 |
| Full article | 10,000 | $0.16 |

## Notes

- If `MISTRAL_API_KEY` is not set, the script exits with an error.
- Model name may need updating — check `GET https://api.mistral.ai/v1/models` if you get a 404.
- Script path (relative to skill root): `scripts/voxtral`
- The script prints the output file path to stdout — capture with `$(...)`
