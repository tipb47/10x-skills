---
name: handoff
description: Produce a complete session handoff as ONE copyable in-chat message that lets a fresh agent continue the work. Use when the user wants to hand off, transfer, or continue this session's task in a new session, or runs /handoff.
---

# handoff — one-message session transfer

Write a handoff the user can paste into a FRESH agent session. Deliver it as ONE
message inside a single markdown code fence, in chat. Never write it to a file; never
create dedicated handoff documents.

The next agent has ZERO context: no conversation history, no memory of decisions. The
handoff must be self-contained — every path absolute, every claim labeled verified or
unverified, every decision stated with its why.

Before writing, refresh reality cheaply: `git status`, current branch, recent commits
if the session touched git; actual state of background tasks; which files really
changed. Hand off facts, not recollections.

Structure of the fenced message:

1. **Role + mission** — one paragraph: who the next agent is, the overall task, where
   it stands.
2. **Context** — the project, key paths (absolute), which docs/files are authoritative
   and must be read first.
3. **Done so far** — completed work, each item with HOW it was verified. Label anything
   unverified as a claim, explicitly.
4. **In flight** — half-done work, its exact state (branch, files, failing test), and
   what finishing it looks like.
5. **Next goals** — remaining work in intended order, with any agreed approach.
6. **Decisions already made** — each with its one-line why. The next agent must not
   silently relitigate these.
7. **Gotchas** — footguns discovered this session: quirky commands, env traps, things
   that look done but are not.
8. **Standing instructions to you, the next agent** — include these, always:
   - Verify, never trust: re-check every claim above against real state (git, files,
     running processes) before building on it.
   - Before substantial work, interview the operator relentlessly about the plan: walk
     every branch of the decision tree, resolve dependent decisions in order, and lead
     every question with your recommended answer. Look up facts yourself; put only
     genuine decisions to the operator.
   - Do not start building until the plan is unambiguous.

Dense but complete — long is fine, hollow is not. No placeholders.
