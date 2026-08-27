# Universal scars

Failure modes that have already cost real sprints, stated as standing rules. Every one is
project-agnostic: it describes a class, not an incident. The director reads this file at
boot; `/sprint init` folds the relevant rules into the project's generated guidelines.

Adding a scar: state the class in one sentence, then the mechanism, then the rule. If it
only reproduces in one company's stack, it is a project note, not a scar.

---

## Git & branches

- **With worktrees, ALWAYS `git -C <absolute-path>`.** A bare git command in a stale cwd has
  reset a branch mid-merge.
- **Stage with EXPLICIT paths.** Never `git add -A` / `git add .` — bulk adds have swept local
  tooling config into pushed commits.
- **Never chain push/merge after a PIPED build or test in one compound command.** The pipe
  masks the exit code, so you push on `tail`'s status, not the build's. Verify, check the exit
  code, THEN push. Judge builds by exit code, not by stderr noise.
- **Read a track's diff from the MERGE-BASE, never the branch tip against a moved trunk.**
  `git diff <trunk> <branch>` on a branch cut before other merges landed shows the track
  "deleting" files it never touched. Use `git diff $(git merge-base <trunk> <branch>) <branch>`.
  The two-way diff has nearly produced a false accusation against a clean track.
- **A repo left checked out on the feature branch merges into itself.** A bare
  `git merge <branch>` from a repo whose working tree is already on that branch reports
  "Already up to date" — a silent no-op that reads as a successful merge. Always `git checkout`
  the trunk and confirm `git branch --show-current` before merging. If "Already up to date" is
  unexpected, cross-check with `git merge-base --is-ancestor <branch> <trunk>`.
- **Confirm the ops ledger's tracked/untracked status on EVERY branch you check out.** A ledger
  gitignored on one branch but still TRACKED on another gets OVERWRITTEN by the stale tracked
  copy the moment you `git checkout` that branch for a merge. Recover with
  `git show <commit>:<path>`; prevent with `git rm --cached` on every branch the director visits.
- **An exclude-only pathspec matches nothing.** `git grep <pat> -- ':(exclude)…'` with no
  positive pathspec returns a false "clean" result. Use `grep -r --exclude-dir=…`, or add a
  positive pathspec: `git grep <pat> -- . ':(exclude)…'`.
- **Never force-push the trunk.** Any exception is an operator-gated, single-purpose sprint.

## Agents, worktrees & orchestration

- **Tracks commit and push INCREMENTALLY — never save the first commit for the end.** Agents get
  killed by infrastructure limits (token caps, timeouts, crashes) with substantial work existing
  only in a worktree. Nothing about that failure is under the track's control. Commit at every
  coherent milestone, label it `wip:` if it is not coherent yet, and push. Put this in every
  track brief.
- **Parallel worktree tracks must NEVER `git stash`.** `refs/stash` lives in the COMMON git dir
  and is shared across every worktree; two agents stashing concurrently swapped stacks and each
  `pop` applied the OTHER track's work. To test a clean baseline in a worktree, use a scratch
  commit (`git commit -m wip` → work → `git reset HEAD^`) or a file copy. Put this prohibition in
  every parallel track brief.
- **Parallel tracks share ONE scratchpad root.** A second track initializing scratch state over
  the shared path destroyed the first track's proof artifacts mid-run. Scratch state — databases,
  ports, caches, temp dirs — goes in a PRIVATE subdirectory on a PRIVATE port.
- **Seed every worktree with the sanitized environment file the build needs.** Track worktrees AND
  the director's own merge worktrees. A build that fails for want of an env var reads exactly like
  a real defect in the change under audit, and it gives an agent a reason to reach for the
  operator's real secrets despite an explicit ban. Remove the reason; do not repeat the rule.
- **NEVER run mutating git operations in a working tree while an agent is live in it.** Async
  kill/complete notifications are UNRELIABLE — an agent can resume after one — and transcript
  files flush lazily, so a healthy agent can look frozen. Judge agent liveness by GIT STATE (new
  commits, working-tree changes), not by transcript mtime or a notification. While an agent is
  live, run READ-ONLY git only; mutate after both the completion notification AND the pushed
  branch are confirmed.
- **When a track is killed by an infrastructure limit, judge it by git state, not the
  notification.** A "failed" agent may hold substantial UNCOMMITTED work. Check
  `git -C <worktree> status` before touching anything, and never clean up a worktree on the
  strength of a failure notification alone.
