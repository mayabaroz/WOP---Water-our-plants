# WOP — Water Our Plants

## Overview
Smart plant watering system: FastAPI backend serving a web dashboard, SQLite storage, optional Gemini AI plant identification, and (disabled) BLE bridge for ESP32/Arduino hardware.

## Running in Base44
- `docker compose -f docker-compose.base44.yml up -d --build` — builds `backend/Dockerfile.dev`, bind-mounts `backend/` to `/app`, runs uvicorn with `--reload` on port 3000.
- Dashboard: `http://localhost:3000/wop` (root `/` redirects there).
- API base: `http://localhost:3000/api/` — endpoints for plants, sensors, pump, BLE/device status.
- SQLite DB and uploaded images persist in the `wop-data` named volume (`/app/data`).

## Key facts
- BLE bridge is **disabled** in code (commented out in `lifespan`). The app runs without Bluetooth hardware.
- `GEMINI_API_KEY` is **optional** — without it, plant identification returns a fallback ("Unknown Plant") and the app boots normally. To enable AI identification, set the key via the dashboard secrets.
- The repo's own `backend/.env` is UTF-16 encoded and contains only a placeholder; Base44 uses `.env.base44-defaults` + `/run/base44/app.env` instead.
- Config is in `backend/app/config.py` (pydantic-settings, `WOP_` env prefix).

## Project structure
- `backend/app/main.py` — FastAPI app, lifespan, static serving, BLE/device endpoints.
- `backend/app/routers/` — plants, sensors, pump API routes.
- `backend/app/static/` — dashboard HTML/CSS/JS (vanilla, no build step).
- `backend/app/database.py` — async SQLite (aiosqlite), includes inline migrations.
- `backend/app/plant_identifier.py` — Gemini AI integration with fallback.
- `arduino/` — Arduino sketches (not part of the backend).
- `desktop/` — deprecated v1 Windows app (not used).
