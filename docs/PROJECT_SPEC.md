# Camino — Travel Planner Project Spec

A node-based, AI-assisted travel planner.
---

## 1. Product Vision 

The user answers a few questions about how they like to travel, gets an AI-generated day-by-day itinerary rendered as draggable nodes on a timeline/map, can swap any node for alternatives, sees live status for flights/trains, tracks a shared budget with friends, and never once feels like they're filling out a form.

---

## 2. High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                        FRONTEND                          │
│  React + TypeScript + Tailwind + Framer Motion           │
│  React Flow (canvas)  │  Google Maps JS (map view)       │
│  Zustand (state)      │  React Query (server cache)      │
└───────────────────────────┬───────────────────────────────┘
                             │ REST + WebSocket
┌───────────────────────────┴───────────────────────────────┐
│                        BACKEND                            │
│  FastAPI (Python)                                          │
│  ├─ /trips        CRUD for trips, preferences              │
│  ├─ /nodes        CRUD for itinerary nodes                 │
│  ├─ /generate     AI itinerary generation                  │
│  ├─ /places       Google Places proxy + caching            │
│  ├─ /tracking     Flight/train/transit status proxy        │
│  ├─ /expenses     Budget + splitting                       │
│  └─ /ws           WebSocket for live collab (Phase 3)      │
└───────────────────────────┬───────────────────────────────┘
                             │
┌───────────────────────────┴───────────────────────────────┐
│                     DATA + EXTERNAL                        │
│  PostgreSQL (Supabase)  │  Redis (cache, Phase 2+)          │
│  Google Places/Maps API │  Claude API (planning brain)      │
│  AirLabs (flights)      │  RailRadar (Indian trains)         │
│  GTFS-realtime (metro)  │  OpenWeather                       │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Repo Structure

```
camino/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── canvas/           # React Flow canvas, custom node types
│   │   │   │   ├── NodeCanvas.tsx
│   │   │   │   ├── nodes/
│   │   │   │   │   ├── FoodNode.tsx
│   │   │   │   │   ├── HotelNode.tsx
│   │   │   │   │   ├── ActivityNode.tsx
│   │   │   │   │   ├── TransportNode.tsx
│   │   │   │   │   └── BaseNode.tsx      # shared card shell
│   │   │   │   └── edges/
│   │   │   │       └── TravelTimeEdge.tsx # custom edge showing transit time
│   │   │   ├── map/
│   │   │   │   ├── TripMap.tsx           # Google Maps view, numbered pins
│   │   │   │   └── RoutePolyline.tsx
│   │   │   ├── onboarding/
│   │   │   │   ├── OnboardingFlow.tsx    # multi-step wizard shell
│   │   │   │   ├── steps/
│   │   │   │   │   ├── DestinationStep.tsx
│   │   │   │   │   ├── BudgetStep.tsx
│   │   │   │   │   ├── InterestsStep.tsx
│   │   │   │   │   ├── DiningStyleStep.tsx
│   │   │   │   │   ├── HotelTierStep.tsx
│   │   │   │   │   ├── MobilityStep.tsx
│   │   │   │   │   ├── TravelingWithStep.tsx
│   │   │   │   │   └── MustAvoidsStep.tsx
│   │   │   ├── sidepanel/
│   │   │   │   ├── NodeDetailPanel.tsx
│   │   │   │   ├── PhotoGallery.tsx
│   │   │   │   └── AlternativesPicker.tsx  # "flexible mode" swap options
│   │   │   ├── tracking/
│   │   │   │   ├── FlightStatusCard.tsx
│   │   │   │   ├── TrainStatusCard.tsx
│   │   │   │   └── TransitStatusCard.tsx
│   │   │   ├── budget/
│   │   │   │   ├── BudgetDashboard.tsx
│   │   │   │   ├── ExpenseList.tsx
│   │   │   │   └── SplitCalculator.tsx
│   │   │   ├── checklist/
│   │   │   │   ├── ChecklistPanel.tsx
│   │   │   │   └── PackingListGenerator.tsx
│   │   │   ├── assistant/
│   │   │   │   └── AIAssistantChat.tsx
│   │   │   └── shared/
│   │   │       ├── Button.tsx, Card.tsx, Slider.tsx, ChipSelect.tsx, etc.
│   │   ├── store/
│   │   │   ├── tripStore.ts              # Zustand: current trip state
│   │   │   ├── canvasStore.ts            # node positions, selection
│   │   │   └── preferencesStore.ts
│   │   ├── api/
│   │   │   ├── client.ts                 # axios/fetch wrapper
│   │   │   ├── trips.ts, nodes.ts, tracking.ts, expenses.ts
│   │   ├── hooks/
│   │   │   ├── useAutosave.ts
│   │   │   ├── useLiveTracking.ts        # geolocation, phase 3
│   │   │   └── useOptimizeRoute.ts
│   │   ├── types/
│   │   │   └── index.ts                  # shared TS interfaces
│   │   └── App.tsx
│   └── package.json
│
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── routers/
│   │   │   ├── trips.py
│   │   │   ├── nodes.py
│   │   │   ├── generate.py               # AI itinerary generation endpoint
│   │   │   ├── places.py                 # Google Places proxy
│   │   │   ├── tracking.py               # flight/train/transit proxy
│   │   │   ├── expenses.py
│   │   │   ├── checklists.py
│   │   │   └── ws.py                     # websocket collab (phase 3)
│   │   ├── services/
│   │   │   ├── constraint_engine.py      # hours/budget/time feasibility rules
│   │   │   ├── route_optimizer.py        # nearest-neighbor / 2-opt
│   │   │   ├── ai_planner.py             # prompt construction + Claude call
│   │   │   ├── places_client.py
│   │   │   ├── flight_client.py          # AirLabs wrapper
│   │   │   ├── train_client.py           # RailRadar wrapper
│   │   │   ├── transit_client.py         # GTFS-realtime wrapper
│   │   │   ├── currency_client.py
│   │   │   └── weather_client.py
│   │   ├── models/
│   │   │   └── (SQLAlchemy models — see schema below)
│   │   ├── schemas/
│   │   │   └── (Pydantic request/response schemas)
│   │   └── db.py
│   ├── requirements.txt
│   └── alembic/                          # DB migrations
│
├── docs/
│   ├── PROJECT_SPEC.md
│   ├── API.md
│   └── demo.gif
├── .env.example
├── docker-compose.yml                    # local Postgres + Redis
└── README.md
```

