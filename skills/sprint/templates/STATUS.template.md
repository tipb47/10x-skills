# Sprint NN — STATUS

**Rule: only this sprint's director writes this file.** Keep the header block current.
`/sprint status` and other directors read only the lines above the first `---`.

**Phase:** drafted
<!-- init/director: drafted → in flight (Gate 0 passed) → closing (close gate started) → done
(CLOSE.md written; in an arc, ready for the arc checkpoint). A STATUS.md is live until its
phase is `done`. -->
**Updated:** {{DATE}}
**Tracks landed on `sNN/integration`:** none of {{TRACK_LETTERS}}
**Promoted:** no
<!-- init/director: in an arc sprint, write "n/a — the arc director merges into aNN/integration". -->
**Blockers:** none
**Deploy needs:** none

---

## Tree ledger

One row per full-suite run, per § Verification budget. A tree recorded green is verified;
never re-run it.

| Tree (`git rev-parse HEAD^{tree}`) | Branch @ commit | Full suite | Failures vs baseline | Date |
|---|---|---|---|---|
| | | | | |

## Tracks

| Track | Branch | Expected duration | Audit | Merged into `sNN/integration` |
|---|---|---|---|---|
| A | `sNN/{{slug}}` | | | |

## Deviations

<!-- init/director: one line per change to SPRINT.md since drafting, with its reason. -->
- none
