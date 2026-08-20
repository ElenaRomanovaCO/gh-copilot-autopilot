# Phase 8 — Touchdown

Landing. Two things happen here, and the first one is the reason this framework exists. A third — the polish loop — happens between them, and only when the `polish` parameter is on.

## 1. Blind acceptance — gate G4

Every check so far has measured the build against the **spec**. But the spec is your own paraphrase of the brief, written several phases ago. If a requirement was lost on the way into it, everything downstream has been faithfully confirming that loss.

So the last check does not use the spec.

**Spawn an `autopilot-blind-check` subagent that receives:**

- `.autopilot/<slug>/<date>-brief.md` — the user's own words (the path is `briefFile` in `state.js`)
- the repository as it now stands
- how to run the project and its tests

**It must not receive:** `spec.md`, `manifest.md`, the tickets, or any summary of them. A checker given the spec inherits the spec's blind spots and will confirm them. Independence is the entire mechanism — take it away and this phase is theatre.

Its brief:

> Read the attached brief file — it is the task the client set. Then study
> the repository and determine what of it is actually implemented.
>
> **Run the project** — the commands are in the attached description — and walk the main
> scenario the way the client would. Reading the code shows intent; running shows the result.
> If the project does not start or the scenario breaks off — that is the main finding
> of this check, put it first. If it cannot be run at all (an account, a key,
> an external service is needed) — say plainly what got in the way, and do not pass off
> a reading of the code as a check that it works.
>
> For each requirement in the brief: implemented / partial / no — and one line
> on where exactly that is visible (what you saw when running, or where it is
> in the code, or why you decided it is absent).
>
> Do not judge code quality. Do not propose improvements. Do not look for excuses
> for what is missing — just record the fact. If a requirement is formally met
> but does not work in substance (the data is saved but never shown to the user) —
> that is "partial", not "implemented".

**Then compare its verdict with `manifest.md`:**

| Manifest says | Blind says | Meaning |
|---|---|---|
| `done` | implemented | agreed |
| `done` | **partial / no** | 🔴 **drift** — the manifest is wrong. Report it, and fix it if it is small |
| `placeholder` | partial | expected — confirm the placeholder is visible, not an invented fact |
| `dropped` / `deferred` | no | expected — must appear in the report as not built |
| — | implemented, but not from the brief | scope that grew without a parent; report it |

Every 🔴 goes in the report **and** in `state.js` under `blind`. A drift found here is not a failure of the run — it is the run working. Hiding it is the failure.

If there are no tickets (tier T0), this check still runs. Small builds drift too, and it is one subagent.

**A build that was never run is a build nobody has seen work.** The tests were written by the same process that wrote the code, so they agree with it by construction; the first time this project meets a user must not be the first time it is launched. If it genuinely cannot be run here — no credentials, a service that needs an account, a platform this machine is not — that goes in the report as an open item under "what I need from you", not silently into the accepted column.

## 2. What outlives the run — memory and decisions

**Launch these at the same time as the blind acceptance.** Up to three subagents in one slot, no contact between them, each answering a different question:

| Agent | Question | Receives | Never receives |
|---|---|---|---|
| blind checker | what of the brief is done | the brief, the repo | `spec.md`, `manifest.md`, tickets |
| memory | how to use this tomorrow | the repo, `interfaces.md`, the memory file, the tier | `spec.md`, tickets |
| ADR *(tier T2+)* | why it was done this way | `spec.md`, `manifest.md` | the repo — it documents decisions, not code |

All three are `autopilot-blind-check`, launched in one message with three different briefs. It is the read-only agent, and here that is not merely a safety rail: **none of these three writes anything.** Each returns its text and you place it — the memory block between the markers, the ADRs into `docs/adr/`. That is inside your hands as `SKILL.md` defines them, and it keeps the one agent deliberately blind to the plan from also being the one holding a keyboard in the repo.

The memory agent composes the full description of the project for `AGENTS.md` — architecture, key files, conventions, environment, tests, gotchas — scaled to the tier, folding in what `interfaces.md` accumulated. Like the blind checker, **it does not receive `spec.md` or the tickets**: a memory written from the plan documents intentions, and the next session has no way to tell the difference.

The ADR agent is the mirror image and that is why it cannot be the same one. **`spec.md` dies with the run**, and with it every "why this way" in it — the reason for the data model, what the build proved wrong at ticket four, which word the project uses for which thing. Six months later the next session reads working code and no reason for any of it, and re-opens decisions that were settled here. At tier T2+ that is worth three files in `docs/adr/`; below it, the memory file carries what little there is.

Everything about all of this — which memory file, the markers, the sections per tier, what an ADR contains, and the verification pass over the commands — is in [`phases/9-memory.md`](./9-memory.md). Read it before spawning.

This is the artifact that decides what the *next* run costs. A project whose second session begins by re-reading the whole codebase paid for that in the first session and got nothing.

## 2a. Polish — only with `polish` on

If the run has the `polish` parameter, the loop goes **here**: after the blind acceptance has said what is and is not built, and before the report describes the result. Both halves of that matter. The blind verdict is the baseline the regression rule compares against, and a report written before the loop describes a build that no longer exists.

Read [`phases/polish.md`](./polish.md) now — and only now. On a run without the parameter, skip this section entirely and do not read the file.

Without `polish`, nothing changes: the blind checker's findings go into the report as open items, exactly as below.

## 3. The final report

Run the full test suite once more first — truncated per the shell table in `SKILL.md`, for the same reason as after every ticket — and wait for both subagents. Then write in the user's language, plain, no jargon.

### Where every line of it comes from

**Re-read the files. Do not write this from memory.**