---

## 4. Data Model

```
trips
  id, owner_id, title, destination, start_date, end_date,
  budget_amount, budget_currency, budget_flexibility (strict|some|flexible),
  pace (relaxed|balanced|packed), created_at, updated_at

trip_members                    -- Phase 3, multi-user
  trip_id, user_id, role (owner|editor|viewer)

preferences
  trip_id, interests[] (chips), dining_style (luxury|street|no_pref|mood_based),
  hotel_star_min, hotel_star_max, mobility (walk_ok|limited|accessible_needed),
  traveling_with[] (partner|kids|pet|solo|friends), must_avoids[]

nodes
  id, trip_id, day_number, order_index, category (food|hotel|activity|transport),
  place_id (google), title, lat, lng, start_time, end_time,
  estimated_cost, currency, photo_url, notes,
  status (planned|confirmed|skipped),
  alternatives[]  (jsonb — for flexible mode swap options)

node_edges                      -- connections between nodes on canvas
  id, from_node_id, to_node_id, travel_mode, travel_time_minutes, distance_km

transport_legs                  -- flights/trains/buses as their own entities
  id, trip_id, node_id, mode (flight|train|bus|metro),
  identifier (flight/train number), origin, destination,
  scheduled_departure, scheduled_arrival, live_status (jsonb, polled)

expenses
  id, trip_id, node_id (nullable), paid_by_user_id, amount, currency,
  category, split_type (equal|custom), split_details (jsonb)

checklists
  id, trip_id, type (packing|todo|shopping|custom), title

checklist_items
  id, checklist_id, text, is_checked, auto_generated (bool)

notes
  id, trip_id, node_id (nullable), content, created_by
```

---

## 5. Node Types on the Canvas

Each is a custom React Flow node component sharing a `BaseNode` shell (rounded card, cover photo, category color strip, drag handle) but with type-specific content:

- **FoodNode** — cuisine tag, price tier, open-now indicator
- **HotelNode** — star rating, price/night, amenities icons
- **ActivityNode** — duration estimate, indoor/outdoor tag (for weather logic)
- **TransportNode** — mode icon, live status badge (on-time/delayed/boarding), connects two location nodes

Edges between nodes are custom (`TravelTimeEdge`) — a labeled line showing transit time, color-shifts to amber/red if the gap is tight.

---

## 6. Feature Breakdown by Version

### 🟢 Phase 1 — MVP (weeks 1–5)

| # | Feature | Key implementation notes |
|---|---|---|
| 1 | Onboarding flow | Multi-step wizard, one question per screen, progress bar, animated transitions |
| 2 | Destination + dates + inter-city travel time | Google Distance Matrix for the "how do I get there" leg |
| 3 | Preferences: chips, dining Q&A (not slider), hotel star-range filter | Dining = single-select 4 options; hotel = min/max star range slider |
| 4 | Budget: number + flexibility level | e.g. `{amount: 2000, currency: "USD", flexibility: "some"}` |
| 5 | AI itinerary generation | Backend filters Places results → sends to Claude → structured JSON back |
| 6 | Node canvas with day swimlanes, drag-to-reorder | React Flow, custom layout algorithm (see §8) |
| 7 | Side panel per node (photo, hours, price, address) | Google Places Details + Photos API |
| 8 | Constraint engine v1: opening hours, travel-time-between-nodes, basic budget check | Rules-based, runs after AI generation to validate/flag |
| 9 | Map view with numbered pins + route polyline | Google Maps JS API, marker numbers = itinerary order |
| 10 | Auto-save | Debounced PATCH on any node/preference change |
| 11 | Cover photos on nodes | Pull top photo from Google Places Photos API |
| 12 | Deploy + demo | Vercel (frontend) + Railway (backend) + Supabase (DB) |

