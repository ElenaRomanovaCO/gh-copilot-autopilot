---
name: autopilot
description: Use when the user dictates an app, site, bot, or feature to build end-to-end and expects a finished result without reviewing specs, tickets, or code — vibecoding sessions, non-technical users, "build it turnkey", "just build it for me", "don't ask unnecessary questions" requests. Also use when the user invokes /autopilot, or asks for a build in a named mode, depth or finish — "fully automatic", "interview mode", "grill me", "manual mode", "strictly as written", "go deep", "polish it to the reference".
argument-hint: "[full|semi|interview|manual] [strict|deep] [polish] what to build, or a path to brief.md"
---

# Autopilot

## Overview

Autopilot flies a dictated idea from words to a working project **in one dialogue**, without making the user approve each stage. It is self-contained: every rule it needs lives in `phases/`. No other skill has to be installed.

Two ideas carry the whole design.

**The order is the product.** Code is written in the second-to-last phase. Everything before it exists to decide *what* to build, and everything after it exists to prove the right thing got built.

**The brief is the contract, not the design.** Two obligations follow from it, and they pull in opposite directions on purpose.

*Nothing may quietly vanish.* The user's original words become a numbered manifest before anything else happens, and every phase is gated on it. What breaks naive vibecoding is not bad code — it is a requirement that stopped existing somewhere around the third rewrite.

*The brief is not the design.* It is a silhouette: it describes the happy path and nothing underneath — no empty states, no failures, no interruptions, no limits. Working those out is legitimate work, not scope creep, and it is where much of the value of this process comes from. **How far to take it is the user's dial**, set by the [depth](#depth) parameter. What is never allowed at any setting is depth that **detaches** from the brief.

## Reading this skill

This file is the orchestrator: modes, phase order, gates. The rules for each phase live in `phases/` and are **read at the moment that phase starts, not before** — that is what keeps the working context small.

**The unit of loading is the file, not the section.** "Read only the block at the top" is not a mechanism — a read pulls in the whole file — so anything needed by one phase and not another is its own file. That is why Phase 0's share of the instruments and of the project memory sit in [`phases/0-instruments.md`](./phases/0-instruments.md) and [`phases/0-memory.md`](./phases/0-memory.md) rather than at the top of the phase-7 and phase-9 files: taking the whole of both would cost thirty-eight thousand characters in the one context that is never refreshed, to answer questions that arrive four phases later.

| Phase | Read | Produces |
|---|---|---|
| 0 Preflight | [`phases/0-preflight.md`](./phases/0-preflight.md), then `0-instruments.md` and `0-memory.md` | repo configured, `.autopilot/` created |
| 1 Manifest | [`phases/1-manifest.md`](./phases/1-manifest.md) | `brief.md`, `manifest.md` |
| 2 Briefing | [`phases/2-briefing.md`](./phases/2-briefing.md) | answers recorded into the manifest |
| 3 Spec | [`phases/3-spec.md`](./phases/3-spec.md) | `spec.md` |
| 4 Plan | [`phases/4-plan.md`](./phases/4-plan.md) | `tickets/NN-*.md` (or none — see tiers), `interfaces.md` seeded |
| 5 Subagents | [`phases/5-subagents.md`](./phases/5-subagents.md) | code, commits, `interfaces.md` grown |
| 6 Review | [`phases/6-review.md`](./phases/6-review.md) | per-ticket review |
| 7 Instruments | [`phases/7-instruments.md`](./phases/7-instruments.md) — **in Phase 4**, when the tickets are cut | `state.js`, `dashboard.html` (opened for the user) |
| 8 Final | [`phases/8-final.md`](./phases/8-final.md) | blind acceptance, final report |
| 9 Memory | [`phases/9-memory.md`](./phases/9-memory.md) — **in Phase 5 and Phase 8** | `AGENTS.md`, `docs/adr/` — the project as the next session will find it |

## The words the user sees

The phases have internal names in this file and the user never sees them. In the chat, on the dashboard and in the final report there is **exactly one word per stage**, and it is this one. Two vocabularies for one process is how a person reads the README and then cannot find any of it on the screen.

| Phase | `stages[].id` | To the user |
|---|---|---|
| 0 Preflight | `preflight` | Preflight |
| 1 Manifest | `manifest` | Requirements |
| 2 Briefing | `briefing` | Briefing |
| 3 Spec | `spec` | Spec |
| 4 Plan | `plan` | Plan |
| 5 Subagents | `build` | Development |
| 6 Review | `review` | Code review |
| 8 Final | `final` | Acceptance |

