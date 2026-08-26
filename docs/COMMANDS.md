# COMMANDS
Axiom's interaction surface. Responses are **ephemeral by default** — visible only
to the caller — so using the tool does not disclose what is being investigated.

## MODERATION

| Command | Minimum tier | Purpose |
| :--- | :--- | :--- |
| `warn` | Moderator | issue a warning and increment the appropriate ladder |
| `infractions` | Moderator | review a member's accumulated record |
| `note` / `mod-note` | Moderator | attach a private annotation to a member |
| `reason` | Moderator | supply or amend the reasoning attached to an action |
| `target` | Moderator | scope a subsequent action to a member |
| `action` | Moderator | apply a selected enforcement action |
| `mod-brief` | Moderator | the moderation summary |

## INVESTIGATION

| Command | Minimum tier | Purpose |
| :--- | :--- | :--- |
| `memberlookup` | Moderator | retrieve what the system holds on a member |
| `userinfo` / `user` | Member | public-facing account information |
| `memberlevel` / `level` | Member | standing and progression |
| `members` | Moderator | membership overview |
| `guildinfo` | Moderator | server-level summary |

## ACCESS CONTROL

| Command | Minimum tier | Purpose |
| :--- | :--- | :--- |
| `grant-access` | Owner / Co-admin | extend access to a gated area |
| `revoke-access` | Owner / Co-admin | withdraw it |
Access changes are deliberately the narrowest surface in the system, restricted
above the moderator tier.

## UTILITY

| Command | Minimum tier | Purpose |
| :--- | :--- | :--- |
| `scan` | Member | invoke the companion read-only web-security scanner |
| `help` | Member | command discovery, filtered to the caller's tier |
| `ping` | Member | liveness |
| `arrow` | Member | utility |

 `help` returns only what the caller can actually invoke. A member cannot enumerate
the moderation surface, which keeps the command list from doubling as
reconnaissance.

## DESIGN NOTES
**Ephemeral-first.** A moderator reviewing a member's history does not broadcast
that they are doing so. A member receiving a correction is not publicly marked.
**Tier is checked at invocation, not at registration.** Commands are visible
according to current authority, not authority at deploy time.
**Every enforcement command writes evidence** — including who invoked it. Human
actions are as auditable as automated ones. See [PRIVACY](PRIVACY.md) for retention.
