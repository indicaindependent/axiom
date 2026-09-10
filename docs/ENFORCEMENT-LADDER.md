# Enforcement ladder for the general channel

Axiom's default posture is *escalate, do not enforce*: deterministic rules hold a message
for a human and the classifier's verdict is advisory. This document covers the single
documented exception — a small set of **solicitation species** in the general channel that
the operator has explicitly authorised the bot to act on, and the ladder that bounds how
hard it may act.

## Why an exception exists

Three species kept grading "clean" or "soft" through the classifier and the carve-outs
that protect ordinary shop talk:

1. **Recruiting with a DM funnel** — "looking for a developer / business partner…
   let's talk in DM". The hiring carve-out (which exists so that real job posts are not
   punished) covered it.
2. **Off-platform commercial adverts** — a service offer whose only contact path is an
   external messenger, often typeset in decorative bold glyphs to defeat text matching.
   Graded as promotion without a price, therefore observed only.
3. **Begging** — personal requests for money, gifts or purchases aimed at members.
   No rule covered it at all.

Each is deterministic, high-precision and high-volume. Leaving them to human review meant
the channel filled up while the queue waited.

## The species

| species | requires | redirect |
|---|---|---|
| `recruit_dm` | a recruiting or partnership ask **and** a DM-funnel phrase | the hiring channel |
| `offplatform_ad` | a commercial offer **and** an external-messenger contact pointer, *or* heavy decorative-glyph usage plus an offer or link | the self-promotion channel |
| `begging` | an *ask* phrase **and** a money-or-payment-channel term | none — the note explains there is nowhere to move it |

Every species needs **two independent signals**. One alone never fires. This is what keeps
"my DMs are open if anyone needs help debugging" and "the app handles money transfers so I
need idempotent writes" out of the ladder.

### Same-author window
A funnel split across two messages defeats any single-message rule. When a message carries
only a partial signal, the author's recent messages in the channel are joined in order and
the detector runs on the joined text. On a hit, **every joined message** is archived and
then deleted as a single offence. Each half alone still does not fire.

## The ladder

| offence | action | strike |
|---|---|---|
| 1st | archive → delete → private note explaining what was removed, why, and where it belongs | none |
| 2nd | the same, plus a short timeout | 1 |
| 3rd+ | the same, plus a longer timeout, and the operator is pinged for the ban decision | n |

The note always ends with an appeal line: reply here, a human reads it.

## Invariants

- **General channel only.** Other channels keep the escalate-only posture.
- **Staff exempt.** Configured staff roles never enter the ladder.
- **Locker first, fail closed.** If the evidence archive does not succeed for *every*
  message about to be deleted, nothing is deleted; the case is held for the operator.
- **Strike memory is per member**, with the full action history retained for review.
- **The ladder never bans.** The third rung asks a human.
- **Detection is text-only and deterministic** — no model output participates in deciding
  whether a species fired. A model that judges attacker-controlled text is itself
  injectable; see [THREAT-MODEL.md](THREAT-MODEL.md).

## Verification discipline

Every species ships with a positive set (real messages that should fire) and a negative set
(real builder messages that must not) run before deployment, and the
[weekly dry replay](OPERATIONS.md#weekly-dry-replay) re-runs the detectors over the prior
week's un-actioned rows so a silent miss surfaces within days rather than when someone
notices the channel is full.
