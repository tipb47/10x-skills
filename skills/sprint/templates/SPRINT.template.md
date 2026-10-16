# Sprint NN — {{SPRINT_TITLE}}

**Goal:** {{ONE_PARAGRAPH — what this sprint makes true that isn't true today.}}

**Definition of done:** {{CONCRETE, VERIFIABLE end-state — phrased so a director can test
it, not vibe it.}}

**Director tier:** {{model tier — one-line why}}
<!-- init: write a model tier (frontier / strong / fast / light), never a model name.
Default strong. Raise the director's effort when ANY hold: 3+ parallel tracks on shared
surfaces; contract decisions expected mid-sprint; prior sprint's audit caught seam
defects its gates missed. Frontier only when the director itself must solve a hard
problem. The kickoff must name this tier and the model it resolves to — the operator
launches the session. -->

<!-- init/arc: in an arc sprint only, add `**Arc:** aNN → arcs/arc-NN/ARC.md` here. Then
rewrite Gate 0 and the close gate: `sNN/integration` cuts from and merges into
`aNN/integration`, not `{{TRUNK}}`. The close gate has no trunk promotion; it ends with
CLOSE.md written and STATUS.md set to `done`. Outside an arc, drop this directive. -->

## Tracks

| Track | Slug/branch | Model tier | Why this tier | Scope |
|---|---|---|---|---|
| A | `sNN/{{slug}}` | {{tier}} | {{work class → tier, one line}} | {{one-line scope}} |
<!-- init: 1-4 rows. Model tier per the model policy in SPRINT_GUIDELINES.md: strong
for contract-defining / architecture / gnarly-debug work, fast for standard
implementation, light only for mechanical fully-specified chores (justify it in the
rationale column), frontier only for a genuinely hard problem. Write tier words, never
model names: the director resolves them at spawn. Note any shared-scaffolding races between tracks and how the
director pre-empts them (pre-create shared skeletons before spawning, or stagger spawns). -->

## Gate 0 (before tracks spawn)

1. {{CHECK — credentials fresh, `{{TRUNK}}` green, store reachable, disk/quota, approvals.}}
2. Director cuts `sNN/integration` from `origin/{{TRUNK}}` and pushes it, empty, before
   any track spawns.

## Mid-sprint gates

- {{GATE — external signups, reviews, anything human that tracks must not block on.
  Structure track scope so unmet mid-gates degrade gracefully (ship tested module +
  documented gap, not a stalled track).}}
- {{EARLY PROMOTION — only if the operator wants part of this sprint on `{{TRUNK}}` before
  close: name the tracks that must have landed on `sNN/integration` first. Otherwise
  state "none — promotion happens at the close gate".}}

## Merge order into `sNN/integration` & rationale

{{ORDER + one line of why (who owns shared scaffolding/migrations, who lands last).
Every track merges into `sNN/integration`; `{{TRUNK}}` moves only at promotion.}}

## Close gate (operator + director together)

1. **Promotion (operator says go → director executes):** merge `origin/{{TRUNK}}` into
   `sNN/integration` if the trunk moved, re-verify, then ONE `--no-ff` merge of
   `sNN/integration` into `{{TRUNK}}` and ONE push. Expected: exactly one pipeline run,
   {{TRUNK_PUSH_EFFECT}}.
2. {{STEP — deploys the operator runs, end-to-end verifications with expected outputs,
   detached jobs to start + record in STATE.md.}}
3. **Sprint git cleanup (director):** promotion verified landed on `origin/{{TRUNK}}` →
   remove this sprint's track worktrees (`git -C <worktree> status` clean first),
   `git worktree prune`, delete the PREVIOUS sprint's merged branches (tracks + its
   integration branch) — local and their `origin` counterparts. This sprint's branches
   survive until the next close, locally and on the remote; never delete an unmerged
   branch, anywhere.

## Risks specific to this sprint

- {{RISK + the mitigation already designed into a track's instructions.}}
