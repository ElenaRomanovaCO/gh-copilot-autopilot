# Phase 5 — Crew

Where the code gets written. **Identical in all four modes — this phase is always hands-free.** Manual mode buys the user control over *what* gets built, not over each edit; once the tickets are agreed, the crew flies to the end without further approvals.

At tier T0 there are no tickets: you are the crew, working straight from the spec in the current context. Everything below about contracts and returns still applies to you — top up `interfaces.md` with what you actually built, run the Phase 6 checklist, commit once.

**T0 does not excuse empty instruments.** Mark the `build` stage `active` before you start and `done` when you finish, record the pass in `state.js` under `singlePass` (files, tests, commit, both timestamps), and update the `requirements` counts exactly as a ticket would. A run that finished the whole project and left the user a dashboard showing nothing but a running clock has failed at the one job the dashboard has. See [`phases/7-instruments.md`](./7-instruments.md).

## One ticket, one subagent, one fresh context

Never two tickets in one context. Accumulated context is precisely what makes long vibecoding sessions start breaking things that used to work — the model stops reading and starts remembering, and its memory is worse than the files.

The corollary is that a subagent knows **nothing** except what you hand it. Hand it the right things.

## Your hands

You dispatch; you do not build. Through the whole of Phase 5 your keyboard reaches exactly three things:

- `.autopilot/**` — state, manifest, interfaces, tickets, dashboard
- the project memory file, between its markers
- git — `add`, `commit`, `--stat`; never the diff itself

Every other file in the repository is written by someone whose context dies with the ticket. **This is rule 5 of the five in `SKILL.md`, and it loses to no argument** — least of all to the two that always arrive: "it's a two-line fix" and "the executor couldn't, I'll finish it myself".

The reason is arithmetic, not taste. Yours is the one context in this design that is never refreshed: it carries the manifest, the plan, every return and every stage transition from the first phase to the last. A subagent spends its context and throws it away; you spend yours and keep it. A diff you read at ticket 02 is still sitting there at ticket 08, competing for room with the requirement you are checking. That is the same mechanism the whole framework is built against — except here it cannot be escaped by starting fresh, because starting fresh means losing the run.

So the material never reaches you. What reaches you is a verdict, a list of names, one contract block per ticket.

**At tier T0 you are the crew**, so the rule cannot apply — there is nobody to hand the keyboard to. That is a cost of T0, not an exemption pattern: it is affordable only because a T0 run ends before the context fills. The moment there are tickets, there is someone else to type.

## What a subagent gets

| | |
|---|---|
| `interfaces.md` | the boundaries from the spec, plus what previous tickets built — read in full, first |
| its ticket file path and body | including the verbatim brief quotes |
| the spec sections its ticket names | not the whole spec |
| the test command and how to run one file | so it does not have to derive them |
| **the testing contract**, verbatim | the block below — it is what makes the tests worth having |
| the working directory and stack constraints | including what it must not touch |
| variable **names** for any credential | never a value, ever |
| the return contract | the block below, as a requirement, not a suggestion |
| **the path of the notes file it must write** | `.autopilot/<slug>/tickets/NN-<slug>.notes.md` — see below; without it a repair on this ticket starts blind |

**The executor is `autopilot-executor`**, defined in `.github/agents/`. It carries the edit and terminal tools; the reviewers do not. One invocation, one ticket, and no second message to it is possible — see *This harness* in `SKILL.md`.

**A rule that lives only in this file does not exist.** These phase files are read by the orchestrator; the code is written by someone who never sees them. Anything the executor must do has to travel in its prompt — and the testing contract is the one that gets left behind most often, because it reads like guidance rather than an input.

## The testing contract — goes into the prompt, word for word

```
Write tests only on the seams named in the attached spec sections.
Do not test everything indiscriminately and do not test internals.

The order: one test — implementation — the next test. Do not write tests
in a batch up front: written in advance, they assert imagined behaviour and
stop reacting to real changes.

A test asserts through the public interface and stays green after a refactor.
If it breaks when the internals moved but the behaviour stayed the same,
it is testing the wrong thing.

Take the expected value from anywhere except the code under test: a known
quantity, a hand-worked example, a line from the spec. An assertion that
computes the answer the same way the code does cannot disagree with it —
and never finds a bug.

An empty catch, a hardcoded happy path and a self-confirming test are
an acceptance criterion unmet, not met.
```