Two rules hold this together:

- **"The build" is the whole run, not one stage.** "The build is running", "the build got interrupted", "continue the build" — all about the process as a whole. That is why the fifth stage is called "Development": otherwise one word would mean both the part and the whole. And "build" in the `npm run build` sense is not it either.
- **The unit of work is a "ticket".** Not a "task", not an "issue". A "task" is what the user set (the brief); one word for two different things breaks both the report and the dashboard.

Phases 7 and 9 are not sequential, and each is split in two along the line where it is read. The instruments are raised in Phase 0 from [`phases/0-instruments.md`](./phases/0-instruments.md) — template, starting state, the update ritual — and [`phases/7-instruments.md`](./phases/7-instruments.md) is opened only when the tickets are cut. The project memory is raised in Phase 0 from [`phases/0-memory.md`](./phases/0-memory.md) — which file, and the skeleton — and [`phases/9-memory.md`](./phases/9-memory.md) is opened when the build discovers something and again in Phase 8, where a subagent writes the full description from the finished code.

## Modes

Everything typed after `/autopilot` splits into four parts: **the mode** (optional bare word — `full`, `semi`, `interview`, `manual`), **the depth** (optional bare word — `strict`, `deep`), **the finish** (optional bare word — `polish`), and **the brief** (everything else). No dashes on any parameter. Text that is not a recognised parameter is always brief.

`/autopilot full deep online store for ceramics` — full mode, deep elaboration. Order does not matter; all three parameters are optional and independent.

| Mode | Triggers | Human gates |
|---|---|---|
| **full** — fully automatic | `/autopilot full`, "fully automatic", "all by yourself", "don't ask me anything", "ask nothing" | none |
| **semi** — semi-automatic **(default)** | `/autopilot semi`, "semi-automatic", nothing specified | questions, on genuine forks only |
| **interview** — interview mode | `/autopilot interview`, "interview mode", "grill me", "interrogate me", "ask me everything", "take the task apart with me" | questions, all of them |
| **manual** | `/autopilot manual`, "manual mode", "approve every step with me" | the same questions + spec + tickets |

**Why four and not three.** A mode decides two separate things: how much the user is asked about the *product*, and how much of the *process* they approve. Those are different kinds of their time — answering questions is the work, approving artifacts is control over how the work runs — and wanting one without the other is the ordinary case, not an exotic one. `interview` is that case: take the task apart with me question by question, then build the rest yourself. `manual` is `interview` plus the two artifact gates, and nothing else.

- **Announce the resolved mode and offer the others, once, before Phase 1.** The user must never discover the mode by noticing questions that did or did not arrive — and they cannot ask for a mode they do not know exists. In a chat client there is no `--help` to read: this block is the only place the dials are ever named, so it is not optional.

  ```
  Mode: semi-automatic · depth: normal — I'll ask only what the task leaves undefined, then build the rest myself.
  Dashboard is open — it refreshes itself.
  Project memory — AGENTS.md. Tell me if you want a different one.

  You can switch at any moment, just say:
  • "fully automatic" — I ask nothing at all
  • "grill me" — I take the task apart with questions to the end, then build the rest myself
  • "manual mode" — the same, plus you approve the specification and the ticket list
  • "strictly as written" / "go deep" — less or more elaboration beyond what you said
  • "polish it" — at the end I compare against the reference and refine; longer and more expensive
  ```

  With `polish` on, the first line names it and its ceiling: "Mode: semi-automatic · depth: normal · polish: up to three rounds".

  One short block, once, at the start. **It is a hint, not a question** — say it and go straight into Phase 1; waiting for a reply to it is exactly the pause this skill exists to remove. Do not repeat it later, do not restate it after a mid-run switch (one line is enough there: "Got it, manual mode from here on").
- **Ambiguity resolves to semi.** A mode word contradicting the rest of the sentence ("manual mode, but don't ask me anything") → the explicit mode word wins; two mode words → ask which one, in one line.
- **The mode can be switched mid-run** ("switch to manual") — it applies from the next phase onward. Phases already passed are not replayed.
- **Extra instructions in the brief** (stack, language, budget, "no database", deadline) are manifest requirements like any other. They constrain the build; they never replace a phase.
- **No mode removes the manifest gates or the safety gates.** Irreversible or outward-facing actions — deploy, publish, pay, send messages to third parties, delete data, rewrite git history — stay a question in **all four** modes, including full.

