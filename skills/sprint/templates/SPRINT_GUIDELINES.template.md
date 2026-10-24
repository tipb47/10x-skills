# Sprint Guidelines

Rules for every sprint and director. Director runbook: `DIRECTOR_GUIDELINES.md`.
Directors obey this file and `DESIGN.md`. Tracks obey `TRACK_RULES.md`. **This file
holds 1–3 line rules, never incident narrative**; the story behind a rule stays in a
sprint's `CLOSE.md`.

## Project shape

Shape tags: {{SHAPE_TAGS}}
<!-- init: from interview facts, the tags this project's shape reaches: `db`, `deploy`,
`ui`, `data`, `jobs`, `parallel`, `team`. Directors read scar sections tagged `[all]` or
with one of these tags. -->

## What a sprint is

- One bounded milestone, run end-to-end by ONE fresh agent session (the
  **director**), which spawns each **track** as a subagent.
- {{TRACK_COUNT_POLICY}} tracks per sprint.
  <!-- init: 1-3 for pipeline-shaped projects with sequential dependencies; 2-4 for
  multi-surface products. State which shape this project is and why. -->
- Sprints contain **operator gates** — explicit pauses for the human
  ({{GATE_EXAMPLES_FROM_INTERVIEW}}). Gates live in `SPRINT.md`, never inside track
  prompts. Gate 0 runs before tracks spawn; the close gate runs after every track has
  landed on the integration branch, and promotion into `{{TRUNK}}` is part of it.

## Branch model

**In an arc (SPRINT.md has `Arc:`):** cut from and merge into `aNN/integration`, never
`{{TRUNK}}`; no promotion, no next-sprint draft.

- **Trunk:** `{{TRUNK}}`. A push to it triggers {{TRUNK_PUSH_EFFECT}}, which is why it
  moves once per sprint, never once per track.
  <!-- init: the branch sprints promote into (main, staging, ...) and what a push to it
  triggers (CI run, preview deploy, production deploy), from interview. -->
- **Integration branch:** `sNN/integration`, cut by the director from `origin/{{TRUNK}}`
  at Gate 0 and pushed once, empty. Every track lands here first. The trunk never
  receives a track directly.
- **Track branches:** `sNN/<slug>`, cut from `origin/sNN/integration`. A track that
  depends on another track's work spawns after that work lands on integration and cuts
  from the integration tip, so it sees the work without the trunk moving.
- **Promotion:** ONE `--no-ff` merge of `sNN/integration` into `{{TRUNK}}`, ONE push, ONE
  pipeline run. It happens at the close gate, or earlier only when the operator asks for
  an early promotion. Promotion is always an operator gate: the director never pushes
  `{{TRUNK}}` unprompted.
- **CI noise:** {{CI_TRIGGER_POLICY}}
  <!-- init: resolve from the interview's CI-trigger answer:
  • CI runs on the trunk only, or on PRs only → "CI runs only on `{{TRUNK}}`; track and
    integration pushes cost nothing."
  • CI runs on every branch and the operator CAN change it → "CI is filtered to
    `{{TRUNK}}` (branch filter + concurrency group with cancel-in-progress)." Put the
    filter change on sprint-01's Gate 0; until it lands, apply the next line.
  • CI runs on every branch and CANNOT change → "Tracks put `[skip ci]` in the subject of
    every `wip:` commit; the director accepts one run per integration push." -->

## Track drafting and verification

Every track rule — push authority, commits, stash ban, scratch state, comments, report
format — lives in `TRACK_RULES.md`. A track reads it and its `TRACK-*.md` once.

1. **Self-contained prompts, one focused context.** A track reads its `TRACK-*.md`,
   `TRACK_RULES.md`, and the `DESIGN.md` sections it names, and needs nothing else. Size
   it to one surface and one contract. Mid-flight human input required = the sprint was
   planned wrong.
2. **The verification contract (DESIGN §5) is the gate, in two tiers.** Tracks run the
   targeted tier: {{TARGETED_TESTS}}. The director runs the full suite,
   {{FULL_SUITE}}, on the integration branch once per distinct tree, and records each run
   in the sprint's `STATUS.md` tree ledger.
   <!-- init: instantiate concretely — exact test commands for both tiers; for data work:
   tracks ship verification queries WITH expected results and reports quote ACTUAL
   output. -->
3. **Never trust, always verify.** "Done"/"pushed"/"loaded" are claims; the report quotes
   command/query output, not assertions. Directors spot-check at audit.
   - **A passing grade is not a verdict until you know WHY it passes.** Clearing a gate is
     necessary, not sufficient — decompose the SOURCE of the result before acting on it, and
     grade against the honest baseline, not a trivial floor. A number whose source you cannot
     name is not a result.
   - **Verify through the ACTUAL path.** A hand-run literal does not exercise the client's
     parameterized path; an unauthenticated request proves nothing about a credential; a mock
     cannot see a bug in the part of reality it omits.
4. {{MIGRATIONS_RULE}}
   <!-- init: if DB: "Migrations are files in <path>, applied ONLY by the director at
   merge time; tracks code against the post-migration schema." If no DB: drop this rule. -->
