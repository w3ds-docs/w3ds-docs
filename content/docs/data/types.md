---
title: Type System
weight: 20
---

# Type System

Without types, services cannot agree on what data means. In W3DS, multiple services touch the same user data and the data outlives them, so shared meaning is essential for interop and for keeping data interpretable as services evolve. This page commits to the fundamentals: what a type is, how types are identified and versioned and how data meaning is semantically grounded.

---

## 1. What a type is

W3DS distinguishes two kinds of types:

- **Entity types** describe a kind of subject (a blog post, a transaction, a profile). An entity type has a name and declares the set of properties an entity of this kind is expected to carry.
- **Value types** describe the kind of value a property holds.

Each **property** has a name, a **range** (the value or entity type it accepts), a **multiplicity** (single- or multi-valued), and may carry value constraints (regex, enumerations, value or cardinality ranges) that further refine its range.

A range is one of three families:

- **Scalar types**: atomic typed values. The architecture provides built-in scalar types (string, integer, decimal, boolean, datetime, URI, ...); publishers may define custom scalar types that refine a built-in scalar type with a pattern, lexical format, or encoded structure. Examples: a `HashtagString` (regex-constrained string), a `WKTPoint` (point geometry encoded as a WKT-formatted string), an `ISBN`.
- **Compound value types**: publisher-defined structures with named properties and no identity. Two values with the same property values are equal. Examples: `Money` (amount and currency), `Coordinate` (latitude and longitude) or an `Address` value.
- **Entity types**: when an entity type appears as a range, the property is a **reference**; its value is the target subject's W3ID, and the target is expected to carry the named type.

| Family               | Form     | Identity |
| -------------------- | -------- | -------- |
| Scalar types         | atomic   | none     |
| Compound value types | compound | none     |
| Entity types         | compound | yes      |

All property names in a write must be declared, otherwise the write is rejected by the eVault. Service-internal data without a declared type lives in the service's local storage.

> [!WARNING]
> **Open question: Undeclared literal properties and types.** W3DS requires every property name in a write to be declared, and every type to resolve to a published declaration. A proposal under discussion is to also allow undeclared, reader-interpreted values where no vocabulary exists yet: a property whose value is a bare literal with no declared type, or a non-resolvable literal type (for example "a mammal") left to reader interpretation.

> [!WARNING]
> **Open question: Ordering on multi-valued properties.** A multi-valued property may be unordered (a set) or ordered (a list); the two have different storage and convergence semantics. Where does ordering sit in the type definition, and does it carry the same weight as value type and multiplicity?

An entity may carry several entity types in its **type set**, accumulated across the records that describe it.

An entity type can extend one or more parent types, inheriting their declared properties. Parents must be entity types defined in imported W3DS models; inheritance graphs stay within W3DS. External vocabularies participate via alignment, not as inheritance targets.

**Inherited property constraints are frozen.** A subtype inherits the parent's properties unchanged: it cannot narrow, widen, or otherwise redefine inherited property constraints (range, multiplicity, required-vs-optional, value constraints). A subtype may declare new properties with whatever constraints it likes. A service that needs stricter validation on a shared property either introduces its own or applies the constraint above the data architecture, in its own service layer.

> [!NOTE]
> Why inherited constraints are frozen: a reader projecting an entity through a parent type must get valid data without resolving the subtype. Freezing keeps every subtype substitutable for its parent, so a parent-type projection never has to know which subtype wrote the data. Allowing a subtype to narrow or widen an inherited constraint would break that substitutability and force readers to resolve the full subtype before trusting the parent-type view.

> [!WARNING]
> **Open question: Conflicting types and model evolution across records.** Records about the same subject may use different (or semantically conflicting) types and different major versions of the same type. The architecture surfaces all records with attribution but does not pick a winner. Specific unresolved cases include semantic conflicts between co-present types, cross-major drift of the same model (where properties may have been renamed or regrounded between majors), and single-valued property values asserted by different records. Whether to commit to default resolution rules, and where those rules belong (reads, concurrency, types), is deferred.

> [!WARNING]
> **Open question: Default read projection.** Reads project through reader-chosen types (one type, a subset, or the union across all). What the architecture returns when a reader specifies no type, and how reads handle multi-major forks of the same model, is deferred to a future reads page.

> [!WARNING]
> **Open question: Does extension induce implicit type-set membership?** When an entity is typed `BlogPost` and `BlogPost` extends `Article`, does the entity's effective type set include both `BlogPost` and `Article`, or only `BlogPost` until `Article` is added explicitly?

