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
  "@id": "w3id:5f8a1c2d-9b40-4f5d-9e6e-8c1a4b7d2e3f",
  "@type": "social:BlogPost",
  "title": "Walking the Camino, day 1",
  "body": "Started today from Saint-Jean-Pied-de-Port. Twenty kilometers in."
}
```

> [!NOTE]
> JSON is the default encoding used throughout this chapter; the model itself is encoding-agnostic. A specific wire transport may serialize entities as protobuf, an RDF serialization, or another format. What the model describes is the logical shape.

The JSON wire form uses the reserved JSON-LD `@id` and `@type` keywords, leaving `id` and `type` available as ordinary property names without the risk of naming collisions.

Four things every entity has:

- An **identifier** (`@id`). A W3ID naming the subject the entity describes. Examples in this chapter use the `w3id:<UUID>` form; W3ID can also be written in other forms in non-JSON-LD contexts.
- A **type set** (`@type`). The set of entity types the subject is declared to be, as the union of types declared by its records (a record may declare one type or several). The set is unordered; all members are equal. On the wire, `@type` renders as a string when the set has one member or an array when it has multiple.
- A set of **properties**. Each property has a name and one or more values. Reads return the entity's current properties, filtered to what the reader has requested and is allowed to access.
- A recoverable **history**. Every accepted write appends an immutable entry to the target record's history, capturing what changed, by whom, when, and against which type version. The entry shape and read interface are elaborated on a future page.

---

## 2. Parallel records

An entity's data lives in one or more **records**, each stored in some eVault. A record carries the subject's W3ID as its `@id` and has its own **record identifier**: an eVault-scoped W3ID (`w3id:<eVault>/<record>`) naming the storage unit itself, surfaced as metadata only when a consumer asks for provenance.

Three consequences follow:

- **An entity's data may be split across many records, in one or many eVaults.** A single eVault may hold several records about one subject when different services write under different models; different eVaults may also hold parallel records about the same subject. Each record contributes properties independently; how much a reader pulls together at read time (one type, a subset, or all) is a reader-side choice.
- **References point at subjects, not records.** Following a reference means asking "which eVaults hold records about this subject?", not "where is the canonical record?". Cross-eVault navigation is part of read resolution, not part of the data shape.
- **A subject's identity outlives any single eVault.** Migration, key rotation, or provider failure does not re-mint the subject's W3ID; the eVault hosting any record about the subject can change while the subject's identity stays the same.

A short scenario, using mnemonic W3IDs for readability. The subject is a specific car (`w3id:car-001`). Four records describe it:

```json
// Car's eVault (authored by manufacturer)
{
  "@id": "w3id:car-001",
  "@type": "auto:Vehicle",
  "name": "Volkswagen Golf",
  "vin": "WVWZZZ1KZ8W123456"
}
```

```json
// Car's eVault (authored by service center)
{
  "@id": "w3id:car-001",
  "@type": "svc:Equipment",
  "lastServicedAt": "2026-04-10",
  "nextDueAt": "2027-04-10"
}
```

```json
// Car's eVault (authored by government body)
{
  "@id": "w3id:car-001",
  "@type": "gov:RegisteredVehicle",
  "licensePlate": "ABC-123",
  "registeredOwner": "w3id:alice"
}
```

```json
// Alice's eVault (authored by Alice)
{
  "@id": "w3id:car-001",
  "@type": "personal:Asset",
  "name": "Goldie",
  "acquiredAt": "2024-06-15"
}
```

In this example, three records live in the car's eVault and one in Alice's; all four use distinct models. A reader resolving `w3id:car-001` chooses how much of the entity to project across them. Each record is under the storage sovereignty of the eVault holding it; merging is read-side and reader-driven.

Property names can collide across models. Both `auto:Vehicle` and `personal:Asset` declare a `name`, but the CURIE prefix resolves them to different IRIs: `auto:name` (the product name, `"Volkswagen Golf"`) and `personal:name` (a pet name, `"Goldie"`). A reader projecting through one type sees one `name`; a multi-type merge exposes both, as the merged view below shows.

Instead of parallel records, the owner's and service center's contributions could be modeled as separate entities (an `Ownership` entity referencing the car, a `ServiceRecord` entity referencing it). Both shapes are architecturally supported.

Overlapping models can collapse parallel contributions into one record. When two services share a base vocabulary (two social platforms both extending `social:Post`, say), a user's post can live as a single multi-typed record (`@type: ["platformA:Post", "platformB:Post"]`) held in the user's eVault. Both platforms have write access granted by the user; shared properties live under `social:`, platform-specific ones under each platform's prefix. The post is one subject, one record, with many cooperating writers.

A person operates exactly one eVault. This constrains how many eVaults a single person runs, not how a subject's records are distributed: a subject's data may still span several eVaults (the car above has records in its own eVault and in Alice's).

> [!NOTE]
> One person, one eVault is a current policy, not a constraint the data model imposes. The model already supports a subject's records being spread across eVaults, so allowing a person several eVaults later (for role separation, risk segregation, jurisdictional placement, or provider redundancy) would not require changing the data model described here.

> [!WARNING]
> **Open question: Cross-eVault record discovery.** Resolving a subject reaches the originator's eVault, but the architecture does not yet specify how a reader discovers parallel records about that subject held in other eVaults (Alice's record about `w3id:car-001`, for example). A subject-keyed registry entry that resolves to the originator may not by itself reveal those other records.

---

## 3. Authority and roles

Three layers of authority are in play when multiple parties interact with a subject:

- **Subject identity** is held by the **originator**: the eVault that first minted the subject's W3ID and registered it. Identity-layer concerns (retirement, key rotation, etc.) belong to the originator's eVault and are the identity chapter's concern. The originator is not encoded in the identifier; it is recovered by lookup. The originator does not control what other eVaults may say about the subject.
- **Record holdership** belongs to the eVault storing the record. The eVault's controller (the principal acting on its behalf) decides who may write to records the eVault holds (the controller itself and any delegated principals) and governs lifecycle decisions about them. Access rules are the access chapter's concern; how the controller is identified and how control transfers is the open question below.
- **Record authorship** belongs to the principal that wrote the record, identified by attribution on each write. An eVault's controller may also author its records, or may grant write access to other principals. Either way, the record lives under the eVault's storage sovereignty while remaining the author's record.

Domain-level roles (author, contributor, editor, commenter, reviewer) belong to the type vocabulary, not to the architecture. A `social:BlogPost` type may declare `author` and `contributors` as properties referencing person W3IDs; the architecture does not name those roles.

> [!WARNING]
> **Open question: Who may hold a parallel record about which subject?** Under the linked-data principle, any eVault may hold a record keyed by any subject W3ID. Whether the architecture restricts this for sensitive subject classes (people in particular), allows the originator to constrain it, or leaves it open and relies on reader-side trust filtering is undecided. Defamation, impersonation, and spam-resistance live in this gap.

> [!WARNING]
> **Open question: eVault control and transfer.** `w3ds:holder` names the eVault storing a record; who controls the eVault (access rules, lifecycle, transfer) is a separate concern. When subject ownership transfers, control of the subject's eVault should transfer too, without relocating data. The representation and transfer mechanism are deferred.

---

## 4. Provenance as opt-in metadata

When a consumer asks for provenance, the merged view exposes record metadata:

```json
{
  "@id": "w3id:car-001",
  "@type": [
    "auto:Vehicle",
    "svc:Equipment",
    "gov:RegisteredVehicle",
    "personal:Asset"
  ],
  "auto:name": "Volkswagen Golf",
  "auto:vin": "WVWZZZ1KZ8W123456",
  "svc:lastServicedAt": "2026-04-10",
  "svc:nextDueAt": "2027-04-10",
  "gov:licensePlate": "ABC-123",
  "gov:registeredOwner": "w3id:alice",
  "personal:name": "Goldie",
  "personal:acquiredAt": "2024-06-15",
  "w3ds:record": [
    {
      "@id": "w3id:car-evault/rec-manufacturer",
      "w3ds:holder": "w3id:car-evault",
      "w3ds:author": "w3id:manufacturer",
      "w3ds:asserts": ["auto:name", "auto:vin"]
    },
    {
      "@id": "w3id:car-evault/rec-service",
      "w3ds:holder": "w3id:car-evault",
      "w3ds:author": "w3id:service-center",
      "w3ds:asserts": ["svc:lastServicedAt", "svc:nextDueAt"]
    },
    {
      "@id": "w3id:car-evault/rec-gov",
      "w3ds:holder": "w3id:car-evault",
      "w3ds:author": "w3id:gov-body",
      "w3ds:asserts": ["gov:licensePlate", "gov:registeredOwner"]
    },
    {
      "@id": "w3id:alice-evault/rec-owner",
      "w3ds:holder": "w3id:alice-evault",
      "w3ds:author": "w3id:alice",
      "w3ds:asserts": ["personal:name", "personal:acquiredAt"]
    }
  ]
}
```

Record provenance surfaces only on request. Each record identifier is an eVault-scoped W3ID (`w3id:<eVault>/<record>`), globally unique and addressable independently of the subject. The `w3ds:` prefix names the foundational W3DS namespace, which houses architecture-level metadata (provenance, system-managed properties); domain vocabularies use their own prefixes (`auto:`, `personal:`, `svc:`, `gov:`).

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

- **A write targets a single record.** The write declares the type it is being made under; validation runs against that type. A record's type set accumulates across writes. An operation that spans multiple records decomposes into one write per record. Because each record lives in a single eVault, a write never crosses an eVault boundary: an operation that spans multiple eVaults decomposes into separate writes per eVault. Since eVaults are independent sovereign stores, no operation commits atomically across them, and observers may see the component writes at different times.
- **A write originates from one service in one request.** Each write therefore carries a single attribution.
- **A write names the properties it changes and commits them atomically.** All named properties on the record commit, or none do; properties not named are untouched. Two services writing different properties (whether to the same record or to different records) do not collide. Outcomes that depend on more than one write are coordinated above the data layer.
- **A write is validated as a whole.** Each named property is validated against the declared type; if any property fails, the entire write is rejected and the record is left unchanged, never partially applied.
- **Writes may opt into state-gating.** A write may declare per-property preconditions; if any fails, the entire write is rejected.

The validation contract is strict on the writer's side, relaxed on the reader's. Each write succeeds only against its declared type; readers project through chosen types and apply their own conformance policy. The [type system](types) page elaborates type extension and inherited-property rules.

> [!WARNING]
> **Open question: Multi-record batching.** Single-record writes leave cross-record consistency as an application concern. Should the architecture provide an additive batching mechanism that commits writes to several records in a single eVault atomically, and if so under what semantics? Batching cannot span eVaults: an eVault is a sovereign store, so cross-eVault operations always decompose into per-eVault writes (see the first commitment above).

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
  "@id": "w3id:7c3e9f1a-2b8d-4e3a-9c1f-6e5d7a8b3c2d",
  "@type": "social:BlogPost",
  "title": "Walking the Camino, day 1",
  "body": "Started today from Saint-Jean-Pied-de-Port.",
  "author": "w3id:9b8e2f4a-1c3d-4e5f-8a7b-6c5d4e3f2a1b"
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
