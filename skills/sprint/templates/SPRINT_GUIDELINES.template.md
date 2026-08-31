# Sprint Guidelines

Rules of work for every sprint, track, and director. Director runbook:
`DIRECTOR_GUIDELINES.md`. Both obey this file and `DESIGN.md`.

## What a sprint is

- One bounded milestone, run end-to-end by ONE fresh agent session (the
  **director**), which spawns each **track** as a subagent.
- {{TRACK_COUNT_POLICY}} tracks per sprint.
  <!-- init: 1-3 for pipeline-shaped projects with sequential dependencies; 2-4 for
  multi-surface products. State which shape this project is and why. -->
- Sprints contain **operator gates** — explicit pauses for the human
  ({{GATE_EXAMPLES_FROM_INTERVIEW}}). Gates live in `SPRINT.md`, never inside track
  prompts. Gate 0 runs before tracks spawn; the close gate runs after merges.

## Track rules

1. **Self-contained prompts.** A track reads its `TRACK-*.md` + `DESIGN.md` and needs
   nothing else. Mid-flight human input required = the sprint was planned wrong.
2. **No blocking questions.** Ambiguity → decide per `DESIGN.md`, log under "Decisions &
   open questions" in the track report.
3. **Branch discipline.** One branch per track: `sN/<slug>`, from latest `origin/main`.
   Push the branch; never merge, never touch main. Same-repo parallel tracks run in
   worktrees. A track's brief states its push authority EXPLICITLY, including the branches
   it must never push, even when that seems obvious.
   - **Commit and push INCREMENTALLY — never save the first commit for the end.** Agents get
     killed by infrastructure limits with substantial work existing only in a worktree, and
     nothing about that failure is under the track's control. Commit at every coherent
     milestone, label it `wip:` if it is not coherent yet, and push.
   - **Never `git stash` in a worktree.** `refs/stash` lives in the COMMON git dir and is
     shared across every worktree; concurrent stashes swap stacks and each `pop` applies the
     OTHER track's work. Baseline-test with a scratch commit or a file copy.
   - **Scratch state goes in a PRIVATE subdirectory on a PRIVATE port.** Parallel tracks share
     one scratchpad root; a second track initializing over it destroys the first track's proof.
   {{WORKTREE_SEED_RULE}}
   <!-- init: if the project needs env files or credentials to build, add: "Seed every
   worktree with the sanitized <file> before building (<copy command>) — a build failing for
   want of an env var reads exactly like a defect in the change under audit, and gives an
   agent a reason to reach for the operator's real secrets." If no such file exists, drop
   this line. -->
4. **The verification contract (DESIGN §5) is the gate.**
   <!-- init: instantiate concretely — exact test commands; for data work: tracks ship
   verification queries WITH expected results and reports quote ACTUAL output. -->
5. **Never trust, always verify.** "Done"/"pushed"/"loaded" are claims; the report quotes
   command/query output, not assertions. Directors re-verify at audit.
   - **A passing grade is not a verdict until you know WHY it passes.** Clearing a gate is
     necessary, not sufficient — decompose the SOURCE of the result before acting on it, and
     grade against the honest baseline, not a trivial floor. A number whose source you cannot
     name is not a result.
   - **Verify through the ACTUAL path.** A hand-run literal does not exercise the client's
     parameterized path; an unauthenticated request proves nothing about a credential; a mock
     cannot see a bug in the part of reality it omits.
6. {{MIGRATIONS_RULE}}
   <!-- init: if DB: "Migrations are files in <path>, applied ONLY by the director at
   merge time; tracks code against the post-migration schema." If no DB: drop this rule. -->
7. **No fabricated data, ever,** outside clearly-marked test fixtures under test dirs.
   A job that can't fetch real data records the failure — it never fabricates rows.
8. **Comments describe the code, not the change.** A code comment exists only to state a
   constraint the code can't show — written in present tense about the code as it is.
   Never reference sprints, tracks, or PRs; never narrate the diff ("removed in…",
   "now X", "used to be Y"). Provenance already has homes: commit messages
   (`[sNN/<slug>]`), the track report, `STATE.md` — never code. Test names and
   `describe`/`it` strings follow the same rule: name the behavior pinned, not the sprint
   that pinned it. No ALL-CAPS emphasis or `──` banner headers inside comments.
   - BAD: `// The legacy narrowing arg was removed in s43 (Track C).`
   - GOOD: `// Single-org scoping comes from the request context; a client-supplied org
     arg here would bypass tenant gating.`
   A deliberate decision that *looks* like a bug IS a constraint — keep it, stated
   behaviorally: `// Intentionally serves the stale value; upstream TTL is 45s and
   blocking would stall every submit.`
