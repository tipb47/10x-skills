---
name: plan
description: Adaptive mini-sprint planning. Interview to zero ambiguity, then a track-shaped plan sized to the work — single-track (implement in-session) or multi-track (isolated subagents on worktrees with per-track model tiers, audited and merged in order). Accepts a described task or a requirements artifact (PRD, ticket, file, URL). Use for /plan <task|artifact> [complexity], or when the user asks to plan an implementation.
argument-hint: <task or artifact> [complexity] [context]
---

# plan — adaptive mini-sprint

A FRESH planning session: disregard prior plans, todos, and implementation discussion
from this conversation. Where the runtime has a plan mode, enter it before analysis and
stay in it until the plan is approved.

The output is a plan sized to the work — and, on approval, its execution. No durable
plan files: track briefs travel inline in subagent prompts. Durable multi-session state
is `/sprint`'s job, not this skill's.

## Parameters

- **Task or artifact** ($1): a described task, or a requirements artifact — file path,
  URL, ticket reference, pasted PRD text.
- **Complexity** ($2, optional): calibrates analysis depth and track granularity.
- **Context** ($3): anything extra.

## Phase 0 — Intake (only when $1 is an artifact)

If $1 is a file, URL, ticket, or document rather than a described task: fetch/read it,
parse it into functional requirements, constraints, acceptance criteria, and implicit
requirements, then run a gap audit:

- **Blockers** — ambiguity that prevents planning. Blockers STOP the session: put them
  to the user (recommended resolution first) and re-audit on answers.
- **Clarifications** — could cause rework; ask alongside the Phase 2 interview.
- **Assumptions** — reasonable inferences, stated explicitly for veto.

## Phase 1 — Analyze the codebase

Search and read what the task touches: affected files, architecture, existing patterns
to follow, dependencies. Calibrate to complexity. For broad sweeps, use read-only
explore scouts to keep this context clean (`/analyze` doctrine: parallel, self-contained
prompts, sonnet default, haiku only for mechanical enumeration). If a fresh `/analyze`
report for this target exists in the conversation, build on it instead of re-deriving.

## Phase 2 — Interview to zero ambiguity

The load-bearing phase. Interview discipline:

- Look up facts yourself; put only genuine decisions to the user (structured question
  tool where the runtime has one).
- Every question leads with your recommended answer. Batch independent questions;
  sequence dependent ones — never make the user answer what a sibling answer could
  invalidate.
- Probe what the user did NOT put on the table: failure modes, edge cases, boundary
  behavior, contradictions between answers.
- If the user's approach is suboptimal, say so and propose the better alternative.
- Loop until nothing material is unresolved. The plan must never inherit a question
  you could have closed here.

## Phase 3 — Shape the plan

Size the plan to the work — this is a drafting judgment, stated explicitly:

**Graduate out.** Work spanning multiple sessions, needing a durable ledger, or
touching many surfaces over weeks → recommend `/sprint init` and stop. A plan that
cannot finish in one session is a sprint wearing a costume.

**Single-track** — one coherent unit, no parallelism to win: a numbered step plan —
files to change (paths), the change per file, order of operations, error handling,
edge cases, verification commands. The main session implements it after approval.

**Multi-track** — separable units that benefit from isolation or parallelism: a
mini-sprint. The plan states:

| Element | Content |
|---|---|
| Tracks | 2–4 rows: scope, branch `plan/<slug>`, model tier + one-line rationale |
| Merge order | order + one line of why |
| Verification | per track: the commands/checks that prove it done |
| Gates | any human action (deploy, signup, approval) — never buried inside a track |

Model tiers per track: contract-defining / architecture / gnarly work → fable;
standard implementation → sonnet; mechanical fully-specified chores → haiku, only with
a written justification. Judgment (audit, merge, synthesis) stays in the main session.

## Phase 4 — Approve, then execute

Present the plan for approval (exit plan mode where the runtime has one). **Assume the
operator is watching:** surface material mid-execution decisions as they arise, and
stop at anything gate-shaped. Unattended execution is never assumed — that grant comes
only from the `/autonomous` skill's negotiated envelope. Then:

**Single-track:** implement in-session, step by step, verifying as you go.

**Multi-track:** the session turns director and does not build:
- One subagent per track, brief inline and self-contained (scope, branch, files, the
  verification commands, push authority, the no-stash and incremental-commit rules).
  Parallel same-repo tracks get worktree isolation; independent tracks spawn in one
  message.
- Audit each track's FULL diff from the merge base; re-run its verification yourself —
  reports are claims. With worktrees, always `git -C <absolute-path>`; stage explicit
  paths; never run mutating git in a tree while an agent is live in it.
- Merge in the stated order, `--no-ff`, suite green after each merge. Then remove the
  plan's worktrees (check `git -C <worktree> status` first) and delete merged
  `plan/<slug>` branches — a one-session plan keeps no branch archaeology.

## Phase 5 — Deliver

Report what shipped vs the plan, verification output (verbatim), a manual testing
guide, and a suggested commit message in the project's convention. Do not commit or
push unless asked.
