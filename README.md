# Dynamic Tug Scheduling — Dispatching Demo

An interactive, browser-based simulation of a container port where a small fleet of tugboats must tow arriving ships from an offshore queue to crane berths and back out to sea. The demo lets you swap between four dispatching rules in real time, disturb the port with traffic surges, high wind and tug breakdowns, and run controlled benchmarks that race every rule against the same arrival stream.

The whole project is a single self-contained HTML file with no dependencies, no build step and no server.

---

## Quick start

**Run locally:** download or clone the repo and open `index.html` in any modern browser (Chrome, Edge, Firefox, Safari).

```bash
git clone https://github.com/<your-username>/tugboat-scheduler.git
cd tugboat-scheduler
open index.html        # macOS  (Windows: start index.html, Linux: xdg-open index.html)
```

**Host it online (GitHub Pages):** in the repository go to *Settings → Pages*, set the source to the `main` branch and `/ (root)` folder, and save. The demo will be live at `https://<your-username>.github.io/tugboat-scheduler/` after a minute or so.

---

## What the simulation models

The port is laid out left to right on a fixed 940 × 540 logical canvas:

| Element | Count | Behaviour |
|---|---|---|
| **Offshore queue** | 15 slots | Arriving ships sail to a free holding slot and wait for a tug. |
| **Tugs** | 3 (T1–T3) | Idle at the tug base; dispatched to fetch a queued ship, tow it to a berth, then later unberth it and tow it to the departure lane. |
| **Crane berths** | 3 (B1–B3) | Each has a different crane rate (2.2, 2.7 and 3.2 moves/s). A berth stays occupied until a tug has pulled the finished ship away. |
| **Ships** | variable | Carry 120–280 TEU, are *normal* or *urgent* (~25 %), and have a due time by which they should be berthed. |

Each ship moves through a state machine:

```
incoming → queued → assigned → towing → berthed → await_out → assigned_out → towing_out → departing → done
```

Environmental pressure comes from three sources. **Wind** above 0.62 halts all cranes (shown by the amber *HIGH WIND* banner), so offloading pauses while berths stay blocked. **Tug breakdowns** take a tug out of service for a period and hand its ship back to the queue. **Traffic level** (Light → Storm) controls the mean arrival interval and the maximum number of ships on screen.

---

## The scheduler "API"

The core design idea is that the dispatching logic is isolated behind a small, JSON-shaped interface, the same shape a real scheduling microservice would expose. Roughly every 1.1 simulated seconds the simulation:

1. **Builds a state snapshot** (`buildState()`), for example:

   ```json
   {
     "time": 42.3,
     "wind": 0.18,
     "waitingShips": [{ "id": "S7", "teu": 210, "priority": "urgent", "waited": 6.1, "deadline": 55.0 }],
     "berths":       [{ "id": "B2", "free": true, "craneRate": 2.7, "x": 820, "y": 270 }],
     "tugs":         [{ "id": "T1", "free": true, "x": 360, "y": 500 }]
   }
   ```

2. **Calls the scheduler** (`schedule(state, rule)`), which greedily picks the highest-priority ship, pairs it with its nearest free tug and the nearest free berth, and repeats until ships, tugs or berths run out.

3. **Applies the returned assignments** (`applyAssignments()`):

   ```json
   { "rule": "gp", "assignments": [{ "ship": "S7", "berth": "B2", "tug": "T1", "score": 0.4 }] }
   ```

Because the scheduler only sees the snapshot and only returns assignments, it could be replaced with a remote HTTP call to a real optimisation service without touching the movement or rendering code.

### Dispatching rules

| Rule | Priority score | Blind spot |
|---|---|---|
| **FCFS** | earliest arrival first | ignores deadlines and tug travel |
| **Earliest deadline (EDD)** | earliest due time first | ignores waiting time and distance |
| **Nearest tug** | shortest tug-to-ship distance first | ignores urgency |
| **GP-evolved ★** | `(w / p) · exp(−slack / (k·p̄)) · exp(−d / (kₜ·d̄))` | — |

The GP-evolved rule is an Apparent Tardiness Cost (ATC) style index extended with a travel term. It multiplies three factors: a **weight-over-processing-time** term that favours urgent ships (`w = 2.0`) and short offloads, a **deadline urgency** term that ramps up exponentially as a ship's slack shrinks, and a **routing** term that favours ships close to a free tug. Its coefficients live in the `GP` object (`k = 2.2`, `p̄ = 2.6`, `kₜ = 1.25`, `d̄ = 340`, `urgentW = 2.0`) and represent the output of a genetic-programming search. No GP training runs inside the demo; the evolved rule is applied as a fixed formula.

---

## Benchmark scenarios

The *Benchmark scenarios* panel offers five load levels. Selecting one generates a single random arrival stream from a seeded PRNG (`mulberry32`), then replays that **identical** stream through all four rules at accelerated speed so the comparison is fair.

| Scenario | Duration | Arrival gap | Wind profile | Breakdowns |
|---|---|---|---|---|
| Light | 82 s | 5.0–6.5 s | calm | 0 |
| Moderate | 88 s | 3.8–5.0 s | light breeze | 0 |
| Busy | 90 s | 3.0–4.0 s | one gust | 0 |
| Heavy | 92 s | 2.4–3.2 s | two gusts | 1 |
| Storm | 96 s | 1.9–2.6 s | sustained high wind | 2 |

