# AGENTS.md

## Base44 dev environment
- Run: `docker compose -f docker-compose.base44.yml up -d`. Single service: FastAPI (`backend/app`) on plain `python:3.12-slim`, `backend/` bind-mounted, `uvicorn --reload`. Host port 3000 -> container 8080.
- The dashboard is static HTML/JS in `backend/app/static`, served by FastAPI at `/wop` (`/` redirects there). It uses relative `/api/...` URLs and a same-origin WebSocket, so no proxy/CORS is needed. Static file edits show on browser refresh; Python edits auto-reload.
- SQLite DB and uploaded images live in the `wop-data` named volume (`WOP_DB_PATH=/data/wop.db`), not in the repo. Schema is created on startup (`database.init_db`); no migrations.
- BLE is disabled in `main.py` (project moved to ESP32 over Wi-Fi), so no Bluetooth/dbus/privileged mode is needed. `bleak` is still imported by `routers/sensors.py` but imports fine without a BT adapter.
- `GEMINI_API_KEY` is optional (plant photo identification); delivered via `/run/base44/app.env`. Don't use the committed `backend/.env` (UTF-16 encoded, meant for the Pi).
- No sensor data appears without real ESP32 hardware posting telemetry.
- Health check: `curl localhost:3000/api/plants` returns `[]` on a fresh DB.
