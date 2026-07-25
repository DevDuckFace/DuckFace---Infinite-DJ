<h1 align="center">DuckFace — Infinite DJ</h1>

<p align="center">
  <em>Endless radio for your desktop: your YouTube playlists, audio only,<br>
  mixed live by an AI DJ that runs entirely on your own machine.</em>
</p>

<p align="center">
  <a href="https://github.com/DevDuckFace/DuckFace---Infinite-DJ/releases/download/infinite1.0/DuckFace-InfiniteDJ-Setup-1.0.0.exe"><b>⬇ Download the installer</b></a>
  &nbsp;·&nbsp; Windows 10/11 64-bit
</p>

<!-- Replace the URL below with your screenshot (drag an image into a GitHub
     issue to get a permanent link, or commit it to docs/ and point here). -->
<p align="center">
  <img src="https://github.com/user-attachments/assets/87b035a9-2470-45b5-993e-72c408ab9f67" alt="DuckFace - Infinite DJ" width="900">
</p>

---

## What it does

Pick a YouTube playlist. It plays **audio only**, shows a Windows-Media-Player
style visualizer, and between tracks a DJ fades the music down, says something
about what just played, announces what's next, and brings the sound back up.

The DJ is not a recording. A language model writes every line, on your machine,
offline, and a neural voice speaks it.

| | |
|---|---|
| **Playlist search** | Search YouTube playlists with thumbnails, infinite scroll |
| **Audio only** | Downloads just the audio track (`bestaudio` → mp3), never the video |
| **Prefetch** | The moment a track starts, the next one is already downloading |
| **Real crossfade** | Constant-power fade between two independent audio decks |
| **Ducking** | Music dips to ~22% while the DJ talks, then ramps back |
| **Offline DJ** | Embedded llama.cpp (GGUF) writes the line, Piper speaks it |
| **Saved playlists** | Save a whole YouTube playlist, or build your own track by track |
| **Visualizer** | Own FFT: spectrum bars with floating peak marks + waveform |
| **Bilingual** | Interface in English and Portuguese; the DJ speaks 6 languages |

The DJ's text **never appears on screen** — it becomes audio and nothing else.
It is written to the log only so you can tell what happened.

---

## Install

Grab the installer from the link at the top. That's it — **everything is
bundled**: `yt-dlp`, `ffmpeg` (LGPL), the full Piper TTS and two voices
(`pt_BR-faber-medium`, `en_US-amy-medium`).

The one thing you choose and download is the **DJ's language model** (a GGUF
file), from **Settings → Models**. Without one the DJ still works, using canned
lines.

> **yt-dlp goes stale.** YouTube changes its API and old builds stop
> downloading. When search or playback starts failing, hit **Update yt-dlp** in
> *Settings → General* — no reinstall needed.

---

## Choosing the DJ's model

Everything lives in **Settings → Models**, in three collapsible sections.

**Download from Hugging Face.** Paste a repository link — or just `author/model`
— and hit *List models*. Every `.gguf` in the repo shows up with its real size,
smallest first, one click to download. All of these work:

```
unsloth/gemma-4-E2B-it-GGUF
https://huggingface.co/unsloth/gemma-4-E2B-it-GGUF
https://huggingface.co/unsloth/gemma-4-E2B-it-GGUF/tree/main
huggingface.co/unsloth/gemma-4-E2B-it-GGUF/blob/main/gemma-4-E2B-it-IQ4_XS.gguf
```

Multi-part models (`-00001-of-00003.gguf`) are flagged and disabled — only
single-file models load.

**Access token.** Only needed for private or gated repositories. Create one with
read permission at [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens).
It is stored encrypted with the Windows DPAPI, tied to your user account — the
file is useless on another account or machine, and the token never touches
`settings.json`. The `Authorization` header is only ever sent to
`huggingface.co`: when a download redirects to the signed CDN URL, the header is
dropped, because S3 rejects requests carrying two auth mechanisms at once.

