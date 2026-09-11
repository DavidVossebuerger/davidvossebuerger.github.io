---
title: "Keymic"
date: 2026-09-11
weight: 4
author: ["David Vossebürger"]
description: "Push-to-talk voice dictation daemon for Linux with Groq Whisper STT and optional LLM cleanup."
summary: "Linux push-to-talk dictation daemon: hold a hotkey, speak, release; transcribed text is typed into the focused window via ydotool/xdotool. Python 3.11+, PipeWire audio, Groq Whisper STT."
editPost:
    URL: "https://github.com/DavidVossebuerger/Keymic"
    Text: "Source code"

---

---

##### Overview

**Keymic** is a push-to-talk voice dictation daemon for Linux. Hold a hotkey (default: Right Ctrl), speak, release — the transcribed text appears in the focused window. It records through PipeWire/PulseAudio, ships audio to Groq Whisper for transcription (no local model), types the result via `ydotool` (Wayland) or `xdotool` (X11), and runs as a background daemon with `setup / run / stop / status / check` subcommands plus a systemd user unit.

---

##### Features

- Configurable hold-to-talk hotkey (default: Right Ctrl).
- PipeWire audio capture (`pw-cat`) with PulseAudio fallback.
- Groq Whisper STT (`whisper-large-v3`, configurable); no local model, no self-hosted server.
- Optional AI post-processing (Groq LLM) fixes mid-sentence self-corrections, e.g. *"Hallo, ich bin um 15, nee sorry, 17 Uhr da."* → *"Hallo, ich bin um 17 Uhr da."* Off by default.
- Fallback: local heuristic cleanup removes filler words when AI post-processing is disabled.
- Text injection: `ydotool` (Wayland), `xdotool` (X11), `wl-clipboard` / `xclip` as fallback.
- Daemonized with PID file, log path, and a `systemd/keymic.service` user unit for autostart.
- TOML config at `~/.config/keymic/config.toml` (XDG-compliant).

---

##### Technical stack

- **Language:** Python 3.11+
- **STT:** Groq API (`whisper-large-v3` or configurable model)
- **Optional LLM cleanup:** Groq (any chat model)
- **Audio:** `pw-cat` (PipeWire), PulseAudio fallback
- **Keyboard:** `python-evdev` (Linux `KEY_*` input event codes; `EVIOCGRAB` + UInput replayer pattern)
- **Text injection:** `ydotool`, `xdotool`, `wl-clipboard` / `xclip`
- **Packaging:** `pyproject.toml` (PEP 621), `requirements.txt`, `setup.sh` (Arch / Debian)
- **Daemonization:** background process with PID file; `systemd/keymic.service` user unit

---

##### Source layout

```
keymic/
  __main__.py        # 8.2 KB CLI entrypoint
  daemon.py          # 5.0 KB
  platform_linux.py  # 12.4 KB evdev/UInput layer
  audio.py           # 1.9 KB
  whisper_api.py     # 1.6 KB
  cleanup.py         # 11.5 KB
  config.py          # 2.9 KB
  setup_cli.py       # 8.2 KB
config/              # default keymic.toml template
systemd/             # keymic.service user unit
tests/               # test suite
README.md (5.8 KB), LICENSE (MIT), pyproject.toml, requirements.txt, setup.sh (1.9 KB), .gitignore
```

The `EVIOCGRAB` + UInput replayer architecture is carried over from the author's earlier private push-to-talk daemon.

---

##### Quick start

```bash
git clone https://github.com/DavidVossebuerger/Keymic.git
cd Keymic
./setup.sh   # Arch / Debian
python -m keymic run
```

Hold Right Ctrl, speak, release. Configure your Groq API key at <https://console.groq.com/keys> before first run.

---

##### External links

- Repo: <https://github.com/DavidVossebuerger/Keymic>
- `evdev`: <https://python-evdev.readthedocs.io/>
- `ydotool`: <https://github.com/ReimuNotMoe/ydotool>
- `xdotool`: <https://www.xdotool.org/>
- PipeWire: <https://pipewire.org/>
- Groq: <https://groq.com>
- License: MIT

---

*Disclosure: developed with Claude Code by Anthropic; human-driven design, AI implementation.*