5. **No fabricated data, ever,** outside clearly-marked test fixtures under test dirs.
   A job that can't fetch real data records the failure — it never fabricates rows.
<!-- init: if the project has long-running jobs, add the AFK-grade standard as a rule:
checkpoint manifest, detached execution, incremental sync, inline validation, idempotent
re-runs, an expected duration and a deadline — tracks deliver the harness verified on a
sample chunk; full runs start at close. -->

## Definition of done (sprint)

- All track branches audited, merged `--no-ff` into `sNN/integration` in SPRINT.md's
  order, pushed; the final integration tree recorded green in the tree ledger.
- `sNN/integration` promoted into `{{TRUNK}}` on the operator's go: one `--no-ff` merge,
  one push, onto a tree the tree ledger recorded green.
- Scoped auto-clean done: this sprint's worktrees removed (each checked for uncommitted
  work first), `git worktree prune` run, the previous sprint's merged branches deleted
  local and on `origin`, and its `done` folder archived. The previous sprint is the
  previous closed sprint in this sprint's own lineage: the sprint this one was drafted
  from. No other sprint is touched. This sprint's branches survive one
  more sprint as recovery points; unmerged branches are never deleted, anywhere.
- {{DB_DONE_CLAUSE}} <!-- init: migrations applied + verified, if DB; else drop. -->
- Operator close-gate checklist delivered/executed.
- `CLOSE.md` written, one STATE index history row added, shipped backlog rows deleted,
  and `STATUS.md` set to `done`; the next sprint's folder drafted from reality.

## Token discipline

- One subagent per track; no orchestration frameworks unless the operator explicitly asks.
- Director audits diffs directly; at most ONE review subagent per sprint, only for a
  genuinely risky merge.
- Tracks read what their file points to, once; no exploratory fan-outs — `DESIGN.md` is
  the map.
- Model policy: {{MODEL_POLICY}}.
  <!-- init: from interview, seeded from the engine's default policy, in model tiers
  (frontier / strong / fast / light), never model names — director and drafter: strong
  minimum; contract-defining / architecture / gnarly-debug tracks: strong; standard
  implementation tracks, scouts, and mechanical audit checks: fast; fix rounds: the
  track's tier, or fast when the director fully specifies the fix; light only for
  mechanical fully-specified chores with a stated justification. Frontier only for
  genuinely hard problems (novel algorithms, modeling engines); state which work in THIS
  project earns it, and raise any default
  the project needs (e.g. core engine tracks on frontier). Recorded in SPRINT.md at
  drafting time, never decided mid-sprint. Every SPRINT.md track row states its model
  tier + a one-line rationale. The tier-to-model map is per-user
  (`~/.config/10x-skills/models.json`), never written here. -->

## Git conventions

- Commits: imperative summary, prefixed with the track tag `[sNN/<slug>]` (director
  ops/merge commits use `[sNN/ops]`); end with the project's standard co-author trailer.
- Comments describe the code, not the change (`TRACK_RULES.md` § Comments). Provenance
  lives in commit messages, the track report, and `CLOSE.md` — never in code.
- Stage with EXPLICIT paths — never `git add -A` / `git add .`.
- **Ops bookkeeping stays out of code commits.** {{OPS_COMMIT_POLICY}}
  <!-- init: resolve from the interview's tracked/untracked answer:
  • UNTRACKED → "The <ops> dir is gitignored; never stage it."
  • TRACKED → "<ops> churn (STATE.md / sprint folders) commits SEPARATELY from
    code, tagged `[sNN/ops]` — never mixed into a code or merge commit. Keeps the code
    history clean and bisectable." -->
- **Confirm the ops folder's tracked/untracked status on EVERY branch the director checks
  out.** An ops folder gitignored on one branch but still TRACKED on another is OVERWRITTEN
  by the stale tracked copy the moment you check that branch out for a merge. Recover with
  `git show <commit>:<path>`; prevent with `git rm --cached` on every branch you visit.
- With worktrees, ALWAYS `git -C <absolute-path>`.
- Diff a track from the MERGE-BASE, never the branch tip against a moved trunk:
  `git diff $(git merge-base sNN/integration <branch>) <branch>`. The two-way diff shows a
  clean track "deleting" files it never touched.
- Before merging, `git checkout` the TARGET branch (`sNN/integration` for a track,
  `{{TRUNK}}` for promotion) and confirm `git branch --show-current`. A repo left on the
  feature branch merges it into ITSELF and reports "Already up to date" — a silent no-op
  that reads as success.
- Never chain push/merge after a PIPED build/test in one compound — pipes mask exit codes.
  Verify, check the exit code, THEN push. Judge builds by exit code, not stderr noise.
- Director merges `--no-ff` per track into `sNN/integration`, and `--no-ff` once into
  `{{TRUNK}}` at promotion. Never force-push `{{TRUNK}}`; never force-push
  `sNN/integration` either — live tracks are cut from it.
<!-- init: if out-of-band human pushers exist: add "pull + check their recent commits at
boot; never clobber; rebase, don't force; keep changes to their hot files minimal+flagged." -->
