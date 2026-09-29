# AGENTS.md — WOP (Water Our Plants)

## What this app is
A smart plant watering system: FastAPI backend serving a static web dashboard + REST API + WebSocket, with SQLite for storage. Designed for Raspberry Pi with BLE hardware, but **BLE is disabled in code** (`app/main.py` lifespan — the `ble_bridge.start()` calls are commented out), so it runs fine in a plain container without Bluetooth.

## Architecture
- **Backend**: FastAPI (`app/main.py`), runs on port 8080 internally.
- **Dashboard**: Static HTML/CSS/JS in `app/static/`, served at `/wop` (root `/` redirects there).
- **Database**: File-based SQLite via `aiosqlite` (`app/database.py`). No external DB service needed. DB path and images dir are set via `WOP_DB_PATH` and `WOP_IMAGES_DIR` env vars.
- **AI plant ID**: Google Gemini (`app/plant_identifier.py`). **Optional** — if `GEMINI_API_KEY` is unset, a fallback identification is returned. Get a key at https://aistudio.google.com/apikey.

## Running in Base44
- `docker-compose.base44.yml` builds `backend/Dockerfile.base44` (python:3.12-slim + bluez libs for bleak imports + pip deps), bind-mounts `./backend` at `/app`, and runs `uvicorn --reload`.
- Port mapping: host `3000` → container `8080`.
- Healthcheck: `curl -fsS http://localhost:8080/wop`.
- Secrets: `GEMINI_API_KEY` (optional) delivered via `/run/base44/app.env`; `.env.base44-defaults` provides an empty placeholder so the app boots without it.

## Verify it works
```bash
docker compose -f docker-compose.base44.yml up -d --build
curl -s http://localhost:3000/wop        # dashboard HTML
curl -s http://localhost:3000/api/plants # → []
```

## Key files
- `backend/app/main.py` — FastAPI entry point, routes, lifespan (BLE disabled here)
- `backend/app/config.py` — pydantic-settings config (reads env vars)
- `backend/app/database.py` — SQLite schema + CRUD + auto-migrations
- `backend/app/static/` — dashboard frontend (index.html, app.js, style.css)
- `backend/app/plant_identifier.py` — Gemini AI integration (optional)
- `backend/app/ble_bridge.py` — BLE manager (imported but not started)
