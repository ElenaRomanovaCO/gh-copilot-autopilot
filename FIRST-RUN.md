# First run — what actually happens

Installation is in [README.md](README.md). This file is the other question: you have typed `/autopilot`, and now what?

Read it once before the first build. It is written for the run you are about to watch, not for the skill's internals — those are in [`SKILL.md`](.github/skills/autopilot/SKILL.md), and the agent reads them so you do not have to.

---

## Where the files go, and where you run it

Two different places, and confusing them is the one setup mistake that costs you a whole build.

### The files go in your user profile — once, ever

```
%USERPROFILE%\.copilot\skills\autopilot\     the skill
%USERPROFILE%\.copilot\agents\*.agent.md     the four agents
```

That is `C:\Users\<you>\.copilot\...`. Create the folders if they are not there.

**Not** into VS Code's own program directory, and not into whichever folder the ZIP happened to land in. VS Code looks in specific places and nowhere else. The skill folder must be named exactly `autopilot`, because VS Code matches the folder name against the `name:` inside `SKILL.md` and will not load it otherwise.

Installed there, it is available in every project you ever open. There is no per-project setup, and no second copy to keep in sync.

**Once that copy is made, the downloaded package has done its job.** You can delete it.

### Nothing is launched, and nothing is filled in

These are not templates and there is no program. They are instruction files an agent reads: `SKILL.md` enters the agent's context when you type `/autopilot`, and each phase file is read off disk as that phase begins.

There is no build step, no `npm`, no runtime and no service. Copy the files, restart VS Code, type `/autopilot`.

### The build runs in the project's own window

| | Where |
|---|---|
| the skill and the agents | `%USERPROFILE%\.copilot\` — installed once |
| a build | a VS Code window opened on **the folder the project should live in** |

Open VS Code on `C:\dev\my-telegram-bot` — an empty folder is the normal case — and type `/autopilot ...` there. Autopilot creates `.autopilot/` in that folder, runs `git init` there, and writes every line of the project there.

> **Do not start a build inside the `gh-copilot-autopilot` folder.** That folder is the delivery package. A build started there would create the project *inside the skill's own repository*, tangled with the skill's files and committed to its history — and the first thing you would notice is the mess, not the mistake.

One project, one window, one folder. Two projects at once means two VS Code windows, and they will not interfere with each other: `.autopilot/` belongs to the folder, not to the installation.

### What has to be in place

- **VS Code, up to date.** This is the real risk, and the failure is silent — a build without Agent Skills support ignores the skill entirely rather than complaining. Help ▸ About, and update if you are behind.
- **The Copilot Chat extension**, and the chat set to **Agent** mode. Ask and Edit modes cannot write files or run the terminal, so they will read your brief and do nothing with it.
- **A model you are entitled to** — Claude Opus 4.6 or similar — selected in the model dropdown.
- **Git installed.** Autopilot commits once per ticket, and those commits are your rollback points.
- **A runtime for whatever gets built** — Node, Python, whatever the project needs. Nothing is needed for the skill itself.

---

## Before you type anything

Installation is done (above). Per run, four things:

- **Copilot Chat is in Agent mode** — not Ask, not Edit. Agent mode is the one that can write files and run the terminal; the other two will read your brief and do nothing with it.
- **Claude Opus 4.6 is selected** in the model dropdown. The four subagents inherit it — their agent files set no `model:` of their own.
- **VS Code is open on the folder the project should live in.** An empty folder is the normal case. An existing repo works too; Autopilot reads what is there first and respects the stack it finds.
- **You have a few minutes to answer questions.** Not hours — the briefing is the one stretch where the build genuinely needs you.

The four `autopilot-*` agents will not appear in your agent dropdown. They are `user-invocable: false` on purpose: they are the crew, not modes for you to fly in.

---

## If you already have a brief, a design, or a diagram

Autopilot is written for someone who has an idea and nothing else. If you arrive with a written brief, a technical design and a system diagram, you have already done the thinking that the first three phases exist to do — and you should change how you invoke it, or it will do that thinking again and reach somewhere slightly different.

### What is still worth having, and what is not

| Still earns its keep | Now largely redundant |
|---|---|
| **The manifest** — your design is prose; this turns it into numbered requirements with a status each, checked at four gates | **The briefing** — your design already answers the forks it would ask about |
| **The plan, the crew, the review** — the cut into tickets, a fresh context per ticket, three axes of review | **The spec** — becomes a transcription of your design rather than an act of invention |
| **Blind acceptance** — someone reads your brief, runs the result, and reports what is not there | |

The first of those is worth having however much design you did. It is the difference between *what I wrote down* and *what got built*, tracked to the end.

### Four changes to how you start

**1. Run it `strict`.** That is the dial for exactly this case: no capabilities invented, deepening only where a requirement cannot work without it. The default `normal` elaborates on top of what you said, which is the right behaviour for a bare idea and the wrong one for a settled design.

**2. Put the design in the repo before you start.** Preflight looks for `CONTEXT.md` and `docs/adr/`, and if it finds them the spec is required to speak that vocabulary rather than inventing synonyms, and to say so out loud where a decision contradicts a recorded one instead of quietly overriding it. A file in the repo is worth more than the same text pasted into chat.

**3. Convert the diagram to text.** This is the one that will actually cost you something. An image may or may not be readable, and the failure is not an error message — it is a spec that quietly ignored the diagram. Put it in the design document as a **mermaid block** or an ASCII sketch. Boxes and arrows written as text are read every time.

**4. Leave the mode at `semi`.** You still want a question when it finds a real gap between the design and what has to be built. `full` would decide it for you and tell you at the end, in the assumptions list.

### The risk worth naming in the brief

That its spec drifts from your design. Say in the brief that the design is a constraint, not a suggestion.

There is a legitimate path for departures — when the code proves the design wrong, the spec is amended and a `D##` row records what the code demonstrated and when, and you are told in one line. That mechanism is worth having. What `strict` plus an explicit instruction closes off is the other thing: a redesign nobody announced.

