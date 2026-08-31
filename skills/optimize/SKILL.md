---
name: optimize
description: Workflow-optimization meta skill — turn repeated manual or copy-paste workflows (e2e verification, service queries, pipeline pokes) into hands-off tooling via CLIs, MCPs, scripts, or generated custom skills. Use for /optimize <workflow>, /optimize scan (mine past work for friction candidates), /optimize audit (re-verify past optimizations); whenever the user says to optimize, automate, or streamline something — including contextual asks like "let's optimize this" or "optimize our <service> interaction"; or proactively, when you notice the session paying a repeated manual tax worth tooling away.
argument-hint: <workflow> | scan | audit
---

# optimize — transform recurring workflows into tooling

Sprints and plans execute through whatever tooling exists; this skill improves the
tooling itself. The target: any workflow where a human repeatedly copy-pastes output,
hand-runs verification, or re-explains the same service interaction to every session.
The outcome: hands-off access (CLI, MCP, script) or a generated custom skill — plus
fewer tokens burned re-deriving the same path.

## Modes

- **`/optimize <workflow>`** — the user names the friction. Go straight to Research.
- **`/optimize scan`** — mine past work for candidates, then propose.
- **`/optimize audit`** — re-verify every registered optimization; flag stale ones.

**Resolving the target.** "This", "our <service> interaction", or a bare `/optimize`
mid-conversation refers to the friction at hand — resolve it from what the session was
just doing or discussing, state your interpretation in one line, and proceed (correct
course if the user objects). A bare invocation with no conversational friction to point
at → offer `scan`. And this skill is not only user-summoned: when you notice the
current session paying a repeated manual tax — the operator pasting output, the same
service poked by hand again — offer to run it, whatever the task at hand.

## Discovery (scan mode)

Mine these sources — and only these — for repeated friction:

1. **Sprint ledgers and scars:** every project in `~/.claude/sprint/projects.json` —
   `STATE.md` learnings and history, generated guidelines, the engine's SCARS. The
   highest-signal record of what repeatedly cost time and tokens.
2. **Agent memory and handoffs:** the persistent memory directory and any handoff or
   session artifacts on disk.
3. **Repo signals:** package scripts, Makefiles, CI configs, existing MCP/CLI configs —
   what is already automated versus quietly done by hand.

Never read shell history. Rank candidates by recurrence × cost per occurrence
(operator time, agent tokens, context burned on re-derivation).

## Research

For each candidate, research the fix — web search, official docs, MCP/CLI ecosystems:

- **Prefer official/first-party tooling** (the vendor's CLI, the vendor's MCP server).
- Third-party tools carry a **provenance note**: maintainer, adoption, last release,
  what it gets access to. An unvetted MCP server is a supply-chain risk — flag it as
  such, never quietly install it.
- **Least privilege by design:** propose read-only roles/keys wherever reading is the
  need (e.g. a read-only IAM role behind an AWS CLI/MCP beats an admin key, and beats
  the operator pasting query results forever).
- Think past tools: a well-placed script, a make target, or a generated skill can beat
  a heavyweight integration.

## Proposal — the approval gate

Present a ranked table, recommended-first, before ANYTHING installs or changes:

| Column | Content |
|---|---|
| Friction | the workflow and what it costs today (time / tokens / copy-paste) |
| Fix | the tool/approach, with provenance note if third-party |
| Effort | setup cost, including any operator gates (logins, account steps) |
| Payoff | what becomes hands-off; what stops being re-explained per session |

Interview with the standard discipline (recommended answer first) on which to pursue.
Nothing proceeds without explicit approval.

## Setup

- Install and configure approved tooling. **Logins, account creation, credential
  issuance, and permission grants are operator gates** — prepare the exact commands,
  have the operator run them (in Claude Code, suggest the `! <command>` prefix so the
  output lands in-session), verify the result yourself. Never handle secret values;
  point them at env files or the tool's own auth flow.
- **Verify end-to-end, never trust setup output:** drive the new path for real — run
  the query, hit the service — and quote the actual output. Record the before/after
  delta (what used to take N manual steps is now one call).

## Generated skills

When a workflow deserves a reusable recipe, generate a custom skill encoding it —
agents then invoke it internally rather than re-deriving the path:

- **Placement by scope:** project-specific → that repo's `.claude/skills/`;
  cross-project → the user's skills directory.
- **Provenance frontmatter** in the generated SKILL.md body header: generated-by
  `/optimize`, date, the workflow it encodes, and a one-line **reverify command**
  proving it still works.
- Generated skills follow the house rules: interview discipline, verify-never-trust,
  no secrets, comments describe the code.
- **Register it** in `~/.claude/optimize/registry.json` (create if missing):

```json
{ "optimizations": [ { "name": "<slug>", "scope": "user|/abs/project/path",
  "workflow": "<one line>", "date": "YYYY-MM-DD",
  "skill_path": "/abs/path or null", "reverify": "<command>" } ] }
```

Tool-only optimizations (no generated skill) register too — the audit needs them.

## Audit mode

1. Read the registry. Empty/missing → say so, point at `/optimize scan`.
2. Per entry: run the reverify command; check the tool/MCP still exists, still
   authenticates, still returns real output.
3. Report: healthy | stale (changed upstream, needs rework) | broken (auth dead, tool
   gone). Propose fixes or retirement per flagged entry — recommended disposition
   first; retire only with approval, removing the registry entry and (if approved) the
   generated skill.

## Hard rules

- Nothing installs, changes, or registers without the proposal's approval gate.
- Never create accounts, spend money, or touch secret values — operator gates, always.
- A generated skill that cannot state its reverify command is not done.
- This skill optimizes workflows; it never rewrites project feature code.
