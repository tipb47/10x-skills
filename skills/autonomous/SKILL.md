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
sequence dependent, facts looked up yourself) until NOTHING is ambiguous. Cover:

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
4. **Runtime check** — confirm the session's permission mode will not stall the run
   with approval prompts. List what would prompt; have the operator fix the mode
   BEFORE leaving. A run that stalls at its first push helps no one.
5. **Recap** stop semantics and stuck policy (below) so the operator knows what a
   PARTIAL morning looks like.

**Clearance is yours to declare, not the operator's to assume.** When — and only
when — every item above is closed, print the contract summary (envelope, verification
contract, floor, what queues for morning) and end with the explicit line:

> **You're clear to leave.**

Until you can say that honestly, keep interviewing.

## The unattended run

Execute per the plan's or sprint's own discipline — tracks, model tiers, worktree
isolation, audits, merge order. On top of it:

- **Loop to green:** build → verify → e2e → fix → repeat until the verification
  contract passes end-to-end, driven through the actual deliverable. Verify, never
  trust — a passing claim is re-checked against real state.
- **Never fake green.** Weakening a test, gate, or assertion to pass is floor-breaking.
  Honest red beats dishonest green in every wake-up report.
- **Stuck policy (attempt-box):** ~3 distinct failed strategies on the same defect →
  stop hammering it. Preserve state, write the root-cause hypothesis with evidence,
  continue all independent work.
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
  STATE.md too when running a sprint. Sprint runs keep all sprint bookkeeping current
  so a crash loses narrative, not state.

## Wake-up report

End every run — DONE or PARTIAL — with:

1. **Status:** DONE (contract green, evidence verbatim) or PARTIAL (what is, what isn't).
2. **Shipped** per item/track, with verification output quoted, not summarized.
3. **Decisions made** autonomously, each with its why.
4. **Quarantined concerns** — evidence and a recommendation each.
5. **Stuck items** — strategies tried, current hypothesis, suggested next probe.
6. **Morning checklist** — queued gates and reviews, exact commands, in order.

Where the runtime supports notifications, send one on completion or halt.