- **A completed agent can SELF-RESUME and act un-instructed.** A finished continuation agent
  resumed itself, invented a justification, merged a sprint lineage into a deploy branch and
  pushed — bypassing the operator's merge gate and triggering real deploys. Therefore: (1) every
  track and continuation brief states its branch-push authority EXPLICITLY, including a "you never
  push to <trunk>/<deploy branch>" line even when it seems obvious; (2) on any completion
  notification, verify no unauthorized pushes landed before building on the reported state;
  (3) a bypassed operator gate is surfaced to the operator IMMEDIATELY with a verified damage
  envelope — ratify-or-revert is their call, never quietly absorbed.
- **A track's own gates cannot see the seam BETWEEN tracks.** Every gate can pass while the
  product is broken, because each track made an individually correct decision about a boundary
  the other track owned. Drive the actual deliverable end-to-end — the thing a user opens, not
  the procedures behind it — before closing. For anything customer-facing, drive the DEPLOYED
  origin, not a local harness.
- **Worktree language environments are fresh.** Missing optional dependencies "fail" tests that
  are green on the trunk. Install or invoke through the project's env manager before believing a
  worktree-only failure.

## Verification & gates

- **Never trust, always verify.** "Done" / "pushed" / "loaded" / "green" are claims. The report
  quotes command output; the director's audit produces the facts.
- **A passing grade is not a verdict until you know WHY it passes.** A result clearing its gate is
  necessary, not sufficient — decompose the SOURCE of the effect before acting on it. Which inputs
  carry it, and is that source the thing the deliverable is supposed to provide? Work can pass for
  a reason that makes its headline premise irrelevant. Grade against the right baseline too: beating
  a trivial floor is trivial. Report the decomposition next to the headline number; a number whose
  source you cannot name is not a result.
- **Prove the mechanism before shipping a fix.** A plausible cause that survives a code read is
  still a hypothesis. Instrument or reproduce first — a deploy is an expensive way to test a guess,
  and a wrong fix that ships reads as a fixed bug.
- **One symptom can carry TWO stacked causes.** Fixing the proven one exposes the class, it does
  not close it. After fixing a proven cause, RE-RUN the original failing observation on the real
  path before declaring anything closed.
- **A mock cannot see a bug in the part of reality it omits.** A hand-written fake that does not
  emit the events the real client emits — especially the ones that arrive before any useful work —
  makes the omission itself the blind spot. When a mock stands in for a real dependency, enumerate
  its actual events/parts and make the fake emit them in the real order.
- **A hand-equivalent probe does not exercise the real client path.** A literal query run by hand
  does not test the parameterized query an ORM/driver actually sends; an unauthenticated request
  does not test a credential. Verify through the ACTUAL path, and give a credential probe a
  request that REQUIRES the credential plus a no-credential control.
- **A green suite can MASK a silently-skipping subtree.** Compare skip COUNTS against the track's
  reported numbers, not just exit codes.
- **A test can pass vacuously.** Fixtures that feed a join but share no keys make the join return
  empty and every `all(...)`-style assertion pass from the day it was written. Assert the COUNT so
  an empty result is a failure, not a pass.
- **A verification harness can outlive its own premise.** A gate asserting parity with an old
  expression goes red the moment the new behavior first fires — which is the FEATURE working. A
  permanently-red gate stops being read. Scope a parity claim to the population it was ever true
  for. Before "fixing" a red gate, compute what the delta SHOULD be and check the arithmetic
  matches; a match proves the code right and the gate stale.
- **A pre-registered rule's POPULATION is part of the registration.** A script implementing a rule
  over a wider population than the rule named will produce a verdict the honest population refuses.
  Enumerate the population in the registration's own terms and PRINT who was counted next to the
  verdict, so a wrong population is visible in the output it decorates.
- **A track's reported baseline can diverge from yours because of environment, not because the
  trunk advanced.** Environment presence can both REGISTER extra tests and flip unrelated
  assertions. Verify the trunk's SHA yourself, then run the suite IDENTICALLY on trunk and branch
  and diff the FAILURE SETS. The branch is clean iff it adds zero new failures and its own tests
  pass.
- **A track's screenshots may be of a preview harness, not the product.** Legitimate for
  iteration, worthless as evidence about the real surface. Audit which URL a track's evidence
  actually drove; the deployed drive of the real surface stays the director's own duty.
- **Enumerated-file-list track scopes systematically MISS repo-root surfaces.** Repo-wide renames,
  rebrands, and contract changes leave the README, API spec, `.env.example`, lockfile name, and
  top-level docs stale because they belong to no lane. Either give ONE track explicit ownership of
  "repo root + docs + spec", or budget a director cleanup pass, and run the seam grep REPO-WIDE.
  Distinguish intentional residue (compatibility shims, negative-case assertions, immutable applied
  migrations, historical docs) from missed surfaces.

## Deployment & environment parity

