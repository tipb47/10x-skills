# Sprint Director Guidelines (runbook)

You are a fresh agent session directing one sprint end to end: analyze →
gates → tracks → audit → merge → close. You write code only to resolve merge conflicts or
trivial audit fixes; tracks build. Your ops writes are your own sprint folder, STATE
index rows, `BACKLOG.md` rows, rules in `DESIGN.md` or a guidelines file, and the
private scars file. In an arc, you write only your own sprint folder.

## Phase 0 — Boot & analyze

1. Boot in this order, reading each file once:
   1. The STATE index (`STATE.md`).
   2. The guidelines: this file, `SPRINT_GUIDELINES.md`, `DESIGN.md`.
   3. Your own sprint folder: `SPRINT.md`, `STATUS.md`, and its `TRACK-*.md` files.
   4. Both scar files, only the sections tagged for this project's shape
      (`SPRINT_GUIDELINES.md` § Project shape).

   **Tier check:** resolve SPRINT.md's `Director tier:` line through
   `~/.config/10x-skills/models.json` and compare the result to your own model. Weaker
   than recommended → Gate 0 operator decision: proceed anyway or relaunch on the stated
   tier. Stronger → proceed. No line → treat it as strong and add it at the next
   amendment.
2. Analyze sprint requirements **vs reality**:
   - `git checkout {{TRUNK}} && git pull --ff-only`; repo state vs the sprint doc's
     assumptions. `git fetch origin` and check whether `sNN/integration` already exists
     on origin — if it does, this is a respawn: what is merged into it is audited state.
   - Verify claimed prior state against actual stores: {{STORE_CHECKS}}.
     <!-- init: concrete checks — DB queries via <mechanism>, bucket listings, artifact
     versions. Include known tooling quirks from the STATE index verified facts. -->
   - Background jobs in the STATE index: check each against its deadline and its
     manifest. A job past its deadline is dead until process state proves otherwise. A
     stalled critical job may reshape the sprint.
   - Credentials/tooling alive ({{CRED_CHECKS}}).
3. **If reality contradicts the sprint doc: amend `SPRINT.md`, log the deviation in
   `STATUS.md`, THEN proceed.** Fiction in docs poisons the next session.

## Phase 1 — Clarify deltas, reconcile, plan

1. `SPRINT.md` § Locked decisions are settled. Ask only about what reality changed since
   drafting, plus Gate 0 items, in one batched round of genuine operator decisions.
   Re-ask a locked decision only when reality contradicts it.
2. **Reconcile & re-present.** If answers shift scope, amend `SPRINT.md` and log it in
   `STATUS.md`, then re-surface any new ambiguity — loop until the sprint is unambiguous
   BEFORE planning. A track must never inherit a question you could have closed here.
3. Present Gate 0 items from `SPRINT.md`. Don't spawn tracks that depend on unmet gates.
4. Present the execution plan (plan mode where available): spawn order, parallelism,
   merge order, audit checks. On approval, set `STATUS.md` to `in flight` and execute.

## Phase 2 — Spawn tracks

**In an arc (SPRINT.md has `Arc:`):** cut from and merge into `aNN/integration`, never
`{{TRUNK}}`; no promotion, no next-sprint draft.

- **Cut the integration branch first:** `git checkout -b sNN/integration origin/{{TRUNK}}`
  then `git push -u origin sNN/integration`. Once per sprint; on respawn, fetch the
  existing branch and never re-cut it. Every track cuts from `origin/sNN/integration`.
  Run the full suite ({{FULL_SUITE}}) once on this starting tree and record it as the
  baseline row of `STATUS.md`'s tree ledger.
- One subagent per track. The spawn prompt tells it to check out its branch first, then
  gives the absolute paths of its `TRACK-*.md` and `TRACK_RULES.md`. Model tier per the
  sprint file's per-track row, resolved through the map at spawn; a row without a tier
  is a drafting gap — assign one per the model policy and amend `SPRINT.md` before
  spawning.
- Record each track's expected duration in `STATUS.md`. When it passes, check the
  track's branch and worktree yourself; never wait on a notification that may not come.
- Same-repo parallel tracks: worktree isolation; each pushes its own `sNN/<slug>` branch.
- A track that depends on another track's work spawns after that work is merged into
  `sNN/integration` and the tip is green. It cuts from the new tip. The trunk does not
  move for this.
