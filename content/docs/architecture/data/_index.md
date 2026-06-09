---
title: Data Architecture
weight: 10
---

# Data Architecture

The data architecture specifies how user-owned data is shaped, accessed, evolved, and propagated.

---

## 1. At a glance

- User data lives in the user's eVault, never centralized; services contribute and read without a central authority.
- An **entity** is the logical representation of a subject: a stable W3ID, one or more types, named properties, and recoverable history. Its data lives in one or more **records**, each in some eVault.
- A **write** changes one or more properties of a single record: atomic across the properties it names, with optional state-gating.
- Types are W3ID-rooted and version-pinned shapes; an entity can carry multiple types and accumulate them as services join.
- Old writes stay valid against their original type; data outlives services.

---

## 2. Scope

A user's data lives in their own eVault, not in the services that act on it. This chapter specifies what that means.

In scope:

- The shape of data and of a change to it.
- How multiple independent services contribute and read without a central authority and without conflict.
- How shared meaning travels with the data so it outlives the services that produced it.
- Authority and accountability (holdership, authorship, attribution of writes).
- Lifecycle (creation, archival, removal, erasure).
- Propagation (how a change becomes visible outside its eVault).

Out of scope:

- Identity (its own chapter).
- Access control: grants, revocation, the grain at which access is evaluated, and sensitive-field protection (its own chapter).
- Concrete components ([Components](../../components)).
- The wire surface developers code against ([API](../../api)).

---

## 3. Glossary

Chapter-scoped quick reference.

- **subject**. The thing being described (a person, post, transaction, concept). Identified by a stable W3ID.
- **entity**. The logical representation of a subject: identifier, type set, named properties, history.
- **record**. A stored unit of an entity in an eVault. An entity's data may be split across multiple records, including multiple records in the same eVault when different services or models contribute. Carries the subject's W3ID as its `@id`, and additionally has its own eVault-scoped record identifier (`w3id:<eVault>/<record>`) naming the storage unit, stable across writes.
- **property**. A named slot on an entity holding one or more values; property values are stored within records and change through writes.
- **type set**. The set of entity types the records declare the subject to be. The effective type set is the union across records.
- **model**. The unit of publication: a coherent bundling of entity types, properties, and value types.

---

## 4. What is queued for elaboration

The following are queued for further refinement.

- **Concurrency.** How concurrent writes from multiple services compose within an eVault, and the event record that captures them.
- **Propagation.** How a change inside one eVault becomes visible to other eVaults and observers, under sovereignty across boundaries.
- **Reads.** Default and opt-in read shapes, including conformance queries and cross-replica or cross-eVault behavior.
- **History.** What is recorded for every accepted write, what is recoverable, whether a particular past snapshot can be addressed (and how that interacts with identity and registry), and how history relates to audit and backup.
- **Lifecycle.** Removal, redaction, and erasure semantics, and how they interact with history, propagation, and access.
- **Further detail on the fundamentals.** Type sharing infrastructure, declared composition, alignment, and runtime conformance.
