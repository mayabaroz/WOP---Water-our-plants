# WOP — Water Our Plants

## Overview
Smart plant watering system: FastAPI backend (Python) serving a single-page web dashboard
(plain HTML/CSS/JS in `backend/app/static/`) plus a REST API and WebSocket. Arduino/ESP32
hardware sends sensor telemetry over Wi-Fi. Optional Google Gemini AI identifies plants
from uploaded photos.

## Running in Base44
- `docker compose -f docker-compose.base44.yml up -d` brings up the backend on host port 3000.
- The compose uses `python:3.12-slim` (NOT the project's own Dockerfile, which bakes a
  production image). Source under `backend/app` is bind-mounted; uvicorn runs with
  `--reload --reload-dir /app/app`, so edits to Python files hot-reload.
- Frontend static files (`backend/app/static/*`) are served by FastAPI. Edits to HTML/CSS/JS
  are picked up on browser refresh (no build step) — call `reload_preview` after frontend edits.
- SQLite DB and uploaded images live in the `wop-data` Docker volume (persisted).

## Secrets
- `GEMINI_API_KEY` (optional): Google Gemini key for AI plant identification. Without it the
  app still boots and manual plant entry works; AI identification falls back to defaults.
  Delivered via `/run/base44/app.env` (platform-managed, outside the repo).

## Architecture notes
- Single FastAPI app serves API (`/api/...`), WebSocket (`/ws/...`), images (`/api/images/...`)
  and the dashboard (`/wop`, root redirects to `/wop`).
- `backend/app/routers/plants.py` — plant CRUD + AI identification on photo upload.
- `backend/app/plant_identifier.py` — Gemini integration; `_fallback_identification()` used
  when no key or on error.
- `backend/app/database.py` — async SQLite (aiosqlite); `init_db()` runs on startup and
  includes inline migrations (auto-water column, FK fixes).
- BLE bridge is disabled (ESP32 Wi-Fi architecture); devices register via `app/state.py`.

## Health check
- `GET /api/plants` returns the plant list (used by the compose healthcheck).

## Tests
- No automated test suite in the repo.
