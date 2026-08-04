# Camino

**Camino** (Spanish for "the way" / "the path") is a node-based, AI-assisted travel planner. Answer a few questions about how you like to travel, get an AI-generated itinerary rendered as a draggable node canvas, and watch your route come together like pins on a map.

Think n8n's canvas, built for travel planning instead of automation workflows.

> 🚧 **Status:** actively in development. See [Roadmap](#roadmap) below for what's built vs. planned.

## Why

Most travel planners are either a bare checklist or a rigid wizard. Camino treats a trip like a graph: each stop — a meal, a hotel, a hike, a flight — is a node you can drag, reorder, swap, and connect, while the app handles the tedious constraint-checking (opening hours, travel time, budget) in the background.

## Tech Stack

- **Frontend:** React, TypeScript, Tailwind CSS, React Flow, Framer Motion, Zustand
- **Backend (from v2):** FastAPI (Python), PostgreSQL
- **AI:** Claude API for itinerary generation
- **Live data (from v2):** Google Places/Maps, AirLabs (flights), RailRadar (Indian trains), GTFS-realtime (metro/bus)

Full architecture and phased feature plan: see [`docs/PROJECT_SPEC.md`](docs/PROJECT_SPEC.md).

## Roadmap

- [x] **v1 — Static prototype:** onboarding flow + draggable node canvas with mock data (no backend)
- [ ] **v2 — MVP:** real backend, AI itinerary generation, live Places data, map view, auto-save
- [ ] **v3 — Depth:** flexible node alternatives, route optimization, live flight/train tracking, currency conversion, checklists
- [ ] **v4 — Collaboration:** multi-user real-time editing, expense splitting, live trip tracking, AI assistant chat

## Getting Started

See [`frontend/README.md`](frontend/README.md) for local setup instructions.

## License

MIT — see [LICENSE](LICENSE).