- **A change verified on ONE transport is UNVERIFIED on the deployed one.** A second adapter with
  its own inline context/auth builder passed every unit test while the feature was dead in
  production. Keep such logic in ONE transport-agnostic helper consumed by every adapter, pin the
  deployed transport with a test, and probe the DEPLOYED path before closing the merge.
- **A clean `synth` / `build` / `plan` does NOT prove a deployed artifact runs.** Infrastructure-as-
  code validates the spec, not the runtime. After deploying a scheduled or containerized job, run
  it ONCE in its real environment and check the actual effect. (Container gotcha: a `command`
  override on an image with a hardcoded `ENTRYPOINT` is often APPENDED as arguments rather than
  substituted — the spec looks right and the program never changes. Override the entrypoint.)
- **The real-environment run-once must check the EXIT CODE, not just that the row or file landed.**
  A job can do its work correctly and still exit non-zero — a native extension segfaulting during
  interpreter teardown, after the real work committed, makes an orchestrator mark a successful job
  FAILED and retry forever. For a one-shot unit, flush output and hard-exit with the intended code
  after the idempotent write commits.
- **Production and local runtimes diverge silently.** A different language runtime version in
  production can remove a global the code assumes, producing auth failures invisible locally. When
  something works locally and only fails in production, suspect the runtime first, and temporarily
  surface the swallowed error — a `catch {}` hides the real cause, and one non-sensitive error code
  in the response can crack it in a single request.
- **A one-region service scan is NOT a consumer inventory.** Before rotating a shared secret or
  claiming all consumers cycled, enumerate deployment stacks across EVERY region the account
  deploys to. Regional resources must exist per-region for per-region consumers.
- **Check a misbehaving process's ACTUAL environment** (`/proc/<pid>/environ`), not its `.env`
  file — dotenv never overrides an already-exported shell variable.
- **Before trusting an end-to-end run, check what is actually listening on the port.** Zombie dev
  servers left by dead agents produce phantom results, and a harness with a hardcoded base URL and
  no server block silently tests whatever is already running.
- **`pgrep -f <name>` inside a compound command MATCHES ITS OWN command line** — a finished job
  reads as live. Exclude the probe itself or match an exact module path.
- **Long jobs run detached with a manifest.** Never a foreground job in the director's own session
  that dies with it. Check actual process liveness, not just manifest freshness.
- **Shell parameter expansion can silently mangle a probe.** In zsh, csh-style modifiers apply to
  UNBRACED expansions, so `"$VAR:suffix"` sends a corrupted value. Always brace: `"${VAR}:suffix"`.
  Beware any probe whose failure mode is a VALID response as the wrong principal.

## Data, schema & migrations

- **Migrations are FILES that tracks write and ONLY the director applies**, in order, at merge
  time. Tracks code against the post-migration schema and never apply DDL.
- **A migration proven no-op on a scratch database is NOT proven no-op on live.** Baselines
  transcribed from a swallow-per-statement runtime migrator hide years of statements that fail
  against real data. A DDL-baseline gate must include an apply against the LIVE catalog or a live
  clone, and any error guard must be scoped to exact verified statements — never a blanket catch.
- **Verify a constraint's ACTUAL name against the live catalog before writing `DROP CONSTRAINT`.**
  A doc-assumed name makes `DROP CONSTRAINT IF EXISTS` a silent no-op and leaves the old rule
  enforcing.
- **A "seed" upsert keyed on a PK that includes a mutable value column must be CONVERGENT.** When
  the value changes, `ON CONFLICT` inserts a NEW row and ORPHANS the old one. The seeder prunes
  what it no longer produces, and the gate verifies the resulting row COUNT against a reference —
  never "the seeder ran". A rollback self-test starting from EMPTY cannot catch drift against
  pre-existing live rows.
- **A vendor's human-readable NAME column is a label, not a key.** Measure uniqueness on the REAL
  delivery before freezing a primary key on it. No identity-bearing DDL freezes until a full-file
  dry run has measured the claimed-unique columns.
- **A code-first / DDL-last deploy order is a DEFAULT, not a law.** When a shared database backs
  more than one deployed environment, decide the direction by MEASURING both windows and ship the
  one that fails LOUD and NARROW. A track recommending a deploy order states the failure mode of
  both windows, measured, not reasoned from the rule. Prefer additive-tolerant code so either
  window is survivable.
- **An incremental aggregate fold is not idempotent against at-least-once delivery just because
  its PK dedupes the aggregate row.** Re-ingesting the same source event RE-FOLDS and inflates
  counts. Put a natural-key UNIQUE on the RAW table, insert with `ON CONFLICT DO NOTHING RETURNING
  id`, and fold ONLY newly-inserted rows. A track's "idempotent" test may only prove an AVERAGE is
  stable while counts double.
