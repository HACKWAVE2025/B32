Silent‑voice
============

Project Submitted in HackWave 2025 Sreenidhi
By Team Almost Optimized(B32)
Turn spoken or written words into an animated American Sign Language (ASL) overlay.

This repo contains:
- A Chrome extension that:
	- Renders ASL on YouTube watch pages using the video’s transcript.
	- Adds a context menu action “Show in Sign Language” so you can select text on any page and see it signed.
- A local Python server that converts text to pose frames via a small ASL database and fingerspelling fallback.

What’s included (current scope)
------------------------------
- YouTube support: a canvas overlay animates pose/hand/face landmarks derived from the transcript.
- Selected text → “Show in Sign Language”: quickly preview signs for any highlighted text in‑page.
- Server endpoint `/pose` that accepts `{ "words": "..." }` and returns a frame sequence.

Removed in this build: Google Meet/MiroTalk integrations and all video‑conferencing code.

How it works
------------
1. The extension requests frames from the local server:
	 - The server optionally rephrases English to ASL Gloss using Gemini (if `GEMINI_API_KEY` is set).
	 - It embeds each word (SentenceTransformers) and looks up the closest sign in Postgres + pgvector.
	 - If no close match is found, it falls back to per‑letter fingerspelling frames (A–Z).
2. The extension plays those frames on a lightweight canvas overlay at ~30 FPS.

Project structure (high‑level)
------------------------------
- `extension/` — MV3 extension (service worker + content script + static assets)
- `server/` — Flask server exposing `/pose`, Postgres integration, and data
- `client/` — Next.js app (not required for core extension usage)

Prerequisites
-------------
- Windows, macOS, or Linux
- Chrome (or Chromium) with Developer Mode for loading unpacked extensions
- Python 3.10+
- Node.js 18+
- PostgreSQL 14+ with pgvector extension

Server setup
------------
1) Environment variables (create a `.env` in `server/`)

```
POSTGRES_PASSWORD=your_postgres_password
# Optional, for ASL gloss rephrasing
GEMINI_API_KEY=your_gemini_key
```

2) Database

Create the database and enable pgvector. From `server/`:

```powershell
# PowerShell (adjust credentials/host as needed)
psql -U postgres -h localhost -f .\setup.sql
```

3) Python dependencies

The server uses Flask, CORS, pgvector, psycopg2, SentenceTransformers, and optional Gemini.

```powershell
cd .\server
python -m venv .venv
. .\.venv\Scripts\Activate.ps1
pip install flask flask-cors python-dotenv psycopg2-binary pgvector sentence-transformers google-generativeai
```

4) Run the server

```powershell
cd .\server
python .\server.py
```

By default it listens on `http://127.0.0.1:5000`.

Extension setup
---------------
Build and load the extension.

```powershell
cd .\extension
npm install
npm run repack
```

Then in Chrome:
- Go to `chrome://extensions`
- Enable “Developer mode”
- Click “Load unpacked” and select `extension/build`

Usage
-----
- YouTube: open a watch page (`https://www.youtube.com/watch?...`). The extension injects a canvas area in the sidebar and animates the sign frames for the current transcript segment.
- Selected text: on any page, select text, right‑click → “Show in Sign Language” to open a small overlay and play frames returned by your local server.

Configuration
-------------
- Server address: the extension calls `http://127.0.0.1:5000/pose` by default (see `extension/src/background.js`). If your server runs elsewhere, you can tweak the URL there and rebuild.
- Host permissions: `extension/public/manifest.json` includes localhost/127.0.0.1 so the service worker can fetch from your local server.
- Gemini rephrasing: set `GEMINI_API_KEY` to enable ASL Gloss rephrasing. If unset or rate‑limited, the server will fall back to direct words.

Demos
-----
Below are inline players pointing to the videos you added in the root `assets/` folder. If the GitHub preview doesn’t play them, use the fallback links beneath each player.

YouTube feature (demo1.mp4)

[Open demo1.mp4 directly](https://github.com/HACKWAVE2025/B32/blob/main/assests/demo1.mp4)

Selective search feature (demo2.mp4)

[Open demo2.mp4 directly](https://github.com/HACKWAVE2025/B32/blob/main/assests/demo2.mp4)

Screenshots
-----------
Add screenshots to visually document the current behavior.

- YouTube overlay (sidebar avatar)
<img width="432" height="316" alt="image" src="https://github.com/user-attachments/assets/f63635df-ad87-47e1-995a-212834279abe" />

```
![YouTube overlay](docs/screenshots/youtube-overlay.png)
```

- Context menu preview (any page → select text → “Show in Sign Language”)
<img width="344" height="476" alt="image" src="https://github.com/user-attachments/assets/9bad684f-c394-4f0d-8ebf-330c0cd579d6" />

```
![Context menu overlay](docs/screenshots/context-menu-overlay.png)
```

Troubleshooting
---------------
- “Could not reach local server”: ensure `python server.py` is running and the extension can reach `http://127.0.0.1:5000/pose`.
- Postgres errors: confirm `setup.sql` ran and `POSTGRES_PASSWORD` matches your local credentials.
- Empty or choppy animation: for unknown words the server falls back to fingerspelling; ensure `server/data/alphabets` is present.
- GPU/Canvas artifacts: reduce FPS or canvas size in the content script if needed.


