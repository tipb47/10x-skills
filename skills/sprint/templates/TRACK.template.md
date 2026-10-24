# Track {{LETTER}} — {{TRACK_TITLE}}

Branch `sNN/{{slug}}`, cut from `origin/sNN/integration`. Push only that branch. Obey
`{{OPS_PATH}}/TRACK_RULES.md`: check out your branch first, then read this file and the
rules once. Read `DESIGN.md` ({{relevant sections}}).
{{ONE PARAGRAPH: what you build and why it matters to the sprint.}}
<!-- init/director: size the track to one focused context: one surface, one contract. A
track that needs two surfaces or two contracts is two tracks. -->

## What exists (verified — do not re-derive)

<!-- init/director: every fact the track needs that's already established — endpoints,
schemas, file IDs, repo patterns to follow, with "verified <date>" stamps. The test of a
good track file: a fresh subagent with NO other context executes end-to-end without
asking anything. For a UI track, paste the `STYLE.md` core plus the 2–3 reference
excerpts this surface needs; the track loads no UI skill. -->
- {{FACT}}

## Build

1. {{STEP — concrete: files/modules to create, behaviors, edge cases to handle.}}
<!-- init/director: include the project's conventions inline where they bite (units,
timezones, batch sizes, naming) rather than assuming the agent internalized DESIGN.md. -->

## Acceptance gates (quote actual outputs in your report)

1. {{GATE — command/query + EXPECTED result. The director spot-checks these at audit;
   mismatches between your quoted output and the director's re-run fail the track.}}

## Relay

**When your context runs heavy, relay.** Heavy means a compaction happened or the
harness reports high context use. Commit and push `wip`, then return a relay brief
(`TRACK_RULES.md` § Report) as your final message. The director spawns a continuation
on the same tier.

## Out of scope

{{EXPLICIT non-goals — neighboring work this track must not touch, future-sprint items
it must not pull forward, other tracks' files it must not edit.}}
