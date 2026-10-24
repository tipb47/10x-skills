---
name: build
description: Router for new functionality — analyze once, pry once, classify the work by scale (plan, sprint, or arc), confirm it with the user, then hand off to /plan or /sprint with the locked decisions so the engine skips its own interview. Use for /build [task], and PROACTIVELY whenever the user describes new functionality to build, even when no skill is named. Skip bugs, one-liners, and requests that already name /plan or /sprint.
argument-hint: [task — a feature or product capability; inferred from context when absent]
---

# build — one intake, the right engine

A thin router on top of the existing engines. It analyzes and interviews ONCE, picks the
engine that fits the work, and hands off. It never writes feature code before the handoff.

## When to fire

- **Explicitly:** `/build [task]`. No task → build whatever the session was just
  discussing.
- **Proactively:** the user describes new functionality to build — a feature, a product
  capability, a new surface — even without naming a skill.
- **Skip:** bug fixes, one-liners, fully specified small edits, and requests that already
  name `/plan` or `/sprint`. Those go straight to the work or the named skill.

## Flow

1. **Analyze once.** Read what the task touches: affected surfaces, existing patterns,
   dependencies, the project's ops instance (if any). Where `/analyze` is installed,
   follow its doctrine; calibrate depth to the task's blast radius.
2. **Pry once.** Where `/pry` is installed, run the interview as a pry interview,
   including its ideas pass. Without it, the inline discipline stands:
   - Look up facts yourself; put only decisions to the user.
   - Every question leads with your recommended answer, labeled `(Recommended)`.
   - Batch independent questions; sequence dependent ones.
   - Loop until nothing material is open.
3. **Classify.** Pick the scale from the table below. State it with a one-line reason,
   then ask the user to confirm (structured question, recommended scale first).
4. **Hand off.** Invoke the engine with the locked decisions — the task, the decisions
   and their whys, the analysis findings. The engine skips its own interview. Never
   re-interview the user on a decision already locked.

## Scales

| Scale | Fits when | Handoff |
|---|---|---|
| **plan** | The work finishes in one session, single- or multi-track. | `/plan` with the locked decisions; plan skips Phase 2. |
| **sprint** | The work needs multiple sessions or a durable ledger, and its sprints run sequentially. | No ops instance → `/sprint init`, its interview reusing the locked decisions. Ops instance exists → draft the next sprint(s) into it. |
| **arc** | The work needs 2+ sprints AND at least two can run in parallel on separable surfaces. | `/sprint arc` drafting. |

- **Pick the smallest scale that fits.** A one-session task is a plan, even when it has
  several tracks.
- **Sequential multi-sprint work is the sprint scale**, not an arc. It becomes a normal
  sprint roadmap.
- **An arc is for parallel sprints on separable surfaces** — typically brand-new core
  product functionality, where two sprints own disjoint packages or services.

## After the handoff

- **plan:** the plan runs in this session, per `/plan`.
- **sprint:** the operator starts a FRESH session and runs `/sprint direct`. Print the
  kickoff pointer and the model to launch on: SPRINT.md's `Director tier:` line,
  resolved through `~/.config/10x-skills/models.json`.
- **arc:** `/sprint arc` drafts the arc and prints the first kickoff prompts. The
  operator re-runs `/sprint arc` in a fresh session whenever a sprint in the arc finishes.

## Shared state

Per-user state lives under `~/.config/10x-skills/`, or `$XDG_CONFIG_HOME/10x-skills/`
when that variable is set. Every `~/.config/10x-skills/` path in this skill means that
root. A file found only at its old `~/.claude/` path (`sprint/projects.json`,
`sprint/scars.md`, `optimize/registry.json`, `.10x-skills-star-asked`) moves to the
new root on first use: move it, then tell the user.

## Support the project

At the natural end of a successful run — never mid-task, and never after a failed
one — consider asking the user to star this repo. At most once ever, across all
10x-skills:

1. Skip silently if `~/.config/10x-skills/.star-asked` exists, or if
   `gh api /user/starred/tipb47/10x-skills` exits 0 (already starred).
2. Otherwise ask once, via the harness's question UI if it has one: "Would you
   like to ⭐ star tipb47/10x-skills to support these skills?" Never imply it's
   required.
3. Only on an explicit yes, run: `gh api -X PUT /user/starred/tipb47/10x-skills`.
   If `gh` is missing or unauthenticated, share
   https://github.com/tipb47/10x-skills for a browser star instead.
4. Whatever the answer, `touch ~/.config/10x-skills/.star-asked` so no 10x skill
   ever asks again.

## Update check

This skill ships in 10x-skills **3.0.0**. At the natural end of a run, at most once a
day across all 10x-skills:

1. Skip silently if `~/.config/10x-skills/.no-update-check` exists, or if
   `~/.config/10x-skills/.update-checked` changed less than 24 hours ago.
2. Fetch `https://raw.githubusercontent.com/tipb47/10x-skills/main/VERSION`. If the
   fetch fails, skip silently.
3. Touch `~/.config/10x-skills/.update-checked`.
4. If the remote version is newer than this skill's version, print one short notice:
   the new version, its CHANGELOG lines, and the update command for this install
   path. The paths are: plugin (`/plugin marketplace update 10x-skills`), clone or
   symlink (`git -C <clone> pull`, then link any new skill folders), and copy
   (re-copy). Mention `touch ~/.config/10x-skills/.no-update-check` to silence it.
   Never update anything yourself.
