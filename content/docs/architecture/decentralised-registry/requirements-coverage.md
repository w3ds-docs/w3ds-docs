---
title: "Requirements coverage"
weight: 3
---

# Requirements coverage

This page maps every published requirement to the part of the design that
satisfies it. It is meant to be read side by side with the two source
documents:

- [W3ID and eName Requirements](https://github.com/w3ds-docs/w3ds-docs/wiki/W3ID-eName-Requirements)
- [W3DS Registry Requirements v2](https://github.com/w3ds-docs/w3ds-docs/wiki/W3DS-Registry-Requirements-v2)

The headline pattern: a registry is an [eVault](/docs/Infrastructure/eVault),
eName records are MetaEnvelopes under an `ENameRecord`
[ontology](/docs/Infrastructure/Ontology), audit history is the eVault's
`/logs` endpoint, and registries gossip via the
[Awareness Protocol](/docs/W3DS%20Protocol/Awareness-Protocol). Most rows below
point at one of these primitives or at the [Architecture](../architecture) or
[Gossip protocol](../gossip-protocol) pages.

## Table A: W3ID and eName requirements

| Section | Requirement | Where it is satisfied |
| --- | --- | --- |
| 2 | Identifier syntax: UUID per RFC 4122, dashes, case-insensitive, `@` prefix for global, local form, expandable space | The registry stores the `ename` and `class` fields verbatim as defined by [W3ID](/docs/W3DS%20Basics/W3ID). It does not redefine the format. |
| 3 | Identifier classes: global W3ID, local W3ID, eName | eName record `class` (`global` / `local`) and `controller` fields. See [Architecture, eName records as MetaEnvelopes](../architecture/#ename-records-as-metaenvelopes). |
| 4 | Persistence and lifetime, stable across key rotation and eVault migration, 500+ year horizon | The eName never changes on key rotation (control-key chain) or on transfer (`transferChain`). |
| 5 | Not derived from a key, key material replaceable, may exist before a key | The eName is independent of keys. The `controlKey` is rotatable via signed updates. Key binding is out of registry scope, see [Overview, note on entropy and key binding](../#background). |
| 6 | Each eVault has its own W3ID, distinct from a user or group eName | Separate `evault` and `controller` fields in the record. The registry's own eVault also has an eVault W3ID, but is deliberately not registered as an eName, see [Architecture, a registry is a special eVault](../architecture/#a-registry-is-a-special-evault). |
| 7 | W3ID URI resolution: dereferenceable via a registry, registry returns current eVault, no dependency on DNS for identity | The resolve function, [Architecture Example B](../architecture/#example-b-resolving-an-ename). `uri` values use HTTPS only during the transition; identity does not depend on DNS. |
| 8 | Creation during provisioning, best-effort duplicate check | [Architecture Example A](../architecture/#example-a-registering-an-ename); duplicate check at registration (`FR11`, `FR12`). |
| 9 | Creation timestamp, timestamp proof, witness quorum such as 7 of 10, earlier timestamps win | `creationRecord.timestampProof` in the [`ENameRecord` schema](../architecture/#ename-records-as-metaenvelopes). |
| 10 | Transfer is not an overwrite, transfer record with evidence, valid transfer chain required | `transferChain` in the record, propagated by [Gossip protocol Example C](../gossip-protocol/#example-c-transfer). |
| 11 | Duplicates are conflicts, detect them, no silent overwrite, resolution priority order | [Gossip protocol Example D](../gossip-protocol/#example-d-a-conflict-and-how-it-is-resolved). |
| 12 | External party assignment to non-participants (a `MAY`) | Supported as an optional creation record carrying external-party identity evidence, with deduplication checks. Future extension, not part of the core flow. |

## Table B: Registry v2 functional requirements

| ID | Requirement | Where it is satisfied |
| --- | --- | --- |
| FR1 | Store global W3ID/eName resolution records | `ENameRecord` MetaEnvelopes in each registry's eVault. |
| FR2 | Resolve eNames to eVault locations | Lookup against the registry's `ENameRecord` store, or a `GET /resolve` shim, in [Architecture Example B](../architecture/#example-b-resolving-an-ename). |
| FR3 | Synchronise records with peer registries | The [Awareness Protocol](/docs/W3DS%20Protocol/Awareness-Protocol) delivers every record change to peer registries. See [Gossip protocol](../gossip-protocol). |
| FR4 | Detect conflicts | [Gossip protocol Example D](../gossip-protocol/#example-d-a-conflict-and-how-it-is-resolved). |
| FR5 | Auditable history for creation, updates, transfers, conflicts | eVault [`/logs` endpoint](/docs/Infrastructure/eVault#logs) records every MetaEnvelope operation. |
| FR6 | Expose a resolution endpoint | Lookup against the registry's `ENameRecord` store, or a `GET /resolve` shim. |
| FR7 | Resolution returns the current accepted target | Example B response. |
| FR8 | Resolution returns conflict metadata when relevant | `conflictStatus` field on the record returned by Example B. |
| FR9 | Never silently return a conflicting record as normal | `conflictStatus` is always part of the response; see [Gossip protocol Example D](../gossip-protocol/#example-d-a-conflict-and-how-it-is-resolved). |
| FR10 | Cached data may be returned but conflict status preserved | The status field travels with the record at every layer. |
| FR11 | Best-effort duplicate check before accepting a record | Registry checks local eVault before accepting; gossip arrival also revalidates. |
| FR12 | Duplicate checks include local and peer records | Local check plus the gossip arrival check across peers. |
| FR13 | May accept a record before full convergence | Eventual-consistency model from the Awareness Protocol. |
| FR14 | Store creation evidence for later audit | `creationRecord` is preserved permanently in the MetaEnvelope. |
| FR15 | Auditable log entry for creation acceptance | eVault `/logs` entry on the MetaEnvelope create. |
| FR16 | Creation record includes a creation timestamp | `creationRecord.creationTimestamp`. |
| FR17 | Creation timestamp backed by timestamp proof | `creationRecord.timestampProof`. |
| FR18 | Timestamp proof from multiple independent witnesses | `timestampProof.witnesses`. |
| FR19 | Support quorum timestamping such as 7 of 10 | `timestampProof.policy`. |
| FR20 | A conflict is the same eName mapping to different targets with no valid transfer chain | Conflict definition used by `GossipConflictReport`. |
| FR21 | Detect and record conflicts | Receiving registry writes a `GossipConflictReport` MetaEnvelope, which is itself logged. |
| FR22 | Notify or expose conflict state to peers and clients | `GossipConflictReport` packet, `conflictStatus` field. |
| FR23 | Do not resolve a conflict unless policy selects a clear winner | Conflict rules; otherwise `conflict_pending_review`. |
| FR24 | Default winner is the oldest valid creation timestamp | Conflict resolution rule. |
| FR25 | Mark `conflict_pending_review` when evidence is insufficient | `conflictStatus` value. |
| FR26 | Support eName transfer records | `transferChain` in the record. |
| FR27 | A transfer points from previous to new target | `previousTarget` and `newTarget` fields in the transfer entry. |
| FR28 | A transfer includes authorisation evidence | `authorizationEvidence` field. |
| FR29 | A transfer references the previous valid state | `creationReference` field. |
| FR30 | Preserve the historical chain, do not overwrite the genesis record | `creationRecord` is never overwritten; `transferChain` is append-only. |
| FR31 | After a valid transfer, resolve to the new eVault | Updated record returned by Example B. |
| FR32 | Exchange records, conflicts, and audit metadata between peers | Awareness Protocol gossip carrying the `Gossip*` ontology messages. |
| FR33 | Maintain source reputation for peers and providers | `PeerRegistry` MetaEnvelopes and `GossipPeerReputation` packets. |
| FR34 | Mark low-reputation or unknown sources as low trust | Reputation score on each `PeerRegistry` entry; records sourced from low-rep peers are downgraded. |
| FR35 | Revise status when peer sync reveals an older valid record | The arrival of a `GossipENameRecord` with an older valid creation timestamp triggers a `conflict_pending_review` reassessment. |
| FR36 | Keep enough history to explain a record's state changes | eVault `/logs`. |
| FR37 | Append-only or tamper-evident audit history | eVault [`/logs`](/docs/Infrastructure/eVault#logs) is append-only by construction. |
| FR38 | Audit entries include type, timestamp, old and new value, source, evidence hashes, registry signature | eVault `/logs` entry shape, see [eVault `/logs`](/docs/Infrastructure/eVault#logs). |

## Table C: Registry v2 non-functional requirements

| ID | Requirement | Where it is satisfied |
| --- | --- | --- |
| NFR1 | The registry is a federated resolver mapping an identifier to the current eVault | [Overview, background](../#background). |
| NFR2 | Conceptually like DNS, but W3DS must not depend on DNS as the identity layer | W3IDs are resolved without DNS; `uri` HTTPS values are transitional only. |
| NFR3 | The registry is not the source of truth for user keys or identity | The registry eVault stores only `ENameRecord` and `PeerRegistry` MetaEnvelopes; key binding is out of scope, see [Overview, note on entropy and key binding](../#background). |
| NFR4 | Registries assume eventual consistency | Inherited directly from the Awareness Protocol delivery model. |
| NFR5 | Tolerate temporarily incomplete peer knowledge | Anti-entropy catch-up across peers reconciles missed gossip. |
| NFR6 | Support later conflict discovery | A `GossipConflictReport` can be raised at any time, including after the fact, when peer gossip reveals a clash. |
| NFR7 | Deterministic timestamp policy, such as median of valid attestations | Conflict resolution rule on `creationTimestamp` + `timestampProof`. |
| NFR8 | A record without timestamp proof has lower trust | Conflict resolution treats records without a valid `timestampProof` as lower priority. |
| NFR9 | Conflict resolution considers timestamps, first-seen, reputation, transfer documents, audit logs | All five inputs are available: timestamps and transfer chain from the record, first-seen from peer logs, reputation from `PeerRegistry`, audit history from `/logs`. |
| NFR10 | Registries operate as peers, no mandatory central root | Federated topology; each registry is just an eVault, no hierarchy. |
| NFR11 | A fake or new registry cannot override older, higher-reputation records | Reputation-weighted gossip + oldest valid creation timestamp wins. |
| NFR12 | A registry publishing false mappings loses reputation | `GossipPeerReputation` downgrades it. |
| NFR13 | The eVault remains the source of truth for its own objects and certificates | Key binding and certificates stay with the user eVault; the registry eVault only stores resolution records. |
| NFR14 | The registry may store public evidence for resolution integrity | `timestampProof` and transfer document hashes are stored in the `ENameRecord`. |
| NFR15 | Security rests on timestamped creation, peer visibility, reputation, conflict detection, transfer evidence | The combination of `creationRecord`, gossip, `PeerRegistry` reputation, `GossipConflictReport`, and `transferChain`. |
| NFR16 | A malicious duplicate is detectable because peers hold the older record | On gossip arrival, every peer compares against its own record set and triggers a `GossipConflictReport` if a clash is found. |
| NFR17 | A fake registry has insufficient reputation to override established records | Same reputation mechanism as `NFR11`. |

## References

- [W3ID and eName Requirements](https://github.com/w3ds-docs/w3ds-docs/wiki/W3ID-eName-Requirements)
- [W3DS Registry Requirements v2](https://github.com/w3ds-docs/w3ds-docs/wiki/W3DS-Registry-Requirements-v2)
- [Overview](../) for the problem statement.
- [Architecture](../architecture) and [Gossip protocol](../gossip-protocol).
- [eVault](/docs/Infrastructure/eVault),
  [Awareness Protocol](/docs/W3DS%20Protocol/Awareness-Protocol), and
  [Ontology](/docs/Infrastructure/Ontology).
