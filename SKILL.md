---
name: JoeFastTubeAI
description: Watch a video (URL or local path) the way /watch does — download with yt-dlp, extract frames with ffmpeg, pull the transcript from captions (or Whisper) — BUT with per-video caching and persistent output. Every run is saved under ./JoeFastTubeAI/<video-id>/<N>/ (prompt.md + result.md + frames), and the heavy reusable artifacts (download, transcript) are cached per video so re-prompts skip re-downloading and re-transcribing.
argument-hint: "<video-url-or-path> [question]"
allowed-tools: Bash, Read, Write, AskUserQuestion
homepage: https://github.com/bradautomates/claude-video
author: bosi.giuseppe (fork of bradautomates/watch)
license: MIT
user-invocable: true
---

# /JoeFastTubeAI — watch a video, cache it, and save the result to disk

This is a personalized fork of the `watch` skill. It does everything `watch` does
(download → frames → transcript → Claude answers), and additionally:

1. **Caches per video.** The downloaded video, subtitles and Whisper transcript are
   stored under `JoeFastTubeAI/<video-id>/` and **reused** on any later request for
   the same video — saving bandwidth (no re-download) and tokens/money (no re-transcribe).
2. **Persists every request.** Each run gets its own numbered folder
   `JoeFastTubeAI/<video-id>/<N>/` containing `prompt.md` (the prompt that triggered it),
   `frames/`, `report.md`, and `result.md` (your final answer). `N` auto-increments,
   so multiple questions about the same video never overwrite each other.

```
JoeFastTubeAI/
  <video-id>/                COMMON, reusable across requests
    download/                yt-dlp output (video + info.json + subtitles)
    transcript.json          cached Whisper transcript (if used)
    audio.mp3                cached extracted audio (if Whisper used)
    1/   2/   3/ ...          NON-common, one folder per request
      prompt.md              the prompt that generated this run
      frames/                extracted JPEG frames
      report.md              frames list + transcript
      result.md              <-- YOU save your final answer here
```

## Step 0 — Setup preflight (silent on success)

On **Windows** substitute `python` for `python3`. Before running, verify deps + key:

```bash
python3 "${CLAUDE_SKILL_DIR}/scripts/setup.py" --check
```

Exit 0 → emit nothing, proceed. On non-zero exit:

| Exit | Meaning | Action |
|------|---------|--------|
| `2` | Missing `ffmpeg` / `ffprobe` / `yt-dlp` | Run `python3 "${CLAUDE_SKILL_DIR}/scripts/setup.py"` |
| `3` | No Whisper API key | Run installer, then ask the user for a Groq/OpenAI key via `AskUserQuestion` |
| `4` | Both missing | Run installer, then ask for a key |

The installer auto-installs ffmpeg/yt-dlp via Homebrew on macOS and scaffolds
`~/.config/JoeFastTubeAI/.env` (a legacy `~/.config/watch/.env` key is reused if present).
Whisper is optional:
videos with native captions (most of YouTube) work with no key. If the user declines a
key, run with `--no-whisper` and tell them caption-less videos come back frames-only.
Within a session, skip Step 0 on follow-up calls once `--check` returned 0.

## How to invoke

**Step 1 — parse input.** Split the video source (URL or path) from the user's question.
Example: `/JoeFastTubeAI https://youtu.be/abc what language is this?`
→ source = `https://youtu.be/abc`, question = `what language is this?`

**Step 2 — run the script.** ALWAYS pass `--prompt` with the user's request verbatim
(or a short description like "riassunto del video" if they didn't ask anything specific),
so it gets saved into `prompt.md`:

```bash
python3 "${CLAUDE_SKILL_DIR}/scripts/joefasttube.py" "<source>" --prompt "<user question>"
```

The script prints (to stderr) the `video id`, the request number `#N`, and whether the
download was reused from cache. It prints the markdown report to stdout.

Optional flags (same semantics as `watch`):
- `--start T` / `--end T` — focus a section (`SS`, `MM:SS`, `HH:MM:SS`); fps auto-densifies.
- `--max-frames N` — lower the cap for a tighter token budget (e.g. `--max-frames 40`).
- `--resolution W` — frame width in px (default 512; 1024 only to read on-screen text).
- `--fps F` — override auto-fps (max 2).
- `--no-whisper` — disable the Whisper fallback (frames-only if no captions).
- `--whisper groq|openai` — force a backend.
- `--base-dir DIR` — change the root output folder (default: `./JoeFastTubeAI`).

**Step 3 — Read every frame path** the report lists, in a single message (parallel
Read calls) so you see them together. Each has a `t=MM:SS` absolute timestamp.

**Step 4 — answer the user** in chat. Combine frames (what's on screen) with the
transcript (what's said), citing timestamps. If they asked nothing, summarize the video.

**Step 5 — SAVE YOUR ANSWER (required).** The script printed a line
`[JoeFastTubeAI] SAVE-RESULT-TO: <path>/result.md` on stderr. Write your final answer
(the same text you gave the user, in Markdown) to that exact `result.md` path using the
Write tool, overwriting the placeholder. This is the whole point of the skill — the
result must end up on disk next to its `prompt.md`.

## Caching behavior (what to tell the user)

- **First time** on a video: it downloads + extracts + (maybe) transcribes. Folder `1/`.
- **Re-prompt** on the same video: the download and transcript are reused from
  `JoeFastTubeAI/<video-id>/`; only new frames + a new `<N>/` folder are produced. Mention
  in your answer when the download was reused ("riusato dalla cache, nessuna banda consumata").
- Do **NOT** delete the `JoeFastTubeAI/` folder — unlike the original `watch` skill, this
  output is meant to persist. The cache is what makes future runs cheap.

## Token efficiency

Frames dominate token cost (~50–80k for 80 frames at 512px). The transcript is cheap.
If you already watched a video this session and the user asks a follow-up that the frames
+ transcript already in context can answer, just answer — no need to re-run the script.

## Security & Permissions

Same as the upstream `watch` skill: runs `yt-dlp`/`ffmpeg`/`ffprobe` locally; only the
extracted audio (never the video) is sent to Groq/OpenAI Whisper, and only when native
captions are missing and Whisper is not disabled. Reads/creates `~/.config/JoeFastTubeAI/.env`
(mode 0600) for API keys. The only new behavior is that output is written to
`./JoeFastTubeAI/...` in the working directory instead of a throwaway temp dir.

**Bundled scripts:** `scripts/joefasttube.py` (entry point, caching + persistence),
plus the reused `download.py`, `frames.py`, `transcribe.py`, `whisper.py`, `setup.py`.
