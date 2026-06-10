---
title: Requirements coverage
weight: 4
---

# Requirements coverage

This page maps every published W3ID and eName requirement to the part of this
section that satisfies it. It is meant to be read side by side with the
source:

[W3ID and eName Requirements](https://github.com/w3ds-docs/w3ds-docs/wiki/W3ID-eName-Requirements)

Each row quotes the requirement and links to the explanation.

## Section 2: Identifier syntax

| Requirement | Where it is satisfied |
| --- | --- |
| "A W3ID MUST use UUID syntax based on RFC 4122." | [Anatomy: the shape](../anatomy/#the-shape) |
| "A W3ID UUID string MUST include mandatory dashes in the standard UUID positions." | [Anatomy: rules to remember](../anatomy/#rules-to-remember) |
| "A W3ID UUID string MUST be treated as case-insensitive." | [Anatomy: rules to remember](../anatomy/#rules-to-remember) |
| Global W3IDs require an `@` prefix | [Anatomy: the shape](../anatomy/#the-shape) and [the three classes](../anatomy/#the-three-classes) |
| Local W3IDs use `/` prefix or path segments within global W3ID URIs | [Anatomy: the shape](../anatomy/#the-shape) and [examples](../anatomy/#examples) |
| "The identifier space MAY be expanded beyond UUID if future collision rates require it." | [Anatomy: rules to remember](../anatomy/#rules-to-remember) |

## Section 3: Identifier classes

| Requirement | Where it is satisfied |
| --- | --- |
| "A global W3ID MUST be resolvable via W3DS registries." | [Lifecycle: resolving a W3ID](../lifecycle/#resolving-a-w3id) |
| "A global W3ID MUST be globally unique after registry convergence." | [Lifecycle: conflicts](../lifecycle/#conflicts) (oldest valid creation timestamp wins) |
| "A global W3ID MUST be suitable for long-term references." | [Lifetime: how a W3ID stays the same](../lifetime/) |
| "A global W3ID MUST NOT depend on DNS for its semantic identity." | [Lifecycle: resolving a W3ID](../lifecycle/#resolving-a-w3id) |
| "A local W3ID MUST be unique inside its owning eVault." | [Anatomy: the three classes](../anatomy/#the-three-classes) |
| "A local W3ID is not globally unique by itself." | [Anatomy: the three classes](../anatomy/#the-three-classes) |
| Local W3IDs become globally addressable when scoped: `@<controller-or-evault-w3id>/<local-w3id>` | [Anatomy: examples](../anatomy/#examples) |
| "A local W3ID MAY be promoted to a global W3ID through registry discovery." | [Anatomy: the three classes](../anatomy/#the-three-classes) |
| Promotion conflicts trigger renaming or resolution procedures | [Lifecycle: conflicts](../lifecycle/#conflicts) |
| "eName SHOULD be used as the public term for globally resolvable W3IDs where a name-like concept is useful." | [Overview: two words you will see a lot](../#two-words-you-will-see-a-lot) and [Anatomy: the three classes](../anatomy/#the-three-classes) |
| "eName SHOULD primarily refer to people, organizations, groups, devices, eVault controllers, or other first-class entities." | [Overview: where W3IDs show up](../#where-w3ids-show-up) |
| "eName MUST NOT imply that the identifier is derived from a cryptographic key." | [Lifetime: a W3ID is not a key](../lifetime/#a-w3id-is-not-a-key) |

## Section 4: Persistence and lifetime

| Requirement | Where it is satisfied |
| --- | --- |
| "A W3ID MUST be designed as a persistent identifier." | [Lifetime: how a W3ID stays the same](../lifetime/) |
| "A W3ID SHOULD remain stable for the lifetime of the referenced entity." | [Lifetime: how a W3ID stays the same](../lifetime/) |
| "The ecosystem SHOULD assume a target lifetime of 500+ years for long-term W3ID references." | [Lifetime: how a W3ID stays the same](../lifetime/) (opening paragraph) |
| "A W3ID MUST remain stable across key rotation." | [Lifetime: what can change without changing the W3ID](../lifetime/#what-can-change-without-changing-the-w3id) |
| "A W3ID SHOULD remain stable across eVault migration when the W3ID identifies a person, group, or controller rather than one physical eVault instance." | [Lifetime: a person and their eVault are different things](../lifetime/#a-person-and-their-evault-are-different-things) and [Lifecycle: transferring to a new eVault](../lifecycle/#transferring-to-a-new-evault) |

## Section 5: Relationship to keys and identity

| Requirement | Where it is satisfied |
| --- | --- |
| "A W3ID MUST NOT be intrinsically derived from a public/private key pair." | [Lifetime: a W3ID is not a key](../lifetime/#a-w3id-is-not-a-key) |
| "A W3ID MAY be bound to public keys through key-binding certificates, binding documents, or similar records." | [Lifetime: a W3ID is not a key](../lifetime/#a-w3id-is-not-a-key) |
| "A W3ID MAY exist before any public key is bound to it." | [Lifetime: a W3ID is not a key](../lifetime/#a-w3id-is-not-a-key) |
| "A W3ID MAY identify an entity that has no key pair." | [Lifetime: a W3ID is not a key](../lifetime/#a-w3id-is-not-a-key) |
| "User key material MUST be replaceable without changing the W3ID." | [Lifetime: what can change without changing the W3ID](../lifetime/#what-can-change-without-changing-the-w3id) |
| Person eName should provide stable anchors between identity evidence, social attestations, and public keys | [Lifetime: a W3ID is not a key](../lifetime/#a-w3id-is-not-a-key) (last paragraph) |

## Section 6: Relationship to eVaults

| Requirement | Where it is satisfied |
| --- | --- |
| "Each eVault MUST have its own globally unique W3ID." | [Lifetime: a person and their eVault are different things](../lifetime/#a-person-and-their-evault-are-different-things) |
| "An eVault W3ID MUST be distinct from a user or group eName." | [Lifetime: a person and their eVault are different things](../lifetime/#a-person-and-their-evault-are-different-things) |
| Long-term references prefer controller eName format over transient eVault W3ID | [Lifetime: a person and their eVault are different things](../lifetime/#a-person-and-their-evault-are-different-things) |
| Resolution process: identify current active eVault for controller eName, then fetch local object | [Lifecycle: resolving a W3ID](../lifecycle/#resolving-a-w3id) |

## Section 7: W3ID URI resolution

| Requirement | Where it is satisfied |
| --- | --- |
| "A W3ID URI MUST be dereferenceable through a registry when it starts with a global W3ID." | [Lifecycle: resolving a W3ID](../lifecycle/#resolving-a-w3id) |
| "The registry MUST resolve the global W3ID to the current eVault location." | [Lifecycle: resolving a W3ID](../lifecycle/#resolving-a-w3id) |
| "The eVault MUST resolve the local path component, if present." | [Lifecycle: resolving a W3ID](../lifecycle/#resolving-a-w3id) (step 3 in the sequence diagram) |
| "W3ID URIs SHOULD gradually replace DNS/URL references inside W3DS." | [Lifecycle: resolving a W3ID](../lifecycle/#resolving-a-w3id) (closing sentence) |

## Section 8: Creation

| Requirement | Where it is satisfied |
| --- | --- |
| "A person or group eName SHOULD normally be created during eVault provisioning." | [Lifecycle: creating a W3ID](../lifecycle/#creating-a-w3id) |
| "The eVault MUST make a best-effort duplicate check before accepting a new global W3ID." | [Lifecycle: creating a W3ID](../lifecycle/#creating-a-w3id) (step 6 in the sequence diagram) |
| Duplicate detection not guaranteed at creation; later conflict resolution required | [Lifecycle: conflicts](../lifecycle/#conflicts) |

## Section 9: Creation timestamp

| Requirement | Where it is satisfied |
| --- | --- |
| "A global W3ID/eName creation record MUST include a creation timestamp." | [Lifecycle: creating a W3ID](../lifecycle/#creating-a-w3id) |
| "The creation timestamp SHOULD be backed by a timestamp proof from one or more trusted timestamp witnesses." | [Lifecycle: creating a W3ID](../lifecycle/#creating-a-w3id) (steps 2 and 3) |
| "For stronger guarantees, the ecosystem SHOULD support timestamp quorum, for example `7 of 10` witnesses." | [Lifecycle: creating a W3ID](../lifecycle/#creating-a-w3id) (rules list) |
| Earlier valid creation timestamps have priority in conflicts unless superseded by valid transfer chain | [Lifecycle: conflicts](../lifecycle/#conflicts) |

## Section 10: Transfer

| Requirement | Where it is satisfied |
| --- | --- |
| "Transfer of an eName from one eVault/controller to another MUST NOT be treated as a simple registry overwrite." | [Lifecycle: transferring to a new eVault](../lifecycle/#transferring-to-a-new-evault) |
| "Transfer MUST be represented by a transfer record or transfer document." | [Lifecycle: transferring to a new eVault](../lifecycle/#transferring-to-a-new-evault) |
| Transfer record fields (eName, previous and new eVault, effective time, original creation reference, document hash, signatures, authorisation evidence) | [Lifecycle: transferring to a new eVault](../lifecycle/#transferring-to-a-new-evault) (fields list) |
| "A registry MUST accept a later target for an eName only when supported by a valid transfer chain." | [Lifecycle: transferring to a new eVault](../lifecycle/#transferring-to-a-new-evault) (last sentence) |

## Section 11: Conflict and deduplication

> Some pieces of this section are explicitly described as **prototype-level**
> by the source document (for example, post-prototype merging behaviour). The
> rows below mark those.

| Requirement | Where it is satisfied |
| --- | --- |
| "The ecosystem MUST treat duplicate global W3IDs/eNames as conflicts." | [Lifecycle: conflicts](../lifecycle/#conflicts) |
| "Duplicates SHOULD be treated as bugs or malicious events until proven otherwise." | [Lifecycle: conflicts](../lifecycle/#conflicts) |
| "Registries MUST detect conflicts where the same global W3ID/eName maps to different targets." | [Lifecycle: conflicts](../lifecycle/#conflicts) |
| "A registry MUST NOT silently overwrite a conflicting existing record." | [Lifecycle: conflicts](../lifecycle/#conflicts) (rules list) |
| Conflict resolution preference order: earlier creation timestamps, valid transfer chains, higher-reputation registry evidence, peer first-seen evidence | [Lifecycle: conflicts](../lifecycle/#conflicts) (rules list) |
| Post-prototype deduplication may merge entities and update references (prototype-level) | [Lifecycle: conflicts](../lifecycle/#conflicts) (deferred to a later iteration) |

## Section 12: External party assignment

> This section is a `MAY`, supported as a prototype-level extension.

| Requirement | Where it is satisfied |
| --- | --- |
| "The system MAY support assigning an eName to a person or organization that is not yet an active W3DS participant." | [Lifecycle: reserving a W3ID for someone who is not on W3DS yet](../lifecycle/#reserving-a-w3id-for-someone-who-is-not-on-w3ds-yet) |
| External party records support identity evidence: legal name, passport numbers, addresses, dates of birth, photographs, organisation registration numbers | [Lifecycle: reserving a W3ID for someone who is not on W3DS yet](../lifecycle/#reserving-a-w3id-for-someone-who-is-not-on-w3ds-yet) |
| "External party assignment SHOULD include deduplication checks." | [Lifecycle: reserving a W3ID for someone who is not on W3DS yet](../lifecycle/#reserving-a-w3id-for-someone-who-is-not-on-w3ds-yet) (last sentence) |

## References

- [W3ID and eName Requirements](https://github.com/w3ds-docs/w3ds-docs/wiki/W3ID-eName-Requirements)
- [Overview](../) for the orientation diagram.
- [Anatomy](../anatomy) for format and classes.
- [Lifetime](../lifetime) for persistence and key independence.
- [Lifecycle](../lifecycle) for creation, resolution, transfer, conflicts.
