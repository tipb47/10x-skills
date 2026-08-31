---
name: analyze
description: Depth-scaled, read-only investigation of a feature, bug, flow, or component. Fans out parallel explore subagents to keep the main context clean, with per-subagent model selection. Use for /analyze <target> <depth>, or when the user asks how something works, wants an issue investigated, or wants a flow traced without changing code.
argument-hint: <target> <depth> [context]
---

# analyze — depth-scaled read-only investigation

This is an ANALYSIS session: investigate and report. Write no code, propose no
implementation, enter no plan mode. The report it produces is shaped to feed `/plan`
directly.

## Parameters

- **Target** ($1): what to analyze — feature, bug, flow, component, issue.
- **Depth** ($2): quick | standard | deep | forensic, or any descriptive phrase
  ("pretty thorough but not crazy") — interpret it sensibly.
- **Context** ($3): anything extra the user supplied.

## Depth calibration

| Depth | Subagents | Scope |
|---|---|---|
| quick | 0 — search and read directly | key files, main path |
| standard | 1–2 scouts | core flows, primary dependencies |
| deep | 2–3 scouts in parallel | all code paths, edge cases, related systems |
| forensic | 3–4+ scouts, + one deep-reasoning subagent | full trace, all references, history |

## Subagent doctrine

Subagents exist to keep the main context clean and to parallelize. Where the runtime
supports them:

- Scouts are READ-ONLY explorers. Independent scouts spawn in ONE message. Each prompt
  is self-contained: the specific aspect to investigate, and the return format —
  key findings, the 5–10 most important files (with paths), anomalies noticed, a
  control/data-flow summary. Scouts return conclusions, not file dumps.
- **Model per task, stated at spawn (tier + one-line why):**

| Work class | Tier |
|---|---|
| Mechanical enumeration — file inventories, call-site listings, no judgment | haiku |
| Scout investigation (default) | sonnet |
| Root-cause deep reasoning: ONE subagent, forensic depth only | opus or fable |
| Synthesis of findings | main session — never delegated |

## Phases

1. **Scope.** Look up facts yourself; put only genuine decisions to the user (use the
   runtime's structured question tool if it has one). Every question leads with your
   recommended answer; batch independent questions; sequence dependent ones. Confirm:
   what exactly is under analysis, boundaries, what prompted this, what the user needs
   to know at the end. Skip what the codebase or the arguments already answer.
2. **Discovery.** Fan out per the depth table. At quick depth: search and read directly,
   no subagents.
3. **Deep read.** Read the files the scouts flagged in the MAIN context; trace the flow
   end to end; capture file:line for every finding.
4. **Clarify as you go.** A genuine fork mid-investigation (two plausible
   interpretations, possible scope change) goes to the user immediately — recommended
   answer first. Never silently assume.
5. **Report** (in chat):
   - **Executive summary** — 2–3 sentences.
   - **How it works** — entry points, core logic, data flow, effects.
   - **Architecture & dependencies** — key files (paths), what it uses, what uses it.
   - **Code flow** — numbered trace with `file:line` per step.
   - **Edge cases & error handling** — covered and not covered.
   - **Concerns** — risks, smells, anomalies, each with location. Flag, don't fix.
   - **Open questions** — what code alone could not answer.

If the user's next step is implementation, say so: the Concerns and Open questions
sections are ready input for `/plan`.

## Hard rules

- Read files before drawing conclusions about them. Claims carry `file:line`.
- Verify, never trust: a scout's summary is a claim — spot-check the load-bearing ones
  against the actual files before building the report on them.
- Respect the depth the user paid for — no forensic sweeps on a quick ask.