### If it is a proof of concept

The tier is read from what has to be built, never from how much you wrote. A POC lands at one of the two smallest:

- **T0** — one surface, one layer, no external service. **No tickets at all**: it is built in one pass, in one context. Autopilot's largest advantage, a fresh context per ticket, does not engage here. What you are buying is the manifest and the blind acceptance.
- **T1** — a few surfaces over one data shape, two or three tickets. This is where the machinery starts paying for itself.

**Be honest about which you have.** If the POC is genuinely one script or one endpoint, this is heavier than the job needs. If it is three or four moving parts you want to actually work together, it fits.

Say *proof of concept* in the brief, in those words — throwaway quality, shortest path to something runnable, no production hardening. It becomes a tracked requirement and constrains the build like any other, rather than a preference the run can drift away from. And leave `polish` off: it exists to finish something to a reference, and it costs real time and real money.

### What that looks like typed

```
/autopilot strict docs/brief.md — docs/design.md and the diagram in it are
the technical design: follow them, don't redesign. If something in the design
can't work, tell me rather than routing around it. This is a POC — shortest
path to something I can run, no production hardening.
```

The path is read as the task; the words beside it join the same brief. Every sentence there becomes a tracked requirement instead of a preference.

---

## The run

### `/autopilot I want a Telegram bot that takes repair requests into a Google Sheet`

Copilot loads `SKILL.md`. That is the orchestrator — modes, gates, the crew roster, the shell table. Every phase file stays on disk until the phase that needs it begins, which is what keeps the run's main context small enough to survive to the end.

**Preflight** runs in your main chat and takes under a minute. It probes which shell your terminal is, checks for git and an existing stack, creates `.autopilot/`, writes `state.js`, copies the dashboard template out of the installed skill, and hands the page to your browser.