## Depth

How far past the brief's own words the spec is allowed to go. The mode decides *how much the user is asked*; depth decides *how much is worked out for them*. They are independent.

| Depth | Triggers | Deepening a requirement (`R##.n`) | New capabilities (`A##`) |
|---|---|---|---|
| **strict** | `/autopilot strict`, "strictly as written", "only what I said", "add nothing extra", "nothing extra" | only what the requirement cannot work without | **not allowed** |
| **normal** **(default)** | nothing specified | freely, by judgement — as much as the feature warrants | allowed, with a parent, within proportion |
| **deep** | `/autopilot deep`, "go deep", "maximum depth", "think it through for me", "think it through" | the full depth pass, every dimension, every requirement | actively encouraged, same two limits |

- **Default is normal, and normal means permitted.** The agent elaborates where elaboration obviously helps and does not chase every edge of every requirement. This is the setting most briefs should run on.
- **`strict` does not mean careless.** Errors and empty states are still handled — a build that crashes on bad input does not satisfy the requirement it was written for. What `strict` removes is anything the user did not ask for: no extra capabilities, no anticipating needs, no "while I'm here, I'll add".
- **`deep` does not lift the attachment rules.** Every `A##` still names its parent requirement; the proportion limit still holds. `deep` buys thoroughness, never a different project.
- **`deep` also turns on the adversarial pass** — the premortem over the brief in [`phases/2-briefing.md`](./phases/2-briefing.md), which asks where the idea itself comes apart rather than where a requirement is underspecified. It runs at `deep` in **every** mode, and in `interview` and `manual` at every depth, because taking the task apart is what those modes are for. The mode then decides what happens to what it finds: a question, or an `ASSUMPTION` decided for the user.
- **Depth is announced with the mode**, in the same opening block: "Mode: semi-automatic · depth: maximum".
- **Depth can be changed mid-run** ("less of your own ideas", "think it through deeper") — applies from the next phase. Already-written spec sections are not retroactively trimmed unless the user asks.

The rules for each level live in [`phases/3-spec.md`](./phases/3-spec.md).

## Polish

**Off by default.** One bare word turns it on, and it is the only parameter that costs the user real money and real time rather than just attention.

| | Triggers | What it adds |
|---|---|---|
| **polish** | `/autopilot polish`, "polish it", "make it perfect", "compare against the reference", "don't stop until it's right", "budget doesn't matter, the result does" | after the blind acceptance, up to three rounds of comparing the running build against the user's own reference and fixing the differences |

**Why this is a third dial and not a value of `depth`.** Depth decides how much is worked out *before* the code exists; polish decides how much is corrected *after* it does. A `strict` brief can deserve a flawless finish, and a `deep` spec can be right the first time. Folding one into the other would tie two independent decisions to one word, which is the same argument that gives this skill four modes instead of three.

Two things are decided here; everything else — the critic's prompt, the filter, the stop conditions, the bookkeeping — is in [`phases/polish.md`](./phases/polish.md), **read only when the parameter is on.**

- **It measures against a reference, never against taste.** No `reference.md` with something comparable in it → the loop says so in one line and does not run. A critic with nothing to compare against invents a standard, and the run then pays for chasing it.
- **Its findings become tickets**, cut and flown and reviewed and committed like any others. Nothing about polish bypasses Phase 6 or the green suite; it is more work of the same kind, not a different kind of work.

Announced with the mode and depth in the opening block, ceiling named: "polish: up to three rounds".

## When to Use

- User dictates what to build and expects the finished thing, not a collaboration on process.
- User is non-technical: will not read specs, judge ticket granularity, or review code.
- "Build it turnkey", "just build it", "don't ask unnecessary questions".
- User wants the idea taken apart with them question by question, and the build done without them — that is **interview** mode, still Autopilot.
- User wants to approve the spec and the tickets but not to run the pipeline by hand — that is **manual** mode, still Autopilot.

**When NOT to use:** the user wants to co-author the code itself line by line (work with them directly); the task is a small single-file change (just do it); the idea is bigger than one project and its destination is unclear (settle the destination first, then return here).

