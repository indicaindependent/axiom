# ARCHITECTURE

## COMPONENTS
Axiom is three cooperating pieces, deliberately split across two failure domains.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/indicaindependent/axiom/main/assets/diagrams/components-dark.svg">
  <img src="https://raw.githubusercontent.com/indicaindependent/axiom/main/assets/diagrams/components-light.svg" alt="Components and failure domains: the Discord gateway websocket feeds a gateway listener, a resident process on operator-controlled hardware forming failure domain A, which feeds a Cloudflare Worker forming failure domain B. The worker handles interaction handling, classification, enforcement and a scheduled sweep, and writes to two D1 databases, BOT_DB holding guild-local state and VM_DB holding the moderation record. The two domains fail independently." width="100%">
</picture>

<details>
<summary>Same diagram as plain text</summary>

```
  Discord Gateway (websocket, persistent session)
            |
            v
   +--------------------+        resident process, operator-controlled hardware
   |  GATEWAY LISTENER  |        holds the live session, survives worker cold starts
   +--------------------+
            |
            v
   +--------------------+        Cloudflare Worker
   |   MODERATION       |        interaction handling, classification, enforcement
   |   WORKER           |        scheduled sweep on a recurring trigger
   +--------------------+
        |          |
        v          v
   +---------+  +---------+
   | BOT_DB  |  |  VM_DB  |     two D1 databases, different lifecycles
   +---------+  +---------+
```

</details>

### Why the listener is not in the worker
A Cloudflare Worker cannot hold a long-lived websocket across cold starts. Discord's
gateway expects a persistent, resumable session with heartbeats and sequence
tracking. So the listener runs as a resident process on operator hardware and the
worker handles everything request-shaped.
This split has a second benefit that matters more than the first: **the two halves
fail independently.** A worker deploy does not drop the gateway session, and a
gateway reconnect does not interrupt interaction handling.

### Why there are two databases

| Binding | Holds | Lifecycle |
| :--- | :--- | :--- |
| `BOT_DB` | guild-local state: members, rate limits, security tiers, legacy records | per-bot, long-lived |
| `VM_DB` | the moderation record: decisions, evidence, strike ladders, per-guild config | shared with the wider platform |

**This split is a documented trap.** Both databases contain a table named
`mod_log`. `BOT_DB` holds a legacy manual-moderation shape; `VM_DB` holds the
current automated record. Reading the wrong one returns a well-formed, plausible,
**empty-looking** answer rather than an error — which reads exactly like "the system
has never acted."
Anyone auditing this system must confirm which binding they queried before drawing
a conclusion from an absence. An empty result from the wrong store is
indistinguishable from a real one.

## DATA FLOW — A MESSAGE

```
message arrives on the gateway
  -> tier lookup            is the author exempt from evaluation?
  -> channel scope check    is this channel in moderated scope?
  -> classification         layer 1, falling through 2 and 3 as needed
  -> verdict assignment     graduated, never binary (see CLASSIFICATION)
  -> backstop reconciliation deterministic rules can RAISE a verdict, never lower it
  -> action selection       ephemeral nudge / warn / timeout / escalate / none
  -> ladder update          the appropriate strike ladder, by category
  -> evidence write         decision, reasoning, model, confidence, excerpt
  -> operator log           to the configured log channel
```
Two properties are load-bearing:
**The backstop can only escalate.** A deterministic rule may raise a verdict that
the model scored too low. It may never clear a verdict the model raised. Failure is
biased toward action, not silence.
**Evidence is written before the operator sees anything.** If the log write fails,
the record still exists. A notification is not a record.

## SCHEDULED WORK
The worker runs a recurring scheduled handler for sweeps and periodic reporting. A
separate out-of-band job on operator hardware handles retry of anything the
classifier could not evaluate during an outage.

Cron cadences and offsets are intentionally omitted from this document. An
adversary who knows when coverage is thinnest knows when to post.

## BINDINGS
See [`wrangler.toml.example`](../wrangler.toml.example) for the binding shape.
Names are published because they are the useful part of a specification.
Identifiers are placeholdered because they are not.
