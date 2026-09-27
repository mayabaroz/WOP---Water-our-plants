# WOP (Water Our Plants) — Agent Notes

- Single service: FastAPI backend (`backend/app`) serves both the REST/WebSocket API
  and the static dashboard (`backend/app/static/index.html`, `style.css`, `app.js`)
  via `StaticFiles`. There is no separate frontend build step.
- `GET /` redirects (307) to `/wop`, which serves the dashboard `index.html`. Use
  `/wop` for health checks, not `/`.
- Dev environment: `docker-compose.base44.yml` builds `backend/Dockerfile`, bind-mounts
  `backend/app` into the container, and runs `uvicorn --reload` on port 8080, mapped to
  host port 3000. Editing any file under `backend/app` (including the static assets)
  is picked up immediately — Python changes via the reloader, static files on next request.
- The repo's own `backend/docker-compose.yml` runs `network_mode: host` with
  `privileged: true` for real Bluetooth/BLE access on a Raspberry Pi — not usable/needed
  in this sandbox, so the Base44 compose file omits BLE/privileged mode entirely.
- `GEMINI_API_KEY` (plant identifier AI feature) is optional; the app boots fine without it.
- SQLite DB lives in the `wop-data` named volume at `/app/data/wop.db` inside the container.