## The flight

| Phase | full | semi (default) | interview | manual |
|---|---|---|---|---|
| 0 Preflight | auto | auto | auto | auto |
| 1 Manifest | auto | auto | auto | auto |
| 2 Briefing | skipped → self-briefing | only what the brief leaves open — sometimes none | the adversarial pass, then every fork it opens | the same |
| 3 Spec | auto | auto | auto | show → wait for explicit "ok" |
| 4 Plan | auto, notify only | auto, stoppable | auto, stoppable | discuss → wait for explicit "ok" |
| 5 Subagents | auto | auto | auto | auto |
| 6 Review | auto | auto | auto | auto |
| 8 Final | report + Assumptions | report | report | report |

**`polish` adds a step inside Phase 8, in every mode** — the polish loop, between the blind acceptance and the report. It changes no cell above: it asks the user nothing, and it approves nothing with them.

**`interview` and `manual` differ in exactly two cells** — the spec gate and the plan gate. If you find yourself treating them differently anywhere else, one of them is wrong.

**The manifest gates run in every mode.** They are checks against the user's own words, not requests for the user's time — no mode buys the right to skip them.

| Gate | After phase | Condition to pass |
|---|---|---|
| **G1** | 2 Briefing | every requirement has a status; none left `open` without a reason recorded |
| **G2** | 3 Spec | every live requirement is `in-spec`, `deferred`, or `dropped`, zero `open` — **and an independent reader given only the brief and the spec finds nothing missing** |
| **G3** | 4 Plan | every `in-spec` maps to ≥1 ticket, **and every ticket traces back to ≥1 requirement** |
| **G4** | 8 Final | blind acceptance run against the brief, spec withheld; every disagreement with the manifest reported |

**G2 and G4 are the same check at the two ends of the flight, and both are needed.** They measure the build against the user's own words, with your paraphrase of them taken away — G2 while the answer is a paragraph of spec, G4 when it is the last chance to know. Everything in between measures against the spec, because that is the contract the executors were actually given; judging a subagent by words it never saw produces findings nobody can act on.

A failed gate is not a warning. It sends the phase back to be redone — see [`phases/1-manifest.md`](./phases/1-manifest.md).

**The plan may be corrected; the brief may not.** When the build proves the plan wrong — a data model that does not hold, an assumed interface that cannot exist — the spec is amended and a `D##` row records what the code demonstrated and when. That is the one thing allowed into the manifest after the briefing, it never retires a requirement, and it is never a route for an idea you had. Rules in [`phases/5-subagents.md`](./phases/5-subagents.md).

## Secrets

Credentials are the user's to hold, not the agent's to handle. This section binds every phase; the phases do not restate it.

- **Never request one.** No key, token, password, connection string, or card number is ever a question. *Which* provider is a question. *Whether* an account exists is a question. The credential is not.
- **Redact at ingest, before anything is written.** The brief, every user answer, and every pasted fragment pass the redaction gate in [`phases/1-manifest.md`](./phases/1-manifest.md) *before* they reach a file. A detected secret becomes `[REDACTED:<VAR_NAME>]` — the variable name survives, the value does not.
- **"Verbatim" always means "verbatim after redaction."** Wherever this skill asks for the user's exact words, it asks for them redacted. The two rules are one rule.
- **Refer to it by name.** `STRIPE_SECRET_KEY`, not the value. The user puts the value in `.env` themselves; `.env` is in `.gitignore` before the first commit; the final report lists which names are still empty.
- **A leaked secret is a stop condition.** A secret that reached a file or a commit is reported immediately, in plain language, with the advice to rotate it. Before the first commit, run the redaction gate over the whole of `.autopilot/`.

## This harness — VS Code + GitHub Copilot

Autopilot runs here as the orchestrator inside Copilot's agent mode. Three things about this harness change how the phases below are carried out, and they bind every one of them.

### The crew is named

You do not improvise a subagent. Four are defined in `.github/agents/`, and every phase names the one it needs. Invoke them with the subagent tool (`agent` / `#runSubagent`), one task per invocation.

| Agent | Used by | Can |
|---|---|---|
| `autopilot-executor` | Phase 5 — one per ticket | read, edit, run the terminal |
| `autopilot-review-spec` | Phase 6 — axes Manifest + Spec | read, and the terminal for `git diff` — no file editing |
| `autopilot-review-craft` | Phase 6 — axis Craft | the same |
| `autopilot-blind-check` | gates G2 and G4, the ADRs and the memory text | the same, plus running the project — and given only what its gate allows it to see |