**A browser window opens.** That is expected, and it is the last time you have to think about it — the page reloads its own data every ten seconds for the rest of the build.

Then one block, which does **not** wait for a reply:

```
Mode: semi-automatic · depth: normal — I'll ask only what the task leaves undefined,
then build the rest myself.
Dashboard is open — .autopilot\dashboard.html, it refreshes itself.
Project memory — AGENTS.md. Tell me if you want a different one.

You can switch at any moment, just say:
• "fully automatic" — I ask nothing at all
• "grill me" — I take the task apart with questions, then build the rest myself
• "manual mode" — the same, plus you approve the specification and the ticket list
• "strictly as written" / "go deep" — less or more elaboration beyond what you said
• "polish it" — at the end I compare against the reference and refine
```

That block is the only place the dials are ever named. There is no `--help` in a chat window.

### The briefing — the part that needs you

A handful of questions about the forks your brief left open: where the data lives, whether an account exists, what should happen when the thing it depends on is down. In the default `semi` mode there are only a few. Say *"grill me"* if you want the idea taken apart properly, or *"fully automatic"* if you want none at all and are content for the answers to be decided for you and listed as assumptions in the final report.

**It will never ask you for a key, token, or password.** Which provider — yes. Whether you have an account — yes. The credential itself is not a question, ever; you put it in `.env` yourself at the end, and the report tells you which names are still empty.

After this, you can walk away.

### Spec, plan, build

Everything from here is hands-free in every mode except `manual`.

The spec gets written, then checked by a subagent that is given **only your original brief and the spec** — no manifest, no conversation — and asked what the spec fails to cover. Then the work is cut into tickets, and the dashboard fills up with the whole road ahead.

---

## Where the subagents show up

From the spec onward, subagent calls appear in the chat as **nested blocks, collapsed by default** — one line each, expandable if you are curious. Their working output never enters your transcript; only their short return block does. That is the entire architecture in one sentence, and it is also why the chat stays readable through a two-hour build.

| Phase | What appears |
|---|---|
| Spec | one `autopilot-blind-check` — gate G2, blind to everything but the brief and the spec |
| Development | up to **three `autopilot-executor` blocks at once**, one per ticket in the wave |
| Code review | two more per ticket — `autopilot-review-spec` and `autopilot-review-craft`, in parallel |
| Acceptance | three `autopilot-blind-check` at once — acceptance, project memory, decision records |

Three of those four have no file-editing tool at all. A reviewer here cannot repair what it found and a blind checker cannot fix what it saw, which is not a rule anyone has to keep — it is simply not reachable.

Terminal commands mostly will not stop you: `git add`, `git commit`, `npm test`, `Get-Date` and the rest of the ordinary run are in the auto-approve list that ships with this package. `git push`, `rm`, `curl`, `docker`, `npm publish` are explicitly denied and will always prompt.

---

## What the chat looks like while it builds

Quiet. **One plain-language line per ticket**, by design:

> The bot takes requests — 3 of 8 done

No diffs, no file lists, no review findings, no jargon. If you want detail, it is on the dashboard, not in the chat.

**The dashboard is where progress lives:** the eight stages of the cycle, which one is running, how much of your brief is covered so far, tickets coloured by state, and live clocks on the run, the stage and each ticket. If it says a ticket has been running for eighteen minutes, that is true; the state is written when work starts, not filled in afterwards.

Meanwhile the folder fills up:

```
.autopilot/<slug>/
├── 2026-08-19-brief.md      your own words, redacted, never edited again
├── manifest.md              every requirement and what became of it
├── spec.md                  the specification
├── interfaces.md            what each ticket built, for the next ticket to read
├── review-log-spec.md       what the reviews established, across tickets
├── review-log-craft.md      the same, for the craft axis
└── tickets/
    ├── 01-shell.md          the ticket
    └── 01-shell.notes.md    why the code came out this way
```

One git commit per ticket, with the ticket number in the subject. Those are your rollback points, and there is one per unit of work on purpose.

