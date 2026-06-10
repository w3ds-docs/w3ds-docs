# TimeStamp Server (TSS) Implementation Requirements

Date: 2026-06-07
Status: Draft requirements
Source: W3DS decentralised registry discussion

## 1. Purpose

This document defines the implementation requirements for the **TimeStamp Server (TSS)** — an independent witness that certifies that an arbitrary cryptographic hash existed no later than a given point in time.

TSS provides reusable time evidence for immutable data structures: W3ID/eName, client audit records, hash chains, Merkle roots, transfer documents, and other content-addressed records.

TSS does not interpret the hash and does not make decisions about the meaning of the hashed content.

## 2. Core Requirement: TSS_Main_Process Implementation

TSS MUST implement the server-side part of the timestamp request process described in [TSS_Main_Process.md](TSS_Main_Process.md) (steps 3.3–3.8, section 4).

## 3. Time Requirements

- All timestamps MUST be in UTC.
- Precision MUST be at least milliseconds.
- TSS MUST maintain a documented clock synchronization mechanism.
- TSS MUST declare clock accuracy/tolerance in every attestation.
- TSS MUST reject or flag requests when clock health is outside accepted bounds.
- TSS MUST maintain monotonically increasing attestation sequence numbers.
- TSS SHOULD use multiple independent time sources.
- TSS SHOULD continuously monitor clock drift.
- TSS SHOULD expose current clock health via an endpoint.

## 4. TSS Responsibility Boundaries

TSS responsibility boundaries are defined by the TSS main process boundaries.

## 5. Auditability

TSS SHOULD publish a signed log head or equivalent transparency checkpoint so that participants can detect deleted, reordered, or rewritten attestations.

## 6. Replay and Abuse Resistance

TSS MUST:

1. Require a fresh `clientNonce`.
2. Bind the nonce into the signed attestation.
3. Make repeated identical requests idempotent when possible.
4. Rate limit clients.
5. Validate the hash algorithm and digest length.
6. Validate canonical digest encoding.
7. Reject malformed signatures and unsupported schema versions.

TSS SHOULD:

1. Detect high-volume attempts to timestamp suspicious or conflicting hashes under the same context.
2. Publish abuse signals or reputation events to relying parties when appropriate.
3. Avoid leaking unnecessary client network metadata in public logs.

## 7. Privacy

TSS MUST minimize retained personal data and MUST NOT require identity documents, user profiles, or private eVault contents to issue a timestamp attestation.

TSS SHOULD support timestamping a bare hash without publishing the preimage or revealing application-specific metadata.

## 8. Failure Handling

- If a TSS key is compromised, TSS clients MUST be able to:
  - identify affected attestations by key validity period;
  - re-score or reject affected attestations;
  - re-evaluate conflicts that depended on affected attestations;
  - preserve the audit trail explaining the decision.

## 9. Independence and Trust

TSS MUST be independent: compromise, misconfiguration, legal pressure, or downtime of one operator MUST NOT determine timestamp priority.

The TSS set SHOULD avoid concentration by:

- operator;
- hosting provider;
- jurisdiction;
- network dependency;
- time source dependency;
- software implementation.

The ecosystem SHOULD maintain TSS reputation and discount attestations from TSS with poor operational or security history.

## 10. Hash-Chain and Log Use

TSS MUST maintain an audit log represented as a hash chain or Merkle log.

A hash-chain log entry MUST define its head as a canonical digest over:

1. The previous head hash.
2. The new event hash.
3. The sequence number.
4. The log identifier.
5. The event type or context.

A timestamped hash-chain head proves that the head existed no later than the attested time. It does not by itself prove that all earlier entries are valid; that proof comes from replaying the hash chain from the genesis entry to the timestamped head.

For audit logs, timestamped heads MUST be used to detect later rewrites, deletions, reordering, or backdating of client events.

Recommended checkpoint frequency for client hash-chain logs: once per hour.
