# PRIVACY

Axiom reads messages in the channels it is scoped to. That is a meaningful thing for
software to do, so this document states what the system retains and who can reach it.

## WHAT IS RETAINED

| Data | Purpose |
| :--- | :--- |
| Decision records | what was classified, the category, the confidence, the action taken |
| Message excerpts | enough context to review a decision afterwards |
| Reasoning | why the classifier reached its verdict |
| Model identity | which model produced the decision |
| Member records | standing, tier, accumulated infractions |
| Strike ladders | separate per category, so promotional and abuse records do not merge |
| Moderator actions | who acted, against whom, and why |

**Human moderator actions are retained on the same terms as automated ones.** The
audit trail is not one-directional.

## WHAT IS NOT RETAINED

- messages in channels outside configured scope
- direct messages
- full message history — retention is **decision-scoped**, meaning a record exists
  because something was evaluated, not because everything is archived

## WHY EXCERPTS ARE KEPT AT ALL

An enforcement decision that cannot be reviewed cannot be appealed or corrected.
Discarding the excerpt would make every past action unfalsifiable, which protects the
system from scrutiny rather than protecting the member.

The trade-off is explicit: a small amount of retained content, in exchange for
decisions that can be checked.

## WHO CAN REACH IT

| Data | Reachable by |
| :--- | :--- |
| A member's own standing | that member, through their own commands |
| Infraction records | moderator tier and above |
| Full evidence records | owner and co-admin |
| Operator log | whoever can read the configured log channel |

Investigative commands are **ephemeral**, so a lookup is not itself broadcast to the
channel.

## NOTES FOR OPERATORS

Axiom is a tool, and deployment decisions belong to whoever runs it. Points worth
considering when configuring an instance:

- **Channel scope is the main privacy control.** Every channel added to scope is a
  channel whose messages may be evaluated and excerpted. Narrow scope is both a
  privacy posture and, per [CONFIGURATION](CONFIGURATION.md), the most effective
  tuning lever.
- **The log channel inherits the sensitivity of its contents.** Evidence written to a
  broadly readable channel is broadly readable.
- **Community norms and applicable law vary**, and no configuration file settles what
  a given community should disclose to its members or how long records should be
  kept. Those are operator decisions.

## WHAT THIS REPOSITORY CONTAINS

No real message content, usernames, member identifiers, guild identifiers or evidence
excerpts appear anywhere in this repository. Every example is synthetic.

Retention windows and evidence-scope values are not published, for the same reason
thresholds are not.
