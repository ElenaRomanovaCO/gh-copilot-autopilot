# Phase 0 — Raising the project memory

Two things happen here and nothing else: **pick the file, write the skeleton.** What may be appended during the build, the full description written from the finished code, and the ADRs all live in [`phases/9-memory.md`](./9-memory.md) and are read when they happen — the second inside Phase 5, the last two in Phase 8. None of it applies until the build is over.

## Which file

The repo needs a file that tells the **next** session what this project is — `CLAUDE.md` or `AGENTS.md`. Decided by detection, in this order. Stop at the first match.

| Check | File |
|---|---|
| `CLAUDE.md` already exists | `CLAUDE.md` |
| `AGENTS.md` already exists | `AGENTS.md` |
| both exist | the one that already holds the project description; if neither does, `AGENTS.md` — and **leave the other file alone** |
| `.cursor/` directory | `AGENTS.md` |
| `.claude/` directory | `CLAUDE.md` |
| nothing matched | `AGENTS.md` |

**In this harness the answer is `AGENTS.md` unless the repo already says otherwise**, and the table above is short for that reason. VS Code Copilot reads `AGENTS.md` by itself, so the memory this run writes is loaded into the next session with nothing to configure — which is the entire job of the file. `.github/copilot-instructions.md` is *not* the place for it: it is loaded into every request in the repository, it is the team's file rather than the run's, and a project description that grows through a build does not belong in a header that ships with every prompt. Leave it untouched if it exists.

The pointer file the upstream skill writes (`CLAUDE.md` containing `See @AGENTS.md`) is dropped here — nothing in this harness reads it, and an unread file that has to be kept in sync is worse than no file.

- **An existing file always wins over detection.** The repo has already answered the question; asking it again is how you end up with two half-filled memory files.
- **Never duplicate the text into both files.** Two copies of a project description drift within one run.
- Record the choice in `state.js` as `memoryFile`, so a resume does not re-derive it.

**This is never a question for the user** — in any mode, including manual. It is a process decision, like where ticket files live, and Phase 0 answers those itself. It is not, however, a *secret* decision: one line in the opening block, together with the mode.

> Project memory — `AGENTS.md`. Tell me if you want a different one.

Say it and move on. **Do not wait for an answer** — if the user names a different file later, switch and move the block; renaming a markdown file costs nothing, which is exactly why this never earned a gate.

## The skeleton

Cheap, written before anything is built, and it is what survives an interrupted run. Only what is already known — everything Autopilot writes sits between the two markers, in every case, including a file it created itself:

```markdown
<!-- autopilot:start -->
# <Project name>

<One line: what this is and who it is for.>

## Commands

| Command | What it does |
|---------|------------|
| `<install>` | Install dependencies |
| `<run>` | Run locally |
| `<tests>` | Run the tests |

## How Autopilot works here

The build is driven by the `/autopilot` skill. Requirements, spec and tickets — in `.autopilot/`.
Progress — `.autopilot/dashboard.html`. The rule: a requirement in `manifest.md`
can be removed only by the user.

If the work is continuing — say "continue autopilot": the state comes back up
from `.autopilot/state.js`, nothing needs to be re-asked.
<!-- autopilot:end -->
```

Commands that are not known yet are simply absent. **An invented command is worse than a missing one** — the next session runs it, it fails, and now the whole file is suspect.

**Anything the user wrote outside the markers is untouchable.** A brownfield repo whose `CLAUDE.md` carries a team's hard-won rules must come out of an Autopilot run with those rules intact. If the markers are missing on a later run but Autopilot's sections are recognisably there, wrap them — do not append a second copy. The reasoning is in [`phases/9-memory.md`](./9-memory.md).

In the third Phase 0 case — a configured repo starting a new feature — the memory file already exists: **top it up, do not rewrite it.** The skeleton is written once, in the run that created the repo.
