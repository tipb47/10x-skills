# Track rules

Every track obeys these rules and its own `TRACK-*.md`. Nothing else applies to a track.

## Start

1. **Check out your branch first.** Cut `sNN/<slug>` from `origin/sNN/integration`, then
   confirm it with `git branch --show-current`. A worktree can start at `{{TRUNK}}`, so a
   brief read before checkout can be a stale copy.
2. **Read your `TRACK-*.md` and this file once.** Do not re-read them. Read the
   `DESIGN.md` sections your brief names, and nothing else unless the brief points at it.
3. **Never ask a blocking question.** On ambiguity, decide per `DESIGN.md` and log the
   decision in your report.

## Git

- **Push authority:** you push your own `sNN/<slug>` branch and nothing else. You never
  merge. You never push `sNN/integration` or `{{TRUNK}}`.
- **Commit and push incrementally.** Commit at every coherent milestone and push. Label
  unfinished work `wip:`. Infrastructure limits can kill you with work still in the tree.
  {{CI_SKIP_RULE}}
  <!-- init: if CI runs on every branch and cannot change, write "Put `[skip ci]` in the
  subject of every `wip:` commit." Otherwise drop this line. -->
- **Never `git stash`.** `refs/stash` is shared across every worktree, so a `pop` can
  apply another track's work. For a clean baseline, use a scratch commit or a file copy.
- **Stage explicit paths.** Never `git add -A` or `git add .`.
- **Always `git -C <absolute-path>`** in a worktree.
- Commit subjects carry the tag `[sNN/<slug>]`.

## Scratch state

- Put databases, caches, temp dirs, and servers in a **private subdirectory on a private
  port**. Parallel tracks share one scratchpad root.
- You cannot write the shared ops folder from a worktree. Put any file the brief asks for
  inside your worktree, and name it in your report.
{{WORKTREE_SEED_RULE}}
<!-- init: if the project needs env files or credentials to build, add: "Seed your
worktree with the sanitized <file> before building (<copy command>)." Otherwise drop this
line. -->

## Verification tiers

- **You run:** typecheck, lint, and the tests that cover what you changed:
  {{TARGETED_TESTS}}. Run them as often as you need.
  <!-- init: the project's targeted-test pattern from the interview, as exact commands. -->
- **The director runs:** the full suite, {{FULL_SUITE}}, on the integration branch,
  once per distinct tree. Do not run it yourself unless your brief says so.
- **Every background command carries a deadline.** Wrap it in a timeout (`timeout` on
  Linux, `gtimeout` on macOS; elsewhere record the deadline and poll), or record the
  deadline in its manifest.
- Quote actual command output. "Done", "pushed", and "green" are claims until output
  backs them.

## Comments

Comments describe the code, not the change. A comment states a constraint the code cannot
show, in present tense. Never reference sprints, tracks, or PRs. Never narrate the diff
("removed in…", "now X", "used to be Y"). Test names follow the same rule. No ALL-CAPS
emphasis and no banner headers in comments.

- BAD: `// The legacy narrowing arg was removed in s43 (Track C).`
- GOOD: `// Single-org scoping comes from the request context; a client-supplied org arg
  here would bypass tenant gating.`

A deliberate decision that looks like a bug is a constraint. Keep it, stated
behaviorally: `// Intentionally serves the stale value; upstream TTL is 45s.`

## Report

Your report is your **final message**. Never write it to a file outside your worktree.
Use these sections:

1. What shipped.
2. Branch and commits, with the last pushed SHA.
3. Verification output, verbatim.
4. Files changed.
5. Decisions and open questions.
6. What the director must verify by hand.

A **relay brief** replaces the report when your context runs heavy (see your
`TRACK-*.md`). It has: branch and last pushed SHA, steps done, steps left, gate status,
and decisions so far.
