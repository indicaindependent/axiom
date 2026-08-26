# CONFIGURATION
Axiom's behaviour is operator-tunable **without a redeploy**. Roughly twenty
configuration keys govern the moderation surface.

**Values, defaults and ranges are deliberately absent from this document.** A
published threshold is a published evasion budget. What follows is the shape of the
control surface and what each key affects.

## MASTER SWITCHES

| Key | Effect |
| :--- | :--- |
| `MOD_AUTOMOD_ENABLED` | master switch for autonomous evaluation |
| `MOD_WARN_ENABLED` | whether warnings are issued automatically |
| `MOD_RAID_ENABLED` | whether raid heuristics are active |
Each is independent, so a single subsystem can be disabled without standing the
whole system down — the operating posture that matters most during an incident.

## SCOPE

| Key | Effect |
| :--- | :--- |
| `MOD_GUILD` | which guild is under management |
| `MOD_CHANNELS` | which channels are in moderated scope |
| `MOD_LOG_CHANNEL_ID` | where the operator log is written |
| `MOD_ADMIN_ROLE_PING` | who is notified on escalation |
| `MOD_RULES` | the rule text the classifier reasons against |

 `MOD_RULES` is the most consequential key in the system. **The community's own
rules are configuration, not code** — so the same engine enforces a different
posture in a different server without modification.

## MODELS

| Key | Effect |
| :--- | :--- |
| `MOD_CLASSIFIER_MODEL` | primary text classifier |
| `MOD_CLASSIFIER_FALLBACK` | used when the primary is unavailable |
| `MOD_VISION_MODEL` | primary image classifier |
| `MOD_VISION_FALLBACK` | used when the vision primary is unavailable |

Changing a model changes behaviour **with no code diff**. This is the reason
[GOVERNANCE](GOVERNANCE.md) exists: a configuration change that alters enforcement
must leave a record, or the system's behaviour drifts untraceably.

## LIMITS

| Key | Governs |
| :--- | :--- |
| `MOD_SPAM_THRESHOLD` | message volume tolerated within the window |
| `MOD_SPAM_WINDOW_SEC` | the window over which volume is measured |
| `MOD_MENTION_LIMIT` | mention density tolerated in a message |
| `MOD_WARN_LIMIT` | warnings before escalation |
| `MOD_TIMEOUT_LIMIT` | timeout ceiling |
| `MOD_RAID_JOIN_RATE` | join velocity that constitutes a raid signal |
| `MOD_RAID_ACCOUNT_AGE` | account age below which a join is treated as suspicious |
**Both raid keys are evaluated together, never singly.** A burst of joins during an
announcement is normal; a burst of *new* accounts is not. Either signal alone
produces false positives, which is the conjunction principle from
[FALSE-POSITIVES](FALSE-POSITIVES.md) applied to behaviour.

## OPERATOR GUIDANCE
**Tune scope before tuning thresholds.** Most unwanted behaviour is a channel-scope
problem, not a sensitivity problem, and narrowing scope has no false-positive cost.
**Change one key at a time.** With graduated verdicts and conjunction gating,
simultaneous changes produce effects that cannot be attributed afterward.
**Record every change.** See [GOVERNANCE](GOVERNANCE.md).
