# Phase 6 — Checklist

Review of each ticket's diff along three axes. Not sequential — it runs inside Phase 5, after every ticket.

Three axes, because a change can pass one and fail another:

| Axis | Question | Fails when |
|---|---|---|
| **Manifest** | does the diff deliver what the user asked for, in their words? | a requirement quietly shrank |
| **Spec** | does it implement what the spec decided? | the executor improvised |
| **Craft** | is the code fit to build on? | it works today and blocks tomorrow |

Report them **separately**. Merging or ranking findings across axes lets one mask another — clean code implementing the wrong thing looks fine until you read the axes apart.

## Which axis a finding belongs to

One question decides it: **could the executor have known?**

It saw its ticket, the spec sections that ticket named, and `interfaces.md`. Nothing else.

- **Yes, it could have known** → axis Spec or Craft. This is a defect of the code, and it is fixed in this ticket.
- **No, it could not have known** → axis Manifest. The requirement was lost on the way down, and the defect is in the spec or in the cut — **not in the executor**. It still gets fixed, but do not re-run the subagent against words it was never given: repair the ticket first, or the spec, then run it.

This is the same line the whole framework runs on, seen from close up. Between the gates everything measures against the spec, because the spec is the contract the crew actually received; only at G2 and G4 does anything measure against the brief, and there the subject is the plan, not the code. **The manifest is what lets axis 1 exist at all** — without it the brief is prose and cannot be checked one ticket at a time.

## Scale to the ticket

Cheap early, delegated once it starts costing. A review that costs more than the ticket is its own kind of waste — and so is a review that quietly spends the one context the run cannot replace.

| Where the run is | How |
|---|---|
| tier T0 — no tickets at all | all three axes yourself, inline: there is nobody to delegate to, and the run ends before it matters |
| the first two tickets, diff under ~150 changed lines | inline still allowed |
| every ticket after that, and anything touching shared modules | Manifest+Spec in `autopilot-review-spec`, Craft in `autopilot-review-craft`, launched in one message so they run in parallel |
| the final whole-project pass at the end | separate subagents, per [`phases/8-final.md`](./8-final.md) |

**Why the threshold is not diff size alone.** A hundred-line diff costs the same to read whenever it arrives; what changes is what you have left to spend it from. The orchestrator's context is the only one in the run that is never refreshed ([`phases/5-subagents.md`](./5-subagents.md)), and inline review is how it fills — undramatically, one ticket at a time, until ticket 08 is being judged by a reader who has been awake since the brief. The concession for the first two tickets is there because a reviewer's setup cost is real and early context is cheap. It expires because neither stays true.

## The review log — the reviewer's memory, on disk

Upstream this section says *keep one reviewer for the whole run*. Here you cannot: a subagent is a single invocation and cannot be sent a second message. So the reviewer is fresh every ticket, and the thing that made keeping one worthwhile is written to a file instead.

What was worth keeping was never the setup. It was this: a reviewer that saw ticket 02 can see that ticket 05 quietly contradicts it — a whole class of defect no per-ticket reviewer can reach, because nothing in its inputs mentions ticket 02 at all. **That is the panoramic view the orchestrator used to have and can no longer afford**, and losing it silently is the worst outcome available here.

Two files under the flight directory, **one owner each, because the two reviewers run in parallel and two writers on one file collide**:

- `.autopilot/<slug>/review-log-spec.md` — owned by `autopilot-review-spec`
- `.autopilot/<slug>/review-log-craft.md` — owned by `autopilot-review-craft`

Every reviewer prompt carries the same two instructions, first and last:

```
Read .autopilot/<slug>/review-log-<axis>.md before you read the diff.
It is what earlier reviews of this project already established.

After your verdict, append one entry to it. Nothing else in the file changes.
```

The entry is four lines, and it records what the *next* reviewer needs rather than what this one found:

```markdown
## Ticket 05 — cart totals
- Established: money is integer cents everywhere; `formatMoney` in `lib/money` is the only formatter.
- Pattern to hold: handlers validate at the boundary, never inside the domain.
- Watch: ticket 03 also has a rounding path — a second one appeared here and was rejected.
```

- **Established / Pattern / Watch, and nothing else.** Not the findings — those went to the executor and are done. A log of findings is a list of things already fixed; a log of what the project has committed to is what catches the contradiction three tickets later.
- **Short, or it stops being read.** Roughly four lines per ticket. Twelve tickets is a page — which a fresh reviewer reads in a second and an essay is not.
- **The Craft log is the one that earns its keep.** Reinvention and divergent change are invisible to a reader who does not know what the earlier tickets built, and `interfaces.md` records the signatures, not the conventions.
- **Each log belongs to one axis, all run.** Never merge them, never let one reviewer write the other's file, and never write to them yourself — a log the orchestrator maintains is the orchestrator reading diffs again, through a different door.

## What a reviewer gets

A reviewer knows nothing you do not hand it — the same rule as for an executor, and it bites harder here, because **axis Manifest is built entirely out of words the subagent has never seen.** A reviewer sent off with just the diff will quietly review two axes and report three.

