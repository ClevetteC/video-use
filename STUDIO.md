# Video Editing Studio — Setup Guide

This document describes how `video-use` is wired into a full AI-driven video editing studio alongside [hyperframes](https://github.com/ClevetteC/hyperframes).

## What video-use handles

- **Transcription** — ElevenLabs Scribe, word-level timestamps, speaker diarization
- **Filler word removal** — `umm`, `uh`, false starts, dead air detected from transcript
- **Cut editing** — EDL-driven, word-boundary snapping, 30ms audio fades
- **Color grading** — per-segment ffmpeg filter chains
- **Subtitle burning** — applied last in the filter chain (Hard Rule 1)
- **Animation overlays** — PIL, Manim, or Remotion sub-agents spawned in parallel

## What hyperframes handles

- **HTML-based motion graphics** — write HTML/CSS/GSAP, render to MP4
- **Typography overlays** — lower thirds, title cards, caption animations
- **Brand-aligned graphics** — 50+ catalog blocks (social overlays, charts, shader transitions)
- **TTS narration** — `hyperframes tts` generates voiceover from text

## Pipeline

```
Raw footage
    │
    ▼
transcribe.py            ← ElevenLabs Scribe, word-level timestamps (cached)
    │
    ▼
pack_transcripts.py      ← Compact takes_packed.md for LLM reasoning
    │
    ▼
Converse + confirm       ← Propose strategy, wait for approval
    │
    ▼
render.py                ← Per-segment extract → grade → 30ms fades → concat
    │
    ├── [parallel] animation sub-agents (PIL / Manim / Remotion / hyperframes)
    │
    ▼
render.py --compose      ← Composite overlays, burn subtitles LAST
    │
    ▼
edit/final.mp4
```

## Local install

```bash
# 1. Clone
git clone https://github.com/ClevetteC/video-use ~/Developer/video-use
cd ~/Developer/video-use

# 2. Python deps
uv sync   # or: pip install -e .

# 3. ffmpeg (Ubuntu/Debian)
sudo apt-get install -y ffmpeg

# 4. Register skill with Claude Code
mkdir -p ~/.claude/skills
ln -sfn ~/Developer/video-use ~/.claude/skills/video-use

# 5. API key
echo 'ELEVENLABS_API_KEY=your_key_here' > .env
chmod 600 .env
```

Get an ElevenLabs key at https://elevenlabs.io/app/settings/api-keys

## Hyperframes integration

For HTML-based motion graphics, install hyperframes alongside:

```bash
git clone https://github.com/ClevetteC/hyperframes ~/Developer/hyperframes
cd ~/Developer/hyperframes
bun install && bun run build

# Register all hyperframes skills
for skill in hyperframes hyperframes-cli hyperframes-registry gsap website-to-hyperframes claude-design-hyperframes; do
  ln -sfn ~/Developer/hyperframes/skills/$skill ~/.claude/skills/$skill
done
```

Then in your edit session, request hyperframes overlays directly:
> "Add a lower third title card at the 0:03 mark using hyperframes"

The rendered overlay MP4 drops into `edit/animations/slot_N/render.mp4` and
`render.py` picks it up via the EDL `overlays` array.

## Skill commands

| Command | What it does |
|---|---|
| `/video-use` | Full pipeline skill — transcribe, cut, grade, animate, subtitles |
| `/hyperframes` | Author HTML video compositions |
| `/hyperframes-cli` | CLI: init, lint, preview, render, tts |
| `/hyperframes-registry` | Install catalog blocks |
| `/gsap` | GSAP animation reference |
| `/website-to-hyperframes` | URL → video |

## Starting a session

```bash
cd /path/to/your/footage
claude
```

Then: *"edit these into a launch video"*

All outputs land in `<footage-dir>/edit/`. This repo stays clean.