At tier T0 you are the executor, so this block applies to you directly.

## interfaces.md — the shared contract

The file that keeps eight independent contexts building one coherent project instead of eight incompatible halves. Without it, ticket 06 invents a second version of what ticket 03 already built, and nobody notices until the end.

Created in Phase 0, **seeded in Phase 4 from the spec's boundaries** — so the first subagent already reads the module map instead of inventing it. **You** — the orchestrator — append to it after each ticket returns, from that ticket's contract block. Subagents never write to it: parallel writers would collide, and a subagent cannot know what the others produced.

```markdown
# What is already built

Read by every executor before starting work. Do not reinvent what is here.

## Boundaries decided in the spec

- `intake` — owns the requests. `createRequest({phone, address, problem}) -> {id, createdAt}`
- `notify` — owns the sending. `send(channel, template, payload) -> {ok}`
- Test seams: `intake` and `notify`, only through these signatures

## Shared project rules

- Stack and versions, run and test commands
- What must not be changed (the config file, the schema, the shared module and its owner)
- If a dependency is missing — do not add it yourself, return `BLOCKED` with the name

## From ticket 01 — the shell

- `db.connect(path) -> Connection` — the single connection point
- Tables `requests`, `clients`; migrations in `migrations/`, owner — ticket 01
- Tests: `npm test`, one file — `npm test -- <path>`

## From ticket 02 — request intake

- `createRequest({phone, address, problem}) -> {id, createdAt}`
- Phone validation — `validatePhone(raw) -> {ok, normalized}`, do not write your own
```

Keep it to interfaces and rules. It is not a log — the log is `state.js`.

## The notes file — what a warm context would have held

Every executor writes `.autopilot/<slug>/tickets/NN-<slug>.notes.md` **before** it returns. Put it in the prompt as a requirement, in the same breath as the return contract.

This exists because a subagent here cannot be continued. Upstream, a shortfall goes back to the executor that still holds *why the code is the way it is* — and that "why" is exactly what a cold reader lacks and what makes it repair the symptom instead of the cause. The notes file is that knowledge, written down instead of held.

```markdown
# Ticket NN — notes

## Decisions
- Why this shape, where the ticket or the spec left the choice open.

## Tried and rejected
- What looked right and was not, and what went wrong when it was tried.

## Traps
- What the next person to touch these files needs to know and cannot see from them.
```

**Ten lines, not a diary.** No summary of the work — that is what the diff is for; no restatement of the ticket — the repairer is given the ticket too. Only what would be lost when the context dies. A ticket that genuinely raised no decisions writes three lines saying so, and that is the correct length.

**You never read it.** It goes from the executor to the executor's replacement, by path. Reading it yourself puts back exactly the material this whole design keeps out of your context.

## The return contract

Every subagent ends by returning exactly this. Put it in the prompt as a requirement, not a suggestion: without it you cannot update the instruments or the manifest, and the next ticket flies blind.

```
STATUS: DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
FILES: created and changed
TESTS: command → result (e.g. `npm test` → 34 passed)
INTERFACES: the public signatures, schemas, event formats you exposed
            — what the following tickets will use
REQUIREMENTS: R01 done | R01.1 placeholder — <what was missing>
CONCERNS: what was done with a caveat, and why
BLOCKERS: what was missing (a dependency, a decision, access)
NOTES: written | nothing worth recording
```

`NOTES` is one word, and it is there so that a missing notes file is caught here rather than discovered by the executor's replacement two hours later. `nothing worth recording` is a legitimate answer and still means the file exists, saying so.

**Demand it short, in the prompt: no more than 25 lines, no code, no diffs, no narrative of the work.** `FILES` is paths only; `INTERFACES` is signatures, not explanations of them. A subagent left to its own judgement returns an essay — it has just spent an hour on the work and wants credit for it — and eight essays cost you exactly what eight diffs would, arriving through a different door. A concern or a blocker that genuinely needs more gets one sentence; the detail stays in the code, where the next reader is anyway.

