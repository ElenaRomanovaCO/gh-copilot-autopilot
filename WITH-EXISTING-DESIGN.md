# When the design is already done

Autopilot is written for someone who arrives with an idea and nothing else. Its first three phases exist to turn words into requirements, ask about the forks, and work out everything the idea did not say.

If you arrive with a written brief, a technical design and a system diagram, you have already done that. Invoked the ordinary way, the run will do it again and land somewhere slightly different from the design you settled on — not wrong, just not yours.

This note is what to change. It applies to any project you bring prior design work to; nothing in it is specific to one.

[FIRST-RUN.md](FIRST-RUN.md) is still the guide to what a run looks like. This one only covers the dials.

---

## What still earns its keep, and what does not

| Still worth having | Now largely transcription |
|---|---|
| **The manifest** — your design is prose; this turns it into numbered requirements with a status each, checked at four gates | **The briefing** — your design already answers the forks it would ask about |
| **The plan, the crew, the review** — the cut into tickets, a fresh context per ticket, three axes of review | **The spec** — a restatement of decisions you have already made |
| **Blind acceptance** — someone reads your brief, runs the result, and reports what is not there | |

The manifest is worth having however much design you did. It is the difference between *what I wrote down* and *what got built*, tracked to the end — and no amount of up-front design produces it, because a design document has no notion of a requirement's status.

---

## Four changes to how you start

### 1. Run it `strict`

That is the dial for exactly this case: no capabilities invented, deepening only where a requirement cannot work without it. The default `normal` elaborates on top of what you said — right for a bare idea, wrong for a settled design.

### 2. Put the design in the repo before you start

Preflight looks for `CONTEXT.md` and `docs/adr/`. If it finds them, the spec is required to speak that vocabulary rather than invent synonyms, and to say so out loud where a decision contradicts a recorded one instead of quietly overriding it.

A file in the repo is worth more than the same text pasted into chat, and it keeps working on the next run.

### 3. Convert the diagram to text

This is the one that will actually cost you something. An image may or may not be readable by the agent, and the failure is not an error message — it is a spec that quietly ignored the diagram.

Put it in the design document as a **mermaid block** or an ASCII sketch. Boxes and arrows written as text are read every time.

### 4. Leave the mode at `semi`

You still want a question when it finds a real gap between the design and what has to be built. `full` would decide it for you and mention it at the end, in the assumptions list — which is the wrong end of the run to discover a design decision you would have made differently.

---

## The risk worth naming in the brief

That the spec drifts from your design. Say in the brief that the design is a constraint, not a suggestion.

There is a legitimate path for departures, and it is worth having: when the code proves the design wrong — a data model that does not hold, an interface that cannot exist — the spec is amended, a `D##` row records what the code demonstrated and at which ticket, and you are told in one line. Reality gets to correct a design; that is not the problem.

What `strict` plus an explicit instruction closes off is the other thing: a redesign nobody announced.

---

## If it is a proof of concept

The tier is read from what has to be built, never from how much you wrote. A POC lands at one of the two smallest:

- **T0** — one surface, one layer, no external service. **No tickets at all**: it is built in one pass, in one context. Autopilot's largest advantage — a fresh context per ticket — does not engage here. What you are buying is the manifest and the blind acceptance.
- **T1** — a few surfaces over one data shape, two or three tickets. This is where the machinery starts paying for itself.

**Be honest about which one you have.** If the POC is genuinely one script or one endpoint, this is heavier than the job needs. If it is three or four moving parts that have to work together, it fits.

Say *proof of concept* in the brief, in those words — throwaway quality, shortest path to something runnable, no production hardening. It then becomes a tracked requirement and constrains the build like any other, instead of a preference the run can drift away from.

And leave `polish` off. It exists to finish something against a reference, and it costs real time and real money.

---

## What that looks like typed

```
/autopilot strict docs/brief.md — docs/design.md and the diagram in it are
the technical design: follow them, don't redesign. If something in the design
can't work, tell me rather than routing around it. This is a POC — shortest
path to something I can run, no production hardening.
```

The path is read as the task; the words beside it join the same brief. Every sentence there becomes a tracked requirement rather than a preference.
