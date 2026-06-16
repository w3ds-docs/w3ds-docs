---
title: "The invariants"
weight: 1
---

# The invariants

Consistency is meaningful only when the rules it enforces are spelled out.
This page lists the invariants the eVault holds. Each one is something a
single log entry can violate — and therefore something the
[validation gate](validation) has to check before letting the entry
through.

> **In plain terms**
>
> Every eVault carries a short list of rules its data has to obey. Most of
> them are obvious in hindsight — entries are signed, ACLs name real users,
> payloads match their declared shape — but writing them down in one place
> turns "the eVault is well-behaved" into a checklist that humans can audit
> and machines can verify.

Most invariants below are **already implicit** in the prototype eVault.
Stating them as invariants does two things: it lets the validation gate
check them uniformly, and it makes their failures distinguishable from
genuine corruption or tampering.

## Log invariants

These are properties of the [log](/docs/architecture/redundant-evaults/durability)
itself, independent of the data it carries.

- **The hash chain is unbroken.** Every log entry includes the hash of the
  previous entry. Removing or reordering any entry breaks the chain and is
  immediately visible. See
  [Durability](/docs/architecture/redundant-evaults/durability/#making-the-log-trustworthy-across-independent-providers).
- **Every entry is signed by the current primary.** A replica cannot
  fabricate an entry the primary never produced; an old primary that has
  been fenced off cannot produce entries the new primary's chain accepts.
  See [Failover](/docs/architecture/redundant-evaults/failover/#only-one-primary-at-a-time).
- **Every transaction is bracketed by `BEGIN` and a signed `COMMIT`.** No
  dangling `BEGIN`, no orphaned change entries floating without a
  transaction id. See [Atomicity](/docs/architecture/atomicity/transactions).
- **Sequence numbers are strictly increasing and gap-free.** The log is a
  total order; gaps would mean a lost entry, a sign of corruption.

## Schema invariants

These describe how the **payload** of a log entry maps to the data model.

- **A MetaEnvelope payload conforms to its declared ontology.** The
  ontology W3ID on a MetaEnvelope resolves (via the
  [Ontology](/docs/Infrastructure/Ontology) service) to a schema; every
  field in the payload has to be present, typed correctly, and within any
  declared constraints of that schema.
- **Envelope `valueType` matches the runtime type of `value`.** A
  `valueType: "number"` cannot carry the string `"42"`.
- **An ontology W3ID is resolvable.** The eVault does not accept a
  MetaEnvelope whose ontology cannot be looked up at write time. A schema
  that later disappears does not retroactively break stored data — the
  schema cache holds onto the resolved shape — but a new write whose
  schema is unknown is refused.

## Access-control invariants

These keep ACLs sane and trustworthy.

- **An ACL is a well-formed list of W3IDs or `*`.** No empty strings, no
  unknown wildcards, no syntactic garbage. The ACL parser is the only path
  to setting an ACL.
- **A grantor can only grant access they themselves hold.** A platform
  acting on user A's behalf cannot widen the ACL on a MetaEnvelope user A
  does not own. (The prototype enforces this through the access guard; the
  invariant makes the enforcement explicit.)
- **An ACL entry's W3ID resolves to a known eName at write time.** Granting
  access to a phantom W3ID is refused; granting access to a W3ID that is
  later deactivated is allowed to remain, but the access guard treats it
  as denied at read time.

## Identity invariants

These tie the eVault's data back to the rest of the W3DS network.

- **An eName resolves to exactly one ingress.** This is enforced by the
  [Registry](/docs/Infrastructure/Registry), but the eVault relies on it
  every time it writes an ACL or an awareness target. See the
  [overview](/docs/architecture/redundant-evaults/#the-registry-stays-dumb).
- **An eVault stores data for exactly one tenant.** The prototype already
  ties each eVault to one W3ID; the invariant says no log entry can ever
  carry data labelled with a different W3ID. Multi-tenancy is a property
  of the provisioning layer, not of any single eVault.
- **A signing key referenced by an entry is bound to an eName the eVault
  knows about.** The key binding certificate (see
  [eVault `whois`](/docs/Infrastructure/eVault#whois)) is the
  authoritative answer; an entry signed by an unknown key is refused.

## Internal-state invariants

These are not visible to clients but matter for the eVault's own integrity.

- **W3IDs are unique within the eVault.** No two MetaEnvelopes share a
  W3ID; no Envelope id collides with another.
- **A MetaEnvelope's Envelope ids are referenced from at most one
  MetaEnvelope.** The flat graph in Neo4j must not develop dangling or
  shared Envelopes.
- **Webhook deliveries are keyed by log position.** A given log position
  triggers at most one webhook per registered platform; replay of the
  webhook side does not generate duplicates. (See
  [Failover](/docs/architecture/redundant-evaults/failover/#what-clients-and-platforms-see).)

## Bounded violations

Two things the eVault deliberately does **not** treat as hard invariants,
because the world makes them messy:

- **Replication lag is bounded but not zero.** A replica can be a few
  seconds behind the primary. Inside the lag window, the state across
  replicas is not identical. The invariant the eVault enforces is "every
  replica catches up to the primary within a bounded time", not "every
  replica matches the primary at all times". This is the difference
  between sequential consistency (what every replica's log enforces) and
  external consistency.
- **Cross-eVault state is eventually consistent.** Two eVaults involved in
  the same saga briefly observe each other in different states (see
  [Cross-eVault atomicity](/docs/architecture/atomicity/cross-evault)).
  Each eVault's local invariants still hold; the cross-eVault invariant
  ("both sides eventually agree") is enforced by the saga, not by either
  eVault on its own.

The next page, [Validation](validation), is about how these invariants
become code that runs on every replica.