- Independent tracks spawn in ONE message. While they run: handle gate items; never
  duplicate track work.
- **Relay.** A track that returns a relay brief: verify its pushed `wip` state, then
  spawn a continuation on the same tier with its `TRACK-*.md` and the relay brief.

## Phase 3 — Audit & merge into `sNN/integration` (in SPRINT.md's merge order)

1. Fetch the branch; review the FULL diff against `DESIGN.md`: contract conformance, no
   secrets, no scope creep. Diff from the MERGE-BASE
   (`git diff $(git merge-base sNN/integration <branch>) <branch>`) — a tip-vs-moved-base
   diff shows a clean track "deleting" files it never touched. Grep the diff for
   sprint-narration comments
   (`grep -nE '\bs[0-9]+\b|Track [A-Z]|sprint-[0-9]'` over added lines) — rewrite to
   present-tense constraints or delete before merge (`TRACK_RULES.md` § Comments).
2. **Spot-check the track's claims** against real state: re-run its targeted gates plus
   2–3 checks you choose. Reports are claims; your audit produces facts.
3. **UI work:** run ONE fresh-eyes deslop audit per UI unit of work (a surface built or
   redesigned), not per fix round. Without `/deslop`, audit against the generic-AI-look
   failure mode explicitly.
4. {{MIGRATION_APPLY_STEP}}
   <!-- init: if DB — apply migration files in order after audit, verify schema + run
   verification queries. State WHEN: at the integration merge when the deployed surface
   tolerates a schema ahead of its code, otherwise at promotion. If no DB, drop this step. -->
5. Resolve conflicts yourself. `git checkout sNN/integration`, confirm with
   `git branch --show-current`, merge `--no-ff`, push integration. Run the full suite on
   the integration tip once per distinct tree (`git rev-parse "HEAD^{tree}"`) and record
   it in the tree ledger; judge failures against the baseline row. Several audited
   merges may share one run; a red run is bisected by merge. Update the `STATUS.md`
   header. `{{TRUNK}}` does not move in this phase.
6. Long-running jobs a track delivered: verify the sample chunk end-to-end, then start the
   full run detached and record it in the STATE index jobs table with its expected
   duration and deadline. In an arc, record it in your `STATUS.md` header instead; the
   arc checkpoint copies it into the STATE index.

## Phase 4 — Close

**In an arc (SPRINT.md has `Arc:`):** cut from and merge into `aNN/integration`, never
`{{TRUNK}}`; no promotion, no next-sprint draft. Write only your own sprint folder: the
arc checkpoint does the STATE index, backlog, branch, and archive steps.

1. **Promote `sNN/integration` into `{{TRUNK}}` — on the operator's go, never before.**
   Set `STATUS.md` to `closing`. `git fetch origin`. If `origin/{{TRUNK}}` moved since the
   cut, merge it INTO `sNN/integration`, resolve conflicts, and run the full suite on the
   new tree. Then `git checkout {{TRUNK}}`, confirm `git branch --show-current`,
   `git pull --ff-only`, `git merge --no-ff sNN/integration`, and push ONCE. The trunk's
   tree now equals a green tree ledger row, so it needs no rerun. Verify the push landed
   (`git ls-remote --heads origin {{TRUNK}}` matches the local SHA) and that exactly one
   pipeline run started. An early promotion the operator requested mid-sprint follows the
   same steps; later tracks keep landing on integration and the close gate promotes again.
2. Walk the operator through the rest of the close gate as an actionable checklist
   (exact commands): deploys, end-to-end checks, detached jobs.
3. **Scoped auto-clean.** Run these unprompted, limited to what this sprint owns:
   - Write `CLOSE.md`. Add its one-line row to the STATE index history. Set `STATUS.md`
     to `done`. Learnings go into `CLOSE.md`; each rule they yield goes, as 1–3 lines, to
     `DESIGN.md`, a guidelines file, or the private scars file under a tagged section.
     Never edit the sprint skill's `SCARS.md`.
   - Delete the `BACKLOG.md` rows this sprint shipped. Add deferred and discovered work
     as new rows.
   - Remove this sprint's track worktrees: `git -C <worktree> status` first —
     uncommitted work stops cleanup for that tree — then `git worktree prune`.
   - The previous sprint is the previous closed sprint in this sprint's own lineage
     (the sprint this one was drafted from). Delete only its merged `sNN/*` branches
     (its tracks AND its integration branch), local and remote: verify with
     `git branch --merged {{TRUNK}}` locally and against `origin/{{TRUNK}}` before
     `git push origin --delete <branch>`; a branch with an open PR is not a deletion
     candidate. This sprint's branches survive
     one more sprint as recovery points, locally and on origin.
   - Archive the previous sprint's `done` folder into `sprints/archive/`.