`NEEDS_CONTEXT` means the ticket was under-specified — the executor could not tell what was wanted. Treat it as a defect in Phase 4, not in the executor: re-cut the ticket with the missing detail and run it again. Two `NEEDS_CONTEXT` in one flight means the tickets are too thin across the board — go back and merge.

## Order of flight — waves, not one at a time

Phase 4 left every ticket with a `wave` and a `zone`. Fly wave by wave, and inside a wave fly everything at once: **launch the whole wave in a single message, one `autopilot-executor` call per ticket.**

That last sentence is the whole section. Two subagent calls sent in two messages run one after the other — the parallelism was computed in the plan and then quietly thrown away in the delivery. This is the default failure, not a rare one: a serial flight looks exactly like a correct one from the inside, and the only visible symptom is a user waiting an hour for work that took twenty minutes of real dependency.

- **Cap at three in flight.** Beyond that the orchestrator's own context fills with returns it cannot usefully hold, and the whole point of the design leaks away. A wave of five goes out as three, then two.
- **Zones must be disjoint.** Phase 4 guarantees it within a wave; check again at launch, because a re-cut ticket may have moved into someone else's files. Overlap → the second one waits for the next slot. **Same files → serialise, no exceptions.**
- **A wave is not a barrier.** The moment one ticket returns, launch the next ticket whose blockers are all done — **and only then** process the one that came back. That order matters: bookkeeping, review and repair all happen while the crew is flying, not while it waits. Waiting for the slowest ticket of a wave gives back exactly what the wave bought.
- **Nothing parallelises with ticket 01.** The shell, the schema, the shared primitives: everything else reads what it built.
- **When in doubt, serialise.** A wrong guess about disjoint files costs silent lost work; a serial run costs minutes.
- **In manual mode the flight is still hands-free.** Waves change how the agreed tickets are ordered, never which tickets get built.

## Before each ticket

Set the ticket's `status` to `in-progress` and its `startedAt` to now in `state.js` **before** launching the subagent. It costs one edit, and it is the difference between the user watching a ticket run and the user watching nothing happen for eighteen minutes.

For a wave, that is **one state write for the whole wave**, before the launch message — all of its tickets flipped together. Two clocks running side by side on the dashboard is what parallel work looks like; two tickets marked `in-progress` an edit apart is the same thing and costs half as much.

## After each ticket

In this order, every time:

1. **Read the contract block.** No block, or no notes file where `NOTES` claims one → the ticket is not finished. There is nobody to ask: re-run it, with the omission named.
2. **Append to `interfaces.md`.**
3. **Update the manifest** — `in-ticket` → `done` or `placeholder`, commit noted.
4. **Send the diff to review** — the ticket goes to `review` in `state.js` first, then the Phase 6 checklist runs, in `autopilot-review-spec` and `autopilot-review-craft`, neither of which can edit a file ([`phases/6-review.md`](./6-review.md)). What comes back to you is a verdict and a list of findings. The diff itself does not.
5. **Run the full test suite**, not just the ticket's own tests — and truncate the output — the command for your shell is in the table in `SKILL.md`, and on Windows PowerShell it is `cmd /c "<test command> 2>&1" | Select-Object -Last 30`. You need two things from it, green-or-red and the names of what failed, and both survive the truncation; the other two hundred lines are pure leak. A regression introduced now costs minutes; found eight tickets later it costs the evening.
6. **Red test, or a finding that has to be fixed → repair** (below): the ticket goes to `repair` and its `repairs` count goes up by one, then re-run 4 and 5 over the repair alone. **Nothing is committed on red**, and nothing is repaired by you.
7. **Commit** — one commit per ticket, the ticket number in the subject, and only now does the ticket become `done`. These are the user's rollback points.
8. **Update the instruments** ([`phases/7-instruments.md`](./7-instruments.md)) — one line of state, one line of the dashboard: the ticket's `finishedAt`, tests and commit, the `requirements` counts, the `build` and `review` stage notes ("3 of 5 tickets done"), `updatedAt`.
9. **Top up the project memory — only if something was discovered.** The real test command, a gotcha that cost time, a new variable in `.env.example`. One line appended between the markers, never a rewrite; the architecture is written once, at the end. Most tickets add nothing, and that is the correct rate. Rules in [`phases/9-memory.md`](./9-memory.md).
10. **Tell the user one plain-language line**: "The bot takes requests — 3 of 8 done". No diffs, no jargon, no file lists.

