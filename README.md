<!--
  ARGUS FORGE — README
  The boot splash is a Three.js particle field rendered in a pywebview (WebView2)
  window. splash/ (splash.html + three.min.js + logo.png) and ui/ (ArgusForge.ico)
  must be bundled as data:
    PyInstaller spec -> datas=[('splash', 'splash'), ('ui', 'ui')]
    Nuitka          -> --include-data-dir=splash=splash
-->

<div align="center">

# ARGUS FORGE

**Red-team workbench. Two tabs. Two deploy targets. Generates captive portals and BadUSB payloads from natural language, then drops them on a WiFi Pineapple or any Bruce-compatible device with one click.**

[![Downloads](https://img.shields.io/github/downloads/MS-XIII/AIForge/latest/ArgusForge.zip?displayAssetName=false&label=downloads)](#)
[![Platform](https://img.shields.io/badge/platform-Windows-blue)](#)
[![Python](https://img.shields.io/badge/python-3.10%2B-3776AB)](#)
[![Built with Nuitka](https://img.shields.io/badge/built%20with-Nuitka-2C2C2C)](#)
[![License](https://img.shields.io/badge/license-MIT-ff3344)](#license)

</div>

---

## Overview

**ARGUS FORGE** is a local, self-contained red-team GUI that turns a plain-text prompt into a finished offensive-tooling artifact. Two tabs, each targeting a different engagement surface: **Evil Portal** for captive-portal phishing infrastructure, **Bad USB** for keystroke-injection payloads. Both sit on top of any OpenAI-compatible chat-completion endpoint — OpenAI, OpenRouter, Groq, Together, vLLM, llama.cpp, Ollama, or a self-hosted box — so the LLM backend is whatever you point it at, not whatever someone else decided.

The generated artifact lands in an output pane, gets validated by you, and ships to a **WiFi Pineapple** over its REST API — or to any **Bruce-compatible device** (M5Stack, LilyGo, ESP32-family boards running Bruce firmware) over its HTTP file interface — with one button. No copy-paste-into-SSH, no scp dance. The target is pinged, authenticated, and written to — all on a background thread so the UI never blocks.

Every generation is routed through a fixed **Repository** system prompt — a persona doctrine that produces code-first output in a consistent format. It is the same prompt on both tabs; a short tab-context suffix tells the model which workspace it's on.

The build pipeline compiles the whole thing to a native Windows binary with **Nuitka**, stripping Python bytecode from the shipped artifact and wrapping it in a set of free runtime anti-analysis guards. No `.pyc` on disk. No `pyinstxtractor` recovery. Constants are the last mile and the repo ships with an XOR path for hiding the system prompt from `strings`.

---

## Features

- **Dual-tab workbench** — `Evil Portal` and `Bad USB` share a single window, a single config file, and a single AI backend. Switch tabs, the context switches with you.

- **Pluggable LLM backend** — any endpoint speaking the OpenAI `/v1/chat/completions` spec. Set `base_url`, `api_key`, and `model` in Settings. Works against hosted providers or a local `llama.cpp` server on `localhost:8080`.

- **Repository system prompt** — every request prepends a fixed operator doctrine. Code-first, consistent format, no hedging. The prompt is a single string at the top of `evilcreations.py` and can be swapped or XOR-obfuscated without touching the rest of the code.

- **Tab-scoped context injection** — a one-line suffix is appended to the system message on each call so the model knows whether it's generating portal code or DuckyScript. Same doctrine, different surface.

- **One-click Pineapple deploy** — hit `SEND // PINEAPPLE`, the tool checks reachability at `172.16.42.1:1471`, logs in with your creds, and writes the current output to `/root/` with a timestamped filename. `.html` for portals, `.txt` for BadUSB payloads. If the Pineapple isn't up, you get an explicit error instead of a silent timeout.

- **One-click Bruce deploy** — the same button routes to any Bruce-compatible device (M5Stack Cardputer, LilyGo T-Embed / T-Deck, ESP32-S3 boards, etc.). ARGUS FORGE authenticates against the Bruce WebUI and pushes the artifact to the correct on-device folder (`/evilportal/` for HTML, `/badusb/` for scripts) with a timestamped filename. Devices that don't respond are skipped cleanly without blocking the queue.

- **Animated splash** — a borderless pywebview (WebView2) window renders a Three.js particle field in purple (`#8B5CF6`) with a glowing centered ARGUS FORGE logo, a letter-spaced title, and a progress bar that fills over ~3 seconds, then fades out into the main window.

- **Bundled icon** — `ui/ArgusForge.ico` ships as data and is applied to the window, the taskbar, and the compiled exe. No runtime generation, no PIL dependency, no external asset pipeline.

- **Dark red-team skin** — near-black base (`#0a0b0e`), crimson accent (`#ff3344`), mono typography throughout. Cards, accent bars, hover states, a status pill that tracks state with a coloured dot. The OS title bar is flipped to dark mode via a single `DwmSetWindowAttribute` call.

- **Config persistence** — API key, base URL, model, GitHub token, Pineapple creds, and Bruce creds all live in `~/.argusforge.json`. Loaded on start, saved on Settings dialog close. Corrupt file falls back to defaults instead of crashing.

- **Runtime hardening** — `hardening.py` runs anti-analysis guards on startup when the binary is compiled. Debugger detection (`IsDebuggerPresent`, `CheckRemoteDebuggerPresent`, `NtQueryInformationProcess` for `ProcessDebugPort` and `ProcessDebugFlags`), VM/sandbox heuristics (MAC OUI, registry markers, driver files), RE-tool process scan, and a timing check for single-stepping. Any trip exits silently with code `0xBEEF`.

- **Nuitka build, no bytecode on disk** — compile with the provided command and the shipped artifact is a native Windows binary. `pyinstxtractor` and `uncompyle6` are useless against it. Standard Nuitka already defeats the common RE pipeline; the hardening layer is the second fence.

- **XOR-hideable system prompt** — optional base64+XOR encoding path for the Repository prompt so `strings ArgusForge.exe | grep Repository` returns nothing. Decode path is built into the file; set `_SYSTEM_PROMPT_B64` and delete the raw literal when you want it.

---

## Repository Structure

```
argus-forge/
├── evilcreations.py          # Entry point. GUI, tabs, AI calls, Pineapple + Bruce deploy.
├── hardening.py              # Runtime anti-analysis guards (called from main).
├── ArgusForge.spec           # PyInstaller build spec (bundles splash/ + ui/).
│
├── splash/                   # Three.js boot splash (rendered in a WebView2 window).
│   ├── splash.html           # Particle field + glow + progress. Calls pywebview.api.finish().
│   ├── three.min.js          # Three.js runtime.
│   └── logo.png              # ARGUS FORGE logo.
│
├── ui/
│   └── ArgusForge.ico        # Window / taskbar / embedded exe icon.
│
├── assets/                   # Optional. Reference-repo style animated SVGs.
│   ├── banner-animated.svg   # Pure SMIL. No JS. Renders on github.com.
│   ├── logo-animated.svg     # Rotating rings / orbiting satellites.
│   └── divider-scan.svg      # Section divider.
│
├── .github/
│   ├── ISSUE_TEMPLATE/       # bug, feature, new-tab templates
│   ├── workflows/
│   │   └── build.yml         # Nuitka build + release artifact
│   └── PULL_REQUEST_TEMPLATE.md
│
├── LICENSE                   # MIT
└── README.md                 # You are here.
```

---

## Tab Reference

Flat catalog of every generation surface. Each tab is scoped to one engagement class and one output format. Both ship to the Pineapple under `/root/` and to Bruce devices under their native folders.

| # | Tab | Generates | Output | Ext | Pineapple Path | Bruce Path |
|---|-----|-----------|--------|-----|----------------|------------|
| 1 | **Evil Portal** | Captive-portal HTML/CSS/JS, login page clones, Flask/Node/PHP capture backends, deployment notes | Portal artifact | `.html` | `/root/portal_<ts>.html` | `/evilportal/portal_<ts>.html` |
| 2 | **Bad USB** | DuckyScript (Rubber Ducky / Hak5), Flipper Zero `.badusb`, O.MG cable payloads, companion PowerShell / bash stagers | Payload script | `.txt` | `/root/badusb_<ts>.txt` | `/badusb/badusb_<ts>.txt` |

**Note:** Both tabs read from the same Repository prompt but append a tab-scoped context line before the model sees the request. Portal requests are biased toward complete HTML with a working backend; BadUSB requests are biased toward target-specific execution chains and clean exits.

---

## Supported Deploy Targets

ARGUS FORGE pushes artifacts to two classes of hardware:

| Target | Auth | Transport | Files land at |
|--------|------|-----------|---------------|
| WiFi Pineapple Mark VII | Basic auth | REST API on `:1471` | `/root/` |
| Bruce firmware devices — M5Stack Cardputer / StickC, LilyGo T-Embed / T-Deck, ESP32-S3 boards | Device WebUI auth | HTTP upload to the Bruce web server (`/api/upload`) | `/evilportal/`, `/badusb/` |

Bruce firmware is a multi-tool suite for ESP32 hardware; ARGUS FORGE speaks to its built-in web interface for file drops. If you can reach it from your workstation, ARGUS FORGE can write to it.

---

## Status Indicators

| Signal | Status | Description |
|--------|--------|-------------|
| ✅ | Active | Shipped, tested, and part of the current build. |
| 🧪 | Experimental | Works, but the behaviour is subject to change between builds. |
| 🔧 | Build-only | Only fires in the compiled Nuitka binary; no-op when running `python evilcreations.py`. |
| 🚧 | Planned | Scoped, not implemented. See [Roadmap](#roadmap). |

**Feature status:**

| Feature | Status |
|---------|--------|
| Dual-tab generation (Evil Portal / Bad USB) | ✅ Active |
| OpenAI-compatible backend | ✅ Active |
| Repository system prompt | ✅ Active |
| Pineapple auto-deploy | ✅ Active |
| Bruce multi-device auto-deploy | ✅ Active |
| Animated Three.js splash | ✅ Active |
| Bundled icon (`ui/ArgusForge.ico`) | ✅ Active |
| Dark red-team skin | ✅ Active |
| Runtime hardening guards | 🔧 Build-only |
| XOR system-prompt encoding | 🧪 Experimental |
| Multi-tab expansion (WiFi scanning, BLE) | 🚧 Planned |

---

## Quick Start

Download the exe and run. Simple.

First launch opens a splash, then the main window. Hit **SETTINGS** in the top-right and fill in:

- **API Key** — your provider key (OpenAI, OpenRouter, Groq, whatever you're pointing at).
- **Base URL** — the API root. Examples:
  - OpenAI: `https://api.openai.com/v1`
  - OpenRouter: `https://openrouter.ai/api/v1`
  - Local llama.cpp / Ollama: `http://localhost:8080/v1`
- **Model** — e.g. `gpt-4o-mini`, `claude-sonnet-4`, `qwen2.5-coder-32b-instruct`.
- **Pineapple Host / User / Pass / Write Endpoint** — defaults target a Mark VII on `172.16.42.1:1471`.
- **Bruce Host / User / Pass** — point at the Bruce device's web interface (default `http://bruce.local`). Leave blank to skip Bruce on deploy.

Save. Type a prompt on either tab. Hit **GENERATE**. Hit **SEND // PINEAPPLE** (or **SEND // BRUCE**) when the output looks right.

---

## Configuration

Config lives at `~/.argusforge.json`:

```json
{
  "api_key": "sk-...",
  "base_url": "https://api.openai.com/v1",
  "model": "gpt-4o-mini",
  "github_token": "",
  "pineapple_host": "http://172.16.42.1:1471",
  "pineapple_user": "root",
  "pineapple_pass": "",
  "pineapple_write_endpoint": "/api/files/write",
  "bruce_host": "http://bruce.local",
  "bruce_user": "admin",
  "bruce_pass": "bruce"
}
```

Delete the file to reset to defaults. A corrupt file is caught and silently replaced with defaults on the next save — no crash, no partial-config limbo.

---

## Hardening

`hardening.py` fires only when `__compiled__` is set in the module globals — i.e. only in the Nuitka-built exe. During development, `python evilcreations.py` runs guard-free, so you don't fight your own anti-analysis.

Layers, in order:

1. **Debugger detection** — `IsDebuggerPresent`, `CheckRemoteDebuggerPresent`, and two `NtQueryInformationProcess` calls (`ProcessDebugPort = 0x07`, `ProcessDebugFlags = 0x1F`).
2. **VM / sandbox heuristics** — MAC OUI match against common hypervisors (VMware, VirtualBox, Hyper-V, Xen, QEMU), registry markers for guest tools, and driver files in `System32`.
3. **RE-tool process scan** — `tasklist` output checked against a list of common RE / packet-capture tool names.
4. **Timing check** — a fixed-cost native loop measured for a debugger-stepping signature. Only enabled when `aggressive=True`, which is only set from the compiled path.

Any trip calls `_fail()`, which redirects stdout/stderr to `os.devnull` and exits with code `0xBEEF`. No message. No hint.

**What this stops:** casual `strings` inspection, `pyinstxtractor` / `uncompyle6` (no bytecode to recover), debugger attachment, VM-sandbox automated analysis, and a good chunk of would-be RE traffic.

**What it doesn't stop:** a Ghidra session against the native binary, or a memory dump of the running process. The Repository prompt has to exist as a Python string in RAM at generation time — that's the fundamental limit. For Nuitka-grade obfuscation of constants, that's a Commercial-license feature and not part of this repo's free stack.

---

## Tech Stack

| Layer | Choice |
|-------|--------|
| Language | Python 3.10+ |
| GUI | Tkinter / ttk (stdlib) |
| Splash | pywebview (WebView2) + Three.js |
| HTTP client | `requests` |
| LLM backend | Any OpenAI-compatible `/v1/chat/completions` endpoint |
| Deploy targets | WiFi Pineapple Mark VII REST API · Bruce firmware HTTP interface (M5Stack, LilyGo, ESP32-S3) |
| Build | Nuitka (`--standalone --onefile`) |
| Icon | Bundled `ui/ArgusForge.ico` (no runtime generation, no PIL) |
| Hardening | `ctypes` against `kernel32`, `ntdll`, `winreg`, `uuid` |

---

## Roadmap

- [x] Dual-tab workbench (Evil Portal / Bad USB)
- [x] OpenAI-compatible backend
- [x] Repository system prompt + per-tab context injection
- [x] WiFi Pineapple auto-deploy
- [x] Bruce-compatible multi-device auto-deploy
- [x] Animated Three.js splash
- [x] Dark red-team skin
- [x] Bundled window/taskbar icon (`ui/ArgusForge.ico`)
- [x] Nuitka build pipeline
- [x] Runtime hardening guards
- [ ] XOR system-prompt encoding — shipped but undocumented in the default build
- [ ] Additional tabs (WiFi scanning surface, BLE payloads, shell stagers)
- [ ] Streaming token output on the current tab
- [ ] Payload history / recall across sessions
- [ ] Multi-target deploy queue (write to N Pineapples and N Bruce devices in sequence)
- [ ] Animated SVG banner + logo (assets/ — pure SMIL, no JavaScript)

---

## License

MIT. Do not do what you want because you cannot. Remember copyright, this tool has been fitted with security and watermarks.

---

<div align="center">

*Built for the operator who'd rather type a prompt than hand-write a DuckyScript.*

</div>
