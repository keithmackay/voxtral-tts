voxtral-tts - Convert text to speech using Mistral's Voxtral model and deliver it as a voice note.

WHAT IT DOES
  Generates an MP3 from text via scripts/voxtral, then optionally sends it as
  a voice note to Discord or Telegram, or hands back a local file path.

WHAT IT NEEDS
  Python with uv installed.
  A MISTRAL_API_KEY environment variable (the script exits with an error if
  it's unset).

USAGE
  uv run scripts/voxtral "Hello, Keith."
    Generates audio, prints the output file path to stdout.

  uv run scripts/voxtral -v nick "Here's your summary."
    Choose a voice. Default is margaret.

  uv run scripts/voxtral -v margaret -o /tmp/reply.mp3 "Your briefing is ready."
    Save to a specific path.

  uv run scripts/voxtral --strip-md -v nick "**Summary:** task complete."
    Strip markdown from AI agent output before speaking it.

  echo "$LONG_OUTPUT" | uv run scripts/voxtral --strip-md --truncate -v nick
    Pipe from stdin, truncate if too long.

  uv run scripts/voxtral --dry-run -v nick "Here's your summary."
    Preview mode. Validates input and reports the voice, character count,
    estimated cost, and output path (noting if it would be served from
    cache) without calling the Mistral API, writing an audio file, or
    sending anything. Combine with --voice ("--voice --dry-run") to also
    preview the destination channel without sending.

  --voice flag / "read that to me" / "send as audio"
    Triggers the same generate-then-send flow from within a coding agent
    conversation: strip markdown, truncate if needed, generate audio, and
    send it to the originating channel.

VOICES
  margaret (British female, default), nick (American male), angele (French
  female), sanchit (Hindi-accented English male), gustavo (Portuguese/Spanish
  male), khyathi (Indian English female), yassir (Arabic-accented English
  male), patrick (American male, casual).

COST
  $0.016 per 1,000 characters. ~$0.003 for a short reply, ~$0.16 for a
  10,000-character article.
