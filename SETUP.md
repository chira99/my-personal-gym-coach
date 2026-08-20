# Setup — publish to GitHub Pages

Two ways in. Both end with a live URL you can open on your phone.

## Option A — you already have git locally (keeps history)

```bash
unzip gym-coach.zip && cd gym-coach
git remote add origin https://github.com/chira99/my-personal-gym-coach.git
git push -u origin main
```

The `.git` folder is included with two commits already made, so this just works.
No remote is configured — nothing of yours is baked in.

## Option B — browser upload (no git needed)

1. Unzip the archive.
2. Go to https://github.com/chira99/my-personal-gym-coach
3. **Add file → Upload files**, drag in `index.html`, `.nojekyll`, `README.md`,
   and the `data` folder.
4. Commit to `main`.

Note: `.nojekyll` starts with a dot, so it may be hidden in your file manager.
On macOS press <kbd>Cmd</kbd>+<kbd>Shift</kbd>+<kbd>.</kbd> to reveal it. It matters —
without it GitHub runs Jekyll over the site, which can skip files.

## Then enable Pages

**Settings → Pages → Source: Deploy from a branch → `main` / `/ (root)` → Save**

Give it a minute. Your URL will be:

```
https://chira99.github.io/my-personal-gym-coach/
```

Open that on your phone and add it to your home screen — it's built mobile-first
and set up to run full-screen with no browser chrome.

## What you get

- **Today** — warm-up, every exercise with form cues, per-set logging, cool-down
- **Program** — the A/B/C/D rotation, weekly rhythm, phase notes, progression rules
- **History** — saved sessions, with a "Copy log for coach" button

Set logging is stored in your browser (`localStorage`), so it stays on whichever
device you use and survives closing the tab.

## Updating the workout

The app reads `data/program.json` at runtime. To change the workout, replace that
one file and commit — GitHub Pages redeploys in about a minute. `index.html` never
needs to change.

Ask Claude for an updated `data/program.json` whenever you want the app refreshed;
it can generate the file but cannot push to GitHub from its sandbox, so the commit
is yours to make.

Meanwhile your **Google Calendar event** and **Todoist task** are written fresh every
night by the Evening Gym Coach, and both carry the complete session with all cues.
Those are the ones that stay current on their own.

## Schema

`data/program.json`:

```
meta      — athlete, goal, phase, week, session window, training/rest days
rotation  — ["A","B","C","D"]
today     — { date, dayName, sessionId, isRest }
sessions  — A / B / C / D / REST, each with:
              name, focus, accent, summary, warmup[], cooldown[], exercises[]
```

Each exercise: `name`, `sets`, `reps`, `rest`, `tempo`, `load`, `muscles[]`,
`how`, `cues[]`, optional `swap`.

`accent` drives the colour theme and must be one of `push`, `legs`, `pull`,
`full`, `rest`.
