---
name: whereami
description: In-chat situation report for the current session — what is done, what is in progress, what comes next. Use when the user asks where we are, what's left, for a session status or recap, or runs /whereami.
---

# whereami — session situation report

Produce a compact situation report for THIS session, in chat. Read-only: never write
files, never mutate anything. This is a glance, not a transfer — a full transfer is
`/handoff`.

Build it from the conversation plus cheap reality checks: if the session touched git,
run `git status` and `git log --oneline -5`; if it started background tasks or agents,
check their actual state. Where a claim is unverified, say so ("edited, not yet
tested") — never upgrade it to done.

Format — short bullets, concrete nouns (file paths, commands, branch names), no
narrative padding:

- **Done** — completed this session, each item with how it was verified (or
  "unverified").
- **In progress** — mid-flight right now, including running background tasks/agents.
- **Next up** — remaining agreed work in order; then anything discussed but not agreed.
- **Blockers / open questions** — decisions waiting on the user, failing checks,
  missing access. Omit the section if none.

End with one sentence: the single most useful next action.
