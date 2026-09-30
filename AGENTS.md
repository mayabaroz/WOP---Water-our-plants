# AGENTS.md — WOP (Water Our Plants)

Notes for working on this repo in the Base44 sandbox.

## What this app is
A FastAPI backend (`backend/`) serving a REST API + WebSocket + a static web dashboard
(`backend/app/static/`). ESP32 hardware sends telemetry over Wi-Fi (not BLE — the BLE
bridge is disabled in `app/main.py`'s lifespan). SQLite via `aiosqlite` stores plants and
sensor readings. Optional Google Gemini integration identifies plants from photos.

## Why the repo's own compose fails to start here
`backend/docker-compose.yml` uses `network_mode: host` + `privileged: true` for Raspberry Pi
Bluetooth/dbus access, and the `backend/Dockerfile` bakes source with `COPY app/` (no live
reload). None of that works in the sandbox. The Base44 dev compose
(`docker-compose.base44.yml`) instead runs `python:3.12-slim`, bind-mounts `backend/` at
`/app`, installs `requirements.txt` on startup, and runs `uvicorn --reload` on port 8080
mapped to host 3000. BLE is disabled in the lifespan so no bluez/dbus deps are needed.

## Run / verify
- Start: `docker compose -f docker-compose.base44.yml up -d`
- Health: container healthcheck curls `http://localhost:8080/wop` inside the container.
- Preview: dashboard is served at `/wop` (root `/` redirects there) on port 3000.
- Data (SQLite db + uploaded images) persists in the `wop_data` named volume at `/app/data`.

## Secrets
`GEMINI_API_KEY` is optional for boot (defaults to empty; AI plant ID falls back gracefully).
Delivered via `/run/base44/app.env`; a placeholder lives in `.env.base44-defaults`.

## Commit 029c96a ("Added auto-watering and few ui fixes")
The auto-watering changes (new `auto_water` column + migration in `database.py`, model
fields, plants/sensors router logic) are valid and do not block startup. The startup
failure was environmental (host networking / privileged mode), not a code defect.
