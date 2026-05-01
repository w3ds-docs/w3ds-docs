---
title: Data Model
weight: 10
---

# Data Model

This page proposes the basic shape of W3DS data: what it is, how it is identified, and what changing it means.

---

## 1. The unit of data is an entity

An entity is what an eVault stores about a single subject: the subject's stable identifier, a set of types and a set of named properties.

```json
{
  "@id": "did:w3ds:5f8a1c2d-9b40-4f5d-9e6e-8c1a4b7d2e3f",
  "@type": "social:BlogPost",
  "title": "Walking the Camino, day 1",
  "body": "Started today from Saint-Jean-Pied-de-Port. Twenty kilometers in.",
  "publishedAt": "2026-04-15T08:30:00Z"
}
```

> [!NOTE]
> JSON is the default encoding used throughout this chapter; the model itself is encoding-agnostic. A specific wire transport may serialize entities as protobuf, an RDF serialization, or another format. What the model describes is the logical shape.

The JSON wire form uses the reserved JSON-LD `@id` and `@type` keywords, leaving `id` and `type` available as ordinary property names without the risk of naming collisions. All W3ID values, whether in `@id` or in reference property positions, use the canonical URI form `did:w3ds:<UUID>`.

Four things every entity has:

- An **identifier** (`@id`). A W3ID in its canonical URI form `did:w3ds:<UUID>`, minted by the eVault on creation.
- A **type set** (`@type`). The set of entity types it is declared to be, accumulated across writes. The set is unordered; all members are equal. On the wire, `@type` renders as a string when the set has one member or an array when it has multiple.
- A set of **properties**. Each property has a name and one or more values. Reads return the entity's current properties, filtered to what the reader has requested and is allowed to access.
- A recoverable **history**. Every accepted write produces a record of what changed, by whom, when, and against which type version.

> [!NOTE]
> The W3ID denotes the **subject**: the thing the entity is about (a person, a post, a transaction, a concept), not the entity itself. The entity is the eVault's record about the subject; reads of the entity yield a **representation** in some wire form.

An entity belongs to one eVault. Crossing eVaults happens by reference, not by copy. A user may operate one eVault or several (for role separation, risk segregation, jurisdictional placement, or provider redundancy).

A service may write to the eVault directly, keep its own local copy synchronized with it, or mix the two for different parts of the same application. The canonical entity is always the one in its eVault; a service's local representation is derived.

> [!NOTE]
> Readers familiar with the TRL6 prototype: the Entity is the TRL7 counterpart of MetaEnvelope's subject role. Properties are attached directly, with no separate Envelope per value.

---

## 2. The unit of change is a property write

Changing an entity means writing one or more of its properties.

```json
{
  "title": "Walking the Camino, day 1 (revised)",
  "body": "Updated notes from the day."
}
```

Six commitments:

- **A write may affect one or more entities, all in the same eVault.** Each named entity carries its own type reference; validation runs per entity. Operations that span eVaults decompose into one write per eVault; observers may see them at different times.
- **A write originates from one service in one request.** Each write therefore carries a single attribution.
- **A write names the properties it changes.** Properties not named are untouched, so two services writing different properties of the same entity do not collide.
- **A write is atomic over everything it names.** Every change on every named entity commits, or none does. Outcomes that depend on more than one write are coordinated above the data layer.
- **Each property has its own access rules.** A write that touches any property the service cannot change is rejected entirely.
- **Writes may opt into state-gating.** A write may declare per-property preconditions; if any fails, the entire write is rejected.

> [!WARNING]
> **Open question: ACL defaults and inheritance.** The commitment names the grain at which access is evaluated (per property); it does not say where each property's effective rule comes from. Defaults could live at the entity level (one ACL covers every property unless overridden per slot), the type level (the type declares default rules for its declared slots), the eVault level (a user-set baseline that types and entities refine), or other levels yet to be named.

> [!WARNING]
> **Open question: Multi-entity write batch size.** Does the architecture impose any limit on how many entities can participate in one write, or leave it to the implementation?

---

## 3. Property operations

A write expresses changes through a small, explicit operation vocabulary. Setting a value is the implicit default; everything else uses a `$`-prefixed operator.

```json
{"title": "Walking the Camino, day 1 (revised)"}
{"coverImage": {"$unset": true}}
{"tags": {"$add": ["#camino"]}}
{"tags": {"$remove": ["#wip"]}}
{"tags": {"$set": ["#camino", "#travel"]}}
{"likeCount": {"$increment": 1}}
```

Operators:

- `$set` is the implicit default for direct values. On a multi-valued property, `$set` replaces the whole collection; this is state-dependent (the new collection depends on knowing the prior state), so safety-conscious writers guard it with state-gating.
- `$unset` removes a property explicitly.
- `$add` and `$remove` modify multi-valued properties incrementally.
- `$increment` and `$decrement` apply to integer properties and compose under concurrent writes from multiple services without read-modify-write.

This is the basic operator set; more are likely to be added as concrete needs emerge.

Whether a multi-valued property is unordered (a set) or ordered (a list) is declared in the type definition (the precise representation is an open question); ordered properties additionally support positional operations (`$insertAt`, `$removeAt`). Positional operations are state-dependent (the position depends on knowing the prior state), so safety-conscious writers should guard them with state-gating.

> [!NOTE]
> JSON Patch (RFC 6902) was considered as a strict superset and not adopted, for three reasons. It treats every collection as an ordered array, forcing array semantics on properties that are conceptually sets. Its addressing is path-based rather than per-property, so it does not match how the data architecture reasons about access, history, and convergence. And it has no native operations for counters, so the architecture would need extension operations anyway, losing the compatibility benefit.

---

## 4. References

A property whose value is another entity's identifier is a reference.

```json
{
  "@id": "did:w3ds:7c3e9f1a-2b8d-4e3a-9c1f-6e5d7a8b3c2d",
  "@type": "social:BlogPost",
  "title": "Walking the Camino, day 1",
  "body": "Started today from Saint-Jean-Pied-de-Port.",
  "author": "did:w3ds:9b8e2f4a-1c3d-4e5f-8a7b-6c5d4e3f2a1b"
}
```

References connect entities by identity rather than by copy. A post by Alice carries Alice's W3ID as the value of its `author` property, not a copy of Alice's display name. When a reader wants the display name, they follow the reference. The reference names the subject (Alice), not any specific representation of her data; it outlasts every change to that data.

A reference does not require the target to exist at write time. The target may live in another user's eVault, may be unreachable for a moment, may be removed later. The reference is still a valid pointer; what it currently points at is a separate question.

References open the multi-service story: any service that writes a post can record who wrote it, without copying the author's data and without coordinating with whatever service owns the author's profile.

When a single write creates multiple entities that reference each other, the request uses a client-supplied placeholder in any reference whose target W3ID has not yet been minted. Each new entity declares its own type reference in the same write. The eVault mints the W3IDs for the new entities and substitutes the placeholders with the minted values as part of the same atomic write, so the cross-reference resolves before any reader observes it.

> [!WARNING]
> **Open question: Reference target validity.** A reference is a W3ID pointer. What happens when (a) the target does not exist, or (b) the target exists but its type does not match the reference property's declared range? Accept silently (conformance is the reader's concern)? Reject the write at the eVault? Or accept with a warning surfaced to the writer?
