# Gym Coach

Mobile workout app for Chirantha. Hosted on Netlify, deployed continuously from this repo.

## How it updates

A nightly scheduled task ("Evening Gym Coach", 21:00 local) reviews the last three
sessions, programs the next one, and commits an updated `data/program.json` to this
repo. Netlify picks up the commit and redeploys automatically.

## Structure

- `index.html` — the whole app (inline CSS/JS, no build step)
- `data/program.json` — the workout data; **this is the only file the nightly task edits**
- `netlify.toml` — publishes the repo root, disables caching on `data/`

`index.html` fetches `data/program.json` at runtime and falls back to an embedded
copy of the same JSON if the fetch fails, so the app still renders if opened offline
or as a bare file.
