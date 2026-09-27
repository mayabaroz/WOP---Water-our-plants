# AGENTS.md

## Running in Base44
- `docker compose -f docker-compose.base44.yml up -d`: runs a single FastAPI service (uvicorn `--reload`) straight from `./backend`. Container port 8080 is mapped to host port 3000.
- The dashboard is plain static HTML/JS/CSS in `backend/app/static/`, served at `/wop`. `/` redirects there. Nothing needs building, and edits show up after a browser refresh.
- SQLite DB and uploaded images live in the `wop-data` named volume (`/data`). Tables are created on startup. There are no migrations.
- `backend/docker-compose.yml` / `Dockerfile` are for the Raspberry Pi (host network, privileged, dbus for BLE). Don't use them here.
- BLE is disabled in `main.py`. ESP32 devices POST telemetry over Wi-Fi, so no hardware is needed to load the UI. The device list stays empty until a device reports in.
- `GEMINI_API_KEY` is optional (plant photo identification). Without it, identification is skipped.
- `backend/.env` is committed as UTF-16 and holds only a placeholder. The Base44 compose ignores it.

## Verify
- `curl -sL localhost:3000/` should return the dashboard HTML, and `curl localhost:3000/api/plants` should return JSON.
