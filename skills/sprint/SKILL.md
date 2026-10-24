---
name: sprint
description: Sprint-directed multi-agent development methodology. Use for `/sprint init` (scaffold the ops/ sprint system into a project, interview-driven), `/sprint direct` (boot as sprint director for the current project's active sprint), `/sprint arc` (draft an arc — a set of sprints, some parallel, toward one large outcome — and run its checkpoints as arc director), `/sprint status` (per-project sprint/backlog status inside a project, cross-project dashboard otherwise), `/sprint clean` (reconcile the STATE index, backlog, sprint folders, worktrees, and branches — interview-driven). Invoke when the user mentions sprint directing, sprint methodology, starting/running a sprint, parallel sprints or an arc, sprint status, or sprint cleanup.
argument-hint: init | direct | arc | status | clean
---

# Sprint System — methodology engine

This skill is the **engine** for a sprint-directed development methodology. Each project
keeps its own **instance**: an `ops/` folder holding its design contract, guidelines,
roadmap, sprint folders, and a thin **STATE index** (`STATE.md`). **The project's local
ops/ docs are always authoritative — wherever they conflict with this skill, the local
docs win.** This skill provides the boot rituals, the scaffolding templates, and the cross-project registry.

## The methodology in one paragraph

Work is divided into **sprints**: bounded milestones, each run end-to-end by ONE fresh
agent session (the **director**) for context hygiene. The director never builds; it
analyzes, gates, spawns **track** subagents (each with a self-contained `TRACK-*.md`
prompt, each on its own git branch, parallel tracks in git worktrees), then audits their
diffs, verifies their claims against real state, resolves conflicts, merges them in a
defined order into the sprint's **integration branch** (`sNN/integration`), promotes
that branch into the trunk with ONE merge at an operator gate, and closes by writing
the sprint's `CLOSE.md`, adding one STATE index row, and drafting the next sprint from
what actually happened. Humans appear at **operator gates** — explicit pauses for
deployments, account signups, approvals — never as mid-flight track interruptions.
Work too large for one sprint, with sprints that can run in parallel, becomes an **arc**:
sprints promote into `aNN/integration`, and an **arc director** merges them at
checkpoints and promotes the arc into the trunk once.

## Subcommand routing

Parse the argument. No argument or unrecognized → print the usage summary below and stop.

```
/sprint init    — scaffold or adopt the sprint system in the current project
/sprint direct  — become the sprint director for this project's current sprint
/sprint arc     — draft an arc of parallel sprints, or run its next arc checkpoint
/sprint status  — where things stand: this project in depth, or all projects
/sprint clean   — reconcile STATE index, backlog, sprint folders; sweep git debris
```

## Model selection

Choosing a model tier per role is part of sprint DRAFTING, not a mid-sprint whim:
every SPRINT.md track row carries a model tier plus a one-line rationale, and the
director honors it at spawn time. The point is token economy in both directions — a
top tier burned on mechanical work is waste, and an underpowered model on
contract-defining work costs more in rework than it saves.

### Model tiers

Skills name a model tier, never a model. The tier says how much reasoning the work
needs; your map says which model delivers it on this harness.

| Tier | Use for |
|---|---|
| **frontier** | Hard problems only: novel algorithms, modeling engines, breakthroughs |
| **strong** | The workhorse: judgment, contracts, architecture, gnarly debugging |
| **fast** | Standard implementation and investigation |
| **light** | Mechanical, fully specified chores; never judgment |

**Resolve a tier at spawn time:**

1. Read `~/.config/10x-skills/models.json`, keyed by harness, then tier:
   `{ "<harness>": { "<tier>": { "model": "...", "effort": "..." } } }`.
2. No entry for this harness → list the models the harness offers, propose a mapping
   (two tiers may share a model at different efforts), confirm it with the user, and
   save it.
3. A mapped model is gone, or a newer one is offered → propose a remap once. Never
   change the map silently.
4. The harness cannot pick a model per subagent → apply the tier's effort only. It
   cannot set effort either → inherit the session model, and say so in one line.

**Every spawn names its tier.** State the tier and a one-line why at each spawn.
Never let a subagent inherit the session model by default. A fork inherits, so never
fork for tiered work. Spawn a typed agent with an explicit tier instead. Directors
and drafters (sprint drafting, `/build`, `/sprint arc`) run on strong or above.
Their effort scales with the work's size and seams; the tier does not drop.

A legacy model name in an old doc reads as its tier: first by reverse lookup in the
map, else opus → strong, fable → frontier, sonnet → fast, haiku → light.

### Default policy

The init interview offers this as the recommendation. Once set, the project's model
policy in `SPRINT_GUIDELINES.md` is authoritative, and it may raise or lower any
default (a modeling-engine project may run its core tracks on frontier):

| Role / work class | Model tier |
|---|---|
| Director and drafter (analyze, audit, merge, close, draft the next sprint) | strong minimum; effort scales with size |
| Contract-defining, architecture, cross-cutting, or gnarly-debug tracks | strong |
| Standard implementation tracks | fast |
| Scouts (read-only investigation the director delegates) | fast |
| Mechanical audit checks (greps, diff scans, file counts) | fast |
| Fix rounds after audit | the track's tier; fast when the director fully specifies the fix |
| Genuinely mechanical chores (single-file, low-risk, fully specified) | light — only with a stated justification in the sprint doc |

**Frontier is for hard problems, never for size.** Any role may run on frontier when
the problem itself needs a breakthrough: a novel algorithm, a modeling or simulation
engine, hard math or performance work. Many tracks, dense seams, or heavy coordination
escalate to strong at a higher effort, not to frontier. Record every frontier choice as
a drafting decision with its one-line why. Light is off the table for judgment.

**Size each track to one focused context:** one surface, one contract. A track that
needs two surfaces or two contracts is two tracks.

The DIRECTOR's model tier is declared per sprint: SPRINT.md carries a `Director tier:`
line (model tier + one-line why), set by the closing director when drafting the sprint.
Strong is the minimum; effort scales with size. Raise the director's effort when 3+ parallel tracks touch shared
surfaces, when contract decisions are expected mid-sprint, or when the prior audit
caught seam defects its gates missed. Frontier only when the director itself must
solve a hard problem.

The operator launches the session, so the tier only binds if the kickoff names it — the
close report and the ops README kickoff prompt both name the tier AND the model it
resolves to on the operator's harness.

---

## /sprint init

Scaffold the sprint system into the current project, interview-driven. Never overwrite
existing files.

1. **Detect existing instance.** Look for an ops folder (`ops/`, `.docs/ops/`, or ask).
   If one exists → **adopt mode**: map its files against the canonical layout (below),
   report gaps (no STATE.md? no guidelines? no TRACK_RULES.md or STATUS.md? missing
   sections?), offer to fill ONLY the gaps using the templates, then register (step 5)
   and stop. Never modify existing docs beyond explicitly approved gap-fills. A STATE.md
   that carries sprint narrative is a `/sprint clean` index split.
2. **Interview** the user (use your harness's structured question tool if it has one — adapt
   wording to what you can already see in the repo; skip questions the codebase answers).
   Where the `/pry` skill is installed, run this interview — and every later question
   round, sprint drafting included — as a pry interview; it is the reference for the
   intensity and mechanics. Without it, the discipline below stands on its own.
   Locked decisions handed in by `/build`, and a repo-root `BLUEPRINT.md` when present,
   are settled: ask only what they leave open.
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
   - Branch model: which branch sprints promote into (`main`, `staging`, ...) and what a
     push to it triggers — fills the templates' trunk placeholders. Then how CI
     triggers (read the workflow files first): trunk-only or PR-only → nothing to do;
     every branch and changeable → recommend a trunk-only branch filter plus a
     concurrency group with cancel-in-progress, as a sprint-01 Gate 0 item; every branch
     and NOT changeable → tracks put `[skip ci]` on `wip:` commits. Fills the CI-policy
     placeholders. The integration branch itself is not a choice: every
     sprint lands on `sNN/integration` and the trunk moves once, at promotion.
   - Team shape: solo, or other humans pushing out-of-band? (Out-of-band pushers need
     pull-and-check-recent-commits rules in the guidelines.)
   - Verification contract: what makes a track's work *provably* done — test suites,
     data-quality gates (row counts / verification queries with expected output), build
     artifacts, or a mix? This shapes the guidelines' "gates" section. Then the two
     verification tiers (§ Verification budget): the targeted-test pattern a track runs
     for only what it changed, and the full suite the director runs on integration.
   - Database/migrations: does the project have a DB? If so: migrations are FILES tracks
     write and ONLY the director applies — confirm the apply mechanism (CLI, MCP, etc.).
   - Model policy: present the engine's default policy (§ Model selection) as the
     recommendation and confirm or adapt it — director tier, per-work-class model
     tiers, what earns frontier in this project, whether light is ever acceptable. The
     answer lands in SPRINT_GUIDELINES.md and in every SPRINT.md track row (model tier +
     one-line rationale). The tier-to-model map is per-user, never per-project.
   - Roadmap seed: the next 3–6 milestone-level goals, and which ONE becomes sprint-01.

   Then a **grill pass**: 2–3 adversarial questions, each with a recommended answer, at
   whatever would most likely sink or reshape sprint-01 — failure modes, boundary cases,
   contradictions between answers — even when nothing looks ambiguous. Then a **clarify
   pass**: follow up until sprint-01 can be written with zero placeholders. Never
   generate hollow docs over unresolved ambiguity.
3. **Generate** from `templates/` (this skill's folder), substituting interview answers
   for the double-brace placeholders and resolving every `init` comment directive (these
   comments instruct YOU what to fill; they never survive into generated files):
   `README.md`, `STATE.md`, `DESIGN.md`, `SPRINT_GUIDELINES.md`, `DIRECTOR_GUIDELINES.md`,
   `TRACK_RULES.md`, `ROADMAP.md`, `BACKLOG.md`, and `sprints/sprint-01/` (`SPRINT.md`,
   `STATUS.md` at phase `drafted`, and one `TRACK-*.md` per track).
   Sprint-01 track files must be genuinely self-contained: a fresh subagent with no other
   context executes one end-to-end.
   Read the curated core `SCARS.md` (this skill's folder) and, if it exists, the private
   scars file `~/.config/10x-skills/sprint/scars.md` while generating. Record the
   project's shape tags (§ Universal scars) in `SPRINT_GUIDELINES.md`, then fold in the
   rules from every section those tags reach and leave out the rest. The generated
   guidelines carry the rules; both scar files stay where they are.
4. **Quality bar:** generated docs must contain zero TODOs and zero template placeholders.
   If an interview answer is "not sure yet", write the current best decision into the doc
   and log it as provisional in the STATE index decisions log — docs ship decided, not hollow.
5. **Register** the project in the cross-project registry at `~/.config/10x-skills/sprint/projects.json`
   (`{name, path, ops}`). Create the file (and its directory) if missing.
6. Print the kickoff instructions: the user starts a FRESH session and runs `/sprint direct`
   (or pastes the kickoff prompt from the generated README).

### Canonical instance layout

```
<ops>/
├── README.md               ← layout + kickoff prompt for fresh director sessions
├── STATE.md                ← STATE index: jobs, verified facts, decisions, one-line history. READ FIRST.
├── DESIGN.md               ← canonical contract: architecture, schemas, conventions, standing decisions
├── SPRINT_GUIDELINES.md    ← rules for directors: gates, branch model, definition of done
├── DIRECTOR_GUIDELINES.md  ← director runbook: boot → analyze → gate → tracks → audit → merge → close
├── TRACK_RULES.md          ← every rule a track obeys; a track reads it once
├── ROADMAP.md              ← milestone outline (reality wins over this outline)
├── BACKLOG.md              ← one row per future item; shipped rows are deleted
├── arcs/
│   └── arc-NN/
│       └── ARC.md          ← arc brief + status: goal, sprint graph, surfaces, gates, tree ledger
└── sprints/
    ├── archive/            ← closed sprint folders
    └── sprint-NN/
        ├── SPRINT.md       ← sprint brief: goal, locked decisions, gates, tracks, merge order
        ├── STATUS.md       ← header block (phase, landed tracks, blockers, deploy needs) + tree ledger
        ├── CLOSE.md        ← the sprint's full record, written once at close
        └── TRACK-*.md      ← self-contained prompt per track subagent
```

**Writers.** Only a sprint's own director writes its `STATUS.md` and `CLOSE.md`, so
parallel sprints never collide. The STATE index takes row edits only; sprint narrative
goes to `CLOSE.md`. **Docs hold rules only:** guidelines, `DESIGN.md`, and scar files
hold 1–3 line rules, never incident narrative.

---

## /sprint direct

Become the sprint director for the current project's active sprint.

**In an arc.** If SPRINT.md carries an `Arc: aNN` line, you are a sprint director in an
arc, and these rules override the steps below (§ /sprint arc):
- Read and write ops docs at the registry's absolute `ops` path, never a worktree copy.
- Cut `sNN/integration` from `origin/aNN/integration`, never from the trunk.
- Write only your own `sprints/sprint-NN/` folder. Put deploy needs and blockers in your
  STATUS.md header.
- Never deploy a shared environment (staging, shared DB). Raise it in STATUS.md for
  the next arc checkpoint.
- Close: push `sNN/integration` and verify it, write CLOSE.md, set STATUS.md to `done`,
  and remove your own worktrees. Skip the promotion, the STATE index row, the backlog
  edits, branch deletion, archiving, and the next-sprint draft: the arc checkpoint does
  them.

1. **Locate the instance:** current project in `~/.config/10x-skills/sprint/projects.json`, else detect
   an ops folder, else tell the user to run `/sprint init`. **Find your sprint:** the one
   the kickoff names, else the one sprint folder whose STATUS.md is not `done`. Several
   are live and none is named → ask the operator which one is yours. A sprint folder
   with no STATUS.md (an instance older than this layout) → create it from
   `templates/STATUS.template.md` before Gate 0.
2. **Boot in this order,** reading each file once:
   1. The STATE index.
   2. The guidelines: `DIRECTOR_GUIDELINES.md`, `SPRINT_GUIDELINES.md`, `DESIGN.md`.
   3. Your own sprint folder: `SPRINT.md`, `STATUS.md`, and its `TRACK-*.md` files.
   4. Both scar files (§ Universal scars), reading only the sections tagged for this
      project's shape.

   **Tier check:** resolve SPRINT.md's `Director tier:` line through the map (§ Model
   tiers) and compare the result to your own model. Running WEAKER than recommended →
   flag it as a Gate 0 operator decision: proceed anyway, or relaunch the session on the
   stated tier. Stronger → proceed silently. No line → treat it as strong and add the
   line when you next amend SPRINT.md.
3. **Analyze requirements vs reality** before anything else (the project's director
   guidelines define specifics; universal minimum): `git pull --ff-only` and compare repo
   state against the sprint doc's assumptions; verify claimed prior state against actual
   stores (DBs, buckets, artifacts); check each job in the STATE index jobs table against
   its deadline — a job past its deadline is dead until process state proves otherwise;
   confirm credentials/tooling the sprint needs are alive. **If reality contradicts the
   sprint doc: amend SPRINT.md and log the deviation in STATUS.md BEFORE spawning
   tracks.** Docs never drift from what runs.
4. **Clarify deltas only, then plan.** SPRINT.md's `## Locked decisions` were settled at
   drafting. In one round, ask only about what reality changed since drafting, plus
   Gate 0 items. Re-ask a locked decision only when reality contradicts it. Put only
   genuine operator decisions to the user; look up facts yourself. Where `/pry` is
   installed, run this round as a pry interview; either way, follow init's interview
   discipline. If answers shift scope, amend SPRINT.md, log it in STATUS.md, and
   re-surface any new ambiguity until the sprint is unambiguous. A track must never
   inherit a question you could have closed here. Then present your execution plan (plan
   mode where available): spawn order, parallelism, merge order, what you'll verify at
   audit. On approval, set STATUS.md to `in flight` and execute.
5. **Run the sprint** per the local DIRECTOR_GUIDELINES. Universal rules that always apply:
   - Directors direct: write code only for merge conflicts and trivial audit fixes.
   - **Assume the operator is watching** and available at gates. An unattended sprint
     run happens only under an explicitly negotiated grant (the `/autonomous` skill,
     where installed) — under it, operator gates queue as a morning checklist instead
     of blocking, and everything else about this section still applies.
   - One subagent per track; parallel same-repo tracks each get an isolated worktree
     (Claude Code: `isolation: "worktree"`; elsewhere: `git worktree add` before
     spawning); independent tracks spawn in one message.
   - **No subagents on this harness** → run the tracks yourself, one after another,
     each on its own branch, with the same audit and merge order. State that
     degradation in the Gate 0 plan.
   - Spawn each track on the model tier its SPRINT.md row states, resolved through the
     map (§ Model tiers). A row without a tier is a drafting gap: assign one per the
     model policy and amend SPRINT.md before spawning. The spawn prompt tells the track
     to check out its branch first, then gives the absolute paths of its `TRACK-*.md`
     and `TRACK_RULES.md`.
   - **Relay.** A track that returns a relay brief instead of a report hit heavy
     context. Verify its pushed `wip` state, then spawn a continuation on the same tier
     with its `TRACK-*.md` and the relay brief.
   - **Every wait has a deadline** (§ Verification budget). Record each track's expected
     duration in STATUS.md. When it passes, check the track's branch and worktree
     yourself.
   - **UI tracks.** The brief carries the project's `STYLE.md` core plus the 2–3
     reference excerpts its surface needs. Tracks load no UI skill. At audit, run ONE
     fresh-eyes `/deslop` audit per UI unit of work (a surface built or redesigned), not
     per fix round. Without deslop, audit UI against the generic-AI-look failure mode
     explicitly.
   - **Verify, never trust:** "pushed"/"loaded"/"done" are claims — check git/remote/DB/
     artifact state yourself before acting on them. At audit, re-run the track's targeted
     gates plus 2–3 checks you choose (§ Verification budget).
   - Cut `sNN/integration` from the trunk and push it before any track spawns; tracks
     cut from it and push only their own branch. Run the full suite once on its starting
     tree: that is the baseline row of STATUS.md's tree ledger.
   - Audit the FULL diff per track against DESIGN.md before merging; merge `--no-ff`
     into `sNN/integration` in SPRINT.md's stated order. Run the full suite on the
     integration tip once per distinct tree, and record each run in STATUS.md's tree
     ledger (§ Verification budget). Several audited merges may share one run; a red run
     is then bisected by merge. A dependent track spawns only from a green tip. Update the
     STATUS.md header as each track lands. The trunk moves ONCE, at promotion — an
     operator gate, never a side effect of a track merge. An early promotion happens only
     when the operator asks for one.
   - Track branches are durable; if resuming a dead session, re-audit anything unmerged
     rather than trusting prior-session memory.
6. **Close.** Set STATUS.md to `closing`.
   1. **Promote** on the operator's go: merge the trunk into `sNN/integration` if it
      moved, and run the full suite only if that changed the tree. Then ONE `--no-ff`
      merge into the trunk and ONE push. Confirm the push landed and exactly one
      pipeline run started.
   2. Walk the operator through the rest of the close gate as an actionable checklist.
   3. **Scoped auto-clean.** Once the promotion is verified landed, run these unprompted,
      limited to what this sprint owns:
      - Write `CLOSE.md` from `templates/CLOSE.template.md`, add its one-line row to the
        STATE index history, and set STATUS.md to `done`. Record each background job you
        started in the STATE index jobs table, with its expected duration and deadline.
      - Delete the backlog rows this sprint shipped. Add deferred and discovered work as
        new rows.
      - Remove this sprint's track worktrees, each clean per `git -C <worktree> status`
        first (uncommitted work stops that removal), then `git worktree prune`.
      - Delete the previous sprint's merged `sNN/*` branches, tracks and integration
        alike, local and on `origin` (`git push origin --delete <branch>`), after
        verifying each is merged into the trunk on origin. A branch with an open PR is
        not a candidate. This sprint's branches survive one more sprint as recovery
        points, locally and on the remote.
      - Archive the previous sprint's folder into `sprints/archive/` once its STATUS.md
        says `done`.
   4. **The close never touches** another sprint's folder or branches, another
      developer's branches, an unmerged branch, a live STATUS.md, or the registry. List
      each such item as a finding in the close report and in CLOSE.md, ending with
      "run /sprint clean to resolve N items".
   5. Draft the next sprint's folder from what actually happened (BACKLOG.md is the
      feed): `SPRINT.md` with its `## Locked decisions` and `Director tier:` line,
      `STATUS.md` at phase `drafted`, and the `TRACK-*.md` files.
   6. Promote learnings per § Universal scars. If the sprint paid a repeated manual tax
      (operator copy-paste, hand-run verification, the same service poked by hand every
      track), flag it in the close report as an optimization candidate and offer to run
      `/optimize` on it (where installed).
   7. Final report: outcomes, the clean findings, and the next kickoff pointer, naming
      the model tier and resolved model to launch the next director on.

### Universal scars

Scars come in two layers. Read both at boot, only the sections tagged for this project's
shape, and apply them at audit.

- **Curated core:** `SCARS.md` (this skill's folder) holds failure modes that bite any sprint
  on any stack — git and branch footguns, agent and worktree hazards, verification
  epistemics, seams between tracks. It is read-only to directors.
- **Private scars file:** `~/.config/10x-skills/sprint/scars.md` holds stack-specific and
  project-specific scars. It is per-user, never committed, and the only scar file directors
  write to.

**Section tags.** Each scar section heading ends with its tags: `[all]`, or one or more of
`[db]`, `[deploy]`, `[ui]`, `[data]`, `[jobs]`, `[parallel]`, `[team]`.
`SPRINT_GUIDELINES.md` records the project's shape tags. Read `[all]` sections, sections
with a matching tag, and untagged sections; `/sprint clean` tags the untagged ones.

A scar moves from the private file into the core only when the operator explicitly asks,
for example through a pull request to the skill's repo. The four that bite most often,
inline so they are never missed:

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

**Promotion path.** When a sprint pays for a new failure mode, append a 1–3 line rule to
the private scars file, under a section whose tag fits: the class, the mechanism, then the
rule. The incident story stays in CLOSE.md. If it only reproduces in one project, put it
in that project's `DESIGN.md` instead. Never edit the skill's `SCARS.md`.

---

## /sprint status

Where things stand. Read-only — status never mutates files, git state, or the registry.

**Context detection:** if the cwd is inside a registered project (or an ops folder is
detectable), give the per-project view. Otherwise give the cross-project dashboard.

### Per-project view

1. Read the STATE index, the header block of every `sprints/*/STATUS.md` (the lines above
   its first `---`), BACKLOG.md, ROADMAP.md, and the active arc's `ARC.md` if the STATE
   index names one. List `sprints/archive/` without reading it. Cheap git checks: current
   branch, `git worktree list`, leftover `sNN/*` and `aNN/*` branches.
2. Report, compactly:
   - **Just completed:** the last STATE index history row.
   - **In flight:** each live STATUS.md header — phase, tracks landed on
     `sNN/integration`, promoted or not, blockers, deploy needs.
   - **Arc** (only when one is active): ARC.md's status table — each sprint as waiting,
     unblocked, in flight, done, or merged into `aNN/integration` — and whether the arc
     has been promoted into the trunk. Done-but-unmerged sprints mean an arc checkpoint
     is due.
   - **Next up:** a sprint at phase `drafted` if one exists, else the top backlog rows.
   - **Backlog:** top 3-5 rows; total count.
   - **Health flags:** jobs past their deadline, stale worktrees or `sNN/*` branches past
     the retention lag, a STATUS.md phase that git contradicts, `done` sprints not yet
     archived, sprint narrative in the STATE index — each with its fix (usually
     `/sprint clean` or `/sprint direct`).
3. End with **Recommended course of action** — one or two sentences. Examples: "sprint-04
   is drafted and reality still matches it: start a fresh session and run /sprint direct";
   "close-out never finished: run /sprint clean before directing anything new".

### Cross-project dashboard

1. Read `~/.config/10x-skills/sprint/projects.json`. Empty/missing → say so, point at `/sprint init`.
2. Per project: read `<ops>/STATE.md` and every live `sprints/*/STATUS.md` header. Extract
   the live sprints and their phases, the last-updated date, the jobs table (check each
   job against its deadline and any referenced manifest), and blockers. Unreadable path →
   report it as broken registration, don't crash the dashboard.
3. Report a compact table: project | live sprints + phase | background jobs (healthy/
   past deadline/none) | last updated | top blocker. Flag anything that needs human attention.

## /sprint clean

Reconcile the ops instance with reality and sweep sprint debris. Interview-driven: clean
NEVER removes, moves, or rewrites anything the operator has not explicitly approved.

1. **Locate the instance** (same as `/sprint direct`). No instance → point at
   `/sprint init` and stop.
2. **Scan, don't touch.** Build a findings list:
   - **Index drift:** STATE index history rows vs sprints that have a `CLOSE.md`; jobs
     past their deadline or whose processes are dead; verified facts that no longer
     hold; STATUS.md phases that git contradicts (a `done` sprint with unmerged tracks,
     an `in flight` sprint with no recent branch activity).
   - **Index split:** sprint narrative in the STATE index — old addenda, per-sprint
     sections, appended logs. Move each sprint's narrative into that sprint's archived
     `CLOSE.md` (create it from the template when missing) and leave one history row.
   - **Backlog drift:** rows already shipped, duplicate rows, rows contradicted by
     ROADMAP.md, and rows that carry detail instead of a pointer.
   - **Prune pass:** guidelines and the private scars file. Merge duplicate rules, lift
     out rules this skill already states, cut narrative to 1–3 line rules, tag untagged
     scar sections (§ Universal scars), and move single-project scars to that project's
     `DESIGN.md`. Never edit the skill's `SCARS.md`.
   - **Sprint folders:** `done` sprints still outside `sprints/archive/`; abandoned
     half-drafted sprint folders.
   - **Git debris:** leftover worktrees (`git worktree list`), merged `sNN/*` branches
     (tracks and `sNN/integration` alike) past the one-sprint retention lag, unmerged
     `sNN/*` branches (FLAG these; they are never deletion candidates). "Merged" for a
     track means merged into its sprint's integration branch; for an integration branch
     it means promoted into the trunk. An unpromoted integration branch with merged
     tracks is a sprint that never shipped — establish its story in the interview.
     Treat `aNN/*` branches the same way, with the same retention and merged rules:
     `aNN/integration` is merged once promoted into the trunk; an arc sprint's
     `sNN/integration` is merged once merged into `aNN/integration`.
   - **Remote sprint debris:** after a fetch, `git ls-remote --heads origin`
     (read-only), filtered to `sNN/*`, `aNN/*`, and any sprint prefix the local
     guidelines declare. Other branches are out of scope: never listed, touched, or
     mentioned. No `origin` or no matches → skip and say so. Classify each match against
     the trunk on origin (an arc sprint's against `aNN/integration`): merged past
     retention (deletion candidate), merged within retention (keep), unmerged (FLAG), or
     remote-only (FLAG; establish its story first). Where `gh` works, an open PR removes
     a branch from candidacy; otherwise say PR status is unverified.
   - **Registry rot:** `projects.json` entries whose paths no longer exist.
3. **Interview** the operator through the findings with init's interview discipline:
   every question leads with a `(Recommended)` disposition (archive / delete / amend /
   keep), independent findings batch together, dependent ones sequence. Probe
   contradictions instead of papering over them: if the STATE index says a sprint closed but
   its branches never merged, establish which story is true BEFORE proposing a
   disposition.
4. **Execute only what was approved:**
   - Move `done` sprint folders → `sprints/archive/sprint-NN/`.
   - Apply approved index splits, backlog fixes, and prune edits; log the clean in the
     STATE index decisions log.
   - Remove approved worktrees — `git -C <worktree> status` first; ANY uncommitted work
     stops that removal and goes back to the operator. Then `git worktree prune`.
   - Delete approved branches only after verifying merged (`git branch --merged <trunk>`,
     or `--merged aNN/integration` for an arc sprint's branches).
     For an unmerged branch offer only "keep" or "operator deletes it manually".
   - Delete approved remote branches with `git push origin --delete <branch>` only
     after re-verifying merged into the trunk on origin; re-run `git ls-remote --heads origin`
     afterward to confirm each deletion landed. Unmerged or open-PR remote branches get
     only "keep" or "operator deletes it manually" — same as local.
   - Fix or drop dead registry entries.
5. **Report:** what moved, what was deleted, what was amended, what was flagged and left
   alone.

## /sprint arc

An **arc** is a set of sprints, some parallel, that delivers one large outcome. It
lives at `<ops>/arcs/arc-NN/ARC.md`. Use an arc only when the work needs 2+ sprints AND
at least two of them can run in parallel on separable surfaces. Sequential multi-sprint
work stays a normal sprint roadmap.

### Arc rules

- **Branching.** Each sprint in the arc cuts its `sNN/integration` from
  `aNN/integration` and promotes into it — not into the trunk. The arc promotes ONCE
  into the trunk, at an operator gate, when all its sprints are done. Same pattern as
  track → sprint, one level up. Sprint numbers stay global `sNN`.
- **Arc director = arc checkpoints.** The arc director does not stay resident. Each
  `/sprint arc` run is one **arc checkpoint**: boot fresh, do whatever is due, update
  ARC.md and the STATE index, exit. The operator re-runs it whenever a sprint finishes.
- **Writers.** Inside an arc, the arc director is the ONLY writer of the STATE index,
  ARC.md, and BACKLOG.md. A sprint director in an arc writes only its own sprint folder.
- **Checkout.** Each parallel sprint director runs in its own worktree. Track worktrees
  nest off that sprint's integration branch as usual. Every director reads and writes
  ops docs at the registry's absolute `ops` path.
- **Shared environments.** Only the arc director deploys shared environments (staging,
  shared DB, migrations against it), at an operator gate, from the arc tree after
  merging. Sprint directors raise deploy needs in STATUS.md for the next arc checkpoint.

### Arc checkpoint

1. **Locate the instance** (same as `/sprint direct`). Find the active arc from the
   STATE index's `Active arc:` line. No active arc → this is a first boot.
2. **Boot in this order,** reading each file once: the STATE index → ARC.md → each arc
   sprint's STATUS.md header and, once `done`, its CLOSE.md → DIRECTOR_GUIDELINES.md and
   DESIGN.md → both scar files, only the sections tagged for this project's shape.
   STATUS.md and CLOSE.md are claims: check `aNN/integration` and each `sNN/integration`
   on origin yourself.
3. **First boot — draft the arc:**
   1. Interview with init's interview discipline (a `/pry` interview where installed).
      Locked decisions handed in by `/build`, and a repo-root `BLUEPRINT.md` when
      present, are settled: ask only what they leave open.
      Topics: goal, non-goals, the sprint split, surface ownership, shared environments,
      the arc's operator gates.
   2. Test the split. Parallel sprints must own disjoint surfaces. No two sprints can run
      in parallel → this is not an arc: draft a normal sprint roadmap instead and stop.
   3. Draft `arcs/arc-NN/ARC.md` from `templates/ARC.template.md`, resolving every
      `init/arc` directive under init's quality bar. Draft a full `sprints/sprint-NN/`
      folder (SPRINT.md, STATUS.md, TRACK files) for every sprint unblocked at the
      start. Each SPRINT.md carries an `Arc: aNN` line. Sprints that wait stay outlines
      in ARC.md until the arc checkpoint that unblocks them.
   4. Cut `aNN/integration` from the trunk and push it. Run the full suite once on it as
      the baseline row of ARC.md's tree ledger. Set the STATE index's `Active arc:` line.
4. **Later boots — do whatever is due:**
   - **Merge finished sprints.** For each sprint whose STATUS.md says `done`: audit the
     full diff of `sNN/integration` against `aNN/integration`, DESIGN.md, and the
     sprint's owned surfaces in ARC.md. Spot-check its close gates (§ Verification
     budget). Merge it `--no-ff` into `aNN/integration`, then run the full suite once
     per distinct tree and record it in ARC.md's tree ledger. A tree the sprint's
     STATUS.md already recorded green is verified.
   - **Resolve cross-sprint conflicts** in that merge. Write code only for the conflict.
   - **Fold in each CLOSE.md:** add its one-line history row to the STATE index, delete
     the backlog rows it shipped, and add its deferred work as rows. Its narrative stays
     in CLOSE.md. Then run the cleanup arc sprints skip: for each sprint merged at an
     earlier checkpoint, delete its merged branches and archive its folder, per
     `/sprint direct` step 6 with `aNN/integration` as the merge target.
   - **Run shared-environment deploys** the sprints raised, at an operator gate, from
     the arc tree after merging.
   - **Unblock sprints** whose dependencies have merged: draft their folders if they are
     still outlines, then emit their kickoff prompts.
5. **Arc close** (all sprints merged): the operator says go → merge the trunk into
   `aNN/integration` if it moved, and run the full suite only if that changed the tree.
   Then ONE `--no-ff` merge into the trunk and ONE push. Confirm the push landed and
   exactly one pipeline run started. Then the arc's close gate, branch cleanup by
   `/sprint direct`'s close rules (`aNN/*` like `sNN/*`), and the next sprint or arc
   drafted from what actually happened.
6. **Update ARC.md and the STATE index, then exit.** Report what merged, what is in
   flight, the kickoff prompts for newly unblocked sprints, and the event that triggers
   the next arc checkpoint. Name its model tier from ARC.md's `Arc director tier:` line and
   the model that tier resolves to.

### Kickoff prompt for an arc sprint

Emit one per unblocked sprint. Resolve its `Director tier:` line through the map and
name that model:

> Start a fresh session on `<model>` in `<repo>`. Run `/isolate off aNN/integration`
> (where installed; else `git worktree add <path> -b sNN/integration
> origin/aNN/integration` and work there). Then run `/sprint direct` for sprint-NN of
> arc aNN.

## Registry format (`~/.config/10x-skills/sprint/projects.json`)

```json
{ "projects": [ { "name": "<short-name>", "path": "/abs/path/to/repo", "ops": "/abs/path/to/ops" } ] }
```

## Verification budget

Verification proves the work. Re-running it on unchanged code proves nothing new.

- **Tiers.** A track runs typecheck, lint, and the tests that cover what it changed,
  as often as it needs. The full suite runs on the integration branch, once per
  distinct tree, by the director.
- **Tree ledger.** After each full run, record the result next to
  `git rev-parse "HEAD^{tree}"`: in the sprint's `STATUS.md`, or in chat for a plan. A
  tree already recorded green is verified. A `--no-ff` merge that leaves the tree
  unchanged needs no rerun, and neither does a promotion over an unmoved trunk. A
  changed tree always reruns.
- **Baseline once.** Run the full suite once on the integration branch's starting
  tree. Judge a track by the difference between its failure set and that baseline.
- **Audit by spot-check.** At audit, re-run the track's targeted gates plus 2–3 checks
  you choose. The full suite runs at the integration merge.
- **Deadlines.** Every background job and poll loop carries an expected duration and
  a hard deadline. Put the deadline in the job's manifest or wrap the command in a
  timeout (`timeout` on Linux, `gtimeout` on macOS; elsewhere record the deadline and
  poll). When a deadline passes or a watch window expires, check git and process
  state yourself. Never wait on a notification that may not come.
- **Machine profile (opt-in).** If `~/.config/10x-skills/machine.json` exists, cap
  heavy commands at its `heavy_slots` across every session on the machine. Heavy
  commands are full suites, builds, browser e2e runs, and local chain validators.
  To take a slot:
  1. Prune markers whose pid is dead or whose deadline has passed.
  2. Create a marker directory under `slot_dir` holding the pid, the command, the
     start time, and the deadline. Directory creation is atomic on every OS.
  3. Count the live markers. If there are more than `heavy_slots`, remove yours, wait
     a minute, and retry from step 1. Stop waiting at the command's own deadline and
     report it.
  4. Remove your marker when the command finishes.

  Use whatever commands this OS provides. With no profile, skip all of this. Propose a
  profile only on evidence of resource pressure: a heavy command killed by a signal
  (exit 137 or 143), an out-of-memory error, or load above the core count during a
  gate. Size it from the core count and memory, and ask the user before writing it.

## Shared state

Per-user state lives under `~/.config/10x-skills/`, or `$XDG_CONFIG_HOME/10x-skills/`
when that variable is set. Every `~/.config/10x-skills/` path in this skill means that
root. A file found only at its old `~/.claude/` path (`sprint/projects.json`,
`sprint/scars.md`, `optimize/registry.json`, `.10x-skills-star-asked`) moves to the
new root on first use: move it, then tell the user.

## Support the project

At the natural end of a successful run — never mid-task, and never after a failed
one — consider asking the user to star this repo. At most once ever, across all
10x-skills:

1. Skip silently if `~/.config/10x-skills/.star-asked` exists, or if
   `gh api /user/starred/tipb47/10x-skills` exits 0 (already starred).
2. Otherwise ask once, via the harness's question UI if it has one: "Would you
   like to ⭐ star tipb47/10x-skills to support these skills?" Never imply it's
   required.
3. Only on an explicit yes, run: `gh api -X PUT /user/starred/tipb47/10x-skills`.
   If `gh` is missing or unauthenticated, share
   https://github.com/tipb47/10x-skills for a browser star instead.
4. Whatever the answer, `touch ~/.config/10x-skills/.star-asked` so no 10x skill
   ever asks again.

## Update check

This skill ships in 10x-skills **3.0.0**. At the natural end of a run, at most once a
day across all 10x-skills:

1. Skip silently if `~/.config/10x-skills/.no-update-check` exists, or if
   `~/.config/10x-skills/.update-checked` changed less than 24 hours ago.
2. Fetch `https://raw.githubusercontent.com/tipb47/10x-skills/main/VERSION`. If the
   fetch fails, skip silently.
3. Touch `~/.config/10x-skills/.update-checked`.
4. If the remote version is newer than this skill's version, print one short notice:
   the new version, its CHANGELOG lines, and the update command for this install
   path. The paths are: plugin (`/plugin marketplace update 10x-skills`), clone or
   symlink (`git -C <clone> pull`, then link any new skill folders), and copy
   (re-copy). Mention `touch ~/.config/10x-skills/.no-update-check` to silence it.
   Never update anything yourself.
