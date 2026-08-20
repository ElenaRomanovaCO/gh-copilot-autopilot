# Autopilot · Copilot Edition

A port of the [Autopilot](https://github.com/nick-vels/skills) skill to VS Code + GitHub Copilot agent mode.

The skill keeps its original name — you invoke it with **`/autopilot`**, and everything the upstream project documents about modes, depth and polish applies here word for word. "Copilot Edition" names the distribution, not the command.

The package folder is `gh-copilot-autopilot`; the skill folder inside it must stay named exactly `autopilot`, because a skill's folder name has to match the `name:` in its `SKILL.md`.

**Installs by copying folders.** No `npx`, no installer, no Node.js, nothing to run. Download the repository, copy two folders, restart VS Code.

---

## What is in this package

```
.github/
├── skills/autopilot/     the skill itself — SKILL.md, phases/, prompts/, the dashboard template
└── agents/               four custom agents the skill dispatches to
    ├── autopilot-executor.agent.md      builds one ticket   (read · edit · terminal)
    ├── autopilot-review-spec.agent.md   Manifest + Spec     (read · terminal, no editing)
    ├── autopilot-review-craft.agent.md  Craft               (read · terminal, no editing)
    └── autopilot-blind-check.agent.md   gates G2 and G4     (read · terminal, no editing)
.vscode/
└── settings.json         terminal auto-approve and subagent limits

README.md                 install, and what differs from the original
FIRST-RUN.md              what a run actually looks like — read this once before the first build
```

---

## Install

### Get the files

On the GitHub page for this repository: **Code ▸ Download ZIP**, then extract it. Windows sometimes marks a downloaded ZIP as blocked — right-click the ZIP ▸ **Properties** ▸ tick **Unblock** before extracting, or the extracted markdown may be unreadable to some tools.

If `git` is available, `git clone` works just as well. Nothing in this package is built or generated.

### Option A — once, for every project *(recommended)*

The skill and the agents live in your user profile, and every repository you open gets them.

```powershell
# from the extracted gh-copilot-autopilot folder
New-Item -ItemType Directory -Force "$env:USERPROFILE\.copilot\skills" | Out-Null
New-Item -ItemType Directory -Force "$env:USERPROFILE\.copilot\agents" | Out-Null

Copy-Item -Recurse -Force ".\.github\skills\autopilot" "$env:USERPROFILE\.copilot\skills\"
Copy-Item -Force ".\.github\agents\*.agent.md" "$env:USERPROFILE\.copilot\agents\"
```

Or do it in Explorer: paste the `autopilot` folder into `%USERPROFILE%\.copilot\skills\`, and the four `.agent.md` files into `%USERPROFILE%\.copilot\agents\`. Create the folders if they are not there.

Then open **Settings (JSON)** — `Ctrl+Shift+P` ▸ *Preferences: Open User Settings (JSON)* — and merge in the keys from [`.vscode/settings.json`](.vscode/settings.json). Read them first; the terminal auto-approve list is the one that decides how often the build stops to ask you.

### Option B — per project

Copy `.github` and `.vscode` from this package into the project folder. If the project already has either, **merge** rather than replace — a `.github` folder usually holds workflows you do not want to lose.

Option B commits the skill to the repository, so everyone on the team gets it. Option A keeps it to you. On a work machine where the repositories are not yours to reshape, Option A is the polite one.

### Restart

**Restart VS Code.** Skills and agents are read at startup.

Installed this way, the skill is available in every project you open, and the downloaded package has done its job — you can delete it. **Builds are run in a VS Code window opened on the project's own folder, never inside this package.** [FIRST-RUN.md](FIRST-RUN.md) opens with why that matters.

### Check it took

Open Copilot Chat, make sure the mode is **Agent**, and type `/`. `autopilot` should be in the list. If it is not:

1. Confirm the path: `%USERPROFILE%\.copilot\skills\autopilot\SKILL.md` must exist — the folder name must be exactly `autopilot`, matching the `name:` in the file.
2. Confirm you restarted, not just reloaded the chat.
3. Check `chat.useAgentSkills` is not set to `false` anywhere — a work machine may have policy settings.

---

## Use

Open VS Code in the folder the project should live in. In Copilot Chat, in Agent mode, with Claude Opus selected:

```
/autopilot I want a Telegram bot that takes repair requests and puts them into a Google Sheet
```

Or point it at a file, which is the better idea for anything real — the more you write, the less gets guessed on your behalf:

```
/autopilot brief.md
```

```
/autopilot deep docs/idea.md just no card payments
```

Autopilot replies with the mode it is flying in, opens a dashboard in your browser, and starts.

**[FIRST-RUN.md](FIRST-RUN.md) is the one to read before you do this for real** — what happens at each stage, where the subagents appear, what the chat looks like while it builds, where it will stop and ask you, how to resume after the session dies, and what to check on the first build. The full rules of modes, depth and polish are in [`SKILL.md`](.github/skills/autopilot/SKILL.md), which the agent reads so that you do not have to.

---

## What is different from the Claude Code original

Four things, and all four are consequences of the harness rather than changes of mind.

**Subagents are stateless here.** Copilot cannot send a second message to a subagent it already ran. Two parts of the original depend on that, and both now keep their memory in a file instead of in a context:

- Every executor writes `.autopilot/<slug>/tickets/NN-<slug>.notes.md` before it returns — the *why* behind the code, which is what a repairer in a cold context would otherwise lack and the reason a cold repair fixes symptoms.
- Reviewers append to `review-log-spec.md` and `review-log-craft.md`, and read their own log before every review. That is the cross-ticket eye — the thing that notices ticket 07 quietly contradicting ticket 03 — relocated to disk.

**The crew is named and its permissions are real.** The original asks a reviewer not to edit files. Here the reviewers and the blind checker have no file-editing tool at all, so the ask is not a rule anyone has to keep. They keep terminal access, because a reviewer that cannot run `git diff` is reviewing the current files rather than the change, and because the blind checker's entire job is to *run* the project.

**The dashboard opens in your browser, not in a pane.** VS Code's Simple Browser cannot be opened from a shell command, and pointed at a `file://` path it cannot load `state.js` beside the page — the result is a dashboard that shows "no state yet" for the whole run. So the run hands the file to Edge or Chrome, which handles it correctly, and leaves no server behind.

**The shell is PowerShell.** Timestamps, output truncation and file copying all differ, and the failure is silent: `date -Iseconds` produces nothing usable in PowerShell, which shows up only as a dead dash on the dashboard for the rest of the build. Phase 0 settles the shell and records it; `SKILL.md` carries the command table.

The project memory file is `AGENTS.md`, which VS Code Copilot reads by itself. `.github/copilot-instructions.md` is deliberately left alone: it is loaded into every request in the repository and belongs to the team, not to one run.

---

## Two things to check before you point this at real work

Both are covered again in [FIRST-RUN.md](FIRST-RUN.md), with what to do when they bite.

**Tool names move between VS Code versions.** The `tools:` lists in the `.agent.md` files use the current namespaced form (`search/codebase`, `edit`, `terminal`). If a reviewer behaves as though it has no tools — or turns out able to edit files — open the agent file in VS Code and use its tools picker to re-select; VS Code writes the names its build actually knows.

**Cost is the real constraint, not capability.** Autopilot is subagent-heavy by design: one executor per ticket, two reviewers per ticket, and up to three more at the finish. A Copilot agentic session is billed at the highest-multiplier model used anywhere in the chain, so pinning the executors to a cheaper model with `model:` in their agent files may not save what you would expect. **Run one throwaway project first and watch your usage** before committing this to anything that matters.

---

## Updating

Re-download and copy over the top. Nothing keeps state outside the folders above, so a copy-over is the whole update.

To remove it: delete `%USERPROFILE%\.copilot\skills\autopilot` and the four `autopilot-*.agent.md` files from `%USERPROFILE%\.copilot\agents`.

`.autopilot/` inside a project is the record of that build, not part of the installation. Deleting the skill does not touch it.

---

## Credit

**The Autopilot skill is Nick Vels's work** — [nick-vels/skills](https://github.com/nick-vels/skills), MIT. The phases, the gates, the manifest design, the dashboard and very nearly all of the prose in `.github/skills/autopilot/` are his, unchanged: of 2,499 lines of skill text, this port rewrites about 250.

What is new here is the adaptation — the four agent definitions, the file-backed replacements for stateless subagents, the Windows and PowerShell work, the VS Code settings, and these two documents.

Both are MIT, and both copyright lines are in [LICENSE](LICENSE). If Autopilot is useful to you, the thanks belong upstream.
