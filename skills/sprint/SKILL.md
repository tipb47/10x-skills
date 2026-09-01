---
name: sprint
description: Sprint-directed multi-agent development methodology. Use for `/sprint init` (scaffold the ops/ sprint system into a project, interview-driven), `/sprint direct` (boot as sprint director for the current project's active sprint), `/sprint status` (per-project sprint/backlog status inside a project, cross-project dashboard otherwise), `/sprint clean` (reconcile ledger, backlog, sprint folders, worktrees, and branches — interview-driven). Invoke when the user mentions sprint directing, sprint methodology, starting/running a sprint, sprint status, or sprint cleanup.
argument-hint: init | direct | status | clean
---

# Sprint System — methodology engine

This skill is the **engine** for a sprint-directed development methodology. Each project
keeps its own **instance**: an `ops/` folder holding its design contract, guidelines,
roadmap, sprint files, and a living state ledger. **The project's local ops/ docs are
always authoritative — wherever they conflict with this skill, the local docs win.** This
skill provides the boot rituals, the scaffolding templates, and the cross-project registry.

## The methodology in one paragraph

Work is divided into **sprints**: bounded milestones, each run end-to-end by ONE fresh
agent session (the **director**) for context hygiene. The director never builds; it
analyzes, gates, spawns **track** subagents (each with a self-contained `TRACK-*.md`
prompt, each on its own git branch, parallel tracks in git worktrees), then audits their
diffs, verifies their claims against real state, resolves conflicts, merges in a defined
order, and closes by updating the ledger (`STATE.md`) and drafting the next sprint from
what actually happened. Humans appear at **operator gates** — explicit pauses for
deployments, account signups, approvals — never as mid-flight track interruptions.

## Subcommand routing

Parse the argument. No argument or unrecognized → print the usage summary below and stop.

```
/sprint init    — scaffold or adopt the sprint system in the current project
/sprint direct  — become the sprint director for this project's current sprint
/sprint status  — where things stand: this project in depth, or all projects
/sprint clean   — reconcile ledger, backlog, sprint folders; sweep git debris
```

## Model selection

Choosing a model per role is part of sprint DRAFTING, not a mid-sprint whim: every
SPRINT.md track row carries a model tier plus a one-line rationale, and the director
honors it at spawn time. The point is token economy in both directions — a top tier
burned on mechanical work is waste, and an underpowered model on contract-defining work
costs more in rework than it saves.

Default tiering. The init interview offers this as the recommendation; once set, the
project's model policy in `SPRINT_GUIDELINES.md` is authoritative:

| Role / work class | Tier |
|---|---|
| Director (analyze, audit, merge, close) | opus |
| Contract-defining, architecture, cross-cutting, or gnarly-debug tracks | fable |
| Standard implementation tracks | sonnet |
| Genuinely mechanical chores (single-file, low-risk, fully specified) | haiku — only with a stated justification in the sprint doc |

Escalation: ANY role — the director included — may run on fable when the work genuinely
warrants the deepest reasoning; record that as a drafting decision in the sprint doc.
Haiku is off the table for anything that requires judgment.

The DIRECTOR's tier is declared per sprint: SPRINT.md carries a `Director tier:` line
(model + one-line why), set by the closing director when drafting the sprint. Default
opus; escalate the director to fable when ANY of these hold:

- 3+ parallel tracks touch shared surfaces — merge/seam risk concentrates in the director;
- architecture or contract decisions are expected to be MADE mid-sprint rather than
  pre-drafted;
- the prior sprint's audit caught seam defects its gates missed.

The operator launches the session, so the tier only binds if the kickoff names it — the
close report and the ops README kickoff prompt both say which model to launch.

---

## /sprint init

Scaffold the sprint system into the current project, interview-driven. Never overwrite
existing files.

1. **Detect existing instance.** Look for an ops folder (`ops/`, `.docs/ops/`, or ask).
   If one exists → **adopt mode**: map its files against the canonical layout (below),
   report gaps (no STATE.md? no guidelines? missing sections?), offer to fill ONLY the
   gaps using the templates, then register (step 5) and stop. Never modify existing docs
   beyond explicitly approved gap-fills.
2. **Interview** the user (use your runtime's structured question tool if it has one — adapt
   wording to what you can already see in the repo; skip questions the codebase answers).
   Interview discipline, here and in every later question round:
   - **Every question carries your recommended answer** as the first option, labeled
     `(Recommended)` — never present a bare option list you have no opinion on.
   - **Batch only independent questions.** If one question's answer determines whether or
     how another applies, sequence them — ask the upstream question first, then shape the
     downstream batch from its answer. Never make the user answer a question that a
     sibling answer in the same batch could invalidate.
   - Look up *facts* yourself (filesystem, git, configs); put only *decisions* to the user.

   The fixed topics:
   - Ops location and git tracking: `ops/` vs `.docs/ops/`; tracked or untracked? If
     tracked, ops bookkeeping (STATE.md/SPRINT.md churn) commits SEPARATELY from code under
     an `[sNN/ops]` tag; if untracked, the ops dir is gitignored. Either way the code
     history stays clean — encode the resulting rule into the generated guidelines.
   - Project surfaces: repos/packages/deploy targets a sprint touches; how deploys happen
     (manual command? CI?) — deploys are usually operator gates.
   - Team shape: solo, or other humans pushing out-of-band? (Out-of-band pushers need
     pull-and-check-recent-commits rules in the guidelines.)
   - Verification contract: what makes a track's work *provably* done — test suites,
     data-quality gates (row counts / verification queries with expected output), build
     artifacts, or a mix? This shapes the guidelines' "gates" section.
   - Database/migrations: does the project have a DB? If so: migrations are FILES tracks
     write and ONLY the director applies — confirm the apply mechanism (CLI, MCP, etc.).
   - Model policy: present the engine's default tiering (§ Model selection) as the
     recommendation and confirm or adapt it — director tier, per-work-class track tiers,
     what escalates to fable, whether haiku is ever acceptable. The answer lands in
     SPRINT_GUIDELINES.md and in every SPRINT.md track row (tier + one-line rationale).
   - Roadmap seed: the next 3–6 milestone-level goals, and which ONE becomes sprint-01.

   After the fixed questions, do a **grill pass** — proactive, not reactive: attack the
   sprint-01 plan's weakest assumptions even when nothing looks ambiguous. Probe what the
   user did NOT put on the table: failure modes ("what happens when the deploy/migration/
   external API fails mid-sprint?"), boundary cases, and contradictions between answers
   ("you chose X earlier, but this goal implies Z — which wins?"). 2–3 adversarial
   questions (each with a recommended answer) aimed at whatever would
   most likely sink or reshape sprint-01.

   Then a **clarify pass**: if the roadmap seed or sprint-01 scope is still ambiguous,
   contradictory, or too thin for a self-contained track, ask focused follow-ups until
   sprint-01 can be written with zero placeholders. Never generate hollow docs over
   unresolved ambiguity.
3. **Generate** from `templates/` (this skill's folder), substituting interview answers
   for `{{PLACEHOLDERS}}` and resolving every `<!-- init: ... -->` directive (these
   comments instruct YOU what to fill; they never survive into generated files):
   `README.md`, `STATE.md`, `DESIGN.md`, `SPRINT_GUIDELINES.md`, `DIRECTOR_GUIDELINES.md`,
   `ROADMAP.md`, `BACKLOG.md`, and `sprints/sprint-01/SPRINT.md` (+ one `TRACK-*.md` per
   sprint-01 track).
   Sprint-01 track files must be genuinely self-contained: a fresh subagent with no other
   context executes one end-to-end.
   Read `SCARS.md` (this skill's folder) while generating: fold in every scar the project's
   shape makes reachable — it has a DB, a deployed surface, parallel tracks, out-of-band
   pushers — and leave out the rest. The generated guidelines carry the rules; `SCARS.md`
   stays the engine's copy.
4. **Quality bar:** generated docs must contain zero TODOs and zero template placeholders.
   If an interview answer is "not sure yet", write the current best decision into the doc
   and log it as provisional in STATE.md's decisions table — docs ship decided, not hollow.
5. **Register** the project in the cross-project registry at `~/.claude/sprint/projects.json`
   (`{name, path, ops}`). Create the file (and its directory) if missing.
6. Print the kickoff instructions: the user starts a FRESH session and runs `/sprint direct`
   (or pastes the kickoff prompt from the generated README).

### Canonical instance layout

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

---

## /sprint direct

Become the sprint director for the current project's active sprint.

1. **Locate the instance:** current project in `~/.claude/sprint/projects.json`, else detect
   an ops folder, else tell the user to run `/sprint init`.
2. **Boot in this order:** `STATE.md` (current-sprint pointer, decisions, running jobs) →
   `DIRECTOR_GUIDELINES.md` → `SPRINT_GUIDELINES.md` + `DESIGN.md` → the current
   `sprints/sprint-NN/SPRINT.md` and its `TRACK-*.md` files → `SCARS.md` (this skill's
   folder) for the failure modes the local docs have not yet absorbed.
   **Tier check:** compare your own model to SPRINT.md's `Director tier:` line. Running
   WEAKER than recommended → flag it as a Gate 0 operator decision: proceed anyway, or
   relaunch the session on the stated tier. Stronger → proceed silently. No line → treat
   as opus and add the line when you next amend SPRINT.md.
3. **Analyze requirements vs reality** before anything else (the project's director
   guidelines define specifics; universal minimum): `git pull --ff-only` and compare repo
   state against the sprint doc's assumptions; verify claimed prior state against actual
   stores (DBs, buckets, artifacts); check any background-job manifests/status files
   STATE.md references — progressed, stalled, failed?; confirm credentials/tooling the
   sprint needs are alive. **If reality contradicts the sprint doc: amend SPRINT.md and
   log the deviation in STATE.md BEFORE spawning tracks.** Docs never drift from what runs.
4. **Clarify, reconcile, then plan:** raise ALL blocking/open/clarifying questions in one
   round (genuine operator decisions only — look up facts yourself). Follow the
   init interview discipline: every question leads with your `(Recommended)` answer, and
   dependent questions are sequenced, not batched with the questions they depend on. If answers
   shift scope, reconcile — amend SPRINT.md and log in STATE.md — then re-surface any new
   ambiguity, looping until the sprint is unambiguous. A track must never inherit a question
   you could have closed here. Present Gate 0 items. Then present your execution plan (plan
   mode where available): spawn order, parallelism, merge order, what you'll verify at
   audit. On approval, execute.
5. **Run the sprint** per the local DIRECTOR_GUIDELINES. Universal rules that always apply:
   - Directors direct: write code only for merge conflicts and trivial audit fixes.
   - **Assume the operator is watching** and available at gates. An unattended sprint
     run happens only under an explicitly negotiated grant (the `/autonomous` skill,
     where installed) — under it, operator gates queue as a morning checklist instead
     of blocking, and everything else about this section still applies.
   - One subagent per track; parallel same-repo tracks get `isolation: "worktree"`;
     independent tracks spawn in one message.
   - Spawn each track on the model tier its SPRINT.md row states (§ Model selection).
     A row without a tier is a drafting gap: assign one per the model policy and amend
     SPRINT.md before spawning.
   - **Verify, never trust:** "pushed"/"loaded"/"done" are claims — check git/remote/DB/
     artifact state yourself before acting on them. Re-run tracks' verification gates
     yourself at audit.
   - Audit the FULL diff per track against DESIGN.md before merging; merge `--no-ff` in
     SPRINT.md's stated order; suites green on main after each merge.
   - Track branches are durable; if resuming a dead session, re-audit anything unmerged
     rather than trusting prior-session memory.
6. **Close:** operator close-gate as an actionable checklist; update STATE.md (shipped,
   deviations, learnings, background jobs) and BACKLOG.md (strike shipped items, add
   deferred and discovered work). **Clean sprint git state:** once every merge is
   verified landed, remove this sprint's track worktrees (`git -C <worktree> status`
   must be clean first — uncommitted work stops that removal) and `git worktree prune`;
   delete the PREVIOUS sprint's merged track branches — local AND their `origin`
   counterparts (`git push origin --delete <branch>`, after verifying merged into
   `origin/main`; a branch with an open PR is not a deletion candidate). This sprint's
   branches survive one more sprint as recovery points, locally and on the remote;
   never delete an unmerged branch, anywhere. Draft/amend the
   next sprint's files from what actually happened (BACKLOG.md is the feed), including
   the next SPRINT.md's `Director tier:` line per § Model selection; final report with
   the next kickoff pointer, naming the model to launch the next director on.
   **Promotion path:** if a learning is project-agnostic (a git scar, an audit technique),
   propose promoting it into this skill's templates — keep the engine improving.
   If the sprint paid a repeated manual tax (operator copy-paste, hand-run
   verification, the same service poked by hand every track), flag it in the close
   report as an optimization candidate and offer to run `/optimize` on it (where
   installed).

### Universal scars

`SCARS.md` (this skill's folder) is the engine's catalog of failure modes that have already
cost real sprints — git and branch footguns, agent/worktree hazards, the epistemics of a
verification gate, deployment and environment parity, data and schema traps, browser
surfaces, comment hygiene. Read it at boot and apply it at audit. The four that bite most
often, inline so they are never missed:

- **Verify, never trust.** "Pushed" / "loaded" / "done" / "green" are claims. Check git,
  the store, the process, the deployed path yourself before acting on any of them.
- **With worktrees, ALWAYS `git -C <absolute-path>`, stage EXPLICIT paths, and never chain a
  push after a PIPED build** — a stale cwd has reset a branch mid-merge, a bulk add has
  pushed local config, and a pipe masks the exit code you are gating on.
- **Never run mutating git operations in a tree while an agent is live in it**, and judge
  agent liveness by git state, not by a notification — kill/complete notifications are
  unreliable and an agent can resume after one.
- **A track's own gates cannot see the seam between tracks.** Drive the actual deliverable
  end-to-end — on the deployed origin for anything customer-facing — before closing.

**Promotion path.** When a sprint pays for a new failure mode, write the class (not the
incident) into `SCARS.md`: one sentence naming the class, the mechanism, then the rule. If
it only reproduces in one company's stack, it belongs in that project's `DESIGN.md`, not here.

---

## /sprint status

Where things stand. Read-only — status never mutates files, git state, or the registry.

**Context detection:** if the cwd is inside a registered project (or an ops folder is
detectable), give the per-project view. Otherwise give the cross-project dashboard.

### Per-project view

1. Read `STATE.md`, `BACKLOG.md`, `ROADMAP.md`, and the `sprints/` folder (including
   `archive/`). Cheap git checks: current branch, `git worktree list`, leftover
   `sNN/*` branches.
2. Report, compactly:
   - **Just completed:** the last closed sprint and its one-line outcome (STATE.md
     sprint history).
   - **Current sprint:** pointer, phase (drafted / in flight / closing), and — if in
     flight — track and merge progress as far as STATE.md and git branches show.
   - **Next up:** the drafted next sprint if one exists, else the top backlog items.
   - **Backlog:** top 3-5 prioritized items; total count.
   - **Health flags:** stalled background jobs, stale worktrees or `sNN/*` branches past
     the retention lag, STATE.md pointer vs `sprints/` mismatch, closed sprints not yet
     archived — each with its fix (usually `/sprint clean` or `/sprint direct`).
3. End with **Recommended course of action** — one or two sentences. Examples: "sprint-04
   is drafted and reality still matches it: start a fresh session and run /sprint direct";
   "close-out never finished: run /sprint clean before directing anything new".

### Cross-project dashboard

1. Read `~/.claude/sprint/projects.json`. Empty/missing → say so, point at `/sprint init`.
2. Per project: read `<ops>/STATE.md` — extract current-sprint pointer, status line,
   last-updated date, background-jobs table (if present, also check any referenced
   manifest/status files for staleness: `updated_at` older than expected cadence = flag),
   and open items. Unreadable path → report it as broken registration, don't crash the dashboard.
3. Report a compact table: project | current sprint | status | background jobs (healthy/
   stalled/none) | last updated | top open item. Flag anything that needs human attention.

## /sprint clean

Reconcile the ops instance with reality and sweep sprint debris. Interview-driven: clean
NEVER removes, moves, or rewrites anything the operator has not explicitly approved.

1. **Locate the instance** (same as `/sprint direct`). No instance → point at
   `/sprint init` and stop.
2. **Scan, don't touch.** Build a findings list:
   - **Ledger drift:** STATE.md current-sprint pointer vs actual `sprints/` folders;
     sprint-history rows vs sprints that actually closed; background-job entries whose
     manifests are stale or whose processes are dead; verified-facts entries that no
     longer hold.
   - **Backlog drift:** BACKLOG.md items already shipped, duplicated in STATE.md's open
     items, or contradicted by ROADMAP.md.
   - **Sprint folders:** closed sprints still outside `sprints/archive/`; abandoned
     half-drafted sprint folders.
   - **Git debris:** leftover worktrees (`git worktree list`), merged `sNN/*` branches
     past the one-sprint retention lag, unmerged `sNN/*` branches (FLAG these; they are
     never deletion candidates).
   - **Remote sprint debris:** `git ls-remote --heads origin` (read-only), filtered to
     the sprint branch convention — `sNN/*` plus any project-specific sprint prefix the
     local guidelines declare. Branches outside the convention are out of scope: never
     listed, never touched, never mentioned. No `origin`, or no matches → skip this
     pass and say so in the report. Classify each match against `origin/main` (fetch
     first): merged past the one-sprint retention lag (deletion candidate), merged
     within retention (keep — the same recovery-point rule as local), unmerged (FLAG;
     never a deletion candidate), or remote-only with no local counterpart (FLAG as an
     anomaly — establish its story in the interview before proposing anything). Where
     `gh` works, check deletion candidates for open PRs: an open PR removes a branch
     from candidacy (deleting the branch closes the PR); where `gh` is unavailable,
     state in that branch's question that PR status is unverified.
   - **Registry rot:** `projects.json` entries whose paths no longer exist.
3. **Interview** the operator through the findings with init's interview discipline:
   every question leads with a `(Recommended)` disposition (archive / delete / amend /
   keep), independent findings batch together, dependent ones sequence. Probe
   contradictions instead of papering over them: if the ledger says a sprint closed but
   its branches never merged, establish which story is true BEFORE proposing a
   disposition.
4. **Execute only what was approved:**
   - Move closed sprint folders → `sprints/archive/sprint-NN/`.
   - Amend STATE.md / BACKLOG.md; log the clean in STATE.md's decisions log.
   - Remove approved worktrees — `git -C <worktree> status` first; ANY uncommitted work
     stops that removal and goes back to the operator. Then `git worktree prune`.
   - Delete approved branches only after verifying merged (`git branch --merged main`).
     For an unmerged branch offer only "keep" or "operator deletes it manually".
   - Delete approved remote branches with `git push origin --delete <branch>` only
     after re-verifying merged into `origin/main`; re-run `git ls-remote --heads origin`
     afterward to confirm each deletion landed. Unmerged or open-PR remote branches get
     only "keep" or "operator deletes it manually" — same as local.
   - Fix or drop dead registry entries.
5. **Report:** what moved, what was deleted, what was amended, what was flagged and left
   alone.

## Registry format (`~/.claude/sprint/projects.json`)

```json
{ "projects": [ { "name": "<short-name>", "path": "/abs/path/to/repo", "ops": "/abs/path/to/ops" } ] }
```