The unit of publication is a **model**: a coherent bundling of entity types, properties, and value types under one W3ID and [Semantic Versioning (SemVer)](https://semver.org).

---

## 2. Identity and versioning

The architecture commits to:

- **W3ID-rooted, version-pinned.** A model is addressed via its W3ID; entity types, properties, and custom value types within the model are addressed via IRIs derived from the model's W3ID (model W3ID plus an anchor). A model's stable identity holds across versions; the version pin names a specific snapshot.
- **Concrete pins, not ranges.** A pin names exactly one version. Range syntax (`^1.2.0`, wildcards) does not appear. The reasons are reproducibility (a published version must compose the same way every time it is read), longevity (data outlives services and must remain interpretable against the exact version it was written for), and supply-chain safety (a pin cannot drift to an unauthorised snapshot).
- **Additive evolution by default.** Minor versions add types or properties; existing properties do not change meaning. Breaking changes (removing or renaming an element, narrowing a value type) require a new major version.
- **Verifiable by content and signature.** Each published version is signed by its publisher over a canonical form of the model. Verification is independent of any specific serialization and of who hosts the bytes: any holder whose bytes project to the signed canonical form can verify intactness and authorship. The publisher's continued existence is not required for a type to remain usable; replicas are authentic by their content and signature, not by their hosting location.
- **Immutable and irrevocable once published.** A published version cannot be mutated and cannot be retracted. Republishing the same version is invalid; withdrawal is invalid. Consumers may resolve a version once and cache or replicate the result indefinitely; subsequent resolutions yield the same canonical form. Errors or improvements are addressed by publishing a new version, not by mutating or withdrawing the existing one.
- **Old versions stay interpretable indefinitely.** Data outlives services; types outlive their authors. Any reader can ask for the definition of any version that data has been written against, even decades later.

> [!WARNING]
> **Open question: Canonicalization and signing scheme.** What canonical form does the publisher's signature cover, and what algorithms compute and verify it? The default proposal is deterministic CBOR ([RFC 8949](https://www.rfc-editor.org/rfc/rfc8949.html)) for the canonical form and COSE ([RFC 8152](https://datatracker.ietf.org/doc/html/rfc8152)) for the signature.

> [!WARNING]
> **Open question: Publication and hosting.** Where is a model published, and how do consumers discover its location? The default proposal is that the architecture does not constrain hosting location: a publisher's eVault is a natural starting point, community-run replica caches are equally legitimate, and identity rests on canonical form, signature, and version pin rather than host. Discovery and minimum durability guarantees (for example, perpetuation when a publisher's eVault is closed) are not yet specified.

---

## 3. Semantic grounding

Every entity type, property, and custom value type has a **W3DS-native identity**: an IRI formed by the model's W3ID plus an anchor naming the element (e.g. `w3id:abc#BlogPost`, `w3id:abc#title`, or `w3id:abc#HashtagString`). These elements do not have their own W3IDs; the model's W3ID is the root from which their IRIs derive. The identity is implicit (derived from the model and the short name) and versionless. Short labels are publisher-defined and fixed for the model version.

The W3DS-native identity resolves to the element's **declaration** in the published model: its structural commitments (properties, ranges, constraints) and a human-readable **description** of what the element means. The declaration is published, signed, and immutable with the rest of the model, so any consumer that resolves the IRI reads the same definition. The IRI is the address; the declaration is the meaning.

Every description must disambiguate any aspect the structural declaration leaves underspecified: units, scale, encoding, controlled sets, or conventional usage. Examples illustrating typical and edge usage are recommended.

An entity type, property, or custom value type can additionally declare one or more **alignments** to IRIs in other published vocabularies (other W3DS models, or external vocabularies such as schema.org, vCard, Activity Streams). Each alignment carries a kind: **exact** (same concept), **close** (near-equivalent), **broad** (the aligned IRI is broader), **narrow** (the aligned IRI is narrower), or **related** (associated but not a match). The W3DS-native identity already carries the element's meaning via its published definition; alignments are additive, broadening reach to consumers that already know the aligned vocabularies.

> [!WARNING]
> **Open question: Are alignment changes breaking?** Once an element declares an alignment, is removing or changing that alignment a breaking change requiring a major version?

> [!NOTE]
> **Inheritance and alignment do different work.** Inheritance connects a W3DS entity type to its parent inside the W3DS-internal graph; the child inherits the parent's declared properties, and the parent must be a W3DS-defined entity type. Alignment records that an element is equivalent (or related) to an IRI in another published vocabulary, whether another W3DS model or an external one; no properties are inherited. An entity type may carry both at the same time.

