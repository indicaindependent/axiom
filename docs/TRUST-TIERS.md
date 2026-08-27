# TRUST TIERS
Authority in Axiom is tiered, and tier is resolved per interaction rather than
cached.

## THE TIERS

| Tier | Who | How it is established |
| :--- | :--- | :--- |
| **Owner** | the operator | identity supplied as a secret, with a compiled fallback |
| **Co-admin** | a designated second administrator | explicit identity |
| **Moderator** | anyone holding real moderation permissions | **derived from Discord permission bits** |
| **Member** | everyone else | default |
The moderator tier is the design decision worth noting. It is **not** a hardcoded
list and **not** a role name. It is computed from the caller's actual Discord
permission bitfield — kick, ban and manage-guild.
Consequences, all of them intentional:

- promoting someone in Discord grants Axiom authority immediately, with no config
  change and no redeploy
- demoting someone revokes it just as immediately
- a renamed or deleted role cannot silently strand authority
- **Discord remains the single source of truth for who moderates**, so there is no
  second permission system to drift out of sync

## WHY TIERS AFFECT EVALUATION, NOT JUST COMMANDS
Tier gates which commands are available, and it also influences how the moderation
path treats a message.
This is a deliberate trade-off rather than an oversight, and it is listed as a known
weakness in [THREAT-MODEL](THREAT-MODEL.md). The reasoning is the asymmetry from
[FALSE-POSITIVES](FALSE-POSITIVES.md): a false positive against a moderator is
unusually expensive, because it undermines the authority the system depends on to
function. A moderator who cannot post without being flagged stops using the tool.
The accepted cost: a compromised trusted account retains its trust until its
behaviour changes.

## RESOLUTION ORDER
Tier is evaluated highest-first and short-circuits on the first match. Owner
identity is checked before anything derived, so an operator cannot be locked out of
their own instance by a permission change in Discord.

## WHAT IS NOT PUBLISHED
Owner and co-admin identifiers, the permission bit values used, and the specific
per-tier evaluation differences are omitted. The model is documented; the
calibration is not. See the note at the end of [CLASSIFICATION](CLASSIFICATION.md).

