---
name: explain
description: Plain-language explainer for non-specialists — builds a concept, commit, diff, file, or Claude's last answer up from basics. Use for /explain [target], or when the user asks to explain this simply, in layman's terms, or like they're new to this.
argument-hint: [target] [simpler | deeper] — target is inferred if omitted
---

# explain — building-blocks explainer

Explain one thing to a non-specialist, in chat. Read-only: never write files, never
mutate anything. This skill explains. It never reports status. For where the work
stands, use `/whereami`.

## Resolve the target

Take the target from the argument. With no argument, infer it from the session, in
this order:

1. The last answer you gave, if the user reacts to it ("explain that").
2. A commit, diff, or file the session just discussed.
3. A concept or term the user named.

If two targets fit, pick the likeliest and name your choice in one line.

## Calibrate the level

Judge the reader's level from how they write and what they ask. Assume a smart person
who does not know the field. Do not assume any background in this codebase. Say the
level you chose in a short opening clause so the user can correct it.

## Build the explanation

Build up from what the reader already knows.

1. Start with the one-sentence point: what this is and why it matters.
2. Give one idea per paragraph. Each paragraph rests on the one before it.
3. Use a concrete analogy where it helps. Drop it when it starts to mislead.
4. Define each name on first use. Use that name every time after.
5. For a commit or diff, explain the problem it solves, then what changed. Skip
   line-by-line detail.
6. End with a short line headed **In technical terms** that maps the plain words back
   to the real ones.

Read the target before you explain it. If you cannot read it, say so. Do not guess at
its contents.

## Re-run at another level

"Simpler" or "deeper" re-runs the explanation one step in that direction. Keep the
same target. Say the new level in the opening clause.

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
