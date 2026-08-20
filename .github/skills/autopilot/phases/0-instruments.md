# Phase 0 — Raising the instruments

Everything Phase 0 needs to know about the dashboard, and nothing else. **The rest of the instruments — the full `state.js` shape, the stage table, what the dashboard computes — lives in [`phases/7-instruments.md`](./7-instruments.md) and is read when the tickets are cut.** Reading it now buys nothing and costs the one context that is never refreshed.

Two files, and the split matters:

- **`.autopilot/state.js`** — the truth, and the only thing you ever write. You read it on resume; the user never opens it.
- **`.autopilot/dashboard.html`** — the only human view. Copied from the template once and **never touched again**. No build step, no dependencies, nothing to generate: in a real browser it opens by double-click, and in an in-app pane it needs one static file server and no more (§3).

The page loads `state.js` from beside it and re-loads that file every ten seconds on its own. So there is exactly one place where state lives, one write per update, and nothing that can drift out of sync — because there is no second copy to drift.

## 1. Copy the template

Never regenerate it, never read it into context, never edit it afterwards:

```powershell
Copy-Item "$env:USERPROFILE\.copilot\skills\autopilot\phases\dashboard-template.html" .autopilot\dashboard.html
```

The skill may be installed per-project instead — then it is `.github\skills\autopilot\phases\dashboard-template.html`. Try the project path first, the profile path second; one of the two exists.

## 2. Write `.autopilot/state.js`

First line exactly `window.STATE =`, then the state as ordinary indented JSON. Keeping the assignment on its own line is what makes an edit further down a small edit, and what lets the file be read as JSON without a parser of its own.

This is the whole file at the moment it is created — copy it and fill in what you know:

```js
window.STATE =
{
  "slug": "telegram-repair-bot",
  "title": "Telegram bot for repair requests",
  "mode": "semi",
  "depth": "normal",
  "polish": null,
  "tier": null,
  "briefFile": "2026-08-07-brief.md",
  "memoryFile": "AGENTS.md",
  "shell": "powershell",
  "startedAt": "2026-08-07T14:02:06+03:00",
  "updatedAt": "2026-08-07T14:02:06+03:00",
  "finishedAt": null,
  "stages": [
    { "id": "preflight", "status": "active", "startedAt": "2026-08-07T14:02:06+03:00" },
    { "id": "manifest",  "status": "pending" },
    { "id": "briefing",  "status": "pending" },
    { "id": "spec",      "status": "pending" },
    { "id": "plan",      "status": "pending" },
    { "id": "build",     "status": "pending" },
    { "id": "review",    "status": "pending" },
    { "id": "final",     "status": "pending" }
  ],
  "requirements": {
    "total": 0, "done": 0, "inTicket": 0, "inSpec": 0,
    "placeholder": 0, "deferred": 0, "dropped": 0
  },
  "tickets": [],
  "singlePass": null,
  "tests": null,
  "debt": { "placeholders": [], "assumptions": [], "emptyEnv": [] },
  "additions": [],
  "coverage": null,
  "blind": null
}
```

**All eight stages are listed from the first minute**, the seven unreached ones as `pending`. That is what makes the dashboard show the whole road instead of a blank page — the template renders every stage it is given and nothing it is not.

`tier` is `null` until Phase 4 decides it. `polish` stays `null` unless the polish parameter was requested — see [`phases/polish.md`](./polish.md), and do not add the block here on speculation.

**Never put a secret value in here.** `emptyEnv` holds names only — the whole point of the list.

**ISO 8601 with the offset**, always: `2026-08-07T14:02:06+03:00`. A bare `14:50` gives an invalid date and a dead dash on the dashboard. Read the clock at the moment the thing happens — `Get-Date -Format "yyyy-MM-ddTHH:mm:sszzz"` in PowerShell, `date -Iseconds` in bash, per the shell table in `SKILL.md` — **seconds are part of the answer**, and a column of times all ending in `:00` is the visible tell that they were written from memory.

## 3. Open it once, by you

