---
title: "Consistency: invariants hold"
weight: 4
---

# Consistency: invariants hold

Consistency means the eVault only ever moves from one valid state to another.
A change that would break a rule is refused; a change that is accepted is one
the rules already permit. This is the **C** in ACID.

> **In plain terms**
>
> The eVault has a short list of rules every piece of data has to satisfy:
> the right shape, the right signatures, sensible permissions. Every change is
> checked against those rules before it is allowed in. Because all three
> replicas check the same rules in the same order, they never disagree about
> what is in the store.

## The invariants the eVault enforces

The eVault has a small, fixed set of invariants. Most of them are not new in
this proposal — they are already implicit in the prototype — but stating them
together makes it clear what "valid state" means.

- **The log hash-chain is unbroken.** Every log entry carries the hash of the
  previous one. See
  [durability](durability/#making-the-log-trustworthy-across-independent-providers).
- **Every log entry is signed by the current primary.** A replica cannot
  fabricate an entry the primary never produced.
- **Every transaction is bracketed by `BEGIN` and a signed `COMMIT`.** No
  dangling `BEGIN`, no orphaned changes — see [atomicity](atomicity).
- **A MetaEnvelope payload conforms to its declared ontology.** The ontology
  W3ID on a MetaEnvelope resolves to a schema (via the
  [Ontology](/docs/Infrastructure/Ontology) service); the payload has to match
  it.
- **An ACL is a well-formed list of W3IDs or `*`.** No garbage strings, no
  unknown wildcards.
- **An eName resolves to exactly one ingress.** This is the Registry's
  invariant, but the eVault relies on it — see the
  [overview](./#the-registry-stays-dumb).
