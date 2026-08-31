# Sprint NN — {{SPRINT_TITLE}}

**Goal:** {{ONE_PARAGRAPH — what this sprint makes true that isn't true today.}}

**Definition of done:** {{CONCRETE, VERIFIABLE end-state — phrased so a director can test
it, not vibe it.}}

**Director tier:** {{model — one-line why}}
<!-- init: default opus. Fable when ANY hold: 3+ parallel tracks on shared surfaces;
architecture/contract decisions expected mid-sprint rather than pre-drafted; prior
sprint's audit caught seam defects its gates missed. The kickoff must name this model —
the operator launches the session. -->

## Tracks

| Track | Slug/branch | Model | Why this tier | Scope |
|---|---|---|---|---|
| A | `sNN/{{slug}}` | {{tier}} | {{work class → tier, one line}} | {{one-line scope}} |
<!-- init: 1-4 rows. Tier per the model policy in SPRINT_GUIDELINES.md: fable for
contract-defining / architecture / gnarly-debug work, sonnet for standard
implementation, haiku only for mechanical fully-specified chores (justify it in the
rationale column). Note any shared-scaffolding races between tracks and how the
director pre-empts them (pre-create shared skeletons before spawning, or stagger spawns). -->

## Gate 0 (before tracks spawn)

1. {{CHECK — credentials fresh, main green, store reachable, disk/quota, approvals.}}

## Mid-sprint gates

- {{GATE — external signups, reviews, anything human that tracks must not block on.
  Structure track scope so unmet mid-gates degrade gracefully (ship tested module +
  documented gap, not a stalled track).}}

## Merge order & rationale

{{ORDER + one line of why (who owns shared scaffolding/migrations, who deploys last).}}

## Close gate (operator + director together)

1. {{STEP — deploys the operator runs, end-to-end verifications with expected outputs,
   detached jobs to start + record in STATE.md.}}
2. **Sprint git cleanup (director):** every merge verified landed → remove this sprint's
   track worktrees (`git -C <worktree> status` clean first), `git worktree prune`, delete
   the PREVIOUS sprint's merged track branches. This sprint's branches survive until the
   next close; never delete an unmerged branch.

## Risks specific to this sprint

- {{RISK + the mitigation already designed into a track's instructions.}}
