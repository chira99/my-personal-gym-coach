# Gym Coach

Mobile workout app. Hosted on GitHub Pages, served straight from `main`.

## How it updates

A nightly scheduled task ("Evening Gym Coach", 21:00 local) reviews the last three
sessions, runs a muscle-group coverage audit, programs the next session, and commits
an updated `data/program.json` to this repo. GitHub Pages redeploys automatically,
usually within a minute.

## Structure

- `index.html` — the whole app (inline CSS/JS, no build step)
- `data/program.json` — the workout data; **this is the only file the nightly task edits**
- `.nojekyll` — skips Jekyll processing so files are served verbatim

`index.html` fetches `data/program.json` at runtime with `cache: "no-store"`, and falls
back to an embedded copy of the same JSON if the fetch fails — so the app still renders
if opened offline or as a bare file.

## Schema

`data/program.json` holds `meta`, `rotation`, `today` and `sessions` (`A`/`B`/`C`/`D`/`REST`).
Each session carries `warmup`, `cooldown` and an `exercises` array, where every exercise has
`name`, `sets`, `reps`, `rest`, `tempo`, `load`, `muscles`, `how`, `cues` and an optional `swap`.
