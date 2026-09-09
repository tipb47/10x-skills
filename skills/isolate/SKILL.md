---
name: isolate
description: Move THIS session into its own git worktree and branch so it can share a repo with other agents without touching their diff. Use for /isolate [off <branch>] (enter isolation), /isolate done (land the work, remove the worktree), /isolate status (who is where — every worktree, branch, and dirty tree). Also trigger without the slash command whenever the user says other agents or sessions are working on this repo, asks you to use a worktree, or warns about overlapping with someone else's changes.
argument-hint: [off <branch>] | done | status
---

# isolate — one session, one worktree

Several agent sessions on one checkout share one working tree and one index. They edit
the same diff, stash each other's work, and commit each other's files. This skill takes
this session OUT of the shared checkout: its own worktree, its own branch, and a set of
standing rules that hold until `/isolate done`.

Three modes: **enter** (`/isolate`, optionally `off <branch>`), **done**, **status**.
Bare `/isolate` with no mode word is enter.

---

## Enter

### 1. Resolve the base

The base is the branch this session's work will land on. Resolve it in this order and
stop at the first hit:

1. **Explicit argument.** Parse a ref out of the free text: `/isolate staging`,
   `/isolate off staging`, `/isolate from release/2.4`, `/isolate on main`. Verify it
   exists (`git rev-parse --verify <ref>`; try `origin/<ref>` if the local ref is
   missing). A non-existent ref is a stop, not a guess.
