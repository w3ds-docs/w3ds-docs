---
title: "Requirements coverage"
weight: 4
---

# Requirements coverage

This page maps every published requirement to the part of the design that
satisfies it. It is meant to be read side by side with the two source
documents:

- [W3ID and eName Requirements](https://github.com/w3ds-docs/w3ds-docs/wiki/W3ID-eName-Requirements)
- [W3DS Registry Requirements v2](https://github.com/w3ds-docs/w3ds-docs/wiki/W3DS-Registry-Requirements-v2)

Unless a row names a specific solution, the requirement is satisfied
identically by both Solution 1 and Solution 2, because they share the eName
record, the conflict rules, the transfer model, and the federated peer model
described in the [Overview](../).

## Table A: W3ID and eName requirements

| Section | Requirement | Where it is satisfied |
| --- | --- | --- |
| 2 | Identifier syntax: UUID per RFC 4122, dashes, case-insensitive, `@` prefix for global, local form, expandable space | The registry stores the `ename` and `class` fields verbatim as defined by [W3ID](/docs/W3DS%20Basics/W3ID). It does not redefine the format. |
| 3 | Identifier classes: global W3ID, local W3ID, eName | eName record `class` (`global` / `local`) and `controller` fields. See [Overview, the shared eName record](../#the-shared-ename-record). |
| 4 | Persistence and lifetime, stable across key rotation and eVault migration, 500+ year horizon | [Design goals](../#design-goals); the eName never changes on key rotation (control-key chain) or on transfer (transfer chain). |
| 5 | Not derived from a key, key material replaceable, may exist before a key | The eName is independent of keys. The `controlKey` is rotatable via signed updates. Key binding is out of registry scope, see [Overview note](../#background). |
| 6 | Each eVault has its own W3ID, distinct from a user or group eName | Separate `evault` and `controller` fields in the record. |
| 7 | W3ID URI resolution: dereferenceable via a registry, registry returns current eVault, no dependency on DNS for identity | The resolve function, [Solution 1 Example B](../federated-dht/#example-b-resolving-an-ename) and [Solution 2 Example B](../ledger-anchored/#example-b-resolving-with-a-proof). `uri` values use HTTPS only during the transition; identity does not depend on DNS. |
| 8 | Creation during provisioning, best-effort duplicate check | [Solution 1 Example A](../federated-dht/#example-a-registering-a-new-ename); duplicate check at registration (`FR11`, `FR12`). |
| 9 | Creation timestamp, timestamp proof, witness quorum such as 7 of 10, earlier timestamps win | `creationRecord.timestampProof`. See [Overview, creation timestamps and witnesses](../#creation-timestamps-and-witnesses). |
| 10 | Transfer is not an overwrite, transfer record with evidence, valid transfer chain required | [Overview, transfer records](../#transfer-records) and the transfer examples on both solution pages. |
| 11 | Duplicates are conflicts, detect them, no silent overwrite, resolution priority order | [Overview, conflict detection and resolution](../#conflict-detection-and-resolution). |
| 12 | External party assignment to non-participants (a `MAY`) | Supported as an optional creation record carrying external-party identity evidence, with deduplication checks. Treated as a future extension, not part of the core flow. |

## Table B: Registry v2 functional requirements

| ID | Requirement | Where it is satisfied |
| --- | --- | --- |
| FR1 | Store global W3ID/eName resolution records | eName record storage in every registry. |
| FR2 | Resolve eNames to eVault locations | `GET /resolve`, Example B on both solution pages. |
| FR3 | Synchronise records with peer registries | [Overview, peer sync](../#audit-log-peer-sync-and-reputation); Solution 1 gossip, Solution 2 log sync. |
| FR4 | Detect conflicts | [Overview, conflict detection](../#conflict-detection-and-resolution). |
| FR5 | Auditable history for creation, updates, transfers, conflicts | Per-registry audit log; provable in Solution 2. |
| FR6 | Expose a resolution endpoint | `GET /resolve`. |
| FR7 | Resolution returns the current accepted target | Example B responses. |
| FR8 | Resolution returns conflict metadata when relevant | Example B and Example E responses. |
| FR9 | Never silently return a conflicting record as normal | Conflict rules; [Solution 1 Example E](../federated-dht/#example-e-a-conflict-and-how-it-is-resolved). |
| FR10 | Cached data may be returned but conflict status preserved | [Solution 1 Example B](../federated-dht/#example-b-resolving-an-ename) note. |
| FR11 | Best-effort duplicate check before accepting a record | Solution 1 and Solution 2 Example A. |
| FR12 | Duplicate checks include local and peer records | Example A, against local store and reachable peers. |
| FR13 | May accept a record before full convergence | Eventual-consistency model; registries accept then converge. |
| FR14 | Store creation evidence for later audit | `creationRecord` is preserved permanently. |
| FR15 | Auditable log entry for creation acceptance | Audit log entry on every registration. |
| FR16 | Creation record includes a creation timestamp | `creationRecord.creationTimestamp`. |
| FR17 | Creation timestamp backed by timestamp proof | `creationRecord.timestampProof`. |
| FR18 | Timestamp proof from multiple independent witnesses | `timestampProof.witnesses`. |
| FR19 | Support quorum timestamping such as 7 of 10 | `timestampProof.policy`. |
| FR20 | A conflict is the same eName mapping to different targets with no valid transfer chain | Conflict definition in the [Overview](../#conflict-detection-and-resolution). |
| FR21 | Detect and record conflicts | Conflict rules; Example E. |
| FR22 | Notify or expose conflict state to peers and clients | `conflictStatus` field and conflict metadata in responses. |
| FR23 | Do not resolve a conflict unless policy selects a clear winner | Conflict rules; otherwise `conflict_pending_review`. |
| FR24 | Default winner is the oldest valid creation timestamp | Conflict resolution rule. |
| FR25 | Mark `conflict_pending_review` when evidence is insufficient | `conflictStatus` value. |
| FR26 | Support eName transfer records | [Overview, transfer records](../#transfer-records). |
| FR27 | A transfer points from previous to new target | `previousTarget` and `newTarget` fields. |
| FR28 | A transfer includes authorisation evidence | `authorizationEvidence` field. |
| FR29 | A transfer references the previous valid state | `creationReference` field. |
| FR30 | Preserve the historical chain, do not overwrite the genesis record | `creationRecord` is never overwritten; `transferChain` is append-only. |
| FR31 | After a valid transfer, resolve to the new eVault | Transfer examples on both solution pages. |
| FR32 | Exchange records, conflicts, and audit metadata between peers | Peer sync in both solutions. |
| FR33 | Maintain source reputation for peers and providers | [Overview, reputation](../#audit-log-peer-sync-and-reputation). |
| FR34 | Mark low-reputation or unknown sources as low trust | Reputation handling. |
| FR35 | Revise status when peer sync reveals an older valid record | Conflict resolution on sync; Solution 2 log comparison. |
| FR36 | Keep enough history to explain a record's state changes | Audit log. |
| FR37 | Append-only or tamper-evident audit history | Solution 1: per-registry append-only log. Solution 2: a Merkle log with inclusion and consistency proofs. |
| FR38 | Audit entries include type, timestamp, old and new value, source, evidence hashes, registry signature | Audit log entry schema in the [Overview](../#audit-log-peer-sync-and-reputation). |

## Table C: Registry v2 non-functional requirements

| ID | Requirement | Where it is satisfied |
| --- | --- | --- |
| NFR1 | The registry is a federated resolver mapping an identifier to the current eVault | [Overview, background](../#background). |
| NFR2 | Conceptually like DNS, but W3DS must not depend on DNS as the identity layer | W3IDs are resolved without DNS; `uri` HTTPS values are transitional only. |
| NFR3 | The registry is not the source of truth for user keys or identity | Key binding is out of scope, see [Overview note](../#background). |
| NFR4 | Registries assume eventual consistency | [Design goals](../#design-goals); both solutions. |
| NFR5 | Tolerate temporarily incomplete peer knowledge | Eventual-consistency model; `FR13`. |
| NFR6 | Support later conflict discovery | Conflict detection on sync; Solution 2 log comparison at any time. |
| NFR7 | Deterministic timestamp policy, such as median of valid attestations | Conflict resolution rule. |
| NFR8 | A record without timestamp proof has lower trust | [Overview, creation timestamps](../#creation-timestamps-and-witnesses). |
| NFR9 | Conflict resolution considers timestamps, first-seen, reputation, transfer documents, audit logs | Conflict resolution inputs in the [Overview](../#conflict-detection-and-resolution). |
| NFR10 | Registries operate as peers, no mandatory central root | Federated topology; both solutions. |
| NFR11 | A fake or new registry cannot override older, higher-reputation records | Source reputation; conflict resolution by timestamp. |
| NFR12 | A registry publishing false mappings loses reputation | Reputation handling. |
| NFR13 | The eVault remains the source of truth for its own objects and certificates | Key binding and certificates left with the eVault. |
| NFR14 | The registry may store public evidence for resolution integrity | `timestampProof` and transfer document hashes are stored. |
| NFR15 | Security rests on timestamped creation, peer visibility, reputation, conflict detection, transfer evidence | The combined conflict, audit, and reputation model. |
| NFR16 | A malicious duplicate is detectable because peers hold the older record | [Solution 1 security model](../federated-dht/#security-model-and-failure-modes), forged creation race. |
| NFR17 | A fake registry has insufficient reputation to override established records | Source reputation. |

## References

- [W3ID and eName Requirements](https://github.com/w3ds-docs/w3ds-docs/wiki/W3ID-eName-Requirements)
- [W3DS Registry Requirements v2](https://github.com/w3ds-docs/w3ds-docs/wiki/W3DS-Registry-Requirements-v2)
- [Overview](../) for the shared record and rules.
- [Solution 1: Federated DHT](../federated-dht) and
  [Solution 2: Ledger-anchored](../ledger-anchored).
