# Changelog

All notable changes to the Dynamic Tug Scheduling demo are recorded here.
Each version has a matching git tag (e.g. `v4.0`), and can be downloaded from the repository's Releases page.

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
