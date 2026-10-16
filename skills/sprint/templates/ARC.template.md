# Arc NN — {{ARC_TITLE}}

**Goal:** {{ONE_PARAGRAPH — the large outcome this arc makes true.}}

**Definition of done:** {{CONCRETE, VERIFIABLE end-state for the whole arc, tested on
`aNN/integration` before promotion.}}

**Arc branch:** `aNN/integration`, cut from `origin/{{TRUNK}}`. Every sprint in the arc
cuts from it and is merged into it. It promotes into `{{TRUNK}}` once, at the arc close
gate.

**Arc director tier:** {{model tier — one-line why}}
<!-- init/arc: a model tier, never a model name. Default strong; raise its effort when
cross-sprint seams are dense or contract decisions land at arc checkpoints. Frontier only
when the arc director itself must solve a hard problem. -->

**Ledger rule:** the arc director is the only writer of this file, `STATE.md`, and
`BACKLOG.md`. Sprint directors write only their own `sprints/sprint-NN/` folder.

## Non-goals

- {{NON-GOAL — neighboring work this arc must not pull in.}}

## Sprint dependency graph

<!-- init/arc: one row per sprint. "Waits on" names sprints that must be merged into
`aNN/integration` first. Two sprints run in parallel only if neither waits on the other
AND their owned surfaces are disjoint. Sprint numbers are global `sNN`. -->

| Sprint | Title | Waits on | Runs in parallel with |
|---|---|---|---|
| sNN | {{TITLE}} | {{— or sNN}} | {{sNN, ... or —}} |

### Outlines for waiting sprints

<!-- init/arc: sprints unblocked at the start get full `sprints/sprint-NN/` folders
instead. The arc checkpoint that unblocks a waiting sprint drafts its folder from this
outline and from what has merged so far. -->
- **sNN — {{TITLE}}:** {{goal + likely tracks + what it waits on}}

## Surface ownership

<!-- init/arc: the files, packages, or services each sprint owns. A sprint never edits
another sprint's surface. Shared files get exactly one owner; others request changes
through the arc checkpoint. -->

| Sprint | Owns | Must not touch |
|---|---|---|
| sNN | {{PATHS / PACKAGES / SERVICES}} | {{OTHER SPRINTS' SURFACES}} |

## Shared-environment plan

<!-- init/arc: every shared environment (staging, shared DB, queues). Only the arc director
deploys to them, at an operator gate, from the arc tree after merging. -->

| Environment | Deployed after | How | Gate |
|---|---|---|---|
| {{ENV}} | {{sNN merged}} | {{COMMAND / PIPELINE}} | {{operator approval}} |

## Operator gates

1. {{GATE — shared-environment deploys, approvals, external signups.}}
2. **Arc close:** operator says go → merge `origin/{{TRUNK}}` into `aNN/integration` if
   the trunk moved, re-verify, then ONE `--no-ff` merge into `{{TRUNK}}` and ONE push.
   Expected: exactly one pipeline run, {{TRUNK_PUSH_EFFECT}}.

## Status

<!-- init/arc: the arc director updates this table at every arc checkpoint. Status values:
waiting / unblocked / in flight / done / merged. -->

| Sprint | Status | `sNN/integration` | Notes |
|---|---|---|---|
| sNN | {{STATUS}} | {{pushed / merged into aNN/integration}} | {{deploy needs, blockers}} |

## Arc checkpoint log

| Date | What the arc checkpoint did |
|---|---|
| {{DATE}} | Arc drafted; `aNN/integration` cut and pushed |
