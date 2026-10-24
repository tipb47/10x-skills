# Sprint NN — {{SPRINT_TITLE}}

**Goal:** {{ONE_PARAGRAPH — what this sprint makes true that isn't true today.}}

**Definition of done:** {{CONCRETE, VERIFIABLE end-state — phrased so a director can test
it, not vibe it.}}

**Director tier:** {{model tier — one-line why}}
<!-- init: write a model tier (frontier / strong / fast / light), never a model name.
Strong is the minimum; effort scales with size. Raise the director's effort when ANY
hold: 3+ parallel tracks on shared surfaces; contract decisions expected mid-sprint;
prior sprint's audit caught seam defects its gates missed. Frontier only when the
director itself must solve a hard problem. The kickoff must name this tier and the model it resolves to — the operator
launches the session. -->

<!-- init/arc: in an arc sprint only, add `**Arc:** aNN → arcs/arc-NN/ARC.md` here. Then
rewrite Gate 0 and the close gate: `sNN/integration` cuts from and merges into
`aNN/integration`, not `{{TRUNK}}`. The close gate has no trunk promotion. Outside an arc,
drop this directive. -->

**Status:** `STATUS.md` in this folder. **Close record:** `CLOSE.md`, written at close.

## Locked decisions

<!-- init/director: every decision settled while drafting (by the drafting interview, by
`/build`, or by the closing director), one line each with its why. The next director asks
only about what reality changed since drafting, and re-asks a locked decision only when
reality contradicts it. -->
- {{DECISION — why}}

## Tracks

| Track | Slug/branch | Model tier | Why this tier | Scope |
|---|---|---|---|---|
| A | `sNN/{{slug}}` | {{tier}} | {{work class → tier, one line}} | {{one-line scope}} |
<!-- init: 1-4 rows. Model tier per the model policy in SPRINT_GUIDELINES.md: strong
for contract-defining / architecture / gnarly-debug work, fast for standard
implementation, light only for mechanical fully-specified chores (justify it in the
rationale column), frontier only for a genuinely hard problem. Write tier words, never
model names: the director resolves them at spawn. Size each track to one focused
context: one surface, one contract. Note any shared-scaffolding races between tracks and
how the director pre-empts them (pre-create shared skeletons before spawning, or stagger
spawns). -->

## Gate 0 (before tracks spawn)

1. {{CHECK — credentials fresh, `{{TRUNK}}` green, store reachable, disk/quota, approvals.}}
2. Director cuts `sNN/integration` from `origin/{{TRUNK}}` and pushes it, empty, before
   any track spawns.
3. Director runs the full suite once on that starting tree and records it as the
   baseline row of STATUS.md's tree ledger.

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
   `sNN/integration` if the trunk moved, and run the full suite only if that changed
   the tree. Then ONE `--no-ff` merge of `sNN/integration` into `{{TRUNK}}` and ONE push.
   Expected: exactly one pipeline run, {{TRUNK_PUSH_EFFECT}}.
2. {{STEP — deploys the operator runs, end-to-end verifications with expected outputs,
   detached jobs to start, each recorded in the STATE index jobs table with its deadline;
   in an arc, in this sprint's STATUS.md header, which the arc checkpoint copies into the
   STATE index.}}
3. **Scoped auto-clean (director):** the close steps of `DIRECTOR_GUIDELINES.md` Phase 4.
   Anything this sprint does not own goes to the close report's findings list.

## Risks specific to this sprint

- {{RISK + the mitigation already designed into a track's instructions.}}