9. **Report format:** what shipped; branch + commits; verification outputs (verbatim);
   files changed; decisions & open questions; what the director must verify by hand.
<!-- init: if the project has long-running jobs, add the AFK-grade standard as a rule:
checkpoint manifest, detached execution, incremental sync, inline validation, idempotent
re-runs — tracks deliver the harness verified on a sample chunk; full runs start at close. -->

## Definition of done (sprint)

- All track branches audited, merged `--no-ff` to main in SPRINT.md's order, pushed;
  verification contract green on main after each merge.
- Sprint git state cleaned: this sprint's worktrees removed (each checked for
  uncommitted work first), `git worktree prune` run, the PREVIOUS sprint's merged track
  branches deleted. This sprint's branches survive one more sprint as recovery points;
  unmerged branches are never deleted.
- {{DB_DONE_CLAUSE}} <!-- init: migrations applied + verified, if DB; else drop. -->
- Operator close-gate checklist delivered/executed.
- `STATE.md` updated; next sprint's files drafted or amended from reality.

## Token discipline

- One subagent per track; no orchestration frameworks unless the operator explicitly asks.
- Director audits diffs directly; at most ONE review subagent per sprint, only for a
  genuinely risky merge.
- Tracks read what their file points to; no exploratory fan-outs — `DESIGN.md` is the map.
- Model policy: {{MODEL_POLICY}}.
  <!-- init: from interview, seeded from the engine's default tiering — director: opus;
  contract-defining / architecture / gnarly-debug tracks: fable; standard implementation
  tracks: sonnet; haiku only for mechanical fully-specified chores with a stated
  justification. Any role, the director included, may escalate to fable when the work
  genuinely warrants the deepest reasoning — recorded in SPRINT.md at drafting time,
  never decided mid-sprint. Every SPRINT.md track row states its tier + a one-line
  rationale. -->

## Git conventions

- Commits: imperative summary, prefixed with the track tag `[sNN/<slug>]` (director
  ops/merge commits use `[sNN/ops]`); end with the project's standard co-author trailer.
- Stage with EXPLICIT paths — never `git add -A` / `git add .`.
- **Ops bookkeeping stays out of code commits.** {{OPS_COMMIT_POLICY}}
  <!-- init: resolve from the interview's tracked/untracked answer:
  • UNTRACKED → "The <ops> dir is gitignored; never stage it."
  • TRACKED → "<ops> churn (STATE.md / SPRINT.md / TRACK-*.md) commits SEPARATELY from
    code, tagged `[sNN/ops]` — never mixed into a code or merge commit. Keeps the code
    history clean and bisectable." -->
- **Confirm the ops ledger's tracked/untracked status on EVERY branch the director checks
  out.** A ledger gitignored on one branch but still TRACKED on another is OVERWRITTEN by the
  stale tracked copy the moment you check that branch out for a merge. Recover with
  `git show <commit>:<path>`; prevent with `git rm --cached` on every branch you visit.
- With worktrees, ALWAYS `git -C <absolute-path>`.
- Diff a track from the MERGE-BASE, never the branch tip against a moved trunk:
  `git diff $(git merge-base main <branch>) <branch>`. The two-way diff shows a clean track
  "deleting" files it never touched.
- Before merging, `git checkout main` and confirm `git branch --show-current`. A repo left on
  the feature branch merges it into ITSELF and reports "Already up to date" — a silent no-op
  that reads as success.
- Never chain push/merge after a PIPED build/test in one compound — pipes mask exit codes.
  Verify, check the exit code, THEN push. Judge builds by exit code, not stderr noise.
- Director merges `--no-ff` per track; never force-push main.
<!-- init: if out-of-band human pushers exist: add "pull + check their recent commits at
boot; never clobber; rebase, don't force; keep changes to their hot files minimal+flagged." -->