Anyone may publish a model. The architecture does not classify models into structural tiers; nothing in the architecture enforces them. However, in practice three patterns of reuse breadth tend to emerge:

- **Foundation-level**: maintained for cross-cutting reuse across all of W3DS.
- **Domain-level**: maintained for cross-platform reuse within a domain (social, health, edu, ...).
- **Platform-level**: maintained by a single platform for its own models, not reused.

The expected practice is to match the broadest reasonable scope: ground shared concepts in existing foundational or domain models rather than re-minting them at a narrower scope. When a narrow-scope type gains broader adoption, the equivalence is recorded as an alignment with a broader-scope type. Existing records keep their original type; the alignment carries the cross-model mapping.

> [!WARNING]
> **Open question: Alignment publishing and opt-in.** Who publishes alignments and how consumers opt in at read time. Cases include: cross-model (`schema:name` ≡ `foaf:name`, or a narrow-scope W3DS model aligning with a broader-scope one), cross-type (two entity types with overlapping properties), and cross-major (a property renamed or regrounded in a new major).

> [!NOTE]
> JSON-LD is a natural wire format for the data: `@context` resolves the short names used in the data to IRIs (typically the external alignment IRI when one is declared, e.g. `BlogPost` to `https://schema.org/BlogPosting`; the W3DS-native IRI otherwise). Other serializations (plain JSON with a separate context, RDF Turtle, Protobuf) are equally valid; the architecture commits to the logical model, not to any particular encoding.

> [!NOTE]
> RDFS/OWL inference rules are not part of the architecture, but consumers can apply their own reasoning over the IRIs (W3DS-native or aligned) if their use case calls for it.

---

## 4. Authoring

Models are authored in **LinkML** (YAML). LinkML's modeling primitives (`class_uri`, `slot_uri`, `is_a`, `range`, mixins, mapping slots like `exact_mappings`, version-pinned imports) map directly onto the architecture's commitments. `class_uri` (for entity types and compound value types) and `slot_uri` (for properties) hold the wire IRI used in JSON-LD serializations; the default is the W3DS-native IRI, and setting these to an external IRI also declares an exact alignment. Additional alignments, with their kinds, live in `exact_mappings`, `close_mappings`, `broad_mappings`, `narrow_mappings`, and `related_mappings`. LinkML compiles to JSON Schema, SHACL, JSON-LD context and other formats, so the JSON Schema ecosystem stays reachable.

A minimal example:

```yaml
id: w3id:5f8a1c2d-9b40-4f5d-9e6e-8c1a4b7d2e3f
name: social
version: 1.0.0

prefixes:
  as: https://www.w3.org/ns/activitystreams#
  linkml: https://w3id.org/linkml/
  schema: https://schema.org/
  social: w3id:5f8a1c2d-9b40-4f5d-9e6e-8c1a4b7d2e3f#

imports:
  - linkml:types
  - w3id:7d2c4f6a-3b81-4e5d-9a2c-8e1b3f5d7a9c/v/2.3.0 # W3DS-published model aligned with schema.org

classes:
  BlogPost:
    is_a: Article # extends the W3DS Article class from the imported model
    class_uri: schema:BlogPosting # wire IRI; also declares exact alignment to schema.org
    exact_mappings:
      - as:Article # additional exact alignment
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
    base: str
    pattern: "^#[a-zA-Z0-9_]+$"
```

> [!WARNING]
> **Open question: Version pinning of imports.** LinkML's `imports:` field accepts a URI or CURIE but has no native version-pinning syntax; the imported schema's `version:` is metadata on the resolved schema, not selectable at import time. The default proposal is a `/v/<version>` suffix on the import URI (as shown in the example), resolved by the W3DS-aware toolchain.

> [!WARNING]
> **Open question: Composition.** Several details remain: how same-named properties from multiple parents resolve when they conflict; whether inheritance is transitive through ancestors by default; whether mixins are supported alongside `is_a` (mixins add properties without establishing an `is_a` relationship); what the published artifact contains (resolved/flattened imports, or only its own declarations and a manifest of imports); and when composition is resolved (at publication, at read, or at write). Resolution at publication makes a type self-sufficient against later upstream evolution; later resolution keeps the artifact small but pushes work to consumers.

> [!WARNING]
> **Open question: Authoring format.** LinkML is proposed as the authoring format. The language is platform-independent by design, but its tooling maturity and conventions (notably base types and default casing) are anchored in the Python ecosystem. Is this trade-off acceptable, are there better-suited alternatives, or should we define a native format instead?
