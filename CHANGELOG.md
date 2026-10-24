# Changelog

Newest first. One line per change.

## 3.0.0

Faster sprints, fewer tokens, clean state.

### Sprint engine

- Each sprint gets its own `STATUS.md` (live header) and `CLOSE.md` (written once at close).
- `STATE.md` is now a thin index. Sprint narrative moves to each sprint's `CLOSE.md`.
- Directors boot in a fixed order and read only the scar sections tagged for the project's shape.
- The clarify round asks only about what changed since drafting. Decisions live in `## Locked decisions` in `SPRINT.md`.
- Verification runs in tiers. Tracks run targeted checks. The director runs the full suite once per distinct tree, recorded in a tree ledger.
- Audits spot-check instead of re-running everything.
- Every background job and poll loop carries a hard deadline.
- New `TRACK_RULES.md` template holds the standing rules for track agents.
- Tracks hand off to a continuation (relay) when their context gets heavy.
- Drafters size each track to one focused context.
- Scouts and mechanical audit checks run on the fast tier. Directors and drafters run on strong or above.
- UI tracks get the `STYLE.md` core in the brief. The director runs one fresh-eyes deslop audit per UI unit of work.
- Close runs a scoped auto-clean of only what the sprint owns. Everything else goes to a findings list.
- Guidelines and scars hold short rules, not incident stories.
- `/sprint clean` gains an index split and a prune pass for guidelines and scars.
- The backlog is one row per item. Shipped rows are deleted.
- `SCARS.md` gets a baseline-once rule, root-anchored ignores for harness folders, and a rule that worktree tracks cannot write the shared ops folder.
- `/sprint status` reads the `STATUS.md` headers.
- Review fixes: the verification budget uses portable timeouts, a retrying slot queue, and a quoted tree ref. Skill descriptions and docs match the 3.0 behavior.

### Intake skills

- `/build` gains a Discover stage for vague ideas. It cycles `/analyze` and `/pry`, then writes `BLUEPRINT.md`.
- `/build` hands locked decisions to `SPRINT.md` or `/plan`. `/sprint init`, `/sprint arc`, and `/plan` read `BLUEPRINT.md`.
- `/analyze` infers its sources (code, web, literature) and cross-checks load-bearing claims.
- `/pry` pitches an idea only if it would recommend "yes, now". Zero ideas is valid.
- `/plan` points at the shared Verification budget and follows the deslop bookends for UI work.
- `/autonomous` sets a wall-clock budget per stage and lists every deadline that fired in the wake-up report.

### UI, status, explain

- `/deslop` runs at the start of new UI surfaces and once at the end of UI work. Tweaks to a styled surface invoke nothing.
- `STYLE.md` has a short core plus a reference part. Audit results go in the report and commit message, not in `STYLE.md`.
- `/whereami` reports in plain language and can cover every recent session across projects.
- New `/explain` skill explains the last answer, a commit, a file, or a concept in plain language. It is read-only.

### Distribution

- Every skill carries a shared spawn-tier rule.
- `/sprint`, `/plan`, and `/autonomous` share one Verification budget section, including an opt-in `~/.config/10x-skills/machine.json` cap on heavy commands.
- Every skill runs a once-a-day update check at the end of a run. Opt out with `~/.config/10x-skills/.no-update-check`.
- Add `VERSION` and this changelog. All plugin versions move to 3.0.0.

## 2.9.0

Previous release: harness-neutral skills, portable model tiers, and `/build`.
