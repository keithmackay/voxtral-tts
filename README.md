# voxtral-tts

A Claude Code skill that converts text to speech using Mistral's Voxtral model, then delivers the audio as a voice note over Discord or Telegram (or hands back a local file path). Triggered by an explicit `--voice` flag or phrases like "read that to me."

## Highlights

- **Cheap** — $0.016 per 1,000 characters, roughly $0.003 for a short reply and $0.16 for a full 10,000-character article
- **Fast** — ~70ms latency for a 500-character response
- **8 voices** — margaret, nick, angele, sanchit, gustavo, khyathi, yassir, patrick, spanning several accents and genders; defaults to margaret
- **Markdown-aware** — `--strip-md` cleans agent output before speaking it, `--truncate` caps overly long input
- **Channel delivery built in** — pairs with the `message` tool to send generated audio straight to a Discord channel or Telegram chat

## Getting Started

### Prerequisites

- Python with [`uv`](https://docs.astral.sh/uv/) installed
- A `MISTRAL_API_KEY` environment variable (the script exits with an error if it's unset)

### Installation

#### Claude Code

```bash
cp -r /path/to/voxtral-tts/ ~/.claude/skills/voxtral-tts/
```

Or symlink:
```bash
ln -s /path/to/voxtral-tts/ ~/.claude/skills/voxtral-tts
```

Then invoke with: `/voxtral-tts`

#### Codex

Place the plugin directory where Codex can find it, then add an entry to your marketplace:

**`~/.agents/plugins/marketplace.json`** (create if absent):
```json
{
  "name": "personal",
  "interface": { "displayName": "Personal Plugins" },
  "plugins": [
    {
      "name": "voxtral-tts",
      "source": { "source": "local", "path": "/path/to/voxtral-tts/" },
      "policy": { "installation": "AVAILABLE", "authentication": "ON_INSTALL" },
      "category": "Productivity"
    }
  ]
}
```

#### Antigravity

**Global install** (all workspaces):
```bash
cp -r /path/to/voxtral-tts/ ~/.gemini/antigravity/skills/voxtral-tts/
```

**Workspace install** (current project only):
```bash
cp -r /path/to/voxtral-tts/ .agents/skills/voxtral-tts/
```

The source skill has Claude Code-specific metadata (`metadata.openclaw`), so use the `antigravity/SKILL.md` version instead of the root `SKILL.md`.

Skills are auto-discovered. You can also mention the skill by name to force activation.

#### Gemini CLI

Gemini CLI installs extensions directly from GitHub:

```bash
gemini extensions install https://github.com/keithmackay/voxtral-tts
```

To update:
```bash
gemini extensions update voxtral-tts
```

The skill is auto-discovered from `GEMINI.md` after installation.

## Compatibility

| Feature | Claude Code | Codex | Antigravity | Gemini CLI |
|---------|:-----------:|:-----:|:-----------:|:----------:|
| Core skill | ✅ | ✅ | ✅ | ✅ |
| `metadata.openclaw` (emoji, required env) | ✅ | ❌ | ❌ | ❌ |

Legend: ✅ Supported · ❌ Not supported

## References

- **Claude Code Skills:** https://code.claude.com/docs/en/skills
- **Claude Code Complete Guide (PDF):** https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf
- **Codex Plugins:** https://developers.openai.com/codex/plugins/build
- **Antigravity Skills:** https://antigravity.google/docs/skills
- **Gemini CLI Extensions:** https://github.com/google-gemini/gemini-cli/blob/main/docs/extension.md
- **Agent Skills open standard:** https://agentskills.io/home

## Usage

```bash
# Generate and get file path (prints path to stdout)
uv run scripts/voxtral "Hello, Keith."

# Choose a voice
uv run scripts/voxtral -v nick "Here's your summary."

# Save to a specific path
uv run scripts/voxtral -v margaret -o /tmp/reply.mp3 "Your briefing is ready."

# Strip markdown from AI agent output before speaking
uv run scripts/voxtral --strip-md -v nick "**Summary:** task complete."

# Pipe from stdin, truncate if too long
echo "$LONG_OUTPUT" | uv run scripts/voxtral --strip-md --truncate -v nick
```

Send the result as a voice note:

```bash
# Discord
FILE=$(uv run scripts/voxtral -v nick "Response text here")
message channel:discord action:send to:"channel:<channelId>" media:"file://$FILE"

# Telegram
FILE=$(uv run scripts/voxtral -v nick "Response text here")
message channel:telegram action:send to:"<chatId>" media:"file://$FILE"
```

When a coding agent sees a `--voice` flag or a request to "send as voice note," it strips markdown, truncates if needed, generates the audio, and sends it to the originating channel automatically.

```
/voxtral-tts --help    # print usage summary, take no other action
```

> [!NOTE]
> If Voxtral requests start returning 404s, the model name may have changed — check `GET https://api.mistral.ai/v1/models`.

## Contributing

Pull requests are welcome — fork the repo, make your change, and open a PR describing what it does and why.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for release history.

## License

[MIT](LICENSE)