Steps 4 through 6 are where the run is usually lost. Done as written, one ticket costs you a verdict, thirty lines of test output and a contract block. Done by hand — "I'll look at the diff myself, it's small anyway" — the same ticket costs you the diff, the test log and every file you opened to fix it, and you pay that eight times.

**Steps 4–7 hold up the commit, not the crew.** The list is the order for *this* ticket; it is not a queue the rest of the flight waits in. The moment a ticket returns, the next launchable ticket goes out — and only then do you walk the list for the one that landed. Its review runs while the next ticket is being written, and the wall-clock cost of reviewing everything drops to roughly nothing.

What this does not buy is a shortcut: the ticket is still committed only after its review and a green suite. Nothing lands unreviewed because something else was in flight; the review simply stopped being the thing everyone waits for. The one ordering that stays strict is a ticket whose dependents are pending — do not launch a dependent on an unreviewed parent, because a finding there invalidates the ground the dependent is standing on.

### When two tickets return together

Process them **one at a time, each through the whole list above**. Two returns are not one event.

- **One commit per ticket, always.** A shared commit takes away a rollback point the user paid for, and blames two tickets for one regression.
- **Run the full suite after each**, not once after both. Otherwise a red test has two possible authors and you have to bisect what you could simply have known.
- **`interfaces.md` is appended by you, in return order**, one block per ticket. Subagents never write to it — parallel writers collide.
- **Two returns claiming the same interface is a plan defect, not a merge problem.** It means the zones overlapped: keep the one that fits `interfaces.md`, and re-cut the other rather than reconciling two versions of the same thing by hand.

## Repair — two kinds, two addresses

A ticket comes back imperfect in two very different ways, and telling them apart is the whole of this section:

- **Shortfall** — a red test, a review finding, an acceptance criterion met in letter and dodged in substance. The executor *could* have done it and did not.
- **Refusal** — `BLOCKED`, `NEEDS_CONTEXT`, or a repair that has already failed. The executor tried and could not.

| | Shortfall | Refusal |
|---|---|---|
| Goes to | a fresh `autopilot-executor` — **plus the ticket's `.notes.md`** | a fresh `autopilot-executor`, **and only with a changed approach** |
| You send | the ticket, `interfaces.md`, the notes file, the condition to satisfy — and nothing else | all of that, plus the error, the failing test named, and the path now spelled out |
| Because | it must know why the code is the way it is, or it repairs the symptom and breaks the reason — and here that knowledge lives in the file, not in a context | the previous approach *is* the failure; the same request produces the same answer, whoever receives it |

**Both columns start cold in this harness** — there is no follow-up to send. That changes what a repair costs, not who does it: a repair still goes down, never onto your keyboard. The notes file is what keeps the left-hand column from collapsing into the right one, which is why an executor that skipped it is a defect worth catching at return time rather than at repair time.

A repair prompt is the ticket prompt again, plus one block:

```
This ticket has already been built once. Read its notes file first:
.autopilot/<slug>/tickets/NN-<slug>.notes.md

The test `parses empty address` is red:
<last 10 lines of the output>

Fix that. Change nothing the finding does not require.
Update the notes file and return the contract block again.
```

