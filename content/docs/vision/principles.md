---
title: Principles
weight: 2
---

# Principles

These principles define the fundamental commitments and invariants of the system.

They are not implementation requirements, but constraints used to evaluate all design and architectural decisions.

---

## 1. Data Sovereignty

Data belongs to the entity it concerns. Services process it; they do not own it.

- Custody does not imply ownership
- No service may operate on data against the entity's wishes
- Processing data does not transfer ownership

---

## 2. Separation of Data and Services

Data and services are separate concerns. Services operate on data; they do not define it.

- Data must exist independently of the services that produce or consume it
- Data is not a byproduct of services; it exists in its own right
- The same user data must be able to serve many services, without the user maintaining separate records for each

---

## 3. Right to Erasure

Users retain the right to remove their data without depending on the cooperation of any service.

- Services must not be able to block or delay erasure
- Erasure must be final, not merely suspended or hidden
- Users must decide whether the data's history is erased with it

---

## 4. Portability

Users must be able to move their data between services and implementations without loss of continuity.

- Data must be exportable in a complete, self-contained form
- Exported data must be re-importable into any conforming service
- Users must not be locked in by any service or provider, regardless of the reason for leaving

---

## 5. Service Independence

Services must not become gatekeepers of user participation.

- Users must remain free to choose among conforming providers of the framework
- Multiple services must be able to operate in parallel on the same data
- No service may obstruct a user's use of other services

---

## 6. Persistent Identity

Identity must exist independently of any particular service.

- Identity must remain stable over time
- Identity must be reusable across contexts
- Identity must not be controlled by a single provider

---

## 7. Interoperability

Interoperability is achieved through shared meaning, not through bilateral agreements between services.

- Shared terms with shared meaning must be enough for services to operate on the same data
- No service should need to negotiate with another to participate in the ecosystem
- Multiple conforming implementations of the framework must be able to interoperate

---

## 8. Semantic Consistency

Meaning must travel with the data, not with the service that produced it.

- Data must carry its meaning, or a reference to where its meaning can be found
- Data must be interpretable without reference to its originating service
- Where meanings differ between services, resolution must come from shared ground, not ad hoc negotiation

---

## 9. Data Longevity

Data must remain interpretable over time, independent of the services or technology that produced it.

- Meaning must outlive the services that first carried the data
- Data must remain usable after the services that produced it are gone
- Evolution of data structures must not invalidate older data

---

## 10. Verifiability and Accountability

Changes to data must be attributable to the actor that made them.

- It must be possible to determine the origin of data and of changes to it
- How the system acts on data must be inspectable
- Historical records must be preserved as long as verifiability depends on them

---

## 11. Minimal and Explicit Control Points

Any point of control in the system must be limited, explicit, and justified.

- No control mechanism may be hidden
- Trust assumptions and critical dependencies must be identifiable
- Any control must be no broader than its purpose requires
- Every control point must have a stated reason

---

## 12. Explicit and Revocable Access

Access to an entity's data must be the result of an explicit grant and must be revocable at any time.

- Every access must be granted, not assumed
- Every grant must be individually rescindable
- Delegation must not become a one-way transfer of control

---

## 13. User Awareness and Control

Entities must be able to understand and influence how the system operates on their behalf.

- Entities must be able to see what data exists about them, what it means, where it lives, and who can access it
- Entities must be able to revisit and change prior decisions about their data
- Entities must be able to anticipate how the system will act on their data

---
