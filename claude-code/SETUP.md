# Running the gym coach as a Claude Code routine

## Why this works better than the cloud sandbox

The Cowork sandbox this was built in sits behind a proxy that blocks every
external host without a first-party connector. Netlify was unreachable; pushing
to GitHub was refused by a git proxy that would not forward a personal access
token. That's what forced the app's workout data to be updated by hand.

Claude Code runs on your machine, with your network and your git credentials.
The routine can `git push` directly, GitHub Pages redeploys, and the app updates
itself every night. Same design, without the wall.

---

## 1. Prerequisites

**Repo cloned locally**

```bash
git clone https://github.com/chira99/my-personal-gym-coach.git
cd my-personal-gym-coach
```

Confirm you can push before wiring up anything automated — this is the step that
failed in the sandbox, so prove it works here:

```bash
git commit --allow-empty -m "push test" && git push
```

If that prompts for credentials, set up a credential helper or SSH remote first.
A routine can't answer a password prompt.

**GitHub Pages enabled**

*Settings → Pages → Source: Deploy from a branch → `main` / `/ (root)`*

**MCP servers connected in Claude Code**

The routine needs two. Add them with `claude mcp add`, or via your MCP config:

| Server | Used for |
|---|---|
| Todoist | reading session history and comments; writing the daily task |
| Google Calendar | finding a free evening slot; writing the event |

Verify with `/mcp` inside Claude Code before scheduling anything. If Todoist
isn't connected the coverage audit has nothing to read and the routine will
program blind.

---

## 2. Create the routine

Schedule it for **21:00 Asia/Colombo**. Cron in UTC:

```
30 15 * * *
```

Take the prompt from `ROUTINE-PROMPT.md`, replace `<REPO_PATH>` and `<PAGES_URL>`,
and paste it in.

> **When you move to Ireland**, change the cron to `0 20 * * *` (21:00
> Europe/Dublin, UTC+1). Cron is evaluated in UTC and won't follow you.

---

## 3. Reference values

| Thing | Value |
|---|---|
| Todoist Health project ID | `6hJ7XMG39g9F8vc3` |
| Google Calendar ID | `chiranthajk@gmail.com` |
| Calendar colorId | `10` (green) |
| Todoist label | `workout` |
| Todoist priority | `p2` |
| Session window | 17:30–19:00, never earlier than 17:30 |
| Session length | 90 min (45 min fallback) |
| Training days | Mon, Tue, Thu, Fri |
| Rest days | Wed, Sat, Sun |
| Rotation | A Push → B Legs → C Pull → D Full Body |

---

## 4. Test before trusting it

Run the prompt manually in Claude Code first, as a normal message. Then check:

- [ ] A calendar event exists for tomorrow, 17:30 or later, 90 minutes
- [ ] Its description is plain text — no visible `<b>` or `&lt;`
- [ ] A Todoist task exists in Health, due tomorrow, labelled `workout`
- [ ] `git log -1` shows a new commit touching only `data/program.json`
- [ ] The JSON validator in `SCHEMA.md` passes
- [ ] The Pages URL shows tomorrow's session after ~1 minute
- [ ] The summary names which muscle groups the audit flagged

The plain-text check matters. HTML entities in a calendar description render
literally and make the event unreadable — it's the one failure that looks fine
in the tool response and broken on your phone.

---

## 5. Feeding it good data

The coverage audit is only as good as its input. Without logged sets it can see
*which* sessions you completed but not how much work each muscle got, so it
progresses conservatively and guesses at gaps.

After a session: **Copy log for coach** in the app → paste as a comment on that
day's Todoist task. That turns the audit from an estimate into a real tally.
The routine will nag you about this whenever comments are empty.

---

## 6. If you also run a morning brief

Tell it to leave workouts alone, or you'll get duplicate health tasks — the brief
fires in the morning, before the coach has written anything, so it won't see the
workout and will invent its own. Add to that routine's prompt:

> A separate "Evening Gym Coach" routine owns all training. Never suggest, time,
> reschedule, or create a Todoist task for a workout, gym session, or exercise
> block. If a workout already exists on the calendar or in Todoist, treat it as a
> fixed commitment you may reference but must not modify. Treat 17:30–19:00 on
> weekdays as reserved.

---

## 7. Files

```
index.html              the whole app — inline CSS/JS, no build step
data/program.json       the only file the routine edits
.nojekyll               stops GitHub running Jekyll over the site
claude-code/
  SETUP.md              this file
  ROUTINE-PROMPT.md     the prompt to paste
  SCHEMA.md             program.json contract — the routine reads this
```

`index.html` embeds a copy of the program JSON as a fallback, so the app still
renders if the fetch fails or you open the file directly off disk. That copy goes
stale by design; the fetched `data/program.json` always wins when it loads.
