# Credits

## My original work (MIT — see LICENSE)

Everything in `src/Application/Racing/`, `src/Application/World/Car.ts`,
`src/Application/carOptions.ts`, and `scripts/` is original code by Yassin Soliman:
the vehicle physics, procedural track, procedural audio synthesis, multiplayer,
leaderboard, ghost-replay systems, and tooling.

- **Nordschleife track geometry** — procedurally generated in-project. Project-owned data.
- **Engine audio** — procedural Web Audio synthesis at runtime. No third-party samples.
- **Drift smoke, road/tire/wind layers, edge markings** — procedural/runtime-generated. No third-party assets.
- **Ghost replay data** — user gameplay telemetry (`localStorage`).

## Host application (not included in this repo)

This racing engine was originally built inside a fork of these MIT-licensed projects.
Neither is included here, and no authorship over them is claimed.

- **Portfolio shell** — Henry Heffernan, https://github.com/henryjeff/portfolio-website (MIT)
- **Inner desktop OS (on the live site)** — Dustin Brett / daedalOS, https://github.com/DustinBrett/daedalOS (MIT)

## Car 3D models (third-party — NOT redistributed in this repo)

The car meshes are third-party assets (Creative Commons, via Sketchfab), loaded by
the live site at runtime and **not committed to this repository**. If you redistribute
them anywhere, CC-BY requires crediting each original author.

> TODO (Yassin): fill in the Sketchfab author + URL + license for each model on the
> live site's CREDITS. Cars in the roster: Mercedes-AMG One, Mercedes-AMG GT63 S
> Edition One, BMW F90 M5 Competition, BMW M8 Competition Coupe, Mercedes-AMG C63 S
> Coupe, Mercedes-AMG C63 507, BMW E92 M3, BMW F82 M4, Toyota Crown Platinum.

## Vehicle performance reference sources (tuning inputs, not imported assets)

Real published specs used to tune each car's top speed / acceleration envelope:

- Mercedes-AMG ONE — https://www.mercedes-amg.com/en/home/vehicles/amg-one/hypercar.html
- BMW M3 Coupe (E92) — https://www.press.bmwgroup.com/middle-east/article/detail/T0048125EN/
- 2014 Mercedes-Benz C63 AMG Edition 507 — https://www.caranddriver.com/reviews/a15111205/
- 2019 Mercedes-AMG C63 — https://www.caranddriver.com/mercedes-amg/c63-2019
- 2015 BMW M4 Coupe — https://www.caranddriver.com/news/a15110475/
- 2023 Toyota Crown Platinum — https://www.caranddriver.com/reviews/a41711747/

## Libraries

- Three.js — https://github.com/mrdoob/three.js (MIT)
- Supabase JS — https://github.com/supabase/supabase-js (MIT)