By the time you reach this phase your context is the most polluted it has been all run — the brief, the manifest, the briefing, the spec, every phase file, and the returns of every subagent, most of it compacted at least once. The report is the one artifact the user actually reads, and writing it from that is how a `deferred` requirement gets reported as done, a placeholder disappears, and an `A##` nobody ordered turns up in the summary as if they had asked for it.

So build each section from its source, opened now:

| Section | Read from |
|---|---|
| Decisions made for you | `manifest.md` — every `ASSUMPTION` in Basis |
| Done | the blind checker's return, not the manifest's `done` rows |
| Polish *(only with `polish`)* | `state.js` → `polish`, and `reference.md` for what was compared against |
| What I need from you | `manifest.md` `placeholder` rows + `state.js` → `debt` |
| What didn't make it in | `manifest.md` `deferred` and `dropped` rows, with their quotes |
| What I added beyond the order | `state.js` → `additions`, cross-checked against `A##` in the spec |
| What didn't go to plan | every `D##` row in `manifest.md` |
| Open questions | `state.js` → `blind`, plus anything in `coverage` that ended up not built |
| Run it / Where things are | `state.js` → `memoryFile`, `briefFile`, and the commands the memory agent verified |

Two of these are worth naming, because memory gets them wrong in a specific direction. **"Done" comes from the blind checker, not from your own bookkeeping** — the manifest says what you believe was delivered, and the whole point of the previous section is that those two can disagree. And **"What didn't make it in" comes from the rows, not from recollection**: a requirement dropped in the first ten minutes of a three-hour run is exactly the one you will not remember, and it is quoted in the file.

Order matters — the user reads the top and skims the rest.

**In full mode, the report opens with "Decisions made for you"** — every `ASSUMPTION` from the self-briefing, in plain language, each with the one-line reason. They never asked for these; they have the right to see all of them in one place, first.

```markdown
## Done

<What works now — 3–6 lines in plain language, from the user's point of view.>

**Run it:**
```
npm install && npm run dev
```
Open http://localhost:3000

## What I need from you

1. Fill in `.env` — `TELEGRAM_BOT_TOKEN`, `GOOGLE_SHEETS_ID`.
   The file `.env.example` is already there; copy it and fill it in.
2. Replace the stubs: the prices in `src/data/prices.ts`, the email copy
   in `src/emails/`. Right now they are visible `[FILL IN]` labels, not invented values.

## What didn't make it in

| What | Why |
|---|---|
| SMS notifications | you said "no SMS, just Telegram" |
| Admin panel for requests | deferred: requests are visible in the sheet, a separate screen is the next run |

## What I added beyond the order

<Every `A##` story that reached the code — in plain language, with the requirement
it was added for. The section is omitted only if there were no additions
(at `strict` depth — always). The user should learn about them from here,
not by stumbling on them in the code.>

| What I added | For the sake of |
|---|---|
| Request number in the confirmation | so the client can refer to it — R01 |

## What didn't go to plan

<Every `D##` row from the manifest — in plain language: what was intended,
what got in the way, and what was done instead. The section is omitted only if
there were no `D##`. The requirement stays the same — the method changes, not the order.>

| What didn't work | What was done |
|---|---|
| One request per address — half the clients have two addresses | Addresses moved into a list, the form takes several |

## Open questions

<The blind acceptance's disagreements, if any. Straight, no softening:
"The requirement 'the client sees the status' I considered done; the independent
check showed the status is saved but displayed nowhere. Fixed /
needs a separate ticket."

Also here — what the coverage check on the spec found that ended up
NOT built. What was found and built is not mentioned here: the gate
did its work, the user has nothing to act on.>

## Where things are

- The project description for next time — `AGENTS.md` at the root
- Why it was done this way — `docs/adr/` (if the project is large)
- Progress and the numbers — `.autopilot/dashboard.html`
- Your original task — `.autopilot/<slug>/<date>-brief.md`
- The requirements and their fate — `.autopilot/<slug>/manifest.md`
- The specification — `.autopilot/<slug>/spec.md`
```

## Rules for the report

- **Placeholders and empty variables are a mandatory section**, even if there are zero (then one line: "everything is filled in"). This is what separates "works" from "works for you".
- **Secrets — by name only.** Never by value, including the ones the user sent themselves.
- **"What didn't make it in" is always written**, even when everything made it in. An empty section with one line is more honest than a missing one: it shows the question was asked.
- **No embellishing.** A failed test, an unfinished ticket, a discovered disagreement — named plainly, with what exactly is broken and what fixing it needs. A report that hides a defect costs more than the defect.
- **No diffs, no code file names, no test names** — they are in the instruments, for those who need them.

## Closing the instruments

The memory file goes in with the final commit, before this. Then, in `state.js` and nowhere else: set `finishedAt`, write the `blind` block, refresh the counts, close every stage — `final` to `done`, and anything still `active` or `pending` to `done`, `skipped` (with a note) or `failed`, whichever is true. A run whose dashboard says "in progress" a day after it landed is lying to the person who trusted it.

The open page picks this up by itself within ten seconds — this is the picture the user is left with, and it arrives without you doing anything more.

`finishedAt` also stops the clocks and the ten-second polling: the page freezes on the final numbers instead of counting time nobody is spending. Leave it `null` on a finished run and the user's total keeps growing overnight.

**There is nothing to shut down.** Phase 0 handed the dashboard to the system browser rather than starting a server ([`phases/0-instruments.md`](./0-instruments.md)), so the run leaves no process behind and the user's tab keeps the final picture — already rendered, no longer polling. `dashboard.html` is a static file: a double-click reopens it any time with every number intact. Say nothing about it; there is no news here.