**The three read-only agents carry no file-editing tool.** "The reviewer repaired it itself" and "the blind checker fixed the thing it found" are failures the phase files can otherwise only warn against; here they are not reachable. They keep the terminal because a reviewer that cannot run `git diff` reviews the current state of the files instead of the change — and because the blind checker's whole job is to *run the project*, not to read it.

**If the subagent tool is unavailable or a call is refused**, say so in one line and fall back to running that role inline, in this context — and tell the user the run will degrade after a few tickets, because it will. What is never the fallback is quietly taking the keyboard.

### Subagents are stateless — there is no follow-up

Every invocation starts cold and cannot be continued, so a subagent's whole context has to arrive in its first message. Two places in the phases assume otherwise, and both have a replacement here. **The rule behind both is one rule: what a warm context would have held, a file holds instead.**

| The phase file says | Here |
|---|---|
| a shortfall goes back to the same executor as a follow-up ([`phases/5-subagents.md`](./phases/5-subagents.md)) | a fresh executor, given the ticket, the finding, **and the ticket's `.notes.md`** — which is why every executor writes one before it returns |
| one reviewer is kept for the whole run ([`phases/6-review.md`](./phases/6-review.md)) | a fresh reviewer per ticket, which **reads its own review log first and appends to it last** — the cross-ticket memory, relocated to disk |

Two logs, never one: `review-log-spec.md` and `review-log-craft.md`, one owner each. The two reviewers run in parallel, and two writers on one file collide.

This costs more than a warm context and buys back most of what it lost. What it does not buy back is a reviewer's own judgement of drift — so the logs stay short and factual, or the next reviewer reads an essay instead of a record.

### Shell

The terminal is PowerShell unless the user's default says otherwise. Settle which one in Phase 0, record it in `state.js` as `shell`, and use this table for the rest of the run.

| Need | PowerShell | bash |
|---|---|---|
| the clock — ISO 8601 with offset | `Get-Date -Format "yyyy-MM-ddTHH:mm:sszzz"` | `date -Iseconds` |
| a test run, truncated | `cmd /c "<test cmd> 2>&1" \| Select-Object -Last 30` | `<test cmd> 2>&1 \| tail -30` |
| copy a file | `Copy-Item <src> <dst>` | `cp <src> <dst>` |
| open a page for the user | `Start-Process .autopilot\dashboard.html` | `open` / `xdg-open` |

`cmd /c` on the test line is not decoration. Windows PowerShell 5.1 wraps a native command's stderr in error records and reports failure on a clean exit, so a bare `npm test 2>&1` reads as red when it is green — and a green suite misread as red sends a finished ticket back for repair.

**A timestamp composed from memory is the failure this table exists to prevent.** Read the clock at the moment the thing happens.

## Files this skill owns

```
.autopilot/
├── <feature-slug>/
│   ├── <YYYY-MM-DD>-brief.md   the user's original words, redacted, never edited again
│   ├── manifest.md      R01…Rnn — requirements and their status
│   ├── reference.md     what the result should be like — the user's comparables, never yours
│   ├── spec.md          the specification
│   ├── interfaces.md    the boundaries from the spec, then what finished tickets built
│   ├── review-log-spec.md    the Manifest+Spec reviewer's memory across tickets — it owns this file
│   ├── review-log-craft.md   the Craft reviewer's memory across tickets — it owns this file
│   └── tickets/
│       ├── NN-<slug>.md        the ticket
│       └── NN-<slug>.notes.md  why the code came out this way — written by its executor, read by its repairer
├── README.md            how to read this folder — for the human, written once in Phase 0
├── state.js             the run state, and the only file you update: stages, tickets, timings, debt
└── dashboard.html       the human view — copied once in Phase 0, then reads state.js by itself

AGENTS.md               the project memory — what the next session reads first
docs/adr/               decisions worth outliving the run — written in Phase 9, tier T2+
```

The three files that do not appear in the upstream skill — the two review logs and the per-ticket notes — exist because subagents here are stateless. They are not documentation; they are the memory a warm context would have carried, and nothing else in the run reads them. See *This harness* above.

