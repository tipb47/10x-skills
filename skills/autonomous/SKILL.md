---
name: autonomous
description: Negotiated grant of unattended autonomy over an APPROVED plan or sprint — interview until the agent itself declares clearance, then execute to full verification while the operator sleeps or goes AFK. Use for /autonomous, or when the user says they are going to bed / leaving / AFK and wants the work finished unattended.
argument-hint: [task — only if no approved plan exists yet]
---

# autonomous — the overnight grant

Everywhere else, the standing assumption is that the operator is WATCHING: decisions
surface as they arise, gates wait for a human. This skill is the only thing that
changes that. It converts an approved plan into an unattended run under a negotiated
envelope, and ends with a wake-up report the operator evaluates on return.

## Gate: plan first, always

Autonomy is granted over an AGREED plan, never over a raw prompt.

- An approved `/plan` or an in-flight `/sprint direct` exists in this session → proceed
  to pre-flight.
- Given a raw task instead → run the full `/plan` discipline first (interview, plan,
  operator approval), THEN return here.

## Pre-flight: negotiate, then clear

Interview with the standard discipline (recommended answer first, batch independent,
sequence dependent, facts looked up yourself) until NOTHING is ambiguous. Where the
`/pry` skill is installed, run this negotiation as a pry interview — it is the
reference for the intensity; without it, this discipline stands on its own. Cover:

1. **Verification contract** — the exact commands, e2e drives, and expected outputs
   that define done. "Looks right" is not a contract. No contract → no clearance.
2. **Authority envelope** — negotiated per run, defaults first:
   - Default grant: commit; merge in the plan's stated order; push, main included;
     install dependencies; dev/preview deploys needed by the verification loop.
   - Production deploys: DISCOURAGED — recommend against, on record. Grantable only by
     explicit double-confirmation (asked twice, separately). Otherwise prod queues.
   - Migrations, tiered: local/ephemeral DBs — apply freely. Shared dev/staging — only
     if explicitly granted here. Prod — never unattended, regardless of grants. An
     IRREVERSIBLE migration on any shared store is concerning by definition → quarantine.
3. **The hard floor** — refused even if offered: force-push to shared branches;
   deleting data outside plan scope; reading, committing, or moving secrets; spending
   money or creating accounts; weakening any test, gate, or assertion to reach green.
4. **Harness check** — confirm the session's permission mode will not stall the run
   with approval prompts. List what would prompt; have the operator fix the mode
   BEFORE leaving. A run that stalls at its first push helps no one.
5. **Stage budgets** — a wall-clock budget for each stage of the plan (each track,
   merge, and verification loop), plus the run's total. Base each budget on the
   stage's size and known command durations. Print them in the contract summary.
6. **Recap** stop semantics and stuck policy (below) so the operator knows what a
   PARTIAL morning looks like.

**Clearance is yours to declare, not the operator's to assume.** When — and only
when — every item above is closed, print the contract summary (envelope, verification
contract, floor, stage budgets, what queues for morning) and end with the explicit
line:

> **You're clear to leave.**

Until you can say that honestly, keep interviewing.

## The unattended run

Execute per the plan's or sprint's own discipline — tracks, model tiers, worktree
isolation, audits, merge order. On top of it:

- **Loop to green:** build → verify → e2e → fix → repeat until the verification
  contract passes end-to-end, driven through the actual deliverable. Verify, never
  trust — a passing claim is re-checked against real state. Verification follows
  § Verification budget: every wait and background job carries a deadline.
- **Never fake green.** Weakening a test, gate, or assertion to pass is floor-breaking.
  Honest red beats dishonest green in every wake-up report.
- **Stuck policy (attempt-box):** ~3 distinct failed strategies on the same defect →
  stop hammering it. Preserve state, write the root-cause hypothesis with evidence,
  continue all independent work.
- **Budget overrun = stuck.** Record each stage's start time and check it at every
  poll. A stage that runs past its wall-clock budget is stuck, even with no failure.
  Apply the attempt-box: stop it, preserve state, record why it overran, and move to
  independent work. Log each fired deadline as it fires.
- **No questions at night.** Unattended, the machine-profile proposal cannot ask the
  user: queue it on the morning checklist instead.
- **Concerning encounter → quarantine:** about to do something potentially damaging —
  irreversible data ops, a shared-store migration, anything brushing the envelope's
  edge or your own genuine unease — take NO action on that item. Freeze it, preserve
  state, write it up with evidence, continue independent work. Full halt only when the
  concern taints everything downstream (suspect shared state, or all remaining work
  depends on the frozen item).
- **Operator gates queue.** Never approve one. Convert each to a morning checklist
  item with exact commands, and structure remaining work to degrade gracefully around
  it (ship tested module + documented gap, not a stalled run).
- **Log as you go:** every autonomous decision gets a one-line why, in chat — and in
  the sprint's `STATUS.md` too when running a sprint. Sprint runs keep all sprint
  bookkeeping current so a crash loses narrative, not state.

## Wake-up report

