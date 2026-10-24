# STATE index

**Last updated:** {{DATE}} (init — system scaffolded)
**Active arc:** none
<!-- init: keep "none". `/sprint arc` sets this line to `aNN → arcs/arc-NN/ARC.md`. -->

**Rule: never append sprint narrative here.** Narrative goes to the sprint's
`CLOSE.md`; in-flight state goes to the sprint's `STATUS.md`. This file holds only the
facts, decisions, jobs, and one-line history rows below. The in-flight view comes from
the header of each `sprints/*/STATUS.md`.

## Background jobs

<!-- init: keep the empty table if the project has no long-running jobs; directors check
it every boot. Every row carries an expected duration and a hard deadline. A job past its
deadline is dead until process state proves otherwise. -->
| Job | Sprint | Status | Manifest / where to check | Started | Expected duration | Deadline |
|---|---|---|---|---|---|---|
| (none) | | | | | | |

## Verified facts

<!-- init: facts established during setup that a fresh director must not re-derive:
store locations, credential mechanisms, known data states, tooling quirks. One line each,
each with its verification date. -->
- {{FACT}} (verified {{DATE}})

## Decisions log

| Date | Decision | Where recorded |
|---|---|---|
| {{DATE}} | Standing decisions from the init interview | `DESIGN.md` § Standing decisions |
<!-- init: add one row per provisional decision (answers that were "not sure yet"). -->

## Sprint history

One line per closed sprint. Each sprint's full record is its `CLOSE.md`, in
`sprints/sprint-NN/` or, once archived, `sprints/archive/sprint-NN/`.

| Sprint | Closed | Outcome (one line) |
|---|---|---|
| (none run yet) | | |
