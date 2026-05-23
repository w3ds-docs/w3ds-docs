---
title: Data Model
weight: 10
---

# Data Model

This page proposes the basic shape of W3DS data: what it is, how it is identified, and what changing it means.

---

## 1. The entity

An entity is the logical representation of a subject: an identifier, a set of types, and a set of named properties. The subject is the thing being described (a person, a post, a transaction, a concept).

```json
{
  "@id": "did:w3ds:5f8a1c2d-9b40-4f5d-9e6e-8c1a4b7d2e3f",
  "@type": "social:BlogPost",
  "title": "Walking the Camino, day 1",
  "body": "Started today from Saint-Jean-Pied-de-Port. Twenty kilometers in."
}
```

> [!NOTE]
> JSON is the default encoding used throughout this chapter; the model itself is encoding-agnostic. A specific wire transport may serialize entities as protobuf, an RDF serialization, or another format. What the model describes is the logical shape.

The JSON wire form uses the reserved JSON-LD `@id` and `@type` keywords, leaving `id` and `type` available as ordinary property names without the risk of naming collisions.

Four things every entity has:

- An **identifier** (`@id`). A W3ID naming the subject the entity describes. Examples in this chapter use the `did:w3ds:<UUID>` form; W3ID can also be written in other forms in non-JSON-LD contexts.
- A **type set** (`@type`). The set of entity types the subject is declared to be, as the union of types declared across its records. The set is unordered; all members are equal. On the wire, `@type` renders as a string when the set has one member or an array when it has multiple.
- A set of **properties**. Each property has a name and one or more values. Reads return the entity's current properties, filtered to what the reader has requested and is allowed to access.
- A recoverable **history**. Every accepted write appends an immutable entry to the target record's history, capturing what changed, by whom, when, and against which type version. The entry shape and read interface are elaborated on a future page.

---

## 2. Parallel records

An entity's data lives in one or more **records**, each stored in some eVault. A record carries the subject's W3ID as its `@id` and has its own **record identifier**, a separate W3ID naming the record itself, surfaced as metadata only when a consumer asks for provenance or attribution.

Three consequences follow:

- **An entity's data may be split across many records, in one or many eVaults.** A car's manufacturer may hold one record in its own eVault; the owner may hold a parallel record contributing ownership attributes; a service center may hold a third contributing maintenance attributes; a government registry may hold a fourth contributing registration attributes. A single eVault may also hold multiple records about the same subject, for example when different platforms operate on the user's eVault using different models. Each record contributes properties independently; how much a reader pulls together at read time (one type, a subset, or all) is a reader-side choice.
- **References point at subjects, not records.** Following a reference means asking "which eVaults hold records about this subject?", not "where is the canonical record?". Cross-eVault navigation is part of read resolution, not part of the data shape.
- **A subject's identity outlives any single eVault.** Migration, key rotation, or provider failure does not re-mint the subject's W3ID; the eVault hosting any record about the subject can change while the subject's identity stays the same.

A short scenario, using mnemonic W3IDs for readability. The subject is a specific car (`did:w3ds:car-001`). Four distinct authorities each hold a record about it: the manufacturer that built it, the owner Alice who bought it, a service center that maintains it, and the government registry that licenses it. None of them is the subject:

```json
// Manufacturer's eVault
{
  "@id": "did:w3ds:car-001",
  "@type": "auto:Vehicle",
  "name": "Volkswagen Golf",
  "vin": "WVWZZZ1KZ8W123456"
}
```

```json
// Owner Alice's eVault
{
  "@id": "did:w3ds:car-001",
  "@type": "personal:Asset",
  "name": "Goldie",
  "acquiredAt": "2024-06-15"
}
```

```json
// Service center's eVault
{
  "@id": "did:w3ds:car-001",
  "@type": "svc:Equipment",
  "lastServicedAt": "2026-04-10",
  "nextDueAt": "2027-04-10"
}
```

