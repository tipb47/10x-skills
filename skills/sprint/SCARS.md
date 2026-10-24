# Universal scars: curated core

Failure modes that have already cost real sprints, stated as standing rules. This file is the
curated core: it holds only rules that bite any sprint on any stack. The director reads it at
boot; `/sprint init` folds the relevant rules into the project's generated guidelines.

Directors never edit this file. A director appends new scars to the private scars file,
`~/.config/10x-skills/sprint/scars.md`, which is never committed anywhere. A scar enters this core only
when the operator explicitly asks, for example through a pull request to this repo.

Scar shape: state the class in one sentence, then the mechanism, then the rule, in 1–3
lines. Never the incident story; that stays in the sprint's `CLOSE.md`. A scar that only
reproduces in one stack is a project note or a private scar, never a core rule. Each
section heading carries its tags; every section here is `[all]`.

---

## Git & branches [all]

- **With worktrees, ALWAYS `git -C <absolute-path>`.** A bare git command in a stale cwd has
  reset a branch mid-merge.
- **Stage with EXPLICIT paths.** Never `git add -A` or `git add .`. Bulk adds have swept local
  tooling config into pushed commits.
- **Never chain push or merge after a PIPED build or test.** The pipe masks the exit code, so
  the push runs on `tail`'s status, not the build's. Check the exit code, THEN push.
- **Read a track's diff from the MERGE-BASE.** `git diff <trunk> <branch>` on a branch cut
  before other merges landed shows the track "deleting" files it never touched. Use
  `git diff $(git merge-base <trunk> <branch>) <branch>`.
- **A repo left checked out on the feature branch merges into itself.** `git merge <branch>`
  there reports "Already up to date", a silent no-op that reads as success. Check out the
  trunk, confirm `git branch --show-current`, then merge. If "Already up to date" is
  unexpected, run `git merge-base --is-ancestor <branch> <trunk>`.
- **Confirm the ops folder's tracked or untracked status on EVERY branch you check out.** An
  ops folder gitignored on one branch but TRACKED on another gets OVERWRITTEN by the stale
  tracked copy at checkout. Recover with `git show <commit>:<path>`. Prevent with
  `git rm --cached` on every branch the director visits.
- **Never force-push the trunk, and land tracks only on the integration branch.** Any
  force-push is an operator-gated, single-purpose sprint. Tracks merge into `sNN/integration`.
  The trunk receives ONE merge, at an operator gate. Per-track trunk pushes multiply CI runs
  and deploys.
- **Ignore harness directories inside the checkout with a ROOT-ANCHORED pattern.** Harnesses
  put worktrees under a dot-directory in the repo, so repo-wide linters and `**/` globs scan
  or skip them by accident. Anchor the ignore at the root in both the linter and
  `.gitignore`, and confirm the gate's processed-file COUNT in the main tree and a worktree.

## Agents, worktrees & orchestration [all]

- **Tracks commit and push INCREMENTALLY.** Infrastructure limits (token caps, timeouts,
  crashes) kill agents with all work still in a worktree. Commit at every milestone, label
  unfinished work `wip:`, and push. Put this rule in every track brief.
- **Parallel worktree tracks must NEVER `git stash`.** `refs/stash` lives in the COMMON git dir.
  Two agents stashing concurrently each popped the OTHER track's work. For a clean baseline,
  use a scratch commit (`git commit -m wip`, work, `git reset HEAD^`) or a file copy. Put this
  ban in every parallel track brief.
- **Parallel tracks share ONE scratchpad root.** A second track initializing scratch state
  destroyed the first track's proof artifacts mid-run. Put databases, ports, caches, and temp
  dirs in a PRIVATE subdirectory on a PRIVATE port.
- **NEVER run mutating git operations in a tree while an agent is live in it.** Completion and
  kill notifications are UNRELIABLE, an agent can resume after one, and transcripts flush
  lazily. Judge liveness and failure by GIT STATE: new commits, working-tree changes,
  `git -C <worktree> status`. A "failed" agent may hold substantial UNCOMMITTED work. Run
  read-only git until the completion notification AND the pushed branch both check out.