---

## Where it will stop and ask you

No mode removes these. Not even `full`.

- **Blocking unknowns**, during the briefing — payment, hosting, an account, where the data lives. These are settled before the build, never at the finish line.
- **Anything irreversible or outward-facing** — deploy, publish, pay, message a third party, delete data, rewrite git history. A question in all four modes.
- **A failed ticket**, after one retry and one further attempt with a changed approach. It tells you what is blocking and what it needs from you, in plain language, and stops. It will not quietly narrow the ticket to whatever happened to work.
- **A leaked credential**, if one ever reaches a file — reported immediately, with the advice to rotate it.
- **The spec and the ticket list**, in `manual` mode only, each needing an explicit "ok". Silence is not approval.

---

## If the session dies

It will, eventually — long agent sessions get compacted, and at some point you will close VS Code.

Say **"continue autopilot"**.

Preflight recognises an unfinished run, reads `state.js`, `manifest.md` and `interfaces.md` **off disk rather than out of the transcript**, tells you in one line where things stand, resets any half-finished ticket to start over, re-checks the work since the last green commit, and carries on from there. Finished phases are not redone and answered questions are not re-asked.

This is why everything gets written down as it happens. The dialogue is a retelling; the files are the run.

---

## Three things to check on your first build

Use a throwaway project. Watch for these, in the order they would bite:

**1. Does `/autopilot` appear** when you type `/` in the chat? If not, the skill did not load — see the troubleshooting table below. Nothing else matters until this works.

**2. Does the first subagent actually spawn** at the spec stage? You are looking for a nested block in the chat. If custom subagent invocation is broken on your VS Code build, the skill says so in one line and falls back to running the role inline — which works, and degrades after a few tickets. You will see that sentence; it does not fail silently.

**3. Do the dashboard's times look real** — seconds that are not all `:00`, durations that are not dashes? That is the tell that Phase 0 picked the right shell. Wrong shell means a build's worth of timestamps the page cannot parse, and it is invisible in the terminal.

---

## When something is off

| What you see | What it is |
|---|---|
| `/autopilot` missing from the slash list | The skill is not installed where VS Code looks, or you reloaded the window instead of restarting. `%USERPROFILE%\.copilot\skills\autopilot\SKILL.md` must exist, and the folder must be named exactly `autopilot`. |
| Skill loads, but no subagents ever spawn | Custom subagents disabled or unsupported on this build. Check `chat.subagents.enabled`. The run continues inline; expect it to degrade past the third or fourth ticket. |
| A reviewer edits files | The `tools:` names in that `.agent.md` were not understood by your VS Code version. Open the file and use its tools picker to re-select — VS Code writes the names its own build knows. |
| A reviewer seems to have no tools at all | Same cause, same fix. |
| Dashboard says "Dashboard has no state yet" | The page was opened somewhere that cannot load `state.js` beside it — almost always VS Code's Simple Browser. Open `.autopilot\dashboard.html` in Edge or Chrome instead. |
| Times all end in `:00`, durations show `—` | Wrong shell detected. Tell the agent which shell your terminal is; it will correct `state.js`. |
| The project's files are appearing inside `gh-copilot-autopilot` | The build was started in the package folder instead of the project folder. Stop it, delete the `.autopilot/` and any generated files from there, open VS Code on the project's own folder and start again. |
| The build stops to confirm every command | The auto-approve settings did not get merged. See README, install step. |
| It asks you a question about *process* — which tracker, where tickets live | It should not, outside `manual`. Tell it to decide for itself; that is what the skill says to do. |

---

## The cost question, honestly

Autopilot is subagent-heavy by design: one executor and two reviewers per ticket, plus three more at the finish. A Copilot agentic session is billed at the **highest-multiplier model used anywhere in the chain**, so pinning executors to a cheaper model with `model:` in their agent files may not save what you would expect.

**Run one small throwaway project first and watch your usage meter.** Capability is not the constraint here; this is.