```json
// Government registry's eVault
{
  "@id": "did:w3ds:car-001",
  "@type": "gov:RegisteredVehicle",
  "licensePlate": "ABC-123",
  "registeredOwner": "did:w3ds:alice"
}
```

The four types come from four different models: the manufacturer uses an auto-industry model, the owner uses a personal-asset model, the service center uses a maintenance model, the government uses a vehicle-registration model. A reader resolving `did:w3ds:car-001` chooses how much of the entity to project. A request that projects through all four types yields a unified view of the entity as an `auto:Vehicle`, a `personal:Asset`, a `svc:Equipment`, and a `gov:RegisteredVehicle`, with properties from every contributing record accumulated. A request that projects through one type returns just the contributions of records carrying that type: a service-only client asking for `svc:Equipment` receives `lastServicedAt` and `nextDueAt`, nothing else. Each record is written in its own eVault under that eVault's sovereignty; merging is read-side and reader-driven. The [type system](types) page elaborates the type-set semantics, including how multi-valued properties such as `serviceHistory: [...]` accrete as additional records contribute entries.

Property names can collide across models. Here, both the manufacturer's `auto:Vehicle` and the owner's `personal:Asset` declare a property labelled `name`, but they live in different models, so the labels resolve to different IRIs: the manufacturer's is `auto:name` (the product name, `"Volkswagen Golf"`); the owner's is `personal:name` (a pet name, `"Goldie"`). The CURIE prefix identifies the model the property belongs to, so the two `name`s are distinct properties at the underlying IRI level. A reader projecting through a single type sees one `name`; a multi-type merge exposes both, disambiguated by their prefix. The record identity section below shows the merged-view wire form with both names surfaced.

Alternative modelling: the owner's and service center's contributions could instead be separate entities (an `Ownership` entity referencing the car, a `ServiceRecord` entity referencing it). Parallel records and separate entities are both architecturally supported.

A user may operate one eVault or several (for role separation, risk segregation, jurisdictional placement, or provider redundancy). Each of the user's eVaults may hold one or more records about a subject; the subject's identity is the same across them.

---

## 3. Authority and roles

Four layers of authority are in play when multiple parties interact with a subject:

- **Subject identity** is held by the **originator**: the eVault that first minted the subject's W3ID and registered it. Identity-layer concerns (W3ID transfer, retirement, key rotation, name binding) belong to the originator's eVault and are the identity chapter's concern. The originator does not thereby control what other eVaults may say about the subject.
- **Record holdership** belongs to the eVault storing the record. The holder controls who may write to records it holds (itself and any delegated principals) and governs lifecycle decisions about them. Access rules are a future page.
- **Record authorship** belongs to the principal that wrote the record, identified by attribution on each write. Holdership and authorship often coincide: in the worked example, each holder also authors its own record. They come apart when the holder grants write access to another principal, for example when a doctor writes a prescription into a patient's eVault. The record then lives under the holder's storage sovereignty while remaining the author's record.
- **Domain-level roles** (author, contributor, editor, commenter, reviewer) belong to the type vocabulary, not to the architecture. A `social:BlogPost` type may declare `author` and `contributors` as properties referencing person W3IDs; the architecture does not name those roles.

Three patterns for relating an actor to a subject:

- **A property of the subject's record.** The actor is named in a property of the record (e.g., `author: did:w3ds:alice`, `contributors: [did:w3ds:bob]`). The record's holder decides whether and how the property is set. Use this when the actor's role is part of what the subject is (a post's author, a transaction's parties, a contract's signatories).
- **A separate subject.** The actor's interaction is its own subject with its own W3ID, referencing the original (e.g., a `social:Comment` entity with `inReplyTo: did:w3ds:<post-W3ID>`). Use this when the interaction is a thing in its own right (comments, replies, reactions, citations). Patterns for referring to subjects external to W3DS that have no W3ID (DBpedia entities, Wikidata items, schema.org concepts, abstract concepts) are a separate concern, deferred to a future revision of this page.
- **A parallel record about the same subject.** The actor writes a record with the same subject W3ID, typically held in their own eVault but optionally in any eVault that has granted them write access. The record may add a new type the subject is also of (multi-typing, as in the worked example above where the owner's record adds `personal:Asset`, the service center's adds `svc:Equipment`, and the government's adds `gov:RegisteredVehicle`) or contribute entries to multi-valued properties of the subject (each reactor's eVault adds one entry to `reactions: [...]`; each reader's eVault adds one entry to `readBy: [...]`). Parallel records and the separate-subject pattern are alternative modellings for many cases; the trade-off is that parallel records merge into the subject on resolve, while separate entities give each contribution its own W3ID and lifecycle.

The architecture supports all three; choosing among them is a type-modeling decision.

> [!WARNING]
> **Open question: Who may hold a parallel record about which subject?** Under the linked-data principle, any eVault may hold a record keyed by any subject W3ID. Whether the architecture restricts this for sensitive subject classes (people in particular), allows the originator to constrain it, or leaves it open and relies on reader-side trust filtering is undecided. Defamation, impersonation, and spam-resistance live in this gap.

---

## 4. Record identity as opt-in metadata

When a consumer asks for attribution (which eVault asserted which property), the merged view exposes record identifiers as metadata:

```json
{
  "@id": "did:w3ds:car-001",
  "@type": [
    "auto:Vehicle",
    "personal:Asset",
    "svc:Equipment",
    "gov:RegisteredVehicle"
  ],
  "auto:name": "Volkswagen Golf",
  "auto:vin": "WVWZZZ1KZ8W123456",
  "personal:name": "Goldie",
  "personal:acquiredAt": "2024-06-15",
  "svc:lastServicedAt": "2026-04-10",
  "svc:nextDueAt": "2027-04-10",
  "gov:licensePlate": "ABC-123",
  "gov:registeredOwner": "did:w3ds:alice",
  "w3ds:record": [
    {
      "@id": "did:w3ds:rec-car-manufacturer",
      "w3ds:asserts": ["auto:name", "auto:vin"]
    },
    {
      "@id": "did:w3ds:rec-car-owner",
      "w3ds:asserts": ["personal:name", "personal:acquiredAt"]
    },
    {
      "@id": "did:w3ds:rec-car-service",
      "w3ds:asserts": ["svc:lastServicedAt", "svc:nextDueAt"]
    },
    {
      "@id": "did:w3ds:rec-car-gov",
      "w3ds:asserts": ["gov:licensePlate", "gov:registeredOwner"]
    }
  ]
}
```

The default wire form omits `w3ds:record`; record provenance surfaces only on request. Each record identifier is a W3ID in its own right, addressable independently of the subject. The `w3ds:` prefix names the foundational W3DS namespace, which houses architecture-level metadata (provenance, attribution, system-managed properties); domain vocabularies use their own prefixes (`auto:`, `personal:`, `svc:`, `gov:`).

A record's W3ID is stable across writes: it names the storage unit, not any particular state of it. Writes mutate the record's properties; the entries describing those writes form a recoverable history (elaborated on a future page) without changing the record's identity.

> [!WARNING]
> **Open question: Provenance vocabulary alignment.** Should `w3ds:record` align with an established provenance ontology, notably W3C [PROV-O](https://www.w3.org/TR/prov-o/)? The current shape is W3DS-internal; alignment is deferred.

---

## 5. Writes

A **write** changes one or more properties of a record. The first write to a record creates it; subsequent writes update it.

```json
{
  "title": "Walking the Camino, day 1 (revised)",
  "body": "Updated notes from the day."
}
```

Five commitments:

- **A write targets a single record.** The record carries its own type reference; validation runs against that type. Operations that span multiple records (whether in the same eVault or across eVaults) decompose into one write per record; observers may see them at different times.
- **A write originates from one service in one request.** Each write therefore carries a single attribution.
- **A write names the properties it changes and commits them atomically.** All named properties on the record commit, or none do; properties not named are untouched. Two services writing different properties (whether to the same record or to different records) do not collide. Outcomes that depend on more than one write are coordinated above the data layer.
- **Each property has its own access rules.** A write that touches any property the service cannot change is rejected entirely.
- **Writes may opt into state-gating.** A write may declare per-property preconditions; if any fails, the entire write is rejected.

> [!WARNING]
> **Open question: ACL defaults and inheritance.** The commitment names the grain at which access is evaluated (per property); it does not say where each property's effective rule comes from. Defaults could live at the record level (one ACL covers every property in the record unless overridden per slot), the type level (the type declares default rules for its declared slots), the eVault level (a user-set baseline that types and records refine), or other levels yet to be named.

> [!WARNING]
> **Open question: Multi-record batching.** Single-record writes leave cross-record consistency as an application concern. Should the architecture provide an additive batching mechanism that commits writes to several records atomically (across one eVault or several), and if so under what semantics?

---

## 6. Property operations

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
- `$increment` and `$decrement` apply to integer properties. They are commutative: two services concurrently writing `$increment(2)` and `$increment(3)` to the same property compose to a net `+5` regardless of arrival order, with no read-modify-write required.

This is the basic operator set; more are likely to be added as concrete needs emerge.

Whether a multi-valued property is unordered (a set) or ordered (a list) is declared in the type definition (the precise representation is an open question); ordered properties additionally support positional operations (`$insertAt`, `$removeAt`). Positional operations are state-dependent (the position depends on knowing the prior state), so safety-conscious writers should guard them with state-gating.

> [!NOTE]
> JSON Patch (RFC 6902) was considered as a strict superset and not adopted, for three reasons. It treats every collection as an ordered array, forcing array semantics on properties that are conceptually sets. Its addressing is path-based rather than per-property, so it does not match how the data architecture reasons about access, history, and convergence. And it has no native operations for counters, so the architecture would need extension operations anyway, losing the compatibility benefit.

---

## 7. References

A property whose value is a subject's W3ID is a reference.

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

Creating records for several new subjects that reference each other is sequenced one write per record: a write creates one record with its newly minted subject W3ID, and a subsequent write creates the next record referencing the first. Acyclic reference graphs work straightforwardly this way. Cyclical references at creation time leave a brief window where one side points at a not-yet-created subject; closing that window atomically would require the multi-record batching mechanism flagged as an open question in the writes section.

> [!WARNING]
> **Open question: Reference target validity.** A reference is a W3ID pointer. What happens when (a) the target does not exist, or (b) the target exists but its type does not match the reference property's declared range? Accept silently (conformance is the reader's concern)? Reject the write at the eVault? Or accept with a warning surfaced to the writer?

---

## 8. Mapping from the TRL6 prototype

TRL6 stored each described thing (a post, user, message) as a **MetaEnvelope** bound to one ontology, with its fields split into per-field **Envelope** nodes in Neo4j. The MetaEnvelope's W3ID served both as the addressable storage handle and as the identifier of the thing being described; these two roles were conflated. TRL6 also allowed MetaEnvelopes to nest (a `VerifiableCredential` containing an `OrganisationRecord` containing an `IdentityProfile`, for example) and aimed to deduplicate shared fields (such as `email`) by referencing the same Envelope from multiple MetaEnvelopes; in practice this turned reads into multi-hop graph traversals and created write-lock contention on shared Envelope nodes.

TRL7 separates and simplifies. **Subject** is the referent (named by a W3ID); **record** is the storage unit (with its own record identifier); **entity** is the logical representation between them. A TRL6 MetaEnvelope maps to a TRL7 record, and the MetaEnvelope's W3ID maps to the subject's W3ID. The per-field Envelope has no TRL7 counterpart; properties attach to the record directly, and shared subjects are referenced by W3ID rather than by sharing inner storage nodes. TRL6 assumed one MetaEnvelope per subject and one ontology per MetaEnvelope; TRL7 allows multiple records per subject (across eVaults and within one eVault) and multi-typing across records, so different platforms or authorities can contribute under their own models without coordinating writes.
