# WOP — Water Our Plants (Base44 dev notes)

## What this is
A FastAPI backend serving a dark-themed web dashboard at `/wop` (static HTML/CSS/JS in `backend/app/static/`). SQLite for storage, WebSocket for live telemetry, optional Google Gemini for AI plant identification. BLE/Wi-Fi hardware bridge code exists but is **disabled** in `backend/app/main.py` (commented out in lifespan), so the app boots with no hardware.

## Running here
- `docker compose -f docker-compose.base44.yml up -d --build` — builds the image from `backend/Dockerfile` (installs bluez + pip deps), bind-mounts `./backend` over `/app`, and runs `uvicorn --reload` so source edits hot-reload.
- Web entry point is host port **3000** → container 8080. Root `/` 307-redirects to `/wop`.
- SQLite DB and uploaded images live in the `wop-data` named volume (`/app/data`), kept out of the repo.
- Healthcheck: `GET /wop` via python urllib inside the container.

## Secrets
- `GEMINI_API_KEY` (optional): enables AI plant identification. Without it, `plant_identifier.py` returns a fallback "Unknown Plant" identification and the app works normally. Delivered via `/run/base44/app.env`; never put it in compose `environment:`.

## Editing
- Frontend: `backend/app/static/{index.html,style.css,app.js}` — served as static files, no build step. Changes appear on reload.
- Backend: edit under `backend/app/`; uvicorn `--reload-dir app` picks up changes automatically.
- If `backend/requirements.txt` changes, rebuild: `docker compose -f docker-compose.base44.yml up -d --build`.

## Hardware note
BLE/ESP32 device connection and pump control endpoints exist but won't find real devices in the sandbox. The dashboard still loads and the plant CRUD + AI identification flow work.
