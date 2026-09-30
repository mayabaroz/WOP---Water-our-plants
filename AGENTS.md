# WOP — agent notes

- The whole app is one FastAPI service in `backend/` (API + WebSocket + static dashboard in `backend/app/static`). No build step; the dashboard is plain HTML/JS served at `/wop` (`/` redirects there).
- Dev run: `docker compose -f docker-compose.base44.yml up -d`. Container port 8080 → host 3000. Uvicorn `--reload` watches `backend/app`; static file edits need only a browser refresh.
- `backend/docker-compose.yml` / `Dockerfile` are the Raspberry Pi production setup (host networking, privileged, BLE via dbus) — don't use them in the sandbox.
- Python deps are installed on container start into the `wop-venv` volume. SQLite DB and uploaded images live in the `wop-data` volume (`WOP_DB_PATH=/data/wop.db`), tables are created automatically on startup.
- Settings are read from env via `os.getenv` in `app/config.py`; `backend/.env` is NOT loaded by the Base44 setup. `GEMINI_API_KEY` (optional, AI plant ID) comes from `/run/base44/app.env`.
- No hardware here: devices (ESP32 over Wi-Fi) post telemetry to the sensors API; without them the dashboard shows plants with no live data. BLE code (`ble_bridge.py`) is imported but not started.
- Verify: `curl localhost:3000/api/plants` returns JSON; `curl localhost:3000/wop` returns the dashboard HTML.
