# ARCHITECTURE

## THREE SERVICES AND A RESIDENT PROCESS

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/indicaindependent/axiom/main/assets/diagrams/components-dark.svg">
  <img src="https://raw.githubusercontent.com/indicaindependent/axiom/main/assets/diagrams/components-light.svg" alt="Components and failure domains: the Discord gateway websocket feeds a resident gateway listener on operator hardware forming failure domain A, which feeds a Cloudflare Worker forming failure domain B. The worker classifies, enforces and sweeps, writing to two D1 databases, BOT_DB for guild-local state and VM_DB for the moderation record. The domains fail independently." width="100%">
</picture>

<details>
<summary>Same diagram as plain text</summary>

```
Discord Gateway  (websocket, resumable session)
                              |
                              v
                   +----------------------+   operator hardware, systemd-supervised
                   |   GATEWAY LISTENER   |   holds the live session
                   |   "dumb plumbing"    |   never acts, never stores
                   +----------------------+
                              |  POST /moderate
                              v
   +--------------------------------------------------+
   |            MODERATION ENGINE (worker)            |
   |  interactions · classification · enforcement     |
   |  scheduled sweep · backfill · evidence           |
   +--------------------------------------------------+
        |                |                     |
        v                v                     v
   +---------+      +---------+        +------------------+
   | BOT_DB  |      |  VM_DB  |        |  RESEARCH        |
   | guild   |      |  the    |        |  GATEWAY (worker)|
   | state   |      |  record |        |  corpus + API    |
   +---------+      +---------+        +------------------+
                                              |
                                              v
                                    +------------------+
                                    | SECURITY SCANNER |
                                    | (worker, r/o)    |
                                    +------------------+
```

</details>

| Service | Role | Relative size |
| :--- | :--- | :--- |
| Moderation engine | classification, enforcement, evidence, commands | ~92% of the codebase |
| Research gateway | authenticated corpus and research API, scanner routing | ~7% |
| Security scanner | read-only web-security audit | ~1% |

## WHY THE LISTENER IS NOT IN THE WORKER
A Worker cannot hold a long-lived websocket across cold starts, and Discord's gateway
expects a persistent, resumable session with heartbeats and sequence tracking. So the
listener is a resident process and the worker handles everything request-shaped.
The split has a second benefit that matters more than the first: **the two halves fail
independently.** A worker deploy does not drop the gateway session, and a gateway
reconnect does not interrupt interaction handling.

### Design notes on the listener, from its own source

- **No Discord library.** Pure websockets plus an HTTP client, deliberately, so the
  process is portable to a container runtime later. Portability designed in, not
  discovered.
- **Never acts, never stores.** It forwards and nothing else. All judgement lives in the
  engine, so there is exactly one place where policy is decided.
- **Refuses to start without credentials** rather than running degraded. A process that
  starts and silently does nothing is the worst available outcome.
- **Auto-reconnect with resume and backoff**, so a network blip does not lose the
  session or hammer the gateway.
Its supervision is covered in [FAILURE-MODES](FAILURE-MODES.md) — the first failure in
that document is about the supervisor, not the listener.

## WHY THERE ARE TWO DATABASES

| Binding | Holds | Lifecycle |
| :--- | :--- | :--- |
| `BOT_DB` | guild-local state: members, rate limits, trust tiers, legacy records | per-bot |
| `VM_DB` | the moderation record: decisions, evidence, strike ladders, per-guild config, harvest queue | shared with the wider platform |
**This split is a documented trap, published because it cost real time.** Both
databases contain a table named `mod_log`. One holds a legacy manual-moderation shape;
the other holds the current automated record.
Querying the wrong one returns a well-formed, plausible, **empty** result rather than an
error — which reads exactly like "this system has never acted." An audit that checked
only the first database concluded the moderation pipeline had never run, while it was
actively enforcing.
**An absence is only evidence once you have confirmed you asked the right store.**

## SCHEDULED AND OUT-OF-BAND WORK

| Job | Where | Purpose |
| :--- | :--- | :--- |
| Engine sweep | worker, recurring | periodic evaluation and reporting |
| Retry queue | operator hardware | re-judge anything an outage left unjudged |
| Liveness check | operator hardware | external black-box supervision of the listener |
| Backfill drivers | operator hardware | batch re-judge history in bounded phases |
Two jobs run on operator hardware rather than in the worker, for the same reason in both
cases: **a worker-side job cannot run when the worker is the thing that is broken.**
Cadences and offsets are intentionally omitted throughout. An adversary who knows
when coverage is thinnest knows when to post.

## ISOLATION POSTURE

- All three workers have their **public platform subdomain disabled**. Traffic arrives
  only through the operator's own hostname, so there is no bypass route to a service
  that skips the intended entry point.
- The engine holds a **guild allowlist** and will not operate in a server it was not
  authorised for, independent of whether it was invited.
- The research gateway authenticates **per consumer**, so a compromised consumer key
  does not become general access.

## BINDINGS
See [`wrangler.toml.example`](../wrangler.toml.example) for the shape. Names are
published because they are the useful part of a specification. Identifiers are
placeholdered because they are not.