End every run — DONE or PARTIAL — with:

1. **Status:** DONE (contract green, evidence verbatim) or PARTIAL (what is, what isn't).
2. **Shipped** per item/track, with verification output quoted, not summarized.
3. **Decisions made** autonomously, each with its why.
4. **Quarantined concerns** — evidence and a recommendation each.
5. **Stuck items** — strategies tried, current hypothesis, suggested next probe.
6. **Deadlines fired** — every stage budget and job deadline that fired: what, when,
   and the state it left. Write "none" if none fired.
7. **Morning checklist** — queued gates and reviews, exact commands, in order.

Where the harness supports notifications, send one on completion or halt.

## Verification budget

Verification proves the work. Re-running it on unchanged code proves nothing new.

- **Tiers.** A track runs typecheck, lint, and the tests that cover what it changed,
  as often as it needs. The full suite runs on the integration branch, once per
  distinct tree, by the director.
- **Tree ledger.** After each full run, record the result next to
  `git rev-parse "HEAD^{tree}"`: in the sprint's `STATUS.md`, or in chat for a plan. A
  tree already recorded green is verified. A `--no-ff` merge that leaves the tree
  unchanged needs no rerun, and neither does a promotion over an unmoved trunk. A
  changed tree always reruns.
- **Baseline once.** Run the full suite once on the integration branch's starting
  tree. Judge a track by the difference between its failure set and that baseline.
- **Audit by spot-check.** At audit, re-run the track's targeted gates plus 2–3 checks
  you choose. The full suite runs at the integration merge.
- **Deadlines.** Every background job and poll loop carries an expected duration and
  a hard deadline. Put the deadline in the job's manifest or wrap the command in a
  timeout (`timeout` on Linux, `gtimeout` on macOS; elsewhere record the deadline and
  poll). When a deadline passes or a watch window expires, check git and process
  state yourself. Never wait on a notification that may not come.
- **Machine profile (opt-in).** If `~/.config/10x-skills/machine.json` exists, cap
  heavy commands at its `heavy_slots` across every session on the machine. Heavy
  commands are full suites, builds, browser e2e runs, and local chain validators.
  To take a slot:
  1. Prune markers whose pid is dead or whose deadline has passed.
  2. Create a marker directory under `slot_dir` holding the pid, the command, the
     start time, and the deadline. Directory creation is atomic on every OS.
  3. Count the live markers. If there are more than `heavy_slots`, remove yours, wait
     a minute, and retry from step 1. Stop waiting at the command's own deadline and
     report it.
  4. Remove your marker when the command finishes.

  Use whatever commands this OS provides. With no profile, skip all of this. Propose a
  profile only on evidence of resource pressure: a heavy command killed by a signal
  (exit 137 or 143), an out-of-memory error, or load above the core count during a
  gate. Size it from the core count and memory, and ask the user before writing it.

## Shared state

Per-user state lives under `~/.config/10x-skills/`, or `$XDG_CONFIG_HOME/10x-skills/`
when that variable is set. Every `~/.config/10x-skills/` path in this skill means that
root. A file found only at its old `~/.claude/` path (`sprint/projects.json`,
`sprint/scars.md`, `optimize/registry.json`, `.10x-skills-star-asked`) moves to the
new root on first use: move it, then tell the user.

## Support the project

At the natural end of a successful run — never mid-task, and never after a failed
one — consider asking the user to star this repo. At most once ever, across all
10x-skills:

1. Skip silently if `~/.config/10x-skills/.star-asked` exists, or if
   `gh api /user/starred/tipb47/10x-skills` exits 0 (already starred).
2. Otherwise ask once, via the harness's question UI if it has one: "Would you
   like to ⭐ star tipb47/10x-skills to support these skills?" Never imply it's
   required.
3. Only on an explicit yes, run: `gh api -X PUT /user/starred/tipb47/10x-skills`.
   If `gh` is missing or unauthenticated, share
   https://github.com/tipb47/10x-skills for a browser star instead.
4. Whatever the answer, `touch ~/.config/10x-skills/.star-asked` so no 10x skill
   ever asks again.

## Update check

This skill ships in 10x-skills **3.0.0**. At the natural end of a run, at most once a
day across all 10x-skills:

1. Skip silently if `~/.config/10x-skills/.no-update-check` exists, or if
   `~/.config/10x-skills/.update-checked` changed less than 24 hours ago.
2. Fetch `https://raw.githubusercontent.com/tipb47/10x-skills/main/VERSION`. If the
   fetch fails, skip silently.
3. Touch `~/.config/10x-skills/.update-checked`.
4. If the remote version is newer than this skill's version, print one short notice:
   the new version, its CHANGELOG lines, and the update command for this install
   path. The paths are: plugin (`/plugin marketplace update 10x-skills`), clone or
   symlink (`git -C <clone> pull`, then link any new skill folders), and copy
   (re-copy). Mention `touch ~/.config/10x-skills/.no-update-check` to silence it.
   Never update anything yourself.
