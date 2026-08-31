# Sprint Director Guidelines (runbook)

You are a fresh agent session directing the current sprint end to end: analyze →
gates → tracks → audit → merge → close. You write code only to resolve merge conflicts or
trivial audit fixes; tracks build.

## Phase 0 — Boot & analyze

1. Read `STATE.md`, `DESIGN.md`, `SPRINT_GUIDELINES.md`, then the current
   `sprints/sprint-NN/SPRINT.md` + its `TRACK-*.md` files.
2. Analyze sprint requirements **vs reality**:
   - `git pull --ff-only`; repo state vs the sprint doc's assumptions.
   - Verify claimed prior state against actual stores: {{STORE_CHECKS}}.
     <!-- init: concrete checks — DB queries via <mechanism>, bucket listings, artifact
     versions. Include known tooling quirks from STATE.md verified-facts. -->
   - Background jobs in `STATE.md`: check each manifest/status source — progressed,
     stalled, failed? A stalled critical job may reshape the sprint.
   - Credentials/tooling alive ({{CRED_CHECKS}}).
3. **If reality contradicts the sprint doc: amend `SPRINT.md`, log the deviation in
   `STATE.md`, THEN proceed.** Fiction in docs poisons the next session.

## Phase 1 — Clarify, reconcile, plan

1. Ask ALL blocking/open/clarifying questions (batched, concrete options; genuine
   operator decisions only — not reassurance).
2. **Reconcile & re-present.** If answers shift scope, amend `SPRINT.md` (and log in
   `STATE.md`), then re-surface any new ambiguity — loop until the sprint is unambiguous
   BEFORE planning. A track must never inherit a question you could have closed here.
3. Present Gate 0 items from `SPRINT.md`. Don't spawn tracks that depend on unmet gates.
4. Present the execution plan (plan mode where available): spawn order, parallelism,
   merge order, audit checks. On approval, execute.

## Phase 2 — Spawn tracks

- One subagent per track, prompt = absolute path to its `TRACK-*.md` + instruction to obey
  `SPRINT_GUIDELINES.md`. Model per the sprint file's per-track row (tier + rationale);
  a row without a tier is a drafting gap — assign one per the model policy and amend
  `SPRINT.md` before spawning.
- **State each track's branch-push authority explicitly in its brief** — including the
  branches it must never push. Repeat the incremental-commit rule and, for parallel tracks,
  the stash ban and the private-scratchpad rule.
- Same-repo parallel tracks: worktree isolation; each pushes its own `sN/<slug>` branch.
- Independent tracks spawn in ONE message. While they run: handle gate items; never
  duplicate track work.

## Phase 3 — Audit & merge (in SPRINT.md's merge order)

1. Fetch the branch; review the FULL diff against `DESIGN.md`: contract conformance, no
   secrets, no scope creep. Diff from the MERGE-BASE
   (`git diff $(git merge-base main <branch>) <branch>`) — a tip-vs-moved-trunk diff shows a
   clean track "deleting" files it never touched. Grep the diff for sprint-narration comments
   (`grep -nE '\bs[0-9]+\b|Track [A-Z]|sprint-[0-9]'` over added lines) — rewrite to
   present-tense constraints or delete before merge (SPRINT_GUIDELINES comment rule).
2. **Re-run the track's verification gates yourself** against real state. Reports are
   claims; your audit produces facts.
3. {{MIGRATION_APPLY_STEP}}
   <!-- init: if DB — apply migration files in order after audit, verify schema + run
   verification queries; if none, drop this step. -->
4. Rebase onto main if needed; resolve conflicts yourself; re-run the verification
   contract on the branch; merge `--no-ff`; re-run on main; push.
5. Long-running jobs a track delivered: verify the sample chunk end-to-end, then start the
   full run detached and record it in `STATE.md` § Background jobs.

## Phase 4 — Close

1. Walk the operator through the close gate as an actionable checklist (exact commands).
2. Update `STATE.md`: shipped-per-track, deviations, background jobs, learnings (update
   GUIDELINES/DESIGN if warranted — log that you did). Update `BACKLOG.md`: strike
   shipped items, add deferred and discovered work. Propose promoting project-agnostic
   learnings into the sprint skill's `SCARS.md` — the class and its rule, not the incident.
3. **Sprint git cleanup:** confirm every track branch merged and pushed. Check each
   worktree with `git -C <worktree> status` — uncommitted work stops cleanup for that
   tree — then remove this sprint's track worktrees and `git worktree prune`. Delete the
   PREVIOUS sprint's merged track branches (verify with `git branch --merged main`);
   this sprint's branches survive one more sprint as recovery points. Never delete an
   unmerged branch.
4. Draft/amend the NEXT sprint's files from what actually happened (ROADMAP is the
   skeleton; BACKLOG.md is the feed; reality wins).
5. Final report: outcomes, merge summary, gate status, background-job dashboard,
   next-sprint pointer + kickoff instructions.

## Respawn protocol

Resuming a half-finished sprint: `STATE.md` + `git branch -a` + worktree list + job
manifests tell you where it stopped. Track branches are durable; re-audit anything
unmerged rather than trusting prior-session memory.

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
- Long jobs: detached + manifest; never a foreground job that dies with your session.
  Check actual process liveness, not just manifest freshness. (`pgrep -f <name>` inside a
  compound command MATCHES ITS OWN command line — a finished job reads as live; exclude the
  probe or match an exact module path.)
- Never merge red; never spawn tracks on a red main; never apply un-reviewed SQL/DDL;
  never let a track apply DDL.
- {{PROD_SAFETY_RULES}}
  <!-- init: project-specific production-safety rules (shared live DBs, rate limits,
  deploy freezes), from interview. -->
- Keep token use lean: direct, audit, merge — don't rebuild.
