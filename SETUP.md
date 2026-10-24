# Slimey for Windows — Setup Guide

Slimey is a local-first voice AI tutor for Windows: it hears your question,
can see your screen, and answers out loud. This guide covers installing it
from source. (For a portable build, see **[Portable build](#portable-build)**.)

## Prerequisites

- **Windows 10 or 11** (x64)
- **Python 3.11+** — download from <https://www.python.org/downloads/> and
  enable "Add Python to PATH" during install
- **Ollama** — <https://ollama.com/download> (free, runs models locally)
- Optional: API keys for cloud providers (Claude / OpenAI / Gemini), stored
  in `.env` — never required

## 1. Get the code

```
git clone https://github.com/vaibhavgaur44/Slimey.git Slimey
cd Slimey
```

## 2. Install Python dependencies

Two requirement files exist. Pick **one**; both launch the same app.

| File | What it is | Use it when |
|------|------------|-------------|
| `requirements-student.txt` | The free/local bundle + offline pointer accuracy extras | You just want Slimey working with no API keys (recommended) |
| `requirements.txt` | The compact core set of providers and utilities | You mainly use cloud APIs and want a smaller install |

```
# Student version (free, no API keys)
pip install -r requirements-student.txt

# … or the full/cloud-leaning core set
pip install -r requirements.txt
```

`requirements-student.txt` additionally pulls `uiautomation`,
`rapidocr-onnxruntime` (offline OCR), `opencv-python-headless` (figure
detection), `ddgs` (web search without rate-limit blocks), `pywhispercpp`
(faster local transcription) and `imageio-ffmpeg`, so lesson recording and
pointing work offline. Both lists include cloud SDKs (openai/anthropic) —
installing them costs nothing until you supply a key.

> Audio uses `sounddevice`; no `PyAudio` build step is needed on any current Python.

Check the install worked:

```
python -c "import PyQt6, sounddevice; print('deps ok')"
```

## 3. Configure (optional)

```
copy .env.example .env
```

Then edit `.env` in a text editor if you want to:

- add API keys (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GOOGLE_API_KEY`,
  `ELEVENLABS_API_KEY`, `DEEPGRAM_API_KEY`, `TAVILY_API_KEY`)
- switch providers (`SLIMEY_ACTIVE_LLM`) — **leave it empty/unset to let
  Slimey pick automatically** (see Provider priority below)
- set the hotkey (`SLIMEY_HOTKEY`, default `ctrl+win`) or microphone mode
  (`SLIMEY_MIC_MODE=hotkey|ambient`)

**Optional custom skills:** the app loads Python skills from
`%USERPROFILE%\.clicky\skills\` (bundled examples live in the repo's `skills\`
folder). The folder keeps its old `.clicky` name in source; you can create it
and drop an `example_self_mode.py` copy there.

## 4. Install Ollama + models

1. Download Ollama from <https://ollama.com/download> and install it.
2. Verify with `ollama --version`, then pull the default model **once**:

   ```
   ollama pull qwen2.5vl:3b
   ```

   `qwen2.5vl:3b` is Slimey's default because it reads your screen *and*
   answers text questions, and stays quick on laptop CPUs/GPUs (~3.2 GB).

   > **Do not use `llama3.2-vision`** — newer Ollama versions dropped its model
   > architecture, so `ollama list` shows it but every screen question fails
   > with an HTTP 500.
3. Optional overrides in `.env`:

   | Var | Purpose |
   |-----|---------|
   | `OLLAMA_HOST` | Server URL (default `http://localhost:11434`) |
   | `OLLAMA_VISION_MODEL` / `OLLAMA_TEXT_MODEL` | Split screen-QA/text models |
   | `OLLAMA_MODEL` | Legacy single override for both slots |

   You can also switch vision/text models at runtime from
   **Tray → Ollama → Vision/Text model** without editing `.env`.

If the end of setup shows a model choice you don't have, install it there —
the first-run wizard does the pull for you.

## 5. Run

```
python main.py
```

- A floating panel + tray icon appear; status text drifts between
  **Listening / Thinking / Speaking**.
- Default voice controls: hold **Ctrl + Win**, speak, release; Slimey
  answers when you stop talking.
- In ambient mode (`SLIMEY_MIC_MODE=ambient`) the mic stays open and scans
  for the wake word **— note: the wake-word list in the current code still
  matches "Clicky" variants**, so prefer the default hotkey flow until that
  rename lands in the app itself. Hold **Ctrl + Win** — speak — release is
  the reliable path today.

## Provider priority (auto-detected, no setup needed)

Each layer falls through in order based on what keys/models it finds:

| Layer | Order |
|-------|-------|
| LLM | Claude → OpenAI → GitHub Copilot → Gemini → Ollama (local) → LM Studio (local) |
| STT  | Deepgram → OpenAI Whisper → whisper.cpp (if installed) → faster-whisper (local) |
| TTS  | ElevenLabs → OpenAI → edge-tts (local, free) |
| Web search | Tavily (if key) → DuckDuckGo (free) |

Leave `SLIMEY_ACTIVE_LLM` unset in `.env` to let this chain decide; set it
only if you want to pin one provider.

## Portable build

`build.bat` produces a self-contained folder you can copy to any Windows PC:

```
build.bat                  # portable folder: dist\Slimey\Slimey.exe
build.bat installer        # + single-file installer: dist\Setup-Slimey.exe
                           #   (requires Inno Setup 6: https://jrsoftware.org/isdl.php)
```

- Run the portable build with `dist\Slimey\Slimey.exe` (keep the whole
  `Slimey` folder together — it relies on the bundled `_internal\` files).
- For a portable build, place `.env` **next to `Slimey.exe`** (the packaged
  app reads it from there, not from inside `_internal\`).

## Logs & diagnostics

Everything runtime-related lands under `%LOCALAPPDATA%\Slimey\`:

| Path | Contents |
|------|----------|
| `%LOCALAPPDATA%\Slimey\slimey.log` | Main rotating log (1 MB × 2) — includes tracebacks |
| `%LOCALAPPDATA%\Slimey\journal.db`   | Q&A history (SQLite, if journaling is on) |
| `%LOCALAPPDATA%\Slimey\diagnostics-*.txt` | One-click diagnostics from Tray → Setup & Diagnostics → Diagnostics |

To inspect the log: paste `%LOCALAPPDATA%\Slimey` into Explorer's address bar,
or use **Tray → Setup & Diagnostics → Save diagnostics report…** — the report
bundles the config/event context, then opens the file for you.

(Note: a few tray labels — the quit item, wake-word toast — still carry the
old "Clicky" wording in source; that rename is tracked separately and does
not affect setup.)

## Troubleshooting

**`python main.py` fails with "ModuleNotFoundError"**
A package is missing: re-run the `pip install -r ...` command from step 2
(from the repo root). If `python` itself isn't found, install Python 3.11+
with "Add to PATH" enabled or use the portable build instead.

**"Ollama doesn't have '<model>' installed" toast**
Run `ollama pull qwen2.5vl:3b`, then either wait for the pull to finish or
pick a different installed model under **Tray → Ollama**.

**Ollama request times out (e.g. "didn't produce an answer within 120 s")**
The server didn't stream an answer in time — usually the model was still
loading, another request was queued, or prompt processing is slow for your
machine. In order:
1. Wait 10–20 s and ask again (the model may simply still be loading).
2. Check the server is alive: `http://localhost:11434` in a browser, or run
   `ollama list` — if Ollama isn't running, start it or reinstall from
   ollama.com/download.
3. Pick a smaller model under **Tray → Ollama**, or close heavy apps.
4. Check the log file above for the exact Ollama response (model dead
   architectures, out-of-memory, etc. — the toast text names what to do next).

**Wrong model after an update / HTTP 500 on every screen question**
Your `.env` may pin a dead model (`llama3.2-vision`). Delete or comment out
`OLLAMA_MODEL=llama3.2-vision` in `.env`, then use the default
`qwen2.5vl:3b` or pull a supported vision model.

**Slimey doesn't launch at all / no tray icon**
Look at `%LOCALAPPDATA%\Slimey\slimey.log`; the first traceback lines almost
always name the missing package or config key.

## What's verified vs. still to test

- ✅ Verified: dependency install, model default (`qwen2.5vl:3b`),
  entry-point `python main.py` wiring, portable build layout (`dist\Slimey\Slimey.exe`
  via `build.bat`, exit code 0), log/diagnostics paths under `%LOCALAPPDATA%\Slimey`.
- ⚠️ Needs testing on your machine: the full first-run wizard flow, the
  `build.bat installer` / Inno Setup path, and any Ollama pull duration you
  haven't seen yet.
