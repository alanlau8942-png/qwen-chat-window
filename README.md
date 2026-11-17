# Qwen Chat Window

A lightweight, single-file chat UI for [Ollama](https://ollama.com) and any OpenAI-compatible API. Built to talk to Qwen models running on free Kaggle GPUs, but it works with any Ollama endpoint — local, remote, or behind a tunnel.

一个轻量级单文件聊天窗口，用于 Ollama 和任何 OpenAI 兼容 API。最初为跑在免费 Kaggle GPU 上的 Qwen 模型而建，但可用于任何 Ollama 端点——本地、远程或经隧道暴露的都行。

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![Single file](https://img.shields.io/badge/Files-1-blue.svg)
![No build step](https://img.shields.io/badge/Build-None-orange.svg)

## Features

- **Single file, zero build step** — just open `index.html` in a browser. No server, no install, no framework.
- **Ollama-native + OpenAI-compatible** — talks to `/api/chat` (streaming) with automatic fallbacks: XHR stream → fetch stream → OpenAI-compatible SSE (`/v1/chat/completions`) → plain non-streaming request.
- **Thinking models, handled** — Qwen3 thinking traces are shown separately from the answer, with a live elapsed timer and thinking-token count.
- **Connection self-check** — on connect it sends a tiny real test message through the same channel and reports the first-byte latency, so you know immediately whether the channel works.
- **Model switcher** — one-tap switch between a main model and a backup (e.g. 27B ↔ 9B), plus a custom model field.
- **Resilient sends** — a 25s zero-data cutoff moves to the next transport automatically, with a 150s total cap and on-page error reporting instead of silent failure.
- **Mobile-friendly, dark mode** — responsive layout, works on phones, respects `prefers-color-scheme`.

## Quick start

1. **Run Ollama** somewhere reachable. For browsers, allow cross-origin requests:
   ```bash
   OLLAMA_ORIGINS=* ollama serve
   ```
   (Ollama rejects non-localhost `Host` headers by default; if you expose it through a tunnel, set the tunnel route's HTTP Host header to `localhost`.)

2. **Open `index.html`** in any modern browser (or serve the folder with any static server).

3. **Enter your endpoint and model**, e.g.:
   - Endpoint: `https://your-ollama-host/v1` (OpenAI-compatible) or the Ollama base URL
   - Model: `qwen3:8b`, `orcarouter/Qwen3.8-27B-Uncensored`, `lukey03/qwen3.5-9b-abliterated`, or any model Ollama has pulled.

4. **Chat.** The self-check runs on connect; if it passes, you are good to go.

## Why this exists

Running a large model on a free GPU (e.g. Kaggle's 2× T4, 30 h/week) is cheap, but the stock Ollama web UIs are heavy, and most chat front-ends assume a fast local model. This window was built for the slow, remote, thinking-model case:

- It never leaves you staring at a blank screen — every send shows status, elapsed time, and which transport it is on.
- It survives flaky tunnels: if streaming yields nothing for 25 s, it falls back to a plain request instead of hanging.
- It is one file you can drop anywhere: a phone, a tablet, a static host, or a USB stick.

## Configuration

All settings live in the top bar of the page:

| Setting | What it does |
|---|---|
| API 地址 (endpoint) | Base URL of your Ollama / OpenAI-compatible API |
| 模型 (model) | Model name as Ollama knows it (e.g. `orcarouter/Qwen3.8-27B-Uncensored`) |
| 备用模型 (backup) | One-tap fallback model |
| API Key | Optional; Ollama ignores it, but some clients require a non-empty value (any placeholder works) |

No data is sent anywhere except to the endpoint you configure. There is no telemetry, no account, and no server component.

## Known gotchas

- **CORS**: browsers block cross-origin calls unless the server sends `Access-Control-Allow-Origin`. Set `OLLAMA_ORIGINS=*` (or your page's origin) before starting Ollama.
- **Host header**: Ollama rejects requests whose `Host` is not `localhost`. Behind Cloudflare Tunnel or a reverse proxy, override the Host header to `localhost`.
- **Thinking models are slow**: a thinking model emits a long thinking trace before any visible answer. The window shows the thinking as it arrives; a full reply on a 27B model over dual T4s can take 45 s or more. For snappy replies, use a smaller model or a non-thinking one.
- **Session lifetime**: free GPU sessions (e.g. Kaggle) stop after ~60 min of notebook-kernel idle or 12 h max. API traffic does not count as kernel activity, so the tunnel can go down while you are still chatting — just restart the session and the tunnel.

## License

MIT. See [LICENSE](LICENSE).
