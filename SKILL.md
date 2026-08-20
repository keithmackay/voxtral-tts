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