- **Two repairs on one ticket is the ceiling.** By the third, the finding is not a shortfall — it is a ticket that cannot be built as written, and it takes the failed-ticket path below: a changed approach, or a stop and a sentence to the user.
- **State the finding as a condition, never as "improve it".** "Make it better" is an invitation to rewrite what already worked. Every repair names something checkable: this test green, this field visible, this error handled.
- **The repair returns the contract block again** — new `FILES`, new `TESTS`. A repair that returns nothing is a ticket you cannot honestly commit.
- **The author repairs; someone else judges.** Sending the finding back to the executor is cheap precisely because it keeps its context — which is also why it cannot review its own repair. Step 4 stays with a subagent that did not write the code.
- **The one thing that is never a fallback is taking the keyboard yourself.** It feels like the cheapest option on the table and it is the most expensive thing in the phase: the diff it puts in your context is still there eight tickets later, competing with the requirement you are checking.

## When a ticket fails

The right-hand column above, and its rules are the strict ones.

Retry **once**, in a fresh context, with the error attached and the failing test named. If that fails too, one further attempt is allowed **only with a changed approach** — a different design decision, a different library, a path the ticket now names explicitly. Running the same attempt again with more hope is not a retry, and it is the only version of this that is forbidden.

After that the flight stops: tell the user in plain language what is blocking and what you need from them. Do not improvise around a blocker, and do not silently narrow the ticket to whatever happened to work — a quietly reduced ticket is a lost requirement, and this whole design exists to make that impossible.

Mark it `failed` in `state.js` and `placeholder` in the manifest, with the reason.

One failure does not abort its wave-mates — they are independent by construction, so let them land. What it does stop is everything **downstream**: its dependents stay `pending`, and naming which ones are now blocked is part of the sentence you tell the user.

## When the build contradicts the plan

The plan was written before the code existed, so sometimes the code is right and the plan is wrong: a data model that does not hold, an interface the spec assumed cannot exist, two requirements that turn out to be incompatible in practice. This is ordinary, it is not the executor's error, and it needs a path — because without one what actually happens is worse. The executor quietly builds something else, the spec keeps claiming otherwise, and every check downstream measures the build against a document that stopped being true at ticket four.

A subagent that hits this returns `BLOCKED` or `DONE_WITH_CONCERNS` with what it found. **You decide, in the orchestrator's context — never the executor**, and never by letting it stand. Deciding is yours; the code that follows from the decision is still written below you, by follow-up or by a fresh ticket:

1. **Amend the spec section.** Edit the affected part of `spec.md` in place, keep the story marks, and add one line saying what the code proved and at which ticket. From the first ticket onward the spec is a living document; the brief and the manifest quotes are not.
2. **Record a `D##` row in the manifest** — *discovered*. Its Basis is the finding, and it names the requirement it serves. This is not a requirement the user made; it is a constraint reality imposed, and it carries a status and appears in the final report like everything else.
3. **Re-cut only what the change invalidates.** Landed tickets stay landed. Unstarted tickets get their spec references updated; a ticket whose whole point disappeared is cut and its requirements go back to `in-spec` to be re-covered.
4. **Tell the user one line, in every mode including full:** "The schema from the plan didn't hold two addresses per request — fixed it, the requirement is the same". They do not need the reasoning. They do need to know the plan moved, because a plan that moves silently is how the final report and their memory of the project stop matching.

Two things this is not:

- **Not a way to drop a requirement.** A requirement the code proves impossible is a question for the user — in full mode, an ASSUMPTION plus a placeholder — never a `D##` that quietly retires it.
- **Not a route for good ideas.** A discovery is something the code demonstrated, not something you thought of while writing it. Ideas are still `A##`, still need a parent and the proportion limit, and at `strict` are still forbidden.

## Testing — what you check when it comes back

The rules themselves went out in the prompt (above). What stays here is the part the executor cannot do for itself: **a green suite is evidence only if the tests could have been red.**

That check reads assertions, so it is not yours either — it belongs to the Craft reviewer, and its three questions are written out in [`prompts/craft-review.md`](../prompts/craft-review.md), which is the file that reviewer is given. It is the one review instruction most easily lost on the way down, because it looks like something the pass count already answered; handing over the file is what makes it arrive.

Your part is two moves: see that the file went down with the reviewer ([`phases/6-review.md`](./6-review.md)), and treat what comes back as a Craft finding of kind *silent narrowing*, fixed in this ticket. A bad test is worse than a missing one: the missing one is visible.
