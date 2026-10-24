# {{PROJECT_NAME}} Ops — Sprint System

Sprint-directed build of {{ONE_LINE_MISSION}}.
<!-- init: state whether this folder is git-tracked, per interview. -->

## How to start a sprint (kickoff for a fresh agent session)

Launch the fresh session on the model that the current SPRINT.md's `Director tier:`
line resolves to in `~/.config/10x-skills/models.json`, then run `/sprint direct` — or paste:

> You are the {{PROJECT_NAME}} sprint director for sprint-NN. Read each file once, in
> this order: `{{OPS_PATH}}/STATE.md` (the STATE index), the guidelines
> (`DIRECTOR_GUIDELINES.md`, `SPRINT_GUIDELINES.md`, `DESIGN.md`), your sprint folder
> `{{OPS_PATH}}/sprints/sprint-NN/`, then the scar sections tagged for this project.
> Analyze the sprint against current reality (repo, stores, background jobs). Ask only
> about what changed since the sprint's Locked decisions, plus Gate 0 items. Present
> your execution plan for approval. Then run the sprint end to end.

Replace `NN` with the sprint the last close report names.

**Contract rule:** `SPRINT.md` is the *what*; the director's plan is the *how*. If
analysis shows the sprint doc no longer matches reality, amend `SPRINT.md` and log the
deviation in the sprint's `STATUS.md` BEFORE spawning tracks.

## Layout

```
{{OPS_DIRNAME}}/
├── README.md                  ← you are here
├── STATE.md                   ← STATE index: jobs, facts, decisions, history. READ FIRST.
├── DESIGN.md                  ← canonical design contract
├── SPRINT_GUIDELINES.md       ← rules for directors
├── DIRECTOR_GUIDELINES.md     ← the director runbook
├── TRACK_RULES.md             ← every rule a track obeys
├── ROADMAP.md                 ← milestone outline
├── BACKLOG.md                 ← one row per future item
├── arcs/
│   └── arc-NN/ARC.md          ← arc brief + status, when sprints run in parallel
└── sprints/
    ├── archive/               ← closed sprints
    └── sprint-NN/
        ├── SPRINT.md          ← sprint brief: goal, locked decisions, gates, tracks
        ├── STATUS.md          ← live header + tree ledger; only this sprint's director writes it
        ├── CLOSE.md           ← the sprint's full record, written once at close
        └── TRACK-*.md         ← self-contained prompt per track subagent
```

The STATE index never holds sprint narrative; that goes to each sprint's `CLOSE.md`.
To see what is in flight, read the header of each `sprints/*/STATUS.md`.

## Branch model

Trunk: `{{TRUNK}}` — a push triggers {{TRUNK_PUSH_EFFECT}}. Each sprint lands on
`sNN/integration`; the director promotes it into `{{TRUNK}}` with one merge and one push,
at the close gate or when you ask for an early promotion. CI: {{CI_TRIGGER_POLICY}}
<!-- init: same values as SPRINT_GUIDELINES.md § Branch model. -->

## Project surfaces

<!-- init: table of repos/packages/deploy targets + how each deploys, from interview. -->
| Surface | Where | Notes |
|---|---|---|
| {{SURFACE}} | {{PATH}} | {{DEPLOY_NOTES}} |

<!-- init: team shape note — solo, or named out-of-band pushers and the rules that implies. -->
