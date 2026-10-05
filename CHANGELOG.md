# Changelog

All notable changes to the Dynamic Tug Scheduling demo are recorded here.
Each version has a matching git tag (e.g. `v4.0`), and can be downloaded from the repository's Releases page.

## v4.5 — 2026-10-06

Traffic control: crossing tows and free tugs now give way instead of driving through each other. Smaller offshore queue, a proper departure zone, and a cleaner exhibition view.

### Added
- **Convoy yielding:** when an inbound and an outbound tow would cross, the convoy farther from the crossing stops short of the conflict zone (`computeYields()`, `CLEAR`, `HOLD_GAP`, `YIELD_MAX`), waits, then continues once the other has passed. A small *YIELD* label marks the waiting ship.
- **Free-tug give-way:** tugs returning to base or heading to pick up a ship wait for moving ships in their way (`freeTugMove()`, `TUG_CLEAR`, `TUG_HOLD`, `TUG_WAIT_MAX`). A small *WAIT* label marks a waiting tug.
- **Departure zone:** the band below the queue is now a labelled departure area with two parallel lanes (`DEPART_TOP`, `DEPART_LANES`, `laneY()`), drawn with direction arrows.
- **Exhibition auto-hide:** the floating controls and legend fade out after 2 s without mouse, touch or key input and return on any activity (`EXH_IDLE_MS`, `exhWake()`).
- `SHOW_CRANE_RATE` flag to show or hide each crane's speed label.

### Changed
- Offshore queue reduced from 15 slots (3 × 5) to **12 slots (3 × 4)**; the manual add-ship buttons now disable at 12 waiting ships.
- Ships without a queue slot no longer wander into the departure zone while waiting.
- Departing ships are released onto one of two lanes instead of a single line.
- While towing, tugs move in straight lines and ignore separation forces; tugs no longer push their own convoy mates away.
- After releasing a ship on the departure lane, a tug backs off sideways before heading home, and the departing ship holds until it is clear.
- Exhibition legend moved from the bottom centre to the bottom-right corner so it no longer covers the departure lanes.
- Crane labels now show only the berth name (B1–B3), centred on the wharf strip. The crane speed label is hidden by default.
- Page title now shows `· V_4_5`.

### Fixed
- Tugs visibly leaving a ship and rejoining it when two tows crossed.
- Free tugs passing through ships that were moving across their path, and tugs cutting across the hull of the ship they had just released.
- Crane label text and the "CRANE BERTHS" title being clipped by the right edge of the canvas.
- The exhibition-mode control panel covering the tug base and other movement.

## v4.0 — 2026-09-25

Multi-tug convoys: every ship now needs two tugs to move, and the fleet grows from 3 to 7.

### Added
- `NUM_TUGS` constant (default **7**) to set the fleet size.
- `TUGS_PER_MOVE` constant (default **2**): each inbound tow and each outbound unberthing requires a convoy of this many tugs.
- `selectTugs()` chooses which tugs form a convoy: the *Nearest tug* rule sends the closest free tugs; FCFS, EDD and GP-evolved pick free tugs at random.
- Convoy movement: tugs are spread laterally either side of the ship (`tugLateral()`), a lead tug drives the ship's position at the convoy's centre, and a move only advances once every tug in the convoy has reached its mark.
- Tow lines are now drawn for outbound (unberthing) tows as well as inbound ones.

### Changed
- **Scheduler API:** assignments now return a `tugs` array (e.g. `"tugs": ["T1", "T4"]`) instead of a single `tug` field. Ships store `tugs: []` instead of `tug`.
- The scheduler only dispatches when at least `TUGS_PER_MOVE` tugs are free; outbound ships wait at their berth until a full convoy is available.
- Tug base moved from the bottom of the harbour to the top (y = 48), clear of the departure lane.
- Departure-lane release points are now staggered per ship rather than per tug.
- Returning tugs head home independently of their former convoy.
- Page title now shows the version (`· V_4_0`).

### Fixed / behaviour
- Tug breakdowns now release the whole convoy: the other tugs return to base, an inbound ship reverts to the queue and frees its berth, and an outbound ship goes back to waiting at its berth.

## v3.2

Baseline version, first published to GitHub.

- Single-file HTML5 / Canvas simulation of a port with 3 tugs, 3 crane berths (2.2 / 2.7 / 3.2 moves/s) and a 15-slot offshore queue.
- One tug per ship movement, inbound and outbound.
- Four dispatching rules: FCFS, Earliest deadline, Nearest tug and a GP-evolved ATC-style index.
- JSON-shaped scheduler interface (`buildState()` → `schedule()` → `applyAssignments()`).
- Live controls: speed, pause/step/reset, manual and surge arrivals, tug breakdowns, five traffic levels and a wind slider (cranes halt above 0.62).
- Five seeded benchmark scenarios (Light → Storm) replaying an identical arrival stream through all four rules, with ranked results.
- Live KPIs and a per-session rule comparison table.
- Exhibition (full-screen kiosk) mode.
