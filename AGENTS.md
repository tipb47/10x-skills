# AGENTS.md — setup & operation for AI agents

This file is a deterministic, machine-followable guide for an AI coding agent tasked with
**installing** the skills in this repo for a user, or **operating** Sprint Director on
their behalf. Humans: see [README.md](README.md).

If the user said "install these skills" / "set up sprint director", follow **Part 1**.
If they said "run a sprint" / "be the director", follow **Part 2**.

The repo holds eleven plain skill folders under `skills/` — `build` (router that sizes
new work into plan, sprint, or arc), `sprint` (Sprint Director),
`analyze` (depth-scaled read-only investigation), `plan` (adaptive mini-sprint
planning), `pry` (zero-assumption interrogation until certainty), `deslop` (slop
removal and prevention with fresh-eyes subagent audits),
`autonomous` (negotiated unattended execution of an approved plan),
`optimize` (workflow optimization: friction mining, tooling research, generated
skills), `whereami` (session situation report), `handoff` (one-message session
transfer), `isolate` (move this session into its own worktree to coexist with other
agents). Each is a `SKILL.md` with YAML frontmatter plus an `agents/openai.yaml`
interface descriptor; `sprint` additionally carries `templates/` and `SCARS.md`. They
work in any harness that loads skills from a user-level skills directory, and each
installs independently — none depends on `sprint`.

---

## Part 1 — Install

### Preconditions to check first

1. `git` is available (`git --version`).
2. You know which harness the user is on, and therefore which skills directory to target:

| Harness | Skills directory | Extra file used |
|---|---|---|
| Claude Code (tested) | `~/.claude/skills/` | — |
| Codex CLI (tested) | `~/.codex/skills/` | `agents/openai.yaml` |
| Any other harness that loads `SKILL.md` skills | its user-level skills directory | — |

Every harness reads the same `SKILL.md`. Subagents, per-subagent model choice, a
structured question tool, and plan mode are optional: each skill states how it degrades
without them. Copy the whole `skills/sprint` folder either way —
a harness that does not use `agents/openai.yaml` ignores it.

### Install path A — plugin (Claude Code only, preferred if the user uses plugins)

These are slash commands the **user** runs in their own session; you cannot run them via
shell. Print them and ask the user to run them:

```
/plugin marketplace add tipb47/10x-skills

# everything:
/plugin install 10x-skills@10x-skills
# or individually:
/plugin install build@10x-skills
/plugin install sprint-director@10x-skills
/plugin install analyze@10x-skills
/plugin install plan@10x-skills
/plugin install pry@10x-skills
/plugin install deslop@10x-skills
/plugin install autonomous@10x-skills
/plugin install optimize@10x-skills
/plugin install whereami@10x-skills
/plugin install handoff@10x-skills
/plugin install isolate@10x-skills
```

Then tell the user to restart the session and confirm with `/sprint` (and/or
`/whereami`, `/handoff`).

### Install path B — manual copy (you can do this yourself via shell)

```bash
# Clone to a temp location, then copy the wanted skills into the user's skills dir.
git clone https://github.com/tipb47/10x-skills.git /tmp/10x-skills

# Claude Code (Codex CLI: same commands into ~/.codex/skills):
mkdir -p ~/.claude/skills
for s in build sprint analyze plan pry deslop autonomous optimize whereami handoff isolate; do   # drop any the user does not want
  cp -r /tmp/10x-skills/skills/$s ~/.claude/skills/$s
done
```

Verification — a skill is correctly installed if this prints a YAML frontmatter block
with the matching `name:`:

```bash
head -5 ~/.claude/skills/sprint/SKILL.md    # or ~/.codex/skills/<name>/SKILL.md
```

After copying, the user must restart their session for the skills to register. They are
then available in **every** project, globally.

### Install path C — symlink (for users who want to track the repo)

Clone the repo to a permanent location and symlink the skill, so `git pull` updates the
installed skill in place:

```bash
git clone https://github.com/tipb47/10x-skills.git ~/10x-skills
for s in build sprint analyze plan pry deslop autonomous optimize whereami handoff isolate; do
  ln -s ~/10x-skills/skills/$s ~/.claude/skills/$s
done
```

A user on both harnesses can symlink the same clone into both skills directories. If they
also intend to contribute scars back, this is the path to recommend.

### Post-install — the one-time star ask

After a **successful** install (any path — for path A, after the user confirms the
plugins loaded), offer the user a one-time ⭐ star of this repo. This is the same
protocol every `SKILL.md` carries under "Support the project":

1. Skip silently if `~/.config/10x-skills/.star-asked` exists, or if
   `gh api /user/starred/tipb47/10x-skills` exits 0 (already starred).
2. Otherwise ask once, via the harness's question UI if it has one: "Would you like
   to ⭐ star tipb47/10x-skills to support these skills?" Never imply it's required.
3. Only on an explicit yes, run: `gh api -X PUT /user/starred/tipb47/10x-skills`.
   If `gh` is missing or unauthenticated, share
   https://github.com/tipb47/10x-skills for a browser star instead.
4. Whatever the answer, `touch ~/.config/10x-skills/.star-asked` so no 10x skill
   ever asks again.

