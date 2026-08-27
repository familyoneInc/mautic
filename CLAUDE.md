# CLAUDE.md

Read and follow all instructions in `./AGENTS.md`.

This project uses AGENTS.md as the single source of truth for all AI coding agents.

<!-- family.one shared operating rules — mirrored from family-one-platform/CLAUDE.md.
     These apply in EVERY family.one repo. Edit the copy in family-one-platform first,
     then re-mirror, so the copies cannot silently diverge. -->

# Plan before you act: walk with your head up

Survey the terrain before the first step. A plan must state **what you expect to happen** —
otherwise divergence is invisible, and you learn the plan was wrong only from the damage.

**Do not discover blockers one at a time by walking into them.** An identity migration was run
reactively and each blocker stopped the work in turn: 142 legacy users vs 2 new, then a missing
sub->sub mapping, then `partner_user` rows keyed to legacy subs, then NOT NULL columns legacy could
not fill, then a missing IP for server-generated ledger events, then the ability to fabricate
consent. One dependency survey up front would have surfaced most of them together.

**A survey does not only find blockers — it corrects the framing.** When that survey was finally
run it showed `program_entry.visitor_id` is a NOT NULL FK to `visitor`, and visitors migrate lazily
on login: the database already enforced the "only migrate on login" rule. The movable set was 7,036
rows, not the 220,379 that had been planned and sized for. The same survey showed `consent_event_id`
is NOT NULL on four tables (`program_entry`, `lead`, `billing_event`, `partner_consent`) — the
consent ledger was the true critical path, found late, after work had been queued behind the wrong
dependency.

**Rehearse in dev before a production write, especially an irreversible one.** A dev rehearsal
caught two defects that would otherwise have been permanent: the S3 consent ledger is Object Lock
COMPLIANCE with 2,555-day retention, which no one — including the root account — can shorten or
delete. A wrong payload there is wrong for seven years and can only be corrected by appending.

## Legal, consent and privacy are planning inputs, not review steps

The team has spent significant effort building these rules. Check them **before** acting. This is
what the knowledge-base query requirement in the Canon section is for, and planning is when it
happens — not after the write.

Reading live infrastructure config is not a substitute for reading our own recorded decisions. A
runbook written that way was wrong three ways, including an instruction that contradicted a standing
privacy decision: IdP attribute mapping is email-only on purpose, because pulling names and pictures
from an IdP collects personal information we were not directed to collect.

## When reality diverges from the plan

1. **STOP.** Do not patch forward and do not improvise past it. The plan's prediction just failed;
   anything built on it now inherits an unknown cause.
2. **Investigate until you understand the actual cause** — not a story that merely fits the symptom.
3. **Reformulate the plan** with the new fact in it, and state what you now expect.

The surprise is the signal, not the inconvenience. Verify the mechanism before building on it — see
*Verification: never trust a green run* above.

## When a plan does not satisfy the rules

- **Stop and examine why a non-compliant plan was formed at all.** It came from somewhere: a stale
  assumption, an unread decision, a rule nobody surfaced. Find that, or you will re-plan into it.
- **Then adjust the plan.** The rules are the constraint; the plan is the variable.
- **If no appropriate adjustment exists** — the work genuinely needs doing and genuinely does not
  fit the rules — examine *how* that is so, and whether the rules themselves need expanding. That is
  a deliberate decision surfaced to the owner. Never a silent exception, and never "it had to ship"
  as the reason a rule moved.
