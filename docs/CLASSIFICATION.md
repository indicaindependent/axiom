# CLASSIFICATION

## VERDICTS ARE GRADUATED, NOT BINARY
Most moderation systems answer allow-or-deny. That forces every ambiguous case into
one of two wrong outcomes: punish a legitimate user, or ignore a real violation.
Axiom's promotional path resolves to one of three states:

| Verdict | Meaning | Typical response |
| :--- | :--- | :--- |
| `hard_ad` | unambiguous commercial solicitation | removal, ladder increment, operator log |
| `soft_promo` | promotional in character but plausibly in good faith | redirect to the appropriate channel, private note |
| clean | no promotional character | no action |

`soft_promo` is the entire point. It is the verdict that lets the system act on
something without treating a member as a spammer.

## THE BORDERLINE RULE

> **When a message is promotional but borderline, it floors at `soft_promo`. It may
> never fall through to clean.**
This is an explicit, dated policy decision rather than an emergent behaviour, and it
encodes a judgement about which error is cheaper:

- **A borderline case wrongly cleared** teaches every observer that the rule is
  optional, and the next message is bolder.
- **A borderline case wrongly redirected** costs one member one moment of mild
  friction, in private, with an explanation.
The second is recoverable. The first compounds. So uncertainty resolves upward.

## WHAT RAISES A VERDICT
Described by role. Values, patterns and weights are withheld — see the note at the
end of this document.

| Signal family | What it observes |
| :--- | :--- |
| commercial intent | whether the message is offering something in exchange for money |
| link presence | whether an outbound destination is attached |
| contact-channel mention | whether the message points to a private or off-platform channel |
| audience targeting | whether the message is addressed at the room rather than a person |
| invite behaviour | whether the message trades in server access |
| earnings framing | whether the message advertises an outcome to be emulated |

**No single signal escalates on its own.** Signals combine, and the combination is
the evidence. This is the subject of [FALSE-POSITIVES](FALSE-POSITIVES.md) and is
the most important design decision in the system.

## OTHER CATEGORIES
The classifier also returns named categories for abuse-type content — harassment
being the most common in practice — each with an independent confidence score and
its own response ladder. These are handled separately from the promotional path
because the appropriate response differs in kind: a promotional message is usually
misplaced, whereas abuse is usually intended.

## CONFIDENCE
Every classification carries a confidence score, retained with the evidence, so a
later review can distinguish a marginal call from an obvious one.

**Confidence thresholds are not published, and are not uniform across
categories.** A score that justifies action on one category does not justify it on
another. Publishing the cut-offs would hand an adversary a target to write beneath.

## A NOTE ON WHAT IS WITHHELD
This document describes the taxonomy, the ordering and the reasoning. It does not
contain trigger patterns, keyword lists, weights, or numeric thresholds.
That is not vagueness for its own sake. A classifier's calibration is the part that
an evader needs and a reader does not.
