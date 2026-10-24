---
name: deslop
description: Remove and prevent AI slop — the statistical-median look and voice of generated work. Frontend path (deep, inspired by samber's frontend-design-deslop): strategy-first design, OKLCH tokens, craft layer, STYLE.md, fresh-eyes slop audits via subagents. Code path (minimal): LLM tells in comments, docs, and structure. Use for /deslop [target] and /deslop audit [target] [intensity]; PROACTIVELY at two bookends only: when a NEW UI surface or a redesign starts with no STYLE.md covering it (design before pixels), and once when a unit of UI work completes (one self-audit). Follow-up tweaks to an already-styled surface do not trigger it; and when the user says deslop, unslop, slop audit, "make it not look like AI", or "less generic".
argument-hint: [target | audit <target> [quick|standard|deep]] — routes frontend vs code from context clues
---

# deslop — slop removal and prevention

AI-generated work converges on the statistical median: in UI, Inter on indigo
gradients, rounded cards, hero + three feature cards; in code, narrated comments and
boilerplate hedging. Slop is not ugliness — it is the absence of committed choices.
This skill routes to the right lens, applies an opinionated discipline while
building, and audits finished work with fresh eyes.

Frontend path adapted from samber/cc-skills `frontend-design-deslop` (MIT — credit
where due); code path is original and deliberately minimal for now.

## Routing

Resolve the target from the argument or, for "this" / no argument, from what the
session was just doing. Then route:

- **Frontend** — pages, components, styling, themes, dashboards, landing pages,
  artifacts, anything rendered. The deep path below.
- **Code** — comments, docs, READMEs, PR descriptions, naming, structure. The
  minimal path at the end.
- Ambiguous target → say which lens you chose and why; both lenses only when the
  target genuinely spans both.

## When to fire without being invoked

Fire at two bookends only. Follow-up tweaks to a styled surface invoke nothing:
STYLE.md applies passively, and you read it and honor it.

- **Start of a new surface or redesign** with no STYLE.md covering it → run the
  frontend build discipline first. Slop prevented is cheaper than slop removed.
  Skip it for trivial touches (a copy fix, a spacing nudge) and for any change to a
  surface that STYLE.md already covers.
- **End of a unit of UI work** (new UI, a redesign, a polish pass) → run ONE
  completion audit, scaled by how much was touched (see Audit mode). Do not re-run it
  for each follow-up tweak that comes after.

## Frontend path — build discipline

Lean core here; depth lives in `references/` and is pulled ONLY when doing that
work. Never preload the library.

**Phase 0 — Discover.** Check for `STYLE.md` at the project root (and `docs/`).
Exists → read it, honor its tokens, skip answered questions, extend rather than
restart. Missing → lock three things before any pixel: WHAT the artifact is
(`references/artifact-types.md`), WHO it serves and the single primary action, and
3–5 committed brand adjectives. Run the questions as a `/pry` interview (where
installed): recommended answer first, batch independent, infer-then-confirm — only
the high-signal subset that changes the design system. Full protocol:
`references/discovery.md`. On a generic brief, widen references past the category
instead of asking for more creativity: `references/divergence.md`.

**Phase 1 — System (the gate).** Translate the adjectives into commitments, stated
briefly in prose before any component:

1. ONE opinionated aesthetic direction (`references/aesthetics-library.md`).
2. Typography from personality — never Inter/Roboto/Arial/system-ui as the primary
   face (`references/typography.md`).
3. Color in OKLCH — dominant + sharp accent + neutrals + semantic states, ~60-30-10;
   the indigo/violet band is a red ocean (`references/color-oklch.md`).
4. Token table BEFORE components: fonts, 6-step type scale with stated ratio,
   spacing base, max two radii, ONE shadow approach, role-named palette including
   `surface`. Starting sets: `references/token-sets.md`, `references/token-core.css`;
   stack syntax: `references/adapters.md`.
5. The signature move — the ONE thing that makes this UI unmistakable. One per
   project.

**Phase 2 — Craft.** Tokens make it consistent; craft makes it good. Apply per
artifact type, pulling the matching reference on demand: layout and composition
(`references/layout.md`), full component state matrices (`references/components.md`),
motion as communication (`references/motion.md`), one coherent icon system
(`references/iconography.md`), art-directed imagery (`references/imagery.md`),
designed-not-inverted dark mode (`references/dark-mode.md`), and WCAG 2.2 AA built
in, not bolted on (`references/accessibility.md`). Mechanisms behind all of it:
`references/design-theory.md` (read once, early). End of conception: suggest a small
artifact-relevant subset of `references/catalogs.md` as inspiration to transpose,
never clone.

**Phase 3 — STYLE.md.** The durable output: one `STYLE.md` at the project root —
discovery context, aesthetic commitment, signature move, tokens, craft decisions.
It holds a short core plus a reference part. Schema: `references/style-md.md`. STYLE.md wins over drifted CSS or
components; later sessions read it instead of re-running discovery. (In sprint
projects it sits beside `ops/DESIGN.md`: STYLE.md is visual design, ops/DESIGN.md is
architecture.)

**Phase 4 — Self-audit.** Every completed build gets audited before presenting — via
the Audit mode below, never in-session.

## Audit mode — `/deslop audit [target] [intensity]`, and every auto-fire

The prime directive: **audits run in fresh-context subagents that report back.** The
session that built the UI cannot audit it — its context is gummed up with build
reasoning; it rubber-stamps its own choices and misses what a fresh eye flags in
seconds. Spawn read-only scouts with self-contained prompts (target files/URLs, the
checklist scope, the report shape — no build history), then synthesize their reports
in the main session. Without subagents on this harness, run the audit as a separate
pass against the checklist alone, and flag the verdict as weaker: it is not fresh eyes.

Intensity — stated explicitly, defaulted by scope when unstated:

| Level | Fan-out | Use |
|---|---|---|
| quick | ONE scout: full checklist pass on the named target | a component, one page, every small completion auto-fire |
| standard | 2–3 scouts split by dimension: color+typography, layout+components, motion+a11y | a feature's UI, a few pages, mid-size completion auto-fire |
| deep | dimension scouts + a rendered-page scout (screenshots/browser where available: layout defects, build correctness) + one strong-tier synthesis judge for verdicts | whole app or site, pre-launch, "unslop everything" |

Auto-fire on completion scales the same way: size the intensity to how much UI the
session just touched. Scouts run on the fast tier by default; judgment never goes to light (§ Model tiers).

Each scout audits against `references/slop-checklist.md` plus the STYLE.md contract
when one exists (render vs commitment, promise by promise — the divergence check in
`references/divergence.md`). Score 0–10, findings worst-first, each with file:line
or a screenshot region and the concrete fix.

**Report, then fix.** Present the synthesized findings. Apply fixes only on approval
— or immediately when invoked with `--fix`. Fixing goes back through the build
discipline (tokens first, not spot-patches), and the audit result goes in the
audit report and the commit message. Never append it to STYLE.md.

## Code path — minimal, grows later

No reference library yet; one tight checklist, applied in-session for builds and via
a fresh-eyes scout for audits (same prime directive):

- **Comment noise:** comments narrating what the next line does, "this function
  does X" headers, section-banner comments, emoji in comments or commit messages,
  comments talking to the reviewer ("added this to fix...").
- **Hedging boilerplate:** defensive try/catch around code that cannot throw,
  needless null-checks on values just constructed, "for safety" duplication.
- **Dead abstraction:** interfaces with one implementation, helpers called once,
  config for things that never vary, premature generality.
- **Doc slop:** README sections that restate the file tree, "## Features" bullets
  nobody asked for, PR descriptions that narrate the diff instead of the why,
  CHANGELOG entries duplicating commit messages.
- **Naming:** `utils`, `helpers`, `data2`, `newX`/`XNew`, `enhanced`/`improved`
  prefixes, synonym rotation for one concept.

The fix is deletion first, rewrite second. Report findings worst-first; apply on
approval or `--fix`.

## Model tiers

Skills name a model tier, never a model. The tier says how much reasoning the work
needs; your map says which model delivers it on this harness.

| Tier | Use for |
|---|---|
| **frontier** | Hard problems only: novel algorithms, modeling engines, breakthroughs |
| **strong** | The workhorse: judgment, contracts, architecture, gnarly debugging |
| **fast** | Standard implementation and investigation |
| **light** | Mechanical, fully specified chores; never judgment |

**Resolve a tier at spawn time:**

1. Read `~/.config/10x-skills/models.json`, keyed by harness, then tier:
   `{ "<harness>": { "<tier>": { "model": "...", "effort": "..." } } }`.
2. No entry for this harness → list the models the harness offers, propose a mapping
   (two tiers may share a model at different efforts), confirm it with the user, and
   save it.
3. A mapped model is gone, or a newer one is offered → propose a remap once. Never
   change the map silently.
4. The harness cannot pick a model per subagent → apply the tier's effort only. It
   cannot set effort either → inherit the session model, and say so in one line.

**Every spawn names its tier.** State the tier and a one-line why at each spawn.
Never let a subagent inherit the session model by default. A fork inherits, so never
fork for tiered work. Spawn a typed agent with an explicit tier instead. Directors
and drafters (sprint drafting, `/build`, `/sprint arc`) run on strong or above.
Their effort scales with the work's size and seams; the tier does not drop.

A legacy model name in an old doc reads as its tier: first by reverse lookup in the
map, else opus → strong, fable → frontier, sonnet → fast, haiku → light.

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