The brief is dated in its filename because a slug directory outlives one sitting. The dashboard is opened for the user, not described to them: it shows the eight stages of the cycle, where the run is now, and a live clock on the run, the current stage and the current ticket.

`.autopilot/` is the record of **this** run; the memory file at the root is the project as it stands, for whoever opens the repo next; `docs/adr/` is why it stands that way. Autopilot's content in the memory file lives between `<!-- autopilot:start -->` markers — everything the user wrote outside them is untouchable. See [`phases/9-memory.md`](./phases/9-memory.md).

The three are not interchangeable, and the split is what keeps the spec throwaway. `spec.md` is worth nothing the day the work ships; the reasoning inside it — why this data model, what the build proved wrong, which word means which thing — is worth something for years, and it dies with `.autopilot/` unless something routes it out. That is what the ADRs are for.

`.autopilot/` is committed, not ignored — it is the user's record of what was promised and what was delivered. A run that leaves nothing under `.autopilot/` did not happen.

## Judgement

This skill describes a process, not the product. Its numbers — tiers, question counts, story counts, wave widths — are **calibration for a first guess, never targets to hit.** A spec written to reach a story count, or a plan cut to land inside a tier, has optimised for the rule instead of for the person who asked.

The rules below are the same kind of thing. Each one is here because it was paid for, and each is an argument — arguments can lose. Where following one would make the result worse for the user, break it deliberately, say so in one line, and carry on. That is a decision, and decisions get recorded. What is never acceptable is breaking one quietly, or keeping one because it is written down.

**Five rules are not calibration and do not lose.** They hold in every mode, at every depth, at every tier:

1. **A requirement is removed only by the user**, in their own words, quoted into the manifest.
2. **A secret is never requested, echoed, or written** — not into a file, a prompt, a commit, or a report.
3. **A fact about the user is never invented.** Prices, texts, addresses, accounts stay visible placeholders until they supply them.
4. **An irreversible or outward-facing action is a question** — deploy, publish, pay, message a third party, delete data, rewrite history.
5. **The orchestrator does not write the project's code.** Its keyboard reaches `.autopilot/`, the memory file and git. Everything else — a fix worth two lines, a red test, a review finding — travels down to a subagent. Rules in [`phases/5-subagents.md`](./phases/5-subagents.md).

Everything else is argument.

## Rationalizations — the ones that cost the user the product

Phase-specific mechanics are not here; they live in the phase that owns them. What follows is the short list of excuses that end with the user getting something other than what they asked for.