**From your computer.** *Choose a .gguf file...* opens the Windows file picker
and uses the file where it is — nothing is copied. *Scan a folder...* lists every
`.gguf` in a directory.

Which architectures load is decided by the llama.cpp version compiled in
(`INFINITEDJ_LLAMA_TAG` in [cmake/Dependencies.cmake](cmake/Dependencies.cmake)).
A model newer than that fails with *unknown model architecture*, and the app
tells you so in plain words.

---

## How it works

```
                 ┌───────── yt-dlp ──────────┐
search ─────────►│ playlists  ·  tracks      │
                 │ bestaudio ──ffmpeg──► mp3 │
                 └───────────┬───────────────┘
                             │  disk cache
                             ▼
   ┌──────────────────── DjDirector ─────────────────────┐
   │  track starts    → download the next one AND have   │
   │                    llama.cpp write the line, Piper  │
   │                    voice it. The audio just waits.  │
   │  track ends      → duck → crossfade → DJ on air     │
   │                    → fade back in                   │
   │  skip pressed    → drop the pending audio, abort    │
   │                    the model, switch immediately    │
   └──────────────────────────┬──────────────────────────┘
                              ▼
   ┌──────────────────── AudioEngine ────────────────────┐
   │  deck A ──┐                                         │
   │  deck B ──┼─ crossfade ─► × duck ─┬─► limiter ────► │──► sound card
   │  DJ voice ────────────────────────┘                 │
   │                                   └──► FFT ──► visualizer
   └─────────────────────────────────────────────────────┘
```

The subtle part is timing. A 3B model takes tens of seconds on an ordinary CPU,
so the line is written **at the start of the track**, on its own thread, and the
finished audio sits and waits. The DJ only ever speaks when a track ends by
itself. Pressing skip never touches the model — it aborts whatever is being
generated and switches on the spot.

---



### Dependencies

Fetched by CMake at configure time: [GLFW](https://github.com/glfw/glfw),
[Dear ImGui](https://github.com/ocornut/imgui),
[miniaudio](https://github.com/mackron/miniaudio),
[stb](https://github.com/nothings/stb),
[nlohmann/json](https://github.com/nlohmann/json),
[llama.cpp](https://github.com/ggml-org/llama.cpp).

Fetched by `build.bat` and shipped in the installer:
[yt-dlp](https://github.com/yt-dlp/yt-dlp) (Unlicense),
[ffmpeg](https://github.com/BtbN/FFmpeg-Builds) (LGPL shared build),
[Piper](https://github.com/rhasspy/piper) (MIT) and
[Piper voices](https://huggingface.co/rhasspy/piper-voices).

---


## Troubleshooting

Everything lives in `%LOCALAPPDATA%\InfiniteDJ`:

| File | What it is |
|---|---|
| `infinitedj.log` | session log, including the lines the DJ spoke |
| `crash.log` | if it ever crashes: exception + stack as RVAs |
| `settings.json` | settings |
| `playlists.json` | your saved playlists |
| `huggingface.dpapi` | Hugging Face token, encrypted with your Windows account |
| `cache/` | downloaded audio (pruned automatically) |
| `models/` | LLM GGUFs and extra Piper voices |

To symbolise a `crash.log`, add `0x140000000` to the RVA:


## Known limits

- Chinese, Japanese and Korean titles render as boxes. The font atlas covers
  Latin, Greek, Cyrillic and punctuation; adding CJK would multiply the texture
  size.
- Small models sometimes ignore the requested language. There is a check that
  detects it and retries at a lower temperature, then falls back to a canned
  line. Larger models make this go away.
- The DJ never speaks on a manual skip. That is deliberate — see *How it works*.

---

<p align="center">
  <b>Dev DuckFace</b><br>
  <a href="https://github.com/DevDuckFace">github.com/DevDuckFace</a>
</p>