When the four runs finish, the rules are ranked by a combined score (`on-time % − 1.2 × avg wait + 0.6 × ships served`) and shown in a results overlay with bars and a table.

### Live KPIs

The sidebar tracks, per rule: ships served, average wait (arrival to berth), on-time percentage (berthed before due time), throughput per minute, berth utilisation and tug idle time. The *Rule comparison* table accumulates these across the session so you can switch rules and compare.

---

## Controls

| Control | Effect |
|---|---|
| Scheduler buttons | Switch the active dispatching rule live |
| Pause / Step / Reset | Pause the loop, advance 0.4 s, or restart the world |
| Speed slider | 0.5×–4× simulation speed |
| + Normal / + Urgent ship | Spawn a ship manually (disabled when the queue is full) |
| Arrival surge | Spawn four ships at once |
| Break a tug | Take one tug out of service for 8 s |
| Traffic level | Light, Moderate, Busy, Heavy, Storm arrival rates |
| Wind slider | Set base wind; above 0.62 the cranes stop |
| Click a ship | Show its ID, priority, arrival and due time |
| ⛶ Exhibition mode | Full-screen kiosk view with a compact rule bar and a one-click Storm benchmark (Esc to exit) |

---

## Technology used

| Technology | How it is used |
|---|---|
| **HTML5** | Page structure: the canvas stage, sidebar cards, KPI grid, comparison table, benchmark overlay and exhibition panel. |
| **CSS3** | All styling is inline in a `<style>` block. Custom properties (`--cyan`, `--amber`, `--panel` …) define the colour theme; CSS Grid lays out the stage/sidebar and KPI tiles; Flexbox handles button rows; `clamp()` and a viewport-based `max-width` keep the canvas sized to the screen; a `body.exhibit` class switches the layout into kiosk mode. |
| **Canvas 2D API** | Draws every frame: the water grid, dashed queue slots, the wharf and cranes, rotating top-down ship and tug silhouettes built from paths and quadratic curves, tow lines, tug wake trails, offload progress rings and the wind gauge. The backing store is rescaled to `devicePixelRatio` so it stays sharp on high-DPI displays. |
| **Vanilla JavaScript (ES6+, strict mode)** | The entire engine — state, scheduler, physics, benchmark harness and UI wiring — with no frameworks or libraries. |
| **`requestAnimationFrame`** | Drives the main loop. Free-play mode advances by real elapsed time × speed; benchmark mode runs eight fixed 0.045 s sub-steps per frame so runs are fast and deterministic in step size. |
| **Seeded PRNG (mulberry32)** | Generates reproducible benchmark arrival streams so every rule faces exactly the same ships. |
| **Fullscreen API** | Used by exhibition mode to take the demo full-screen for displays and kiosks. |
| **DOM events** | Click handling on the canvas (hit-testing ships by converting screen to world coordinates), button and slider listeners, and the Esc key. |

### Simulation techniques

Movement uses simple kinematic steering: `moveToward()` advances an object along the straight line to its target at a fixed speed. Tugs use `tugMove()`, which adds a **separation force** from nearby tugs so convoys never overlap or deadlock. Ships use `blockedAhead()` for lightweight collision avoidance, pausing only when another *moving* ship is directly in front, and `turnToward()` to rotate smoothly between bow-in and bow-out headings. Outbound tugs release ships at staggered points in the departure lane so they don't converge on one spot.

---

## Project structure

```
tugboat-scheduler/
├── index.html    # the complete simulation (HTML + CSS + JS)
├── README.md
└── .gitignore
```

### Code map (inside `index.html`)

| Section | Key functions / objects |
|---|---|
| Layout anchors | `WHARF_X`, `BERTHS_Y`, `QUEUE`, `TUG_BASE`, `SPAWN` |
| State & config | `TRAFFIC`, `GP`, `DECISION_EVERY`, `RULES`, `FORMULAS` |
| World setup | `init()`, `spawnShip()` |
| Scheduler API | `buildState()`, `schedule()`, `applyAssignments()`, `assignOutbound()` |
| Movement | `moveToward()`, `tugMove()`, `blockedAhead()`, `turnToward()`, `update()` |
| Rendering | `draw()`, `rr()` |
| UI panels | `updatePanels()`, `updateShipInfo()` |
| Benchmark | `SCENARIOS`, `WIND`, `generateScenario()`, `startBenchmark()`, `renderResults()` |
| Main loop & events | `frame()`, `activateRule()`, exhibition mode handlers |

---

## Customising

Most behaviour is controlled by constants near the top of the script:

- **Scheduler tuning:** edit the `GP` object to change the evolved rule's coefficients.
- **Decision frequency:** `DECISION_EVERY` (default 1.1 sim-seconds).
- **Traffic presets:** `TRAFFIC` sets the mean arrival gap and on-screen cap per level.
- **Berth crane rates:** the `rate` array in `init()`.
- **Benchmark scenarios:** add or edit entries in `SCENARIOS` and wind profiles in `WIND`.
- **Adding a rule:** add a key to `RULES`, `RULE_NAMES` and `FORMULAS`, add a scoring branch in `schedule()`, and add matching buttons to the scheduler and exhibition panels.

---

## Version

Current version: **3.2**
