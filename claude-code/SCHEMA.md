# `data/program.json` contract

The routine rewrites this file nightly. `index.html` fetches it at runtime and
never needs to change. Get this shape wrong and the app renders blank.

## Top level

```jsonc
{
  "meta":     { ... },              // header text, phase info, weekly rhythm
  "rotation": ["A", "B", "C", "D"], // drives the Program tab order
  "today":    { ... },              // which session is showing right now
  "sessions": { "A": {}, "B": {}, "C": {}, "D": {}, "REST": {} },
  "log":      []                    // unused by the app; leave as []
}
```

## `meta`

| Key | Type | Notes |
|---|---|---|
| `athlete` | string | Shown nowhere; kept for context |
| `goal` | string | Program tab |
| `level` | string | Program tab |
| `phase` | string | `"Phase 1 — Home Gym"`. The header pill strips the `"Phase N — "` prefix |
| `phaseNote` | string | One or two sentences on the phase's constraint |
| `phaseEnds` | string | `YYYY-MM-DD` |
| `nextPhase` | string | What comes after |
| `sessionTime` | string | `"17:30"` — the slot actually booked |
| `sessionWindow` | string | `"17:30–19:00"` — the target window |
| `duration` | string | `"75–90 min"` |
| `trainingDays` | string[] | `["Mon","Tue","Thu","Fri"]` |
| `restDays` | string[] | `["Wed","Sat","Sun"]` |
| `week` | number | Week number since starting |
| `weekLabel` | string | e.g. `"Week 3 — first load increase"` |
| `generated` | string | `YYYY-MM-DD` |

## `today`

```json
{ "date": "2026-08-21", "dayName": "Friday", "sessionId": "B", "isRest": false }
```

- `date` — `YYYY-MM-DD`. The app formats it for the header and keys logging off it.
- `sessionId` — must be a key in `sessions`.
- `isRest` — `true` forces the `REST` session regardless of `sessionId`.

## A session

```jsonc
{
  "id": "B",
  "name": "Legs",                       // the big H1
  "focus": "Quads · Hamstrings · Glutes · Calves",
  "accent": "legs",                     // push | legs | pull | full | rest
  "summary": "One or two sentences...", // the tinted banner under the header
  "warmup":   ["...", "..."],           // plain strings; [] hides the section
  "cooldown": ["...", "..."],
  "exercises": [ ... ]
}
```

`accent` **must** be one of `push`, `legs`, `pull`, `full`, `rest` — it maps to a
CSS custom property. Any other value leaves the page unstyled.

## An exercise

```jsonc
{
  "name": "DB Romanian Deadlift",
  "sets": "4",                 // string; parsed with parseInt to build set rows
  "reps": "12–15",
  "rest": "90 s",              // "—" hides the tag
  "tempo": "3-1-1",            // "—" hides the tag
  "load": "Start 14 kg/hand",  // "—" hides it from the subtitle
  "muscles": ["Hamstrings", "Glutes"],
  "how": "Plain-language description of the movement.",
  "cues": ["Cue one", "Cue two", "Cue three"],
  "swap": "Optional alternative if equipment is missing."  // omit if none
}
```

- `sets` drives how many logging rows render. `"4"` → four rows. A non-numeric
  value (`"3 rounds"`, `"—"`) renders no rows, which is correct for finishers
  and rest-day items.
- `cues` should be 3–4 entries. They're the main value of the app over a plain list.
- `swap` is optional — omit the key entirely rather than passing `null`.

## Rest days

Keep a `REST` entry present at all times. Its `exercises` carry optional recovery
items with `sets: "—"` so no logging rows appear. On a rest day set
`today.sessionId` to `"REST"` and `today.isRest` to `true`.

## Validate before committing

```bash
python3 -c "import json;d=json.load(open('data/program.json'));assert d['today']['sessionId'] in d['sessions'];assert d['sessions'][d['today']['sessionId']]['accent'] in {'push','legs','pull','full','rest'};print('ok',d['today'])"
```

## Checking it rendered

GitHub Pages takes about a minute. Hard-refresh on mobile, or append `?v=2` to
the URL — `data/program.json` is fetched with `cache: "no-store"`, but the HTML
shell itself can sit in the browser cache.
