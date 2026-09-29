# Agent notes (Base44 dev environment)

- Run: `docker compose -f docker-compose.base44.yml up -d`. Single FastAPI service (`backend/`) serves the API, WebSocket and the static dashboard; host port 3000 -> container 8080. `/` redirects to `/wop`.
- Source is bind-mounted; `uvicorn --reload` watches `backend/app`. Static files (`backend/app/static`) are served directly, so just refresh the browser.
- The repo's own `backend/docker-compose.yml` is for the Raspberry Pi (host networking, privileged, dbus for BLE) — don't use it here.
- No hardware needed: BLE bridge is disabled in `main.py`; ESP32 devices push telemetry over HTTP to the sensors router. Without a device, plants show as disconnected.
- SQLite DB + uploaded images live in the `wop-data` named volume (`/app/data`); tables are created on startup (no migrations).
- `GEMINI_API_KEY` is optional; `plant_identifier.py` returns a fallback identification when unset. `backend/.env` is a committed UTF-16 placeholder file and is not used by the Base44 compose.
- Health check: `GET /api/plants` returns 200 (JSON list).
