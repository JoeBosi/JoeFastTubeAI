# Changelog

All notable changes to JoeFastTubeAI are documented here.
The format is based on [Keep a Changelog](https://keepachangelog.com/).

## [1.0.0] — 2026-06-02

First release. Fork of the upstream [`watch`](https://github.com/bradautomates/claude-video)
skill by Bradley Bonanno, extended with caching and persistent output.

### Added
- **Per-video cache.** The downloaded video, subtitles and (if used) the Whisper
  transcript are stored under `JoeFastTubeAI/<video-id>/` and reused on any later
  request for the same video — no re-download, no re-transcription.
- **Persistent, numbered results.** Every request gets its own folder
  `JoeFastTubeAI/<video-id>/<N>/` containing `prompt.md` (the prompt that triggered
  it), `frames/`, `report.md`, and `result.md` (Claude's final answer). `N`
  auto-increments so multiple questions about the same video never overwrite.
- **Independent configuration** at `~/.config/JoeFastTubeAI/.env`, with a read-only
  fallback to the legacy `~/.config/watch/.env` so existing Whisper keys keep working.
- New entry point `scripts/joefasttube.py` and slash command `/JoeFastTubeAI`.

### Unchanged from upstream
- yt-dlp download, ffmpeg auto-scaled frame extraction, caption/Whisper transcript
  pipeline, `--start`/`--end` focus mode, and the macOS/Homebrew setup preflight.
