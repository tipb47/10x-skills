# my-claude-skills

**tipb47's skills for agentic coding CLIs.** Runs on
[Claude Code](https://claude.com/claude-code) and [Codex CLI](https://developers.openai.com/codex/cli).

Three skills, installable together or one at a time:

| Skill | Command | What it does |
|---|---|---|
| [Sprint Director](#sprint-director--sprint) | `/sprint` | Sprint-directed multi-agent development methodology: `init`, `direct`, `status`, `clean` |
| [Where Am I](#where-am-i--whereami) | `/whereami` | In-chat situation report for the current session |
| [Handoff](#handoff--handoff) | `/handoff` | One copyable message that transfers the session to a fresh agent |

---

## Install

### Option A — plugin (Claude Code)

From inside Claude Code:

```
/plugin marketplace add tipb47/my-claude-skills
```

Then install everything:

```
/plugin install my-claude-skills@my-claude-skills
```

…or just the skills you want:

```
/plugin install sprint-director@my-claude-skills
/plugin install whereami@my-claude-skills
/plugin install handoff@my-claude-skills
```

Restart the session so the skills load.

### Option B — manual copy (any runtime)

Copy any skill folder into your user skills directory:

```bash
git clone https://github.com/tipb47/my-claude-skills.git

# Claude Code — pick the skills you want:
cp -r my-claude-skills/skills/sprint    ~/.claude/skills/sprint
cp -r my-claude-skills/skills/whereami  ~/.claude/skills/whereami
cp -r my-claude-skills/skills/handoff   ~/.claude/skills/handoff

# Codex CLI: same, into ~/.codex/skills/
```

Restart your session. Both runtimes read the same `SKILL.md`; Codex additionally reads
each skill's `agents/openai.yaml` for its display name.

### Option C — symlink (track the repo)

`git pull` then updates the installed skills in place:

```bash
git clone https://github.com/tipb47/my-claude-skills.git ~/my-claude-skills
for s in sprint whereami handoff; do
  ln -s ~/my-claude-skills/skills/$s ~/.claude/skills/$s
done
```

---

## Sprint Director — `/sprint`

A sprint-directed, multi-agent development methodology. It turns a messy "spawn a bunch
of agents and hope" workflow into a disciplined assembly line.

Work is divided into **sprints** — bounded milestones. Each sprint is run end-to-end by
ONE fresh agent session, the **director**. The director never writes feature code. It
analyzes, gates, spawns **track** subagents (each isolated on its own git branch /
worktree), audits their diffs against a design contract, verifies their claims against
real state, merges in a defined order, and closes by updating a living state ledger and
drafting the next sprint from what actually happened.

Humans appear only at **operator gates** — explicit pauses for deployments, account
signups, and approvals — never as mid-flight interruptions of a running agent.

> One engine, many instances. The skill is the reusable engine. Each project you run it
> on gets its own self-contained `ops/` folder (design contract, guidelines, roadmap,
> backlog, sprint files, state ledger). **The project's local `ops/` docs are always
> authoritative** — where they disagree with the engine, the local docs win.

### Why use it

- **Context hygiene.** A fresh director per sprint means no context rot across a long
  build. The ledger (`STATE.md`), not the chat history, is the source of truth.
- **Parallelism without collisions.** Independent tracks run as separate subagents in
  separate git worktrees, each on its own branch. The director merges `--no-ff` in a
  planned order, and close-out sweeps the worktrees and lagged branches away.
- **Verify, never trust.** "Done" / "pushed" / "loaded" are treated as *claims*. The
  director re-runs every track's verification gates against real state before merging.
- **Deliberate model spend.** Every track row in a sprint doc states a model tier plus a
  rationale — fable for contract-defining work, sonnet for standard implementation,
  the director on opus — so top-tier tokens go where the judgment is.
- **No hollow docs.** `/sprint init` interviews you and refuses to generate
  placeholder-laden scaffolding — every sprint ships decided, not "TODO".
- **Hard-won scars baked in.** Git footguns, zombie dev servers, self-resuming agents,
  gates that pass while the product is broken — real failure modes, each encoded as a
  standing rule in [`SCARS.md`](skills/sprint/SCARS.md).

### Subcommands

- **`/sprint init`** — scaffold the system into a project. Interviews you (surfaces,
  team shape, verification contract, model policy, roadmap seed), then generates a
  complete `ops/` instance with a fully-detailed `sprint-01`. Projects with an existing
  ops folder get **adopt mode**: gaps filled, nothing overwritten. Registers the project
  in `~/.claude/sprint/projects.json`.
- **`/sprint direct`** — become the director for the current sprint, in a **fresh
  session**. Boot the ledger → analyze the sprint against reality → clarify → plan →
  spawn tracks → audit → merge → close.
- **`/sprint status`** — where things stand. Inside a registered project: last closed
  sprint, current sprint and phase, what's next, top backlog items, health flags, and a
  recommended course of action. Outside a project: a compact cross-project dashboard.
  Read-only.
- **`/sprint clean`** — reconcile the instance with reality. Scans for ledger drift,
  backlog drift, unarchived sprints, leftover worktrees and stale track branches, and
  dead registry entries — then interviews you through each finding (recommended
  disposition first) and executes only what you approve. Closed sprints move to
  `sprints/archive/`.

### The `ops/` instance layout

```
<ops>/
├── README.md               ← layout + kickoff prompt for fresh director sessions
├── STATE.md                ← living ledger: current sprint, decisions, history, running jobs. READ FIRST.
├── DESIGN.md               ← canonical contract: architecture, schemas, conventions, standing decisions
├── SPRINT_GUIDELINES.md    ← rules for tracks + directors: gates, branch discipline, definition of done
├── DIRECTOR_GUIDELINES.md  ← director runbook: boot → analyze → gate → tracks → audit → merge → close
├── ROADMAP.md              ← sprint arc at milestone level (reality wins over this outline)
├── BACKLOG.md              ← prioritized queue of future work; feeds each next-sprint draft
└── sprints/
    ├── archive/            ← closed sprint folders, moved here by /sprint clean
    └── sprint-NN/
        ├── SPRINT.md       ← sprint brief: goal, gates, tracks, merge order, close checklist
        └── TRACK-*.md      ← self-contained prompt per track subagent
```

The engine's templates live in [`skills/sprint/templates/`](skills/sprint/templates);
the failure-mode catalog is [`skills/sprint/SCARS.md`](skills/sprint/SCARS.md).

### How the pieces fit

| Role | Who | Responsibility |
|---|---|---|
| **Engine** | The skill (`/sprint`) | Boot rituals, scaffolding templates, cross-project registry |
| **Director** | One fresh agent session per sprint | Analyze, gate, spawn, audit, merge, close — never builds |
| **Track** | A subagent per unit of work | Builds on its own branch; reports verifiable output; never merges |
| **Operator** | You, the human | Appears only at gates: approvals, deploys, signups |

Model tiers are a drafting decision, stated per track in `SPRINT.md`: director on
**opus**; contract-defining / architecture / gnarly-debug tracks on **fable**; standard
implementation on **sonnet**; **haiku** only for mechanical fully-specified chores with
a written justification. Any role — the director included — escalates to fable when the
work genuinely warrants it.

State lives in files, not chat:

- `~/.claude/sprint/projects.json` — the cross-project registry (per-user, never committed).
- `<project>/ops/STATE.md` — the living ledger the director reads first and updates last.

### Concepts in one screen

- **Sprint** — a bounded milestone with a goal, a definition of done, gates, tracks, and a merge order.
- **Track** — a self-contained unit of work. Its `TRACK-*.md` is a complete prompt: a
  fresh subagent with no other context can execute it end-to-end. One branch per track
  (`sNN/<slug>`); parallel tracks run in git worktrees.
- **Operator gate** — an explicit pause for the human. Gate 0 runs before tracks spawn;
  the close gate runs after merges. Gates live in `SPRINT.md`, never inside track prompts.
- **Verification contract** — what makes work *provably* done (test commands,
  data-quality queries with expected output, build artifacts). Tracks quote actual
  output; the director re-runs it at audit.
- **Scar** — a hard-won failure mode encoded as a standing rule, so the same footgun is
  never fired twice. The engine keeps them in [`SCARS.md`](skills/sprint/SCARS.md);
  `init` folds the reachable ones into each project's guidelines.

---

## Where Am I — `/whereami`

An in-chat situation report for the current session: what is **done** (and how it was
verified), what is **in progress**, what comes **next**, and what is **blocked**. Ends
with the single most useful next action. Read-only; writes nothing.

Use it mid-session to reorient — especially when juggling several agents at once.

## Handoff — `/handoff`

Produces ONE copyable message — in chat, never a file — that transfers the session to a
fresh agent: mission, context, verified-vs-claimed work, in-flight state, next goals,
decisions with their whys, and gotchas. It closes with standing instructions that make
the next agent verify every claim against real state and interview the operator
relentlessly (recommended answers first) before building anything.

---

## FAQ

**Does Sprint Director lock me into a folder structure?** No. The local `ops/` docs are
authoritative; the engine adapts. `init` has an adopt mode for projects that already
have an ops folder.

**Do I need GitHub / a remote?** No. It's `git`-centric (branches, worktrees, `--no-ff`
merges) but works fine on a purely local repo.

**Can other AI agents set this up?** Yes — see [`AGENTS.md`](AGENTS.md) for a
deterministic, machine-followable install and bootstrap procedure.

**Where does my data go?** Nowhere external. Everything is local files: the per-project
`ops/` folder and the per-user registry at `~/.claude/sprint/projects.json`.

---

## Contributing

Issues and PRs welcome. The most valuable contributions are **new scars** — concrete,
reproducible failure modes with the rule that prevents them — and template improvements
that generalize across projects. Keep additions project-agnostic: no traces of any
specific company's infrastructure.

A scar belongs in [`SCARS.md`](skills/sprint/SCARS.md) if it states a **class**, not an
incident: one sentence naming the class, then the mechanism, then the rule. If it only
reproduces in one company's stack, it belongs in that project's own `DESIGN.md`.
