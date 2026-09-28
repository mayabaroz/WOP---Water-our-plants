# Base44 dev notes

- Only the FastAPI app in `backend/` runs here. It serves the API (`/api/*`), a WebSocket, and the static dashboard (`backend/app/static`, plain HTML/JS) at `/wop`; `/` redirects to `/wop`.
- `docker-compose.base44.yml` runs `python:3.12-slim` with `backend/` bind-mounted and `uvicorn --reload`, port 8080 mapped to 3000. Edits to static files take effect on a browser refresh; Python edits reload automatically.
- BLE is disabled in `main.py` (ESP32 devices now push telemetry over Wi-Fi), so no dbus/bluetooth/privileged mode is needed. `arduino/` and `desktop/` are not part of the running app.
- SQLite DB and uploaded images are stored in the `wop-data` named volume (`/data`), not in the repo. Tables are created on startup; there are no migrations.
- `GEMINI_API_KEY` is optional; without it, plant identification is skipped. `backend/.env` is committed with a placeholder in UTF-16 encoding. It is NOT used here, because the key comes from `/run/base44/app.env`.
- Quick check: `curl localhost:3000/api/plants` returns `[]` on a fresh DB.
