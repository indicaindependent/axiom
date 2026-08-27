# RESEARCH GATEWAY
A separate service that fronts a document corpus and a research API, with
**per-consumer authentication**. It is how automated consumers reach curated material
without any of them holding general access.

## WHY IT IS A SEPARATE SERVICE
The moderation engine holds the credentials that let it act on a community. Anything
that merely needs to *read* research should not sit behind those credentials.
Splitting them means a compromise of the research surface yields a corpus reader, not a
moderation actor. Blast radius, not tidiness, is the reason.

## PER-CONSUMER AUTHENTICATION
Each consumer authenticates with its **own** key rather than a shared secret.

| Property | Consequence |
| :--- | :--- |
| Distinct key per consumer | one compromised key does not become general access |
| Independently revocable | a consumer can be cut off without disrupting the others |
| Attributable | audit records show which consumer performed a request |
The key **names** are not published, because a key named after its consumer discloses
which systems exist and how they relate. That is architecture disclosure disguised as a
configuration detail.

## SURFACES

| Surface | Purpose |
| :--- | :--- |
| `/health` | liveness |
| `/research` | authenticated query over the corpus |
| `/admin/*` | operator-only maintenance, separately gated |
Administrative surfaces verify the invoker independently of consumer authentication, so
a research key can never reach them.

## STORED QUERY SAFETY
Queries reaching the corpus pass through an explicit guard rather than being interpolated
into statements. This is unglamorous and it is the single most valuable line of defence
in a service whose entire job is accepting queries from elsewhere.
Every request is written to an audit trail, so the question "what did this consumer
actually read?" has an answer.

## OBJECT STORAGE FOR THE CORPUS
Documents live in object storage rather than in the database, with the database holding
metadata and access records. The database stays small and queryable while documents stay
cheap and immutable.

## WHAT IS NOT PUBLISHED
Consumer key names, the bucket name, database identifiers, the administrative path
shapes beyond their existence, and the corpus contents.