| Excuse | Reality |
|--------|---------|
| "The user said not to ask questions" | They said not to ask UNNECESSARY ones. Decisive questions are part of the work, not a discussion of the process. |
| "KISS — just build it" | A simple result comes from the order, not from skipping stages. Without a spec, every fix becomes "that's not what I meant". |
| "The brief is all here in the dialogue, why rewrite it into a file" | The dialogue gets compacted, and the brief is the oldest thing in it. Three phases from now you'll be synthesising from a retelling of a retelling. |
| "This requirement is obviously unimportant, I'll skip it" | The importance of requirements is decided by the user. You may propose `deferred` — only they may cross one out. |
| "The user never mentioned it again — so they cancelled it" | Silence cancels nothing. A cancellation is their words, quoted into the manifest. |
| "I'll stub it, they'll clarify later" | Blocking unknowns (payment, hosting, accounts) are settled in the briefing — in fully automatic mode in the self-briefing — but always before the build. |
| "Let them send the key, I'll paste it into the code" | Keys are entered by the user, and only into `.env`. You work with the variable name. |
| "The key is already in the context, so it can be written down" | The opposite: it means it must be redacted, and the user warned. Context is not permission. |
| "It's faster to do everything in one context" | Faster in the first hour. After that the model goes in circles and breaks what worked. |
| "The executor wrote that it couldn't — I'll finish it myself, I have the context anyway" | You hold the context of the whole run — which is exactly why you can't. Every fix made with your hands leaves a diff in your context until the end of the build and degrades every following ticket. If it couldn't — a fresh executor with the ticket's notes file, but never your hands. |
| "It's a two-line fix — dispatching a subagent costs more" | More for this ticket. Every following one pays: the orchestrator's context is spent once and never comes back. By the eighth ticket the difference is between "built it" and "broke what worked". |
| "The brief is short — so the spec is short too" | The brief is a silhouette: the user described the happy path and described neither empty states, nor errors, nor interruptions. At normal and maximum depth, thinking those through is your job. |
| "It's obvious anyway, I won't write it down" | What is obvious to you is not recorded, and every subagent will fill it in its own way: three executors — three different "obviouslys". The manifest and the spec are the only points of alignment. |
| "I thought of a useful feature, I'll add it" | Deepening what was ordered (`R##.n`) — yes. A new capability (`A`) — only with a parent requirement, within proportion, and into the report. At `strict` — not at all. |
| "Fully automatic — so I can deploy too" | Automatic removes questions about the product, not the right to the irreversible. Deploy, payment, mass messaging, deletion — a gate in every mode. |
| "In fully automatic mode I can make up everything for the user" | Decisions — yes, and all of them into ASSUMPTIONS. Facts about the user (prices, texts, accounts) — no: a placeholder and a line in the report. |
| "I'll write 'launching in 60 seconds'" | You cannot wait — the promised pause will not happen. The honest phrasing: "starting now, say stop". |
| "In manual mode I'll also start and wait for objections" | In manual, approval is an explicit "ok". Silence is not it, and work already started even less so. |
| "I'll check the result against the spec, that's enough" | The spec may already have lost a requirement. The final check runs against the brief and without the spec — otherwise it confirms its own mistake. |
| "I'll check coverage myself — I just wrote the spec" | The one who wrote it cannot see what they did not write. At G2 the brief and the spec are read by a subagent that is given neither the manifest nor the conversation. |
| "The testing rule is written in the phase file — so it's in force" | Only what reached the executor's prompt is in force. The phase file is read by the orchestrator, and the orchestrator doesn't write the code. |
| "The interfaces will settle as we go — the first ticket will set them" | Then they'll be set by whoever saw one eighth of the task. Module boundaries are decided before the cut, otherwise eight contexts negotiate after the fact. |
| "I'll write the report from memory — I did all of it myself" | By the eighth phase your context is the most polluted of the whole run. The report is assembled from `manifest.md` and `state.js`, re-read from disk. |
| "The reasoning behind the decisions will stay in the spec" | The spec dies with the run. What should outlive it goes into ADRs — otherwise the next session re-opens the same decisions. |
| "The user said 'grill me' — I'll show them the spec too" | Interview mode buys questions, not gates. Gates are "manual mode", and that is a separate word they did not say. |
| "The tickets and the spec are visible in the chat — why files" | The file in `.autopilot/` is the artifact; the chat is only a retelling of it. The dialogue will die, the files will remain. |
| "The user didn't ask about modes — I won't burden them" | They never will ask: there is no `--help` in a chat. Five lines at the start are the only place they even learn the build has dials. |
| "They asked for polish — the critic will figure out what to compare against" | It won't: it will invent a reference and drive the build toward it. No reference from the user — no polish, and that is an answer, not a refusal. |
| "The polish round found a trifle — I'll fix it myself, it's not a ticket" | Then the fix goes in without review, without a green run and without a rollback point. Polish is more tickets, not a licence to take the keyboard. |
| "The critic is still unsatisfied — so it's too early to stop" | It will always be unsatisfied: that's what it's paid for. Stopping is the absence of findings, the round ceiling, or the user's word. |
| "There's no follow-up in this harness, so it's cheaper to fix it myself" | The notes file exists so that a fresh context is cheap. Stateless subagents raise the price of a repair; they do not transfer the keyboard to you. |
| "The reviewer's log is bookkeeping — I'll skip it this ticket" | It is the only cross-ticket memory this harness has. Skipped once, ticket 07 contradicting ticket 03 becomes invisible, and nothing later can see it either. |
| "`date -Iseconds` will probably work in PowerShell too" | It does not, and the failure is silent in the terminal and visible only as a dead dash on the user's dashboard for the rest of the run. |
| "The project is built, the tests are green — so it works" | The tests were written by the same process as the code. Until someone has run the project, "works" is a hypothesis — and the first one to run it will be the user. |

## Red Flags — start the phase over

Every line here means something the user asked for is at risk. Phase mechanics — instruments, timestamps, wave bookkeeping, memory-file detection — are checked in the phase files that own them, not here.