The user should not have to be told where a file is and then go find it. **Open the dashboard yourself, immediately after the first write**, before Phase 1 asks anything.

**Wherever it opens, it keeps itself fresh** — you never refresh it, re-open it or re-point it. The page appends a fresh `<script src="state.js?t=…">` every ten seconds instead of reloading itself, which costs nothing, keeps the scroll position, and survives in panes that silence navigation.

**Hand it to the system browser.** One command, no server, no port, nothing left running:

```powershell
Start-Process .autopilot\dashboard.html
```

```bash
open .autopilot/dashboard.html 2>/dev/null \
  || xdg-open .autopilot/dashboard.html 2>/dev/null \
  || echo "open it yourself: .autopilot/dashboard.html"
```

Edge and Chrome open `file://` as a page and let it load `state.js` from the same directory, so the ten-second poll works with nothing running behind it. A background tab may be throttled to roughly one poll a minute — the data lags by a minute at worst; it does not freeze.

**Why not VS Code's Simple Browser**, which is the tempting answer since the dashboard would then sit in a tab beside the code. Two reasons, and the second is the one that wastes a run. It is opened by a VS Code command rather than a shell command, so you have no way to invoke it at all. And handed a `file://` path it inlines the HTML into a data URL, from which `<script src="state.js">` resolves to nothing and `window.STATE` stays `undefined` — the user watches the template's "Dashboard has no state yet" screen for the whole build, with a perfectly good `state.js` lying beside it. If the user opens Simple Browser themselves and wants the dashboard in it, it needs a static server first (`npx --yes serve .autopilot`): offer that when they ask, and never start it uninvited.

**Rules:**

- **Opened exactly once.** It keeps itself current. Never a second window or tab, never re-pointed.
- **Never on resume into a window that is already open.** On a resume, open it again only if the previous session had ended (`finishedAt` was set).
- **A failure is not an error.** No default browser, a headless machine — print the path in one line and carry on. Do not retry, do not install anything, do not try a second launcher.
- **Do not open it in a remote session.** `$SSH_CONNECTION` or `$CI` set, or a VS Code Remote / Codespaces window → skip opening and print the path. A browser window on the host's machine helps nobody.

Say it in one line, once, inside the opening block:

> Dashboard is open — `.autopilot\dashboard.html`, it refreshes itself.

## 4. The update ritual — the same three moves for the rest of the run

This is here rather than in [`phases/7-instruments.md`](./7-instruments.md) because **you will need it long after that file would have left your context**, and it is the whole of what most updates require:

| When | What |
|---|---|
| entering a phase | that stage → `active` + `startedAt`; the one you left → `done` + `finishedAt` |
| launching a ticket (or a whole wave) | those tickets → `in-progress` + `startedAt` **before** the subagent goes out |
| a ticket returns, review starts | that ticket → `review` |
| a finding goes back for repair | → `repair`, and `repairs` + 1 |
| committed | → `done` + `finishedAt` + tests + commit |

Every one of them: **edit the affected rows** of `state.js` and move `updatedAt`. Not a rewrite of the file — roughly thirty tokens, one tool call, and the screen follows within ten seconds wherever it is open. No mirroring, no second file, no re-opening anything.

- **`startedAt` on a ticket is that ticket's own launch time** — not the run's, not the build stage's. Copying the run's `startedAt` into a ticket is the one mistake that looks harmless and makes every per-ticket duration on the dashboard wrong from the first row.
- **`startedAt` goes in when the thing starts, not when it ends.** An interval with a start and no end is what makes the timer run; filling both in at the end means the user watched a frozen clock while the work was happening.
- **`updatedAt` moves on every write.** The dashboard shows "updated N ago" from it and marks it in warning colour after five silent minutes — that is the user's only way to tell "work is happening" from "the agent died".
- **Never touch `dashboard.html` after copying it**, and never hand-maintain a progress table in prose.

That is all of Phase 0's business with the instruments. When the tickets are cut, read [`phases/7-instruments.md`](./7-instruments.md) for the ticket shape and the rest of the reasoning.
