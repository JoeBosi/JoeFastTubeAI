# Changelog

All notable changes to JoeFastTubeAI are documented here.
The format is based on [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased] — Roadmap

### Planned
- **Auto-zoom (two-pass).** After the panoramic pass, read the transcript to find
  the moments where the narrator points at something on screen ("look here", "this
  level", "this chart"), then run a focused, higher-density second pass on exactly
  those segments. _(internal tag: "Problema 2")_
- **Full HD frames.** Optional `1920px` capture for readable on-screen text in the
  auto-zoom pass. _(internal tag: "Problema 2")_
- **Chunked Whisper.** Split the extracted audio so caption-less videos longer than
  ~50 min stop hitting the 25 MB Whisper upload limit. _(internal tag: "Problema 1")_

## [1.0.1] — 2026-06-02

### Fixed
- **macOS SSL verification (`CERTIFICATE_VERIFY_FAILED`).** python.org Python builds
  ship without an initialized CA bundle, so every Whisper call (Groq/OpenAI) failed
  before authentication. The HTTPS context now uses the `certifi` bundle when
  available, falling back to common system bundles (`/etc/ssl/cert.pem`, Homebrew's).

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