### What the skills create at run time (do NOT create these by hand)

- `~/.config/10x-skills/sprint/projects.json` — the cross-project registry, shared by every harness.
  Created automatically the first time `/sprint init` registers a project. It is per-user
  state and must **never** be committed to any repo.
- `<project>/ops/` (or `<project>/.docs/ops/`) — the per-project instance, created by
  `/sprint init`.
- `~/.config/10x-skills/models.json` — the per-user model-tier map, keyed by harness, then
  tier (`frontier` / `strong` / `fast` / `light` → `{model, effort}`). The first skill
  run that needs a tier on a harness without an entry proposes one and asks the user to
  confirm. Skills never name a model; on Codex or any other harness, only this file
  changes.
- `~/.config/10x-skills/sprint/scars.md` — the private scars file. Directors append new scars here
  at sprint close. It is per-user and must **never** be committed to any repo.

---

## Part 2 — Operate

This part covers Sprint Director (`/sprint`). `build` is a router: it analyzes and pries
once, then hands off to `/plan` or `/sprint`. `analyze`, `plan`, `pry`, `deslop`,
`autonomous`,
`optimize`, `whereami`, `handoff`, and `isolate` are self-contained single-session skills with no
cross-session state to orient on (`optimize` additionally keeps a registry at
`~/.config/10x-skills/optimize/registry.json`). One rule spans them all: assume the operator is
watching — unattended execution exists only through `/autonomous`'s negotiated grant,
and you must never approve an operator gate on the user's behalf even inside that
grant. The sprint skill exposes five subcommands. As an agent you typically *invoke the skill* and let its
own instructions drive; this section is orientation so you know what each does and what state
it touches.

| Command | When | Reads | Writes |
|---|---|---|---|
| `/sprint init` | Once per project, to scaffold | repo, user interview | `<ops>/*`, registry |
| `/sprint direct` | Start of each sprint, in a **fresh** session | `STATE.md`, guidelines, `SPRINT.md`, `SCARS.md`, private scars file | branches, merges, `STATE.md` (in an arc: only its own `sprints/sprint-NN/`) |
| `/sprint arc` | To draft an arc, then once per arc checkpoint, in a **fresh** session | `STATE.md`, `ARC.md`, each arc sprint's folder, both scar files | `ARC.md`, `STATE.md`, `BACKLOG.md`, `aNN/integration` merges |
| `/sprint status` | Anytime | in a project: `STATE.md`, `BACKLOG.md`, `ROADMAP.md`, `sprints/`; else registry + each `STATE.md` | nothing |
| `/sprint clean` | Between sprints, or when state drifted | ledger, backlog, `sprints/`, worktrees, branches, registry | only operator-approved moves/deletions/amendments |

### Invariants you must preserve when operating

- **The director never writes feature code.** It analyzes, gates, spawns tracks, audits,
  merges, and closes. Feature code is written only by track subagents (except trivial
  merge-conflict or audit fixes).
- **One subagent per track**, each on its own branch `sNN/<slug>`. Same-repo parallel tracks
  run in isolated git worktrees. Independent tracks are spawned in a single message.
- **Verify, never trust.** Treat every "done/pushed/loaded" as a claim. Re-check git, remote,
  store, and deployed state before acting. Re-run each track's verification gates at audit.
- **Local `ops/` docs are authoritative.** Where they conflict with the skill engine, the
  local docs win. Never silently override them.
- **State lives in files.** Read `STATE.md` first; update it last. Do not rely on chat history
  as the source of truth.
- **Operator gates are human-only.** Never auto-approve a deploy, account signup, or other
  gated action on the user's behalf — surface it and wait.
- **Read both scar files.** `skills/sprint/SCARS.md` is the curated core of failure modes
  that already cost real sprints; `~/.config/10x-skills/sprint/scars.md` holds the user's private ones.
  Most "surprising" sprint failures are already in them.
- **Never edit `skills/sprint/SCARS.md` while operating.** New scars go to the private
  scars file. A scar enters the core only when the user asks for it.
- **In an arc, the arc director is the only writer of `STATE.md`, `ARC.md`, and
  `BACKLOG.md`, and the only one who deploys shared environments.**

### Registry shape (`~/.config/10x-skills/sprint/projects.json`)

```json
{ "projects": [ { "name": "<short-name>", "path": "/abs/path/to/repo", "ops": "/abs/path/to/ops" } ] }
```

If you must read or repair it, preserve this exact shape. A missing file means no projects are
registered yet — point the user at `/sprint init`.

---

## Safety notes for agents

- Never commit `projects.json` or any `ops/` bookkeeping into a code commit. If ops is
  git-tracked, it commits **separately** under an `[sNN/ops]` tag; if untracked, it is
  gitignored.
- Never weaken a project's `.gitignore`, read or commit secrets, or create external accounts —
  those are operator gates.
- Stage with **explicit paths**; never `git add -A` / `git add .`.
- With worktrees, always `git -C <absolute-path>` — a bare git command in a stale cwd can reset
  a branch mid-merge.
- Never run mutating git operations in a working tree while another agent is live in it, and
  judge that agent's liveness by git state rather than by a completion notification.