2. **Context.** A branch the user named earlier in this session as the thing being
   worked on ("we're on the staging branch", "branch off the sprint integration
   branch").
3. **Current HEAD of the shared checkout** (`git branch --show-current`). State the
   choice in one line; no question.

If the argument is ambiguous or the context names more than one candidate, ask, with
the recommended answer first. Never fetch-and-reset the base; use it as it is locally.

### 2. Look before touching

Run and read, in the shared checkout:

- `git status --porcelain` — the dirty tree. Classify every entry: **mine** (this
  session edited it earlier in the conversation, before `/isolate`) or **not mine**
  (everything else — assume another agent owns it).
- `git worktree list` — who already has a worktree. Do not reuse or enter any of them.
- `git stash list` — report the count. Never pop, never drop, never add.

### 3. Create and enter

Name: `iso/<topic>`, where `<topic>` is a 2–4 word kebab-case slug of this session's
task. No task yet: `iso/<yyyymmdd>-<4 random chars>`. If the branch already exists,
append `-2`, `-3`.

Location: `.claude/worktrees/<topic>` under the repo root. Add `.claude/worktrees/` to
`.git/info/exclude` if it is not already ignored — the local exclude, never the tracked
`.gitignore`.

```bash
git -C <repo-root> worktree add <repo-root>/.claude/worktrees/<topic> -b iso/<topic> <base>
```

Then switch the session in:

- **Claude Code:** `EnterWorktree` with `path: <repo-root>/.claude/worktrees/<topic>`.
- **Any other runtime:** there is no session switch. Every command from now on uses
  `git -C <worktree>` and absolute paths; a plain `cd` does not persist and must not be
  relied on.

### 4. Carry only what is mine

Files classified **mine** in step 2 move to the worktree and leave the shared checkout:

```bash
git -C <repo-root> diff -- <path>... > /tmp/iso-carry.patch   # tracked edits
git -C <worktree> apply /tmp/iso-carry.patch
cp <repo-root>/<new-file> <worktree>/<new-file>               # untracked new files
git -C <repo-root> checkout -- <path>...                       # revert tracked in shared
rm <repo-root>/<new-file>                                      # remove untracked in shared
```

Explicit paths every time. No `git add -A`, no `git stash`, no `git checkout .`. Verify
with `git -C <repo-root> status --porcelain` that only **not mine** entries remain.

Everything **not mine** stays exactly where it was. Do not read intent into it, do not
"clean it up", do not commit it for anyone.

### 5. Report and adopt the standing rules

One short block in chat: worktree path, branch, base, files carried over, files left
behind in the shared checkout (paths, not judgment), stash count, other worktrees seen.

From here until `/isolate done`, these rules hold for the whole session, including any
subagents it spawns:

- **Every git command is `git -C <absolute worktree path>`.** No exceptions, no bare
  `git`, no reliance on cwd.
- **The shared checkout is read-only.** Never `checkout`, `switch`, `reset`, `stash`,
  `clean`, `merge`, or `rebase` in it. Never edit files there. Reading is fine.
- **Never check out the base branch anywhere.** It belongs to whoever has it now.
- **Stage explicit paths.** Never `git add -A` or `git add .`.
- **Never touch a worktree, branch, or stash this session did not create.** No
  `git worktree prune`, no `git branch -D` on others' branches, no `git stash` at all.
- **Commit on `iso/<topic>` only.** A push goes to `origin iso/<topic>`, never to the
  base.
- **Subagents inherit the worktree.** Spawn them with cwd pinned to it, or with their
  own worktree branched from `iso/<topic>` — never from the shared checkout.

---

## Done

Land the work, then leave nothing behind. Run each step; stop and report on the first
failure instead of continuing.

1. **Commit what is left.** `git -C <worktree> status --porcelain`. Stage by explicit
   path, commit. Anything you would not want landed (scratch files, local config) gets
   deleted, not committed.
2. **Verify.** If the project has a test or build command that the session has been
   using, run it and judge by exit code. A failing check stops here: report, do not
   land.
3. **Pick the landing path.**
   - **Remote exists** (`git -C <worktree> remote get-url origin` succeeds): push and
     open a PR.
     ```bash
     git -C <worktree> push -u origin iso/<topic>
     gh pr create --base <base> --head iso/<topic> --fill   # GitHub remotes only
     ```
     No `gh`, or a non-GitHub remote: push, print the branch name and the compare
     URL if one can be derived, and say the PR is on the user.
   - **No remote**, and the base is not checked out in any worktree
     (`git worktree list` shows no entry on `<base>`): fast-forward it without
     checking it out.
     ```bash
     git -C <repo-root> merge-base --is-ancestor <base> iso/<topic> && \
     git -C <repo-root> push . iso/<topic>:<base>
     ```
     Not a fast-forward, or the base IS checked out somewhere: do not merge. Leave the
     branch, print the branch name, and hand the merge to the user.
4. **Leave the worktree first.** Only after step 3 succeeded. Claude Code:
   `ExitWorktree` with `action: "keep"` — it restores the session cwd and leaves the
   directory for the next step. Other runtimes: nothing to do; the next steps use
   `git -C <repo-root>` anyway.
5. **Remove the worktree.**
   ```bash
   git -C <repo-root> worktree remove <worktree>
   ```
   Do not run `git worktree prune`; other sessions may have stale entries they still
   count on.
6. **Delete the local branch** only when it is safe: `git branch -d iso/<topic>`
   (lowercase `-d`; it refuses unmerged branches). After a push, confirm
   `git branch -r --contains iso/<topic>` lists `origin/iso/<topic>` before using
   `-D`. If the branch was neither pushed nor merged, keep it and say so.
7. **Report.** Branch, where it landed (PR URL, or fast-forwarded into `<base>`, or
   left for manual merge), and confirmation that `git worktree list` no longer shows
   this session's entry.

**Abort** — the user wants the work discarded rather than landed: say what will be lost
(uncommitted files, commits not on the base), get an explicit yes, then
`git worktree remove --force <worktree>` and `git branch -D iso/<topic>`. Never on your
own judgment.

---

## Status

Read-only. Never writes, never mutates. One compact block:

- **Worktrees** — every entry from `git worktree list --porcelain`: path, branch, and
  whether its tree is dirty (`git -C <path> status --porcelain | wc -l`). Mark this
  session's own entry.
- **Shared checkout** — branch, dirty file count, stash count.
- **`iso/*` branches** — local and remote, each with ahead/behind against its
  merge-base target where derivable, and whether it still has a worktree.
- **This session** — isolated or not; if isolated, worktree path, branch, base,
  commits ahead of base.

End with one sentence: the single thing that most needs attention (a dirty worktree with
no session, an `iso/*` branch with no worktree, nothing).

---

## Non-negotiables

- **Stash is never the answer.** Not to carry work, not to clean up, not to "just move
  it aside". The stash is shared across every worktree of the repo and every agent on
  it; it is the mechanism of the collision this skill exists to prevent.
- **A dirty diff you did not make is someone's work in progress.** Leave it. Report
  it. Do not interpret it.
- **Refuse silently-wrong success.** "Already up to date" on a merge you expected to
  change something, a `ROLLBACK`-style warning, a push that says everything is
  up-to-date — each is a stop-and-check, not a done.
- **Do not become the sprint.** Multi-track fan-out is `/plan` and `/sprint`; they
  already isolate their subagents. `/isolate` isolates this one session so it can
  coexist with them and with other top-level sessions.