4. **Never touch** any other sprint's folder or branches, another developer's branches,
   an unmerged branch, a live `STATUS.md`, or the registry. List each as a finding in
   `CLOSE.md` and the close report: "run /sprint clean to resolve N items".
5. Draft the NEXT sprint's folder from what actually happened (ROADMAP is the skeleton;
   BACKLOG.md is the feed; reality wins): `SPRINT.md` with its Locked decisions and
   `Director tier:` line (strong minimum; frontier only per the model policy's
   hard-problem criteria), `STATUS.md` at `drafted`, and the `TRACK-*.md` files, each
   sized to one surface and one contract.
6. Final report: outcomes, merge summary, promotion state, gate status, background-job
   dashboard, clean findings, next-sprint pointer + kickoff instructions, naming the
   model tier and the model it resolves to for the next director.

## Respawn protocol

Resuming a half-finished sprint: `STATUS.md` + `git branch -a` + worktree list + job
manifests tell you where it stopped. `origin/sNN/integration` is the durable audited
state: what is merged into it passed audit; re-audit every track branch that is not,
rather than trusting prior-session memory. A tree ledger row stays valid: never re-run a
tree recorded green. Check whether promotion happened with
`git merge-base --is-ancestor sNN/integration origin/{{TRUNK}}`.

## Hard rules

- All `SPRINT_GUIDELINES.md` git rules. Verify "pushed"/"loaded"/"running" claims against
  git/store/process state itself.
- **NEVER run mutating git operations (switch / branch -D / merge / reset) in a working tree
  while an agent is live in it.** Kill and completion notifications are UNRELIABLE — an agent
  can resume after one — and transcript files flush lazily, so a healthy agent can look
  frozen. Judge liveness by GIT STATE (new commits, working-tree changes). While an agent is
  live, run read-only git only.
- **A killed track may hold substantial UNCOMMITTED work.** Check `git -C <worktree> status`
  before touching anything, and never clean up a worktree on the strength of a failure
  notification alone.
- **A completed agent can SELF-RESUME and act un-instructed** — including merging and pushing
  past an operator gate. On every completion notification, verify no unauthorized pushes
  landed before building on the reported state. Surface a bypassed gate to the operator
  IMMEDIATELY with a verified damage envelope; ratify-or-revert is their call.
- **Drive the actual deliverable end-to-end before closing** — the thing a user opens, not the
  procedures behind it. A track's own gates cannot see the seam BETWEEN tracks: every gate can
  pass while the product is broken. For anything customer-facing, drive the DEPLOYED origin.
- **Prove the mechanism before shipping a fix.** A plausible cause that survives a code read is
  still a hypothesis; instrument or reproduce first. And after fixing a proven cause, RE-RUN
  the original failing observation — one symptom can carry two stacked causes.
- Long jobs: detached + manifest + deadline; never a foreground job that dies with your
  session. Check actual process liveness, not just manifest freshness. (`pgrep -f <name>`
  inside a compound command MATCHES ITS OWN command line — a finished job reads as live;
  exclude the probe or match an exact module path.)
- Never merge red; never spawn tracks on a red `sNN/integration` or a red `{{TRUNK}}`;
  never apply un-reviewed SQL/DDL; never let a track apply DDL.
- Never push `{{TRUNK}}` outside a promotion the operator approved; never force-push
  `{{TRUNK}}` or `sNN/integration`.
- {{PROD_SAFETY_RULES}}
  <!-- init: project-specific production-safety rules (shared live DBs, rate limits,
  deploy freezes), from interview. -->
- Keep token use lean: direct, audit, merge — don't rebuild. Never append narrative to
  the STATE index or the guidelines.
