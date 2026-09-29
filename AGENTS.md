# AGENTS.md

## Project Overview
WOP (Water Our Plants) — a smart plant watering system. FastAPI backend serving a static web dashboard + REST API + WebSocket on a single origin. SQLite for storage (no external DB). Originally designed for Raspberry Pi with BLE hardware; BLE is currently **disabled** (commented out in `app/main.py` lifespan) — telemetry now comes via ESP32 Wi-Fi POST to `/api/devices/{device_id}/readings`.

## Running the app
```bash
docker compose -f docker-compose.base44.yml up -d
```
- Single service (`wop`) on **host port 3000 → container 8080**.
- Base image `python:3.12-slim`; source bind-mounted at `/app`; deps installed on startup from `backend/requirements.txt`.
- Uvicorn runs with `--reload` watching `/app/app`, so backend edits hot-reload.
- SQLite DB and uploaded images persist in the `wop-data` Docker volume (`/app/data`).
- Dashboard served at `/wop` (root `/` redirects there). API under `/api/`.

## Key facts
- **Single-origin app**: frontend (`app/static/app.js`) uses relative URLs (`/api/...`, `/ws/...`), so no CORS or separate API origin is needed.
- **No secrets required to boot.** `GEMINI_API_KEY` is optional (defaults to `""`). Without it, the AI plant-identification feature returns an error but the app works. Provide it via the Base44 secrets dashboard if you want AI identification.
- `ble_bridge.py` imports `bleak` at module level (installed via requirements) but is never started, so no Bluetooth hardware/dbus is needed.
- Healthcheck: `python` urllib GET to `http://localhost:8080/` (expects 307/200). `curl` is not in the slim image.

## Where things live
- `backend/app/main.py` — FastAPI app, lifespan, static mounting, BLE/device endpoints.
- `backend/app/routers/` — `plants.py` (CRUD + AI), `sensors.py` (readings + WebSocket), `pump.py`.
- `backend/app/config.py` — pydantic-settings config (env prefix `WOP_`).
- `backend/app/database.py` — async SQLite via aiosqlite (singleton connection).
- `backend/app/static/` — dashboard HTML/CSS/JS (vanilla JS, Chart.js via CDN).
- `arduino/` — Arduino sketch (not run in this environment).
- `desktop/` — deprecated v1 Windows app (not run).