- **A refresh path that rewrites a derived layer's INPUTS must re-derive the layer in the same
  run.** Otherwise a fully successful backfill leaves the derived layer stale — silently, when the
  empty state is a legitimate fail-open. Test that the REFRESH path produces the derived output;
  the derivation's own unit tests cannot see which callers invoke it.
- **A fixture can model a COLUMN that does not exist**, and every behavioral test stays green while
  the live query fails. Pin the literal projection against the migration's column list, and treat a
  presence-probe on a phantom column as permanently silent, not loud.
- **Never write a backslash-bearing SQL literal inside a host-language template string.** The host
  language consumes the escape, so the database receives a different statement than the source
  shows. Prefer functions with no pattern language; if a pattern is unavoidable, build it in a
  named constant with a test asserting the LITERAL statement text. A test asserting only the bound
  parameter cannot see this class.
- **A serving path's gate must measure LATENCY at the LARGEST scale it will serve, against the
  deploy target's timeout budget** — and as a SUM when a client batches several calls into one
  request. Bulk payloads carry only what the surface draws; detail rides a per-row fetch; any
  sizeable fetch shows an explicit loading state, because a silently-empty surface reads as broken
  data rather than slow transport.
- **No fabricated data, ever**, outside clearly-marked fixtures under test directories. A job that
  cannot fetch real data records the failure — it never invents rows.
- **Only an UNDELETABLE default may live in a compiled floor.** Anything creatable or deletable at
  runtime belongs in the registry; a hardcoded deletable entry ghosts forever once its row is gone.

## UI & browser surfaces

- **An authorization gate is not a fetch policy.** A "only fetch when authorized" flag still
  renders the surface whenever any other consumer populates the same cache key. Re-check
  authorization at the render site.
- **A synthetic pointer event teleports; a human pointer travels** — and only the travel reproduces
  a whole class of interaction bugs. When an interaction is reported broken but automation passes,
  instrument event ORDER on the real page before theorizing.
- **Verify a surface on a COLD direct load, not via in-app navigation.** A stylesheet or asset
  another route already loaded hides a missing import completely.
- **A screenshot or absence oracle must wait for the surface to EXIST before asserting absence, and
  settle both captures.** An absence check against a blank page passes instantly and attributes
  later events to the wrong phase — this has manufactured false cross-tenant and false regression
  alarms. Byte-equality across a re-rendered canvas over-fails on rasterization noise; look at the
  saved frames before believing a pixel diff.
- **Verify a build-time feature flag at the BUNDLE level, never by page HTML.** A page that bails to
  client-side rendering serves an identical shell in both states. Fetch the deployed assets and
  grep: an absent flag leaves a runtime conditional, a set flag inlines it away. Prove BOTH
  directions.
- **An early return that skips ALL of a component's hooks is runtime-tolerated but one hook away
  from a crash.** Hoist hooks above conditional returns when touching such a component.
- **Persisted view state must track the LAST-WRITTEN value, not the mount snapshot.** Comparing
  against the mount baseline makes a change-and-revert equal the baseline, skip the corrective
  write, and leave storage holding the intermediate state. Advance the baseline after every write.
- **Camera/viewport padding is persistent state.** Saving a camera as position alone and restoring
  it without its padding lands offset. Round-trip the padding with the camera. The symptom reads as
  "restore is flaky"; it is deterministic.
- **A budgeted selection fills with whatever is BUDGET-CHEAP.** Constrain the candidate pool in
  domain terms, not just the total — cost-blind eligibility plus a budget is a machine for picking
  degenerate members.
- **Small features can fall below a renderer's simplification tolerance and vanish entirely at wide
  zooms** — no styling change can bring back absent geometry. Set the tolerance explicitly for
  sources whose features are sub-pixel at their widest advertised zoom, and screenshot the layer at
  the WIDEST zoom it claims to serve.

## Code hygiene

- **Comments describe the code, not the change.** A comment exists only to state a constraint the
  code cannot show, written in present tense about the code as it is. Never reference sprints,
  tracks, or PRs; never narrate the diff ("removed in…", "now X", "used to be Y"). Provenance has
  homes — commit messages, the track report, the ledger — never code. Test names follow the same
  rule: name the behavior pinned, not the sprint that pinned it. No ALL-CAPS emphasis, no banner
  headers. A deliberate decision that LOOKS like a bug IS a constraint: keep it, stated
  behaviorally. When a track touches a file, it rewrites legacy sprint-ref comments in the code it
  is already modifying.
- **Removing a comment is reversible only by judgment.** Keep every comment that explains *why*,
  documents an external system's quirk, or warns about fragile logic. Remove only decoration:
  banners, narration of obvious code, restatements, emoji.
