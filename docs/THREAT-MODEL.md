# THREAT MODEL
What Axiom is built to stop, and what it does not attempt.

## IN SCOPE

| Threat | Approach |
| :--- | :--- |
| **Commercial spam and solicitation** | classification plus a deterministic backstop; graduated verdicts so soft promotion is redirected rather than punished |
| **Off-platform funnelling** | signal conjunction — a contact-channel mention only escalates when it co-occurs with selling or link intent |
| **Invite farming** | pattern and behaviour signals, on their own ladder |
| **Harassment and targeted abuse** | classification with confidence scoring, evidence retention, escalation path to human moderators |
| **Coordinated raids** | join-behaviour and account-age heuristics evaluated together, never singly |
| **Message flooding** | windowed rate evaluation |
| **Mass-mention abuse** | mention-density evaluation within a window |
| **Image-borne violations** | separate vision classification, with its own fallback model |
| **Moderator error and disputes** | every action writes evidence including the model, confidence and reasoning, so a decision can be reviewed rather than argued about |

## OUT OF SCOPE
Stated plainly, because a tool that implies total coverage is worse than one that
names its edges.

- **Determined human adversaries with patience.** Behavioural signals are tuned to
  volume and co-occurrence. Someone willing to build reputation slowly and violate
  rarely is a different problem. Axiom raises the cost of abuse; it does not
  eliminate it.
- **Content requiring outside context.** A message that is only abusive because of
  something said elsewhere, or on another platform, is not visible to the system.
- **Off-platform coordination.** Anything organised in DMs or another service is
  outside the gateway's view entirely.
- **Account compromise.** Axiom evaluates behaviour, not identity.
- **Legal or policy adjudication.** Axiom enforces a configured posture. It does not
  decide what a community's rules should be.

## DESIGN POSTURE

- **Fail closed on enforcement, fail open on speech.** When the system cannot
  evaluate, it does not silently allow — it queues for later evaluation. But an
  ambiguous case is redirected rather than punished.
- **Escalate on uncertainty, never dismiss.** A borderline case has a floor and
  cannot fall through to clean. See [CLASSIFICATION](CLASSIFICATION.md).
- **Evidence before notification.** The record is written first, so a failed
  notification never costs a record.
- **Private correction first.** The default response is ephemeral. Public
  enforcement is an escalation, not an opening move.
- **No single signal decides anything.** Every escalation path requires
  co-occurrence. This is the core design commitment and the subject of
  [FALSE-POSITIVES](FALSE-POSITIVES.md).

## ACKNOWLEDGED TRADE-OFFS
Two design tensions are worth naming, because they are choices rather than
oversights.
**Accuracy versus availability.** The fallback layers exist so that coverage
degrades rather than disappearing during an inference outage. A simpler layer is by
definition a blunter one. The alternative — no coverage at all when a model is
unreachable — was judged worse.
**Sensitivity versus trust.** A moderation system that frequently flags legitimate
conversation loses the community's cooperation, and a system the community works
around is not protecting it. Calibration reflects that, which is why
[FALSE-POSITIVES](FALSE-POSITIVES.md) is the longest document here.

## WHAT IS NOT PUBLISHED
Detection calibration is deliberately absent throughout: thresholds, limits,
confidence cut-offs, backstop patterns, classifier prompts, scheduled timings, and
the specific evaluation differences between trust tiers.
**Documentation of a detector is a manual for evading it.** The architecture is
published so the design can be judged. The calibration is withheld so it cannot be
worked around. Requests for specific values will be declined.

