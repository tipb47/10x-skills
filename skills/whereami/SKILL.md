---
name: whereami
description: In-chat situation report in plain words — what this session (or every recently active session across projects) has done, is doing, and does next. Use when the user asks where we are, what's left, what the agents did, for a status or recap, or runs /whereami.
argument-hint: [this | all | project name] — optional; scope is inferred from the prompt
---

# whereami — situation report

Produce a plain-language situation report in chat. Read-only: never write files,
never mutate anything. This is a glance, not a transfer — a full transfer is
`/handoff`. This skill reports status only. To explain a concept, a commit, or an
answer, use `/explain`.

## Scope

Infer the scope from the prompt. Do not add a subcommand.

- **This session** — "where are we", "what's left", "recap". Default when unclear.
- **Every recently active session** — "what did my agents do", "status of everything",
  "I'm back". Cover sessions active in the last day across projects, unless the user
  names another window or project.

## Sources

Verify before you claim. Where a claim is unverified, say so ("edited, not yet
tested") — never upgrade it to done.

- **This session:** the conversation, plus cheap reality checks. If the session
  touched git, run `git status` and `git log --oneline -5`. If it started background
  tasks or agents, check their real state.
- **Other sessions:** the harness's session store, where readable. On Claude Code,
  read the newest entries in `~/.claude/projects/<project>/*.jsonl`. Read the last
  user goals and the last results only. Do not read whole transcripts.
- **Sprint projects:** each active sprint's `STATUS.md`.
- **Always:** `git log` and `git status` in each project, to confirm what landed.

If a source is unavailable, say which one and what the report lacks because of it.
If you cannot read other sessions at all, say so and report this session only.

## Tone

Write for someone who left for hours and wants to know how far things got.

- Lead with progress, not mechanism. Use counts and plain verbs.
- Keep jargon out. Use a technical name only when the reader must act on it.
- No buzzwords. One term per thing.

Example: "Sprint 12: 3 of 4 parts done. One went wrong and was fixed. Now fixing a
payout bug."

## Format

For one session, use short bullets:

- **Done** — completed, each item with how it was verified (or "unverified").
- **In progress** — mid-flight now, including running background tasks and agents.
- **Next up** — remaining agreed work in order; then anything discussed but not agreed.
- **Needs you** — decisions waiting on the user, failing checks, missing access. Omit
  if none.

For several sessions, give one short block per session: project name, then one or two
plain sentences on what it did and what it is doing now. Put sessions that need the
user first.

End with one sentence: the single most useful next action.

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
