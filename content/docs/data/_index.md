---
title: Data Architecture
weight: 40
---

# Data Architecture

The data architecture specifies how user-owned data is shaped, accessed, evolved, and propagated.

---

## 1. At a glance

- User data lives in the user's eVault, never centralized; services contribute and read without a central authority.
- The unit of data is the **entity**: one subject, one stable W3ID, one or more types, named properties, recoverable history.
- The unit of change is the **property write**: atomic across everything it names, per-property access, optional state-gating.
- Types are W3ID-pinned shapes; an entity can carry multiple types and accumulate them as services join.
- Old writes stay valid against their original type; data outlives services.

---

## 2. Scope

A user's data lives in their own eVault, not in the services that act on it. This chapter specifies what that means.

In scope:

- The shape of data and of a change to it.
- How multiple independent services contribute and read without a central authority and without conflict.
- How shared meaning travels with the data so it outlives the services that produced it.
- Access and authority (explicit grant, revocability, accountability).
- Lifecycle (creation, archival, removal, erasure).
- Propagation (how a change becomes visible outside its eVault).

Out of scope:

- Identity (its own chapter).
- Concrete components ([Components](../components)).
- The wire surface developers code against ([API](../api)).

---

## 3. Glossary

Chapter-scoped quick reference.

- **entity**. The unit of data: a single subject with a stable W3ID, a type set, named properties and history.
- **property**. A named slot on an entity holding one or more values. The unit of change is a property write.
- **type set**. The set of entity types an entity carries; accumulated across writes and unordered.
- **model**. The unit of publication: a coherent bundling of entity types, properties, and value types.

---

## 4. What is queued for elaboration

The following are queued for further refinement.

- **Concurrency.** How concurrent writes from multiple services compose within an eVault, and the event record that captures them.
- **Propagation.** How a change inside one eVault becomes visible to other eVaults and observers, under sovereignty across boundaries.
- **Reads.** Default and opt-in read shapes, including conformance queries and cross-replica or cross-eVault behavior.
- **History.** What is recorded for every accepted write, what is recoverable, and how history relates to audit and backup.
- **Access.** Per-property grants and revocation, and the user-facing legibility surface.
- **Lifecycle.** Removal, redaction, and erasure semantics, and how they interact with history, propagation, and access.
- **Further detail on the fundamentals.** Type sharing infrastructure, declared composition, alignment, and runtime conformance.
