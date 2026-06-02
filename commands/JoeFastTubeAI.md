---
description: Watch a video (URL or local path) with per-video caching and persistent output. Downloads with yt-dlp, extracts frames with ffmpeg, transcribes from captions or Whisper, answers in chat, and saves prompt.md + result.md + frames under ./JoeFastTubeAI/<video-id>/<N>/.
argument-hint: <video-url-or-path> [question]
allowed-tools: [Bash, Read, Write, AskUserQuestion]
---

Invoke the `JoeFastTubeAI` skill (defined in SKILL.md) with the user's arguments: $ARGUMENTS

Follow the skill's full pipeline: preflight setup check → run `scripts/joefasttube.py "<source>" --prompt "<user question>"` (reuses the per-video cache if present) → Read each frame the report lists → answer the user grounded in frames and transcript → then SAVE that answer to the `result.md` path the script printed (`SAVE-RESULT-TO:`). Do NOT delete the `JoeFastTubeAI/` output folder. If the user provided no arguments, ask them for a video URL or local path before proceeding.
