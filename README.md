# Nordschleife Racer

A browser-based arcade-sim racing game built from scratch in **TypeScript + Three.js** — a full lap of the Nürburgring Nordschleife with custom vehicle physics, real-time multiplayer lobbies, ghost replays, and a global leaderboard.

> **▶ Play it live: [yassin.app](https://yassin.app)** — open the site, then **Play Solo** to time-attack, or **Create Lobby** to race others in real time.

This repository is the **engine source** behind that game. It is the original racing code I wrote on top of my personal site — the physics, netcode, audio, track, and game systems are all mine. (See [About this repository](#about-this-repository) for scope and [Credits & licensing](#credits--licensing) for the third-party car meshes.)

---

## At a glance

- **~12,700 lines** of original TypeScript across a modular engine (a ~4,900-line vehicle physics core alone).
- **Custom vehicle physics**: raycast suspension, per-wheel ground probing, a speed-scaled drift model, airborne handling, and automatic fall/upside-down recovery.
- **Real per-car simulation**: 9 cars, each tuned to real-world drivetrain, gearing, mass, and top-speed data — an RWD M3 and an AWD AMG One feel genuinely different.
- **Real-time multiplayer** over Supabase Realtime (presence + broadcast) with live opponent cars.
- **Procedurally generated** Nordschleife track geometry and **procedurally synthesized** engine audio (no third-party track or audio samples).
- **Ghost replays** and a **global leaderboard** (Postgres with row-level security), with graceful solo/offline fallback.
- **Tested**: a Playwright regression harness drives the live game to validate physics behavior.

---

## About this repository

This is a **source + showcase** repo, not a standalone build:

- It contains the racing engine modules exactly as they run inside [yassin.app](https://yassin.app). A handful of imports (`Application`, `Resources`, `EventBus`) are the host-site interfaces the engine plugs into — they are intentionally **not** included here, because they belong to the portfolio shell, not to the racing game.
- **Car 3D models are not committed here.** They are third-party meshes (see [Credits](#credits--licensing)) loaded at runtime by the live site. This repo ships only my own code.
- To **play**, use the live build at **[yassin.app](https://yassin.app)** — the multiplayer and leaderboard need the deployed Supabase backend.

In other words: read the code here, play the game there.

---

## Architecture

The engine is split into focused, single-responsibility modules orchestrated by `RaceManager`.

| Module | File | Responsibility |
|--------|------|----------------|
| Race orchestration | `Racing/RaceManager.ts` | Wires every subsystem together; owns race state, mode (solo/multiplayer), and the update loop |
| Vehicle physics | `Racing/Vehicle/RaceVehicle.ts` | Suspension, grounding, steering, drift, gearing, recovery — the simulation core |
| Per-car tuning | `carOptions.ts` | Drivetrain, gear ratios, RPM band, mass, wheel radius, steer rates per car |
| Model loading | `World/Car.ts` | GLB loading, per-car material/paint/wheel handling, model caching |
| Track | `Racing/Track/NordschleifeTrack.ts` | Procedural Nordschleife geometry, surface sampling, checkpoints |
| Chase camera | `Racing/Camera/RaceChaseCamera.ts` | Speed-reactive follow camera |
| Input | `Racing/Input/DrivingInput.ts` | Keyboard/touch driving input |
| Lap timing | `Racing/Lap/LapTimer.ts` | Sector/lap timing and validation |
| Engine audio | `Racing/Audio/RaceEngineAudio.ts` | Procedural Web Audio engine sound from live telemetry |
| Drift effects | `Racing/Effects/DriftSmoke.ts` | Tire-smoke particle effect |
| Ghost replay | `Racing/Ghost/GhostReplay.ts` | Records and replays your best lap as a translucent ghost |
| Leaderboard (local) | `Racing/Leaderboard/LocalLeaderboard.ts` | `localStorage` leaderboard + offline fallback |
| Leaderboard (global) | `Racing/Leaderboard/LeaderboardService.ts` | Supabase-backed global leaderboard |
| Multiplayer | `Racing/Multiplayer/MultiplayerService.ts` | Supabase Realtime lobbies, presence, position broadcast |

---

## Vehicle physics

`RaceVehicle.ts` is a from-scratch arcade-sim model — not a physics-library wrapper. Highlights:

- **Raycast suspension & grounding** — each wheel probes the track via raycasts with suspension travel, anti-sink correction, and predictive high-speed look-ahead so the car hugs Nordschleife's elevation changes and cambered corners instead of clipping through them.
- **Surface-aware orientation** — body pitch/roll blend toward the road's surface normal and grade, speed-scaled so it's stable at low speed and responsive at high speed.
- **Drift model** — speed-gated drift entry/release with a visual slip angle up to ~68°, feeding tire smoke and audio slip.
- **Speed-scaled steering** — low-speed steer boost for tight hairpins, tightened response at high speed for stability.
- **Airborne handling** — reduced control authority while airborne (transient vs sustained), so jumps over crests behave plausibly.
- **Automatic recovery** — checkpoint-based fall recovery (with look-back history, cooldowns, and failure limits) and upside-down recovery, so a bad crash respawns you cleanly on the racing line.
- **Per-car drivetrain simulation** — gear ratios, final drive, idle/redline/shift RPM, brake decel, and RWD/AWD/FWD behavior are read per car from `carOptions.ts`.

### Car roster

Nine cars, each tuned from real published performance data (sources in [`CREDITS.md`](CREDITS.md)):

| Car | Drivetrain | Top speed (km/h) |
|-----|-----------|------------------|
| Mercedes-AMG One | AWD | 352 |
| Mercedes-AMG GT63 S Edition One | AWD | 315 |
| BMW F90 M5 Competition | AWD | 305 |
| BMW M8 Competition Coupe | AWD | 305 |
| Mercedes-AMG C63 S Coupe | RWD | 290 |
| Mercedes-AMG C63 507 | RWD | 280 |
| BMW E92 M3 | RWD | 250 |
| BMW F82 M4 | RWD | 250 |
| Toyota Crown Platinum | AWD | 208 |

---

## 3D models & asset pipeline

- Cars are **glTF/GLB** meshes, loaded and cached at runtime by `World/Car.ts`, which also handles per-car paint colors, glass tint, wheel-node mapping, and decal hints.
- Models are optimized through a **Blender → DRACO-compressed GLB** pipeline to keep download size and draw cost low for the web.
- **The car meshes are third-party assets and are not included in this repository.** They are sourced under Creative Commons (Sketchfab) and loaded by the live site only. See [Credits & licensing](#credits--licensing).

---

## Procedural track & audio

- **Track:** the Nordschleife geometry is **procedurally generated in-project** (`NordschleifeTrack.ts` + project-owned JSON), not imported from a third-party track asset.
- **Audio:** engine sound is **synthesized at runtime with the Web Audio API** (`RaceEngineAudio.ts`) — per-car frequency/gain profiles driven by live telemetry (RPM, throttle, gear, slip ratio, drift intensity, drivetrain). No audio samples are used.

---

## Multiplayer, leaderboard & ghosts

Backed by **Supabase**, with full graceful degradation — if no backend config is present, the game runs solo with a `localStorage` leaderboard.

- **Lobbies** (`MultiplayerService.ts`): Supabase Realtime **presence + broadcast**. Each player broadcasts car id, position, and orientation so opponents render as live cars on track.
- **Global leaderboard** (`LeaderboardService.ts`): Postgres table with **row-level security** — anon read + validated insert (name length, plausible lap-time bounds, car id) and **no service-role key in the client**. Schema and RLS policies in [`docs/RACING_SUPABASE.md`](docs/RACING_SUPABASE.md).
- **Ghost replay** (`GhostReplay.ts`): your best lap is sampled (~45 ms) as position+quaternion keyframes and replayed as a translucent ghost car to race against.

---

## Controls

| Action | Keys |
|--------|------|
| Accelerate / Brake-Reverse | `W` / `S` (or `↑` / `↓`) |
| Steer | `A` / `D` (or `←` / `→`) |
| (Touch) | On-screen controls on mobile |

Start from the site: **Play Solo** for time attack, **Create Lobby** to host multiplayer.

---

## Testing & tooling

- `scripts/run-regression-validation.js` — a **Playwright** harness that launches the live race and asserts physics/behavior (grounding, lap validity, recovery) against regressions.
- `scripts/game-audit.mjs` — automated game-state audit.
- `scripts/security-check.mjs` — security checks against the deployed site.
- `scripts/write-racing-config.js` — build-time generation of the runtime Supabase config from environment variables.

---

## Tech stack

**TypeScript**, **Three.js** (WebGL), **Web Audio API**, **Supabase** (Realtime + Postgres), GLB/glTF + **DRACO**, **Blender** (asset pipeline), **Playwright** (testing), Webpack.

---

## Credits & licensing

My original work in this repository (all racing engine code, the procedural track, the procedural audio, the multiplayer/leaderboard/ghost systems, and the tooling) is released under the [MIT License](LICENSE).

This game was originally developed inside a fork of [Henry Heffernan's portfolio](https://github.com/henryjeff/portfolio-website) (MIT); the inner desktop experience on the live site is [Dustin Brett's daedalOS](https://github.com/DustinBrett/daedalOS). **Neither is included or claimed here** — this repo is only the racing engine I authored.

**Car 3D models** are third-party assets (Creative Commons, via Sketchfab), used on the live site and **not redistributed in this repo**. Vehicle performance values are tuned from publicly published manufacturer/press data, cited in [`CREDITS.md`](CREDITS.md). Libraries (Three.js, Supabase JS) are MIT-licensed by their authors.

---

## Author

**Yassin Soliman** — [yassin.app](https://yassin.app) · [github.com/yassinsolim](https://github.com/yassinsolim)

Software Engineering @ University of Calgary.
