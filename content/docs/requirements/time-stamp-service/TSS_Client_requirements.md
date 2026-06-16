# TSS Client Implementation Requirements

Date: 2026-06-07
Status: Draft requirements
Source: W3DS decentralised registry discussion

## 1. Purpose

This document defines the implementation requirements for the client side of the timestamp request process. The TSS client is the request initiator: a wallet, eVault, platform, log operator, or other application that requires a signed time proof from an independent TSS.

## 2. Core Requirement: Client-Side Implementation of TSS_Main_Process

The TSS client MUST implement the client-side part of the timestamp request process described in [TSS_Main_Process.md](TSS_Main_Process.md) (steps 3.1–3.2, 3.6, 3.9–3.10, section 4).

## 3. Request Formation

The client MUST form a request according to the specification in step 3.1 of [TSS_Main_Process.md](TSS_Main_Process.md).

## 4. Sending the Request

The client MUST send the request according to the specification in step 3.2 of [TSS_Main_Process.md](TSS_Main_Process.md).

## 5. Receiving and Verifying the Attestation

The client MUST receive and verify the attestation according to the specification in step 3.6 of [TSS_Main_Process.md](TSS_Main_Process.md).

## 6. Quorum Aggregation

The client MUST perform quorum aggregation according to the specification in step 3.9 of [TSS_Main_Process.md](TSS_Main_Process.md).

## 7. Storing and Using the Timestamp Proof

The client decides independently how to store and use the timestamp proof (step 3.10 of [TSS_Main_Process.md](TSS_Main_Process.md)).

The client SHOULD store timestamp proofs or their hashes for:

- creation records;
- transfer records;
- audit log entries;
- hash-chain heads;
- Merkle roots.

The client's eVault SHOULD keep its own copies of timestamp proofs relevant to its records.

## 8. Failure Handling

If a TSS is unavailable, the client SHOULD request attestations from other TSS in the pool.

If a TSS key is compromised, the client MUST be able to:

- identify affected attestations by key validity period;
- re-score or reject affected attestations;
- re-evaluate conflicts that depended on affected attestations;
- preserve the audit trail explaining the decision.

## 9. TSS Selection and Reputation

The client SHOULD consider TSS reputation:

- avoid concentrating requests with a single operator, hosting provider, or jurisdiction;
- discount attestations from TSS with poor operational or security history;
- request current verification metadata before sending a request.

The client MUST obtain verification metadata according to the specification in step 3.8 of [TSS_Main_Process.md](TSS_Main_Process.md).

## 10. Hash-Chain and Log Use

The client may use TSS for timestamping hash-chain heads.

A hash-chain log entry MUST define its head as a canonical digest over:

1. The previous head hash.
2. The new event hash.
3. The sequence number.
4. The log identifier.
5. The event type or context.

The client MAY timestamp every log entry, but SHOULD normally timestamp periodic checkpoints or important heads to reduce TSS load.

A timestamped hash-chain head proves that the head existed no later than the attested time. To verify all entries, the hash chain must be replayed from the genesis entry to the timestamped head.

For client audit logs, timestamped heads MUST be used to detect later rewrites, deletions, reordering, or backdating.

Recommended checkpoint frequency for client hash-chain logs: once per hour.

## 11. Privacy

The client may disclose operation context through the `context` field. The client SHOULD use generic context labels to avoid revealing sensitive information.

For enhanced privacy, the client may use `metadataHash` to bind private data to the timestamp without disclosing it.