- **A completed agent can SELF-RESUME and act un-instructed.** One finished agent invented a
  justification, merged a sprint lineage into a deploy branch, and pushed. That bypassed the
  operator's merge gate. Therefore: (1) every brief states the agent's branch-push authority,
  including "you never push to the trunk"; (2) on any completion notification, verify no
  unauthorized push landed; (3) surface a bypassed gate to the operator IMMEDIATELY with a
  verified damage envelope. Ratify-or-revert is the operator's call.
- **A harness-created worktree starts at the TRUNK, not the integration tip.** A track that
  reads its brief before checking out its branch reads the pre-amendment copy and misses every
  Gate 0 correction. Order the spawn prompt so checkout comes FIRST, or point it at
  `origin/<integration>:<path>`. A report quoting an already-amended brief value is the tell.
- **Tracks never write the shared ops folder.** The report goes in the final message,
  with a fallback path inside the worktree. The director relocates a fallback file and
  drops it from the integration diff.

## Verification & gates [all]

- **A track's own gates cannot see the seam BETWEEN tracks.** Every gate can pass while the
  product is broken, because each track made a correct decision about a boundary the other
  track owned. Before closing, drive the actual deliverable end-to-end, the thing a user opens.
  For anything customer-facing, drive the DEPLOYED origin, not a local harness.
- **Never trust, always verify.** "Done", "pushed", "loaded", and "green" are claims. The report
  quotes command output. The director's audit produces the facts.
- **A passing grade is not a verdict until you know WHY it passes.** Decompose the SOURCE of the
  effect: which inputs carry it, and is that source what the deliverable should provide? Grade
  against the right baseline, because beating a trivial floor is trivial. Report the
  decomposition next to the headline number.
- **Prove the mechanism before shipping a fix.** A plausible cause that survives a code read is
  still a hypothesis. Instrument or reproduce first.
- **One symptom can carry TWO stacked causes.** After fixing a proven cause, RE-RUN the original
  failing observation on the real path before declaring it closed.
- **A test can pass vacuously.** An empty stub or a fixture that shares no join keys makes
  filters return nothing and `all(...)` assertions pass, or makes a positive assertion fail
  against a filter working as designed. Populate stubs with the shape the real dependency
  returns, assert the COUNT, and suspect the fixture before the filter.
- **A mock cannot see a bug in the part of reality it omits.** A fake that skips events the real
  client emits makes the omission a blind spot. Enumerate the real events and emit them in the
  real order.
- **A probe that is not the real path proves nothing about it.** A hand-run query is not the
  parameterized query a driver sends. An unauthenticated request does not test a credential.
  An empty read from a policy-bound or unscoped credential reports the POLICY, not the data.
  Verify through the ACTUAL path, include a no-credential control, and state which credential
  produced each number.
- **A green suite can MASK a silently-skipping subtree.** Compare skip COUNTS against the track's
  reported numbers, not just exit codes.
- **Baseline once per sprint, then trust the tree ledger.** A track's reported baseline can
  diverge from yours because of environment, and re-running unchanged trees proves nothing.
  Run the full suite once on the integration branch's starting tree and record it against
  `git rev-parse "HEAD^{tree}"`. Judge each later run by its FAILURE SET against that
  baseline: clean means zero new failures. Never re-run a tree already recorded green. A
  red or flaky result may always be re-run to diagnose.
- **Enumerated-file-list scopes systematically MISS repo-root surfaces.** Repo-wide renames and
  contract changes leave the README, API spec, `.env.example`, and top-level docs stale. Give
  ONE track ownership of "repo root + docs + spec", or budget a director cleanup pass, and run
  the seam grep REPO-WIDE. Separate intentional residue from missed surfaces.