| | Manifest + Spec | Craft |
|---|---|---|
| the diff — `git diff` over the ticket's range, or the files its contract block named | ✓ | ✓ |
| **the manifest rows the ticket names, with the verbatim brief quotes** | ✓ | — |
| the spec sections the ticket named — the same ones the executor got | ✓ | — |
| `interfaces.md` | ✓ | ✓ — the only way Reinvention is visible |
| the ticket body and its acceptance criteria | ✓ | ✓ |
| whatever the repo documents about how code is written | — | ✓ |
| **[`prompts/craft-review.md`](../prompts/craft-review.md), by path** — the smells, the assertion-level testing check, the return format | — | ✓ |
| what it must not do: repair nothing, refactor nothing, open no files outside the diff to "understand better" | ✓ | ✓ |

**Give each one only its own axes.** A reviewer handed material for an axis it was not asked to judge will judge it anyway, badly and without saying so — and two overlapping half-reviews are what the separation of axes exists to prevent.

**That table is every ticket, not just the first.** A cold reviewer holds nothing, so the standing material is sent every time — and this is the cost of a stateless harness, paid once per ticket, visibly. What it must *not* be sent is the review log's contents pasted into the prompt: give the path and require it to be read, exactly as with `prompts/craft-review.md`. The difference matters, because a log the reviewer read is a record it can reason against, and a log you pasted is a wall of text at the top of its context that it will skim.

**Give the Craft reviewer the path, and require it to read the file before the diff.** A path is not a delivery: what makes the check exist is the reviewer having read it, so say so as a requirement and expect the return format from that file. Every ticket, since every reviewer here is cold — `autopilot-review-craft` has the read tools to do it, which is why the path is enough.

### What a reviewer returns

```
AXIS: manifest | spec | craft
LOG: appended
VERDICT: clean | findings
FINDINGS: <axis> · <file:line> · what is wrong · what condition must hold
          — one sentence per finding, as a condition, not a wish
BLOCKING: which of the findings block the commit
```

**No more than 20 lines, no code fragments and no diff.** A finding phrased as a condition can be forwarded to the executor's replacement unchanged; a finding phrased as "could have been tidier" has to be rewritten by you before it can go anywhere, and rewriting it means reading the diff — which is the whole thing this arrangement exists to avoid.

## Axis 1 — Manifest

The axis that does not exist in ordinary code review, and the one this framework is built around.

Take the ticket's Requirements line, pull those rows from `manifest.md`, and read the **verbatim brief quotes** — not the spec's version, not the ticket's summary. Then, for each:

- Is it delivered end to end, or only the easy half?
- Was it narrowed on the way down? A requirement that entered as "the client sees the status" and left as a status stored in the database but shown nowhere is a shrunk requirement, not a done one.
- Does a `placeholder` sit exactly where a user fact belongs — and is it visibly a placeholder, not a plausible invention?

Verdict per requirement: `done` / `partial` / `missing`, and `partial` or `missing` means the ticket is not finished.

## Axis 2 — Spec

Against the spec sections the ticket named:

- **Missing** — a decision the spec made that the diff does not implement.
- **Extra** — behaviour in the diff that no one asked for. Scope creep is not a bonus; it is untested surface with no requirement behind it and nobody to maintain it.
- **Wrong** — implemented, but not the way the spec decided. Especially: a second version of something `interfaces.md` already provides.

A diff that departs from the spec because the spec turned out to be wrong is **not** a finding on this axis — but it is only legitimate once the spec has been amended and a `D##` row exists. An undocumented departure is `Wrong`, however good the reason: see [`phases/5-subagents.md`](./5-subagents.md).

Quote the spec line for every finding.

## Axis 3 — Craft

**The whole of what this reviewer judges by is [`prompts/craft-review.md`](../prompts/craft-review.md), and it goes down as a path, not as your retelling of it.** The list of smells, the assertion-level testing check and the return format live there because they are the subagent's material, not yours: an orchestrator that reads them keeps them until the end of the run and gains nothing, and an orchestrator that paraphrases them ships a weaker version of the check than the one that was written.

What stays here is the part you decide:

- Whatever the repo documents about how code is written **wins over that file** — hand the reviewer the repo's conventions too, and say which wins.
- Everything in it is a **judgement call** — "possible Feature Envy", never a hard violation — and anything tooling already enforces is out of scope: a linter finding is not a review finding.
- Three of its entries exist because subagents cause them — *Reinvention*, *Silent narrowing*, *Invented fact* — and the first is the reason the Craft reviewer gets `interfaces.md`. Without it that whole class of finding is invisible.
- The testing check in it is the line most often lost on the way down, because a pass count looks like it already answered the question. Handing over the file is what makes it arrive; nothing about it is checkable from the count.

## What to do with findings

**Every "fix" below happens in the ticket, not in the orchestrator and not in the reviewer.** A finding goes back to the executor that wrote the code, as a **follow-up** stating the condition — [`phases/5-subagents.md`](./5-subagents.md). The reviewer judged and is done; a reviewer that also repairs is a check that has stopped being one. Two follow-ups is the ceiling, after which the finding becomes a failed ticket and takes that path instead.

- **Manifest `partial`/`missing`, or Craft "invented fact"** → repaired now, in this ticket, before the commit.
- **Spec "extra"** → removed, unless it is genuinely required for the rest to work — then say so in one line in the commit message.
- **Craft judgement calls** → repaired if the fix is small and local. If it is structural, note it in `state.js` under `concerns` and carry it to the final report. Do not start a refactor inside a ticket that was not about refactoring.

Refactoring belongs here, not inside the red-green loop. Cleaning up while chasing a failing test is how both jobs get done badly.

## Reporting

To yourself, structured, per axis. **To the user, nothing** — unless something is being carried to the final report as a concern. The user gets one plain line per ticket from Phase 5, not a review.
