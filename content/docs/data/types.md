---
title: Type System
weight: 20
---

# Type System

Without types, services cannot agree on what data means. This page commits to the fundamentals: what a type is, how types are identified and versioned, how property meaning is semantically grounded.

---

## 1. What a type is

Two kinds of type:

- **Entity types** describe a kind of subject (a blog post, a transaction, a profile). An entity type declares the set of properties an entity of this kind is expected to carry.
- **Value types** describe the kind of value a property holds. Three families: **scalars** (string, integer, decimal, boolean, datetime, URI, ...), **references** (a W3ID pointing at another entity, optionally constrained to a target entity type), and **nested objects** (a structured value embedded inline in a parent property, conforming to an entity type's shape but without its own W3ID).

Each property has a name, a value type, a multiplicity (single- or multi-valued), and a canonical IRI from a published source. A property may carry value constraints (regex, enumerations, value ranges) that further refine its value type.

> [!WARNING]
> **Open question: Validation strictness.** Whether the eVault enforces declared-only writes (closed-world per referenced type), accepts open-world writes (containing non-declared properties), or supports per-type opt-in. Whether required-ness is enforced at write time, at query time, or both.

> [!WARNING]
> **Open question: Ordering on multi-valued properties.** A multi-valued property may be unordered (a set) or ordered (a list); the two have different storage and convergence semantics. Where does ordering sit in the type definition, and does it carry the same weight as value type and multiplicity, or is it a separate annotation?

An entity may carry several entity types, accumulated across the services that write to it.

> [!WARNING]
> **Open question: Storage and forks under multi-typing.** When two services contribute to the same entity under different types or versions, when do their writes share storage and when do they diverge into parallel streams? Cross-version (same model, different majors) and cross-model (independent declarations of the same IRI) are the two main cases.

> [!WARNING]
> **Open question: Read modes.** What a default read returns when an entity carries multiple types or forked storage. Whether the architecture commits to one read mode or several (e.g. type-bound or global), and which is the default.

The unit of publication is a **model**: a coherent bundling of entity types, properties, and value types under one W3ID and [Semantic Versioning (SemVer)](https://semver.org).

---

## 2. Identity, versioning, immutability

The architecture commits to:

- **W3ID-identified, SemVer-pinned.** Models, entity types, and user-defined value types are addressed via W3IDs. A type's stable identity holds across versions; the version pin names a specific snapshot.
- **Concrete pins, not ranges.** A pin names exactly one version. Range syntax (`^1.2.0`, wildcards) does not appear. The reasons are reproducibility (a published registration must compose the same way every time it is read), longevity (data outlives services and must remain interpretable against the exact version it was written for), and supply-chain safety (a pin cannot drift to an unauthorised snapshot).
- **Additive evolution by default.** Minor versions add properties; existing properties do not change meaning. Breaking changes (removing a property, narrowing a value type, changing an IRI) require a new major version.
- **Immutable once published.** A same-SemVer republish is invalid. Consumers may resolve a version once and cache the result indefinitely; subsequent resolutions yield identical bytes. Errors or improvements are addressed by publishing a new SemVer, not by mutating the existing one.
- **Old versions stay interpretable indefinitely.** Data outlives services; types outlive their authors. Any reader can ask for the definition of any version that data has been written against, even decades later.

> [!WARNING]
> **Open question: Where types are published.** Type registrations must be resolvable and durable. Where do they reside? A registry, the author's eVault or somewhere else?

---

## 3. Semantic grounding

Every property carries a canonical IRI from a published source: schema.org, Dublin Core, vCard, Activity Streams, or W3DS-native where existing sources leave a gap. The IRI is the canonical identity of the property; local labels are rendering choices. The default authoring instinct is to reuse external sources before minting W3DS-native IRIs.

W3DS-native IRIs are governed in three tiers, visible in every type reference via the namespace prefix so the governance origin of any IRI is obvious at a glance:

- **Core** (`w3ds`): foundational types every platform depends on. Maintained by the W3DS project.
- **Domain** (`social`, `health`, `edu`, ...): cross-platform types shared across a community. Maintained by domain communities.
- **Platform** (`blabsy`, `pictique`, ...): platform-specific types. Maintained by the individual platform.

A platform type that gains cross-platform adoption graduates to a domain type via alignment, not by moving data; existing entities keep their original type and an alignment records the equivalence.

> [!WARNING]
> **Open question: Alignment.** How equivalences between properties are recorded, who publishes them, and how consumers opt in at read time. Cases include: cross-vocabulary (`schema:name` ≡ `foaf:name` for the same concept), cross-type (two entity types with overlapping properties), cross-major (a property renamed in a new major), and cross-tier (a platform property graduating to a domain term).

> [!NOTE]
> Encoding is **JSON-LD**. W3DS publishes a JSON-LD context for every type. Developers see plain JSON; the architecture uses the context at boundaries (semantic consumers, alignment, regulators).

---

## 4. Authoring

Types are authored in **LinkML** (YAML). LinkML's modeling primitives (`slot_uri`, `class_uri`, `is_a`, `range`, mixins, SemVer-pinned imports) map directly onto the architecture's commitments, and it compiles to JSON Schema, SHACL, JSON-LD context and other formats, so the JSON Schema ecosystem stays reachable. A minimal example:

```yaml
id: did:w3ds:5f8a1c2d-9b40-4f5d-9e6e-8c1a4b7d2e3f
name: social
version: 1.0.0
prefixes:
  linkml: https://w3id.org/linkml/
  social: did:w3ds:5f8a1c2d-9b40-4f5d-9e6e-8c1a4b7d2e3f
  schema: did:w3ds:7d2c4f6a-3b81-4e5d-9a2c-8e1b3f5d7a9c
imports:
  - linkml:types
  - did:w3ds:7d2c4f6a-3b81-4e5d-9a2c-8e1b3f5d7a9c/v/2.3.0 # W3DS-registered schema.org model

classes:
  BlogPost:
    is_a: schema:Article
    slots: [id, title, body, tags]

slots:
  id:
    identifier: true
    range: uri
  title:
    range: string
    slot_uri: schema:headline
  body:
    range: string
    slot_uri: schema:articleBody
  tags:
    range: HashtagString
    slot_uri: schema:keywords
    multivalued: true

types:
  HashtagString:
    uri: social:HashtagString
    base: str
    pattern: "^#[a-zA-Z0-9_]+$"
```

> [!WARNING]
> **Open question: Composition.** How entity types compose (inheritance, mixins, cross-model imports), what the published artifact looks like to consumers, and when composition is resolved (at publication, at read, or at write). Resolution at publication makes a type self-sufficient against later upstream evolution; later resolution keeps the artifact small but pushes work to consumers.

> [!WARNING]
> **Open question: Authoring format.** LinkML is proposed as authoring format. Do we see other more appropriate alternatives?