- Writing code before the spec exists.
- The brief was never written to its file — the run is anchored to nothing.
- A requirement left the manifest without a status, or was marked `dropped` without a quote of the user saying so.
- Past gate G3: a ticket that traces to no requirement, or a requirement that traces to no ticket.
- Spec or tickets that exist only in the dialogue — nothing written under `.autopilot/`.
- Instruments that disagree with the chat: a stage still `active` after you moved on, a ticket running while the dashboard calls it `pending`, a ticket carrying the run's `startedAt` instead of its own, timestamps filled in afterwards from memory. The user believes the screen over your sentences, which is the whole reason it exists.
- The announced depth and the actual spec diverge: a bare restatement of the brief at normal or deep, or an invented capability — any `A##` — at strict.
- Gate G2 passed on your own reading of your own spec — the independent coverage check skipped, or its checker handed the manifest.
- Final acceptance measured against the spec instead of blind against the brief.
- The blind checker, the coverage checker or the memory subagent handed `spec.md`, the manifest or the tickets — whichever of those it was supposed to be blind to. Independence is the entire mechanism; without it each of them confirms the plan instead of the thing.
- The final report composed from memory instead of from `manifest.md`, `state.js` and the two subagents' returns, re-read from disk.
- A T2+ run that ended with no ADR: every `D##` and every load-bearing implementation decision left to die with `.autopilot/`.
- The finished project was never actually run — accepted on green tests and a reading of the code.
- Starting without announcing mode and depth, or announcing one and behaving as another: questions in full, a spec put up for approval in interview, a start-and-see instead of "ok" in manual.
- With `polish` on: a polish round run against no reference, its findings applied outside the ticket path, a fourth round, or a round that broke something and was patched instead of reverted.
- A comparable in `reference.md` that the user never named — your taste entered as though it were theirs, and everything downstream now judges the build against it.
- The adversarial pass skipped in `interview` because the brief "looked well thought through" — or used to argue the user out of a requirement instead of into a decision.
- A blocking unknown — payment, hosting, an account, where the data lives — left unasked in semi, interview or manual because the brief "looked clear". Asking nothing is legitimate only when nothing is open; a manufactured question and a skipped blocking one are both defects, in opposite directions.
- Promising the user a wait — a countdown, "in a minute", "if you don't reply within N seconds" — that you have no way to honour.
- In full: an invented fact about the user standing where an ASSUMPTION, a stub, or a PLACEHOLDER belongs.
- Asking the user a process question — which tracker, which doc file, which memory file, ticket granularity, code review — outside manual, where spec and tickets are gates by design.
- A requirement quietly narrowed to whatever happened to work, or the spec amended mid-build with no `D##` row recording why.
- Two tickets in one subagent context, or two tickets in one commit.
- The orchestrator editing a file outside `.autopilot/`: a "two-line" fix, a red test, a review finding applied by hand instead of sent down. One such edit is the whole failure — the diff stays in its context for the rest of the run.
- A ticket's diff, or the raw output of a full test run, read into the orchestrator's context. It needs a verdict and the names of what failed, not the material.
- A repair started from an empty context when the ticket's own executor was still reachable — or the mirror failure, a third follow-up into an executor that has already failed to do it twice.
- Parallel subagents editing the same files — or the mirror failure, independent tickets flown one at a time with the plan's parallelism thrown away in the delivery.
- A subagent launched without `interfaces.md`, or finishing without returning the contract block.
- The first wave launched with `interfaces.md` still empty — module boundaries left for whichever ticket happens to reach them first.
- A subagent prompt with no testing rules in it: the discipline written in the phase file the orchestrator reads, and absent from the handoff to the one who writes the code.
- Payment, hosting, or accounts first mentioned at the finish line.
- A secret value asked for, repeated back, or written into any file, prompt, commit, or report.
- Installing a package or fetching remote code without the user asking for it.
- Text outside the `autopilot` markers edited, moved or dropped, or the run ending with no project memory file at all.
- An executor that returned without writing its ticket's `.notes.md` — every repair on that ticket now starts blind, and in this harness there is no warm context to fall back on.
- A reviewer launched without its own review log, or finished without appending to it — the cross-ticket eye is the one thing this harness had to buy back by hand, and it is bought one ticket at a time.
- A review finding, an executor's task, or a gate check run inline "just this once" while the subagent tool was working. Once is how the orchestrator's context starts filling, and it never empties.
- Timestamps that are all whole minutes, or a dead dash where a duration belongs: the clock command in *This harness* was not run, or was run in the wrong shell.