**Definition of done for Phase 1:** a stranger can enter a destination, answer some quick questions, get a real AI-generated itinerary rendered as draggable cards synced to a map, and it doesn't fall apart if they drag things around.

---

### 🟡 Phase 2 — Depth & Delight (weeks 6–9)

| # | Feature | Key implementation notes |
|---|---|---|
| 13 | Flexible mode: 2–3 alternative options per node | Store `alternatives[]` on the node (jsonb); "swap" UI in side panel |
| 14 | Quick recommendations sidebar | Nearby + high-rated + matches-preferences + passes constraint engine |
| 15 | Optimize route button | Nearest-neighbor heuristic in `route_optimizer.py`, optionally 2-opt pass for quality |
| 16 | Currency conversion for running totals | exchangerate-api.com, cache rates for 24h |
| 17 | Checklists (packing/todo/shopping) + trip notes | Simple CRUD, packing list auto-seeded from weather + activity types |
| 18 | Flight tracking (real) | AirLabs API, poll every few minutes while trip is "upcoming/active" |
| 19 | Train tracking (Indian Railways) | RailRadar API |
| 20 | Metro/bus tracking (major cities only) | GTFS-realtime for 2–3 demo cities (e.g. NYC MTA, DC WMATA); manual entry fallback elsewhere |
| 21 | Weather-aware swaps | If rain forecast, flag outdoor nodes and suggest indoor alternative from `alternatives[]` |

**Definition of done for Phase 2:** the app now *reasons* — it optimizes, adapts to weather, tracks real transport, and offers real choices instead of one fixed plan.

---

### 🔵 Phase 3 — Collaboration & Intelligence (weeks 10–14)

| # | Feature | Key implementation notes |
|---|---|---|
| 22 | Multi-user collaborative editing | Socket.io or Yjs (CRDT) for conflict-free simultaneous node edits |
| 23 | Expense tracking + splitting | Per-node costs, "who owes whom" settlement calculation |
| 24 | Live "am I on track" tracker | Browser Geolocation API, compares current position/time to next node's window |
| 25 | AI assistant chat | Sidebar chat, function-calling into your own `/nodes` and `/generate` endpoints so it can actually modify the trip |
| 26 | Paste-link / paste-email import | LLM extracts place names or reservation details from pasted text |
| 27 | Reservation display (non-live modes) | Manual entry UI for bus/train info in unsupported cities, styled identically to live cards |

**Definition of done for Phase 3:** multiple friends can plan the same trip together in real time, split costs, and get nudged by the app while actually traveling.

---

## 7. AI Integration Details

**Itinerary generation prompt structure (backend, not exposed to user):**
```
System: You are a travel planning assistant. Given user preferences and a
list of candidate places (already filtered for opening hours and budget
tier), produce a day-by-day itinerary. Respond ONLY with valid JSON
matching this schema: [...]

User message includes:
- Trip dates, destination, pace
- Preferences object (interests, dining style, hotel tier, mobility, etc.)
- Candidate places array (from Places API, pre-filtered by constraint engine)
- Budget + flexibility
```

**AI assistant (Phase 3)** uses Claude's tool-calling: expose your own `add_node`, `remove_node`, `move_node`, `swap_node` functions as tools, so "move dinner to 8pm instead" becomes a real, safe, structured edit rather than the model trying to freeform-edit JSON.

---

## 8. The Canvas Layout Algorithm 

1. Group nodes by `day_number` → each day is a horizontal row (swimlane)
2. Within a day, sort by `order_index`, position left-to-right
3. On drag-drop: recalculate `order_index` for affected nodes, debounce a save
4. Cross-day drag: update both `day_number` and `order_index`
5. Render `TravelTimeEdge` between consecutive nodes in the same day, colored by feasibility (green = comfortable, amber = tight, red = infeasible)

---

## 9. Route Optimization

```python
# route_optimizer.py — conceptual outline
def optimize_day(nodes: list[Node]) -> list[Node]:
    # 1. Keep fixed-time nodes anchored (e.g. a 7pm dinner reservation)
    # 2. Nearest-neighbor greedy pass on the remaining flexible nodes
    # 3. Optional 2-opt improvement pass (swap pairs if it reduces total travel time)
    # 4. Re-check opening-hours feasibility after reordering
    ...
```

---

## 10. Design System

- **Palette:** one warm accent (terracotta `#E07856` or teal `#2A9D8F`), neutral sand/off-white background, near-black text — not pure black
- **Typography:** one display font for headings (e.g. `Fraunces` or `Cabinet Grotesk`), one clean sans for body (e.g. `Inter`)
- **Cards:** rounded-2xl, soft shadow, cover photo top, no harsh borders
- **Motion:** 150–250ms ease-out transitions on drag, hover, panel open
- **Category colors:** food = warm orange, hotel = deep blue, activity = green, transport = purple — consistent across nodes, map pins, and sidebar

---