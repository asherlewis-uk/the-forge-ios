# The Forge — AI Narrative Engine

**The Forge** is a production-ready AI narrative engine that transforms roleplay chat logs into screenplay-style scenes with AI-generated imagery. It runs as a Next.js web app and is packaged as a native iOS app via Capacitor.

---

## Features

- **Chat Log Importer** — Parse Open WebUI JSON exports into screenplay scenes, automatically split by 2-hour conversation gaps.
- **Storyboard Navigator** — Browse, create, edit, reorder, and delete scenes in a visual grid layout.
- **Production View** — Edit scene text, tag characters, set mood, and manage scenes one at a time.
- **AI Image Generation** — Generate cinematic scene images via a locally-running Stable Diffusion instance (Automatic1111, Forge, or ComfyUI).
- **Stable Diffusion Prompt Builder** — Constructs rich, context-aware prompts from director style, scene composition, mood, and character visual keywords.
- **Story Engine** — Regenerate scene text using a locally-running Ollama LLM (e.g. `dolphin-mistral:7b`).
- **Character Engine** — Manage a cast of characters with roles, visual keywords, system prompts, avatars, and LoRA model weights.
- **Vision Scanning** — Auto-generate Stable Diffusion visual keywords from a character avatar image using an Ollama vision model (e.g. LLaVA).
- **Character Import** — Import character definitions from Open WebUI JSON exports, with optional avatar scanning.
- **PDF Export** — Export all scenes as a formatted director's script PDF.
- **Project Save / Load** — Export and import the full project (scenes, cast, settings) as a `.forge` JSON file.
- **LocalStorage Persistence** — Project data is automatically saved and restored between sessions.
- **Connection Guide** — Step-by-step in-app guide for connecting local Stable Diffusion and Ollama backends.

---

## Architecture

```
┌─────────────────────────────────────┐
│  Next.js Frontend  (port 3001)      │
│  app/page.tsx — main UI & logic     │
└────────────┬───────────────┬────────┘
             │               │
    HTTP POST│               │HTTP API
             ▼               ▼
┌────────────────┐  ┌──────────────────────┐
│  Python Bridge │  │  Stable Diffusion    │
│  server.py     │  │  (A1111/Forge/ComfyUI│
│  (port 8000)   │  │   port 7860)         │
└────────┬───────┘  └──────────────────────┘
         │
    Ollama Client
         │
┌────────▼───────┐
│  Ollama        │
│  (port 11434)  │
│  LLM + Vision  │
└────────────────┘
```

| Service | Default URL | Purpose |
|---------|-------------|---------|
| Next.js frontend | `http://localhost:3001` | Main app UI |
| Python bridge (`server.py`) | `http://localhost:8000` | Vision scanning, story generation |
| Stable Diffusion | `http://127.0.0.1:7860` | Scene image generation |
| Ollama | `http://localhost:11434` | LLM story rewriting, vision scanning |

---

## Getting Started

### 1. Frontend

```bash
npm install
npm run dev
```

Open [http://localhost:3001](http://localhost:3001) in your browser.

### 2. Python Backend

The Python backend bridges the frontend to Ollama for story generation and vision scanning.

```bash
pip install -r requirements.txt
python server.py
```

The server starts on port 8000.

### 3. Stable Diffusion

Run Automatic1111 or SD Forge with CORS enabled:

```
--listen --cors-allow-origins=*
```

Then enter your machine's IP address in the app's Settings panel.

### 4. Ollama

```bash
OLLAMA_HOST=0.0.0.0 ollama serve
```

Pull the default models:

```bash
ollama pull dolphin-mistral:7b   # story generation
ollama pull llava:latest         # vision scanning
```

Then enter your machine's IP address in the app's Settings panel.

---

## App Logic Overview

### Chat Log Parsing (`parseChatLog`)

Converts an Open WebUI JSON export into screenplay scenes:

1. Detects and extracts the messages array from multiple Open WebUI structures (flat array, `history.messages`, dictionary-keyed messages, bulk export arrays).
2. Normalises each message — extracting `role`, `content`, and `timestamp` (ISO 8601, Unix seconds, or index-based fallback).
3. Filters empty messages and sorts chronologically.
4. Chunks messages into scenes whenever the gap between consecutive messages exceeds **2 hours**.
5. Formats each scene as `[ROLE]: content` dialogue blocks.

### Stable Diffusion Prompt Builder (`constructSdPrompt`)

Combines multiple context layers into a single rich SD prompt:

```
Director Style | Scene composition: <first 2 sentences> | Mood: <mood> | Characters present: <names> | Visual details: <character keywords>
```

### Python Bridge (`server.py`)

A FastAPI server with two endpoints:

- `POST /scan-face` — Sends a character avatar image to an Ollama vision model and returns Stable Diffusion visual keywords.
- `POST /generate-story` — Sends a scene rewrite instruction and context to an Ollama LLM and returns the generated text.

### Project Persistence

- Project data (scenes, cast, settings) is serialised to `localStorage` on every state change and restored on load.
- Full projects can be exported as `.forge` files (JSON) and re-imported later.

---

## iOS Deployment

The app is packaged for iOS using [Capacitor](https://capacitorjs.com/).

```bash
npm run build        # builds the Next.js static export
npx cap sync ios     # syncs the web build into the iOS project
npx cap open ios     # opens Xcode
```

App ID: `uk.asherlewis.theforge`

---

## Project Structure

```
app/
  page.tsx          — Main application (all views and logic)
  layout.tsx        — Root layout with metadata and fonts
components/
  connection-guide.tsx  — In-app setup guide dialog
  ui/               — shadcn/ui component library
server.py           — Python FastAPI bridge (vision + story generation)
capacitor.config.ts — iOS packaging configuration
```

---

## Available Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start the Next.js development server |
| `npm run build` | Build for production / static export |
| `npm run lint` | Run ESLint |
| `python server.py` | Start the Python backend on port 8000 |
