# TSS Main Process: Timestamp Request

Date: 2026-06-04
Status: Draft requirements
Source: W3DS decentralised registry discussion

## 1. Purpose

Request a signed proof from an independent TSS that a specific cryptographic hash existed no later than a given point in time.

The result of the process is a **signed timestamp attestation**, which the client can:
- present as evidence of genesis timing for a W3ID/eName;
- attach to an audit log record or hash-chain head;
- combine with attestations from other TSS into a quorum proof.

The process does not interpret the hash.

## 2. Participants and Roles

| Participant               | Role in the Process                                                                                                                                                                                                     |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Client**                | Initiator: wallet, eVault, registry, platform, log operator. Forms the request, writes it as a metaenvelope into the TSS's eVault, receives the attestation from its own eVault and verifies it, assembles a quorum if needed. |
| **Timestamp Server (TSS)**| Processor: reads the request from a metaenvelope in its own eVault, validates, measures time, writes the signed attestation as a metaenvelope into the client's eVault, maintains audit log, publishes verification metadata. |
| **TSS Set**               | A group of independent TSS (e.g., 10 servers). Used for quorum timestamping — the client collects `M of N` attestations for the same hash.                                                                               |

## 3. Interaction Flow

The process consists of sequential steps. Steps 1-8 form the mandatory base scenario. Step 9 is optional quorum aggregation.

### 3.1. Request Formation (Client)

The client assembles the request structure, mandatorily specifying the hash to be timestamped.

Minimum request structure:

```json
{
  "requestId": "2f17f83b-0d96-45fb-a625-3f3c5872e2b1",
  "hashAlgorithm": "sha-256",
  "subjectHash": "base64url-or-hex-digest",
  "context": "w3ds.registry.creation",
  "clientNonce": "base64url-random-nonce",
  "requestedAt": "2026-05-23T10:15:30Z",
  "client": "@50e8400-e29b-41d4-a716-446655440000"
}
```

The client **MUST** include:
- `requestId` — globally unique request identifier
- `hashAlgorithm` — digest algorithm (e.g. `sha-256`)
- `subjectHash` — digest for which the timestamp is requested
- `clientNonce` — fresh random string for replay protection
- `requestedAt` — request time per the client's clock
- `client` — client's eName where the TSS will write the attestation

The client **SHOULD** include (recommended fields):
- `context` — context label string (`w3ds.registry.creation`, `w3ds.registry.hashchain-head`, `w3ds.transfer-document`)
- `proof` — optional client signature over the request
- `metadataHash` — optional hash of private metadata that the client wants to bind to the timestamp without disclosing it

### 3.2. Sending the Request (Client → TSS)

The client packages the formed request into a **metaenvelope** and writes it into the target TSS's eVault. The metaenvelope metadata indicates that the request is addressed to this TSS. The request remains in the TSS's eVault until processed.

The client repeats this step for each TSS in the pool if a quorum is required. Each TSS processes the request independently and asynchronously.

### 3.3. Reception and Validation (TSS)

The TSS detects a new metaenvelope in its eVault according to the W3DS protocol rules, reads the request contents, and records the reception time (`receivedAt`). It then performs validation:

- checks that the hash algorithm is approved and the encoding is correct;
- checks the digest length;
- checks syntactic correctness of all fields;
- checks that the `clientNonce` is fresh (not previously used);
- if `proof` is present — verifies the client's signature;
- checks that the request schema version is supported;
- checks that `client` is a valid eName and writable.

If the request is invalid, the TSS rejects it and writes a metaenvelope with an error into the client's eVault at the `client` address.

The TSS **MUST** accept any syntactically correct hash that uses an approved algorithm and encoding, regardless of understanding of the source application.

### 3.4. Time Binding (TSS)

The TSS measures the current time from its clock and sets `attestedAt`.

Before binding, the TSS checks its clock health:
- if accuracy is outside acceptable bounds — the request is rejected or flagged;
- the clock is synchronized with an independent source (NTP, GPS, etc.).

### 3.5. Attestation Formation and Signing (TSS)

The TSS forms the attestation, including all request fields, the measured time, TSS information, and signature.

Minimum attestation structure:

```json
{
  "tssId": "@a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "sequence": 17841291,
  "tssPublicKey": "zTSSPublicKey...",
  "hashAlgorithm": "sha-256",
  "subjectHash": "base64url-or-hex-digest",
  "context": "w3ds.registry.creation",
  "clientNonce": "base64url-random-nonce",
  "receivedAt": "2026-05-23T10:15:31.245Z",
  "attestedAt": "2026-05-23T10:15:31.400Z",
  "clockAccuracyMs": 250,
  "policy": "timestamp-tss-v1",
  "signature": "zSignedByTSS..."
}
```

The attestation **MUST** include:
1. `tssId` — TSS eName (format `@<UUID>`)
2. `sequence` — monotonically increasing local TSS sequence number
3. TSS public key or a reference to a verifiable key
4. `hashAlgorithm` from the request
5. `subjectHash` from the request
6. `context`, if provided in the request
7. `clientNonce` from the request
8. `receivedAt` — reception time measured by the TSS
9. `attestedAt` — attestation time measured by the TSS
10. Declared clock accuracy/tolerance (`clockAccuracyMs`)
11. `policy` — identifier of the protocol version and rules under which the attestation was issued (schema version, signature algorithms, validation rules)
12. TSS digital signature

The TSS signature **MUST** cover all fields of the attestation except those added by the transport or storage layer.

### 3.6. Attestation Delivery (TSS → Client)

The TSS packages the signed attestation into a **metaenvelope** and writes it into the client's eVault at the `client` address obtained from the request.

The client detects a new metaenvelope in its eVault according to W3DS protocol rules and verifies:
- correctness of the TSS signature;
- that `subjectHash` and `hashAlgorithm` match the request;
- that `attestedAt` ≥ `receivedAt` (accounting for clock accuracy);
- that the TSS public key is current.

### 3.7. Audit Log (TSS)

The TSS stores a record of the issued attestation in an append-only or tamper-evident audit log. The log is kept in the TSS's eVault. The record includes:
- `tssId` and `sequence`
- request hash (hash of the entire request)
- `subjectHash` and `hashAlgorithm`
- `receivedAt` and `attestedAt`
- TSS signing key
- result status (success/error)
- error reason (if applicable)

### 3.8. Metadata Publication (TSS)

The TSS publishes (or updates) verification metadata in a public metaenvelope at address `verification_metadata@TSS_eName`. Clients request this metaenvelope through the TSS's eVault to verify attestations.

Metadata includes:

- current TSS public key;
- historical public keys and their validity periods;
- supported attestation schema versions;
- current clock health;
- service identification and operator metadata;
- revocation or key compromise notifications.

### 3.9. Quorum Aggregation (Optional, Client)

If the protocol requires a quorum proof (e.g., `7 out of 10` independent TSS), the client repeats steps 3.1-3.6 with different TSS from the TSS pool. Requests are written asynchronously to each TSS's eVault; attestations arrive asynchronously in the client's eVault. The client collects M attestations and aggregates them.

A quorum proof **MUST** contain:
1. Quorum policy identifier
2. Set of TSS attestations
3. Definition of the TSS pool at the time of attestation
4. A deterministic aggregation rule (e.g., median of `attestedAt`)

If the quorum is not reached, the proof is considered weak and marked as incomplete.

### 3.10. Final Process Result

The result of the process is a **timestamp proof**:

- base: a single TSS attestation;
- strengthened: a quorum set of attestations.

The client decides independently how to store and use the timestamp proof.

## 4. Interaction Requirements

### 4.1. Request

- The request **MUST** precisely identify the hash (fields `hashAlgorithm` + `subjectHash`)
- `requestId` **MUST** be globally unique
- `client` **MUST** contain a valid eName of the client's eVault for response delivery
- `clientNonce` **MUST** be fresh to prevent replay
- The TSS **MUST NOT** require knowledge of the hash preimage
- The TSS **MUST** accept only the hash, not the full content (current implementation limitation)

### 4.2. Validation

- The TSS **MUST** verify the hash algorithm and encoding
- The TSS **MUST** verify the digest length
- The TSS **MUST** verify the client's signature (if `proof` is provided)
- The TSS **MUST** reject unsupported schema versions
- The TSS **MUST** accept any syntactically correct hash with an approved algorithm, regardless of source application understanding

### 4.3. Time

- All timestamps **MUST** be in UTC
- Precision must be at least milliseconds
- The TSS **MUST** maintain a documented clock synchronization mechanism
- The TSS **MUST** specify clock accuracy/tolerance in every attestation
- The TSS **MUST** reject or flag requests if clock health is outside acceptable bounds
- The TSS **MUST** maintain monotonically increasing attestation sequence numbers
- The TSS **SHOULD** use multiple independent time sources
- The TSS **SHOULD** continuously monitor clock drift
- The TSS **SHOULD** publish clock health status via an endpoint

### 4.4. Attestation

- The attestation **MUST** be digitally signed by the TSS
- The signature **MUST** cover all fields except transport/storage layer fields
- The attestation **MUST** contain tssId (TSS eName) and sequence at the beginning of the field list
- The attestation **MUST** contain the TSS public key
- The attestation **MUST** include all significant request fields
- The attestation **MUST** contain `receivedAt` and `attestedAt`
- The attestation **MUST** contain the declared clock accuracy
- The attestation **MUST** contain `policy` (protocol version and rules identifier)

### 4.5. Quorum

- The ecosystem **SHOULD** support quorum timestamping (`M of N`)
- A quorum proof **MUST** contain a policy, set of attestations, TSS pool definition, and aggregation rule
- Competing proofs **SHOULD** be compared by the median of `attestedAt` after discarding invalid, outlier, and low-reputation TSS
- Without a quorum, a proof **MAY** be accepted but **MUST** be considered weaker

### 4.6. TSS Independence

- TSS **MUST** be operated with sufficient independence such that compromising one does not determine the timestamp priority
- The TSS set **SHOULD** avoid concentration by operator, hosting provider, jurisdiction, network, time source, or software implementation
- The ecosystem **SHOULD** maintain TSS reputation and discount attestations from TSS with poor operational/security history

### 4.7. Public Verifiability

- The TSS **MUST** publish verification metadata in a public metaenvelope `verification_metadata@<TSS_eName>`: current keys, historical keys, supported schema versions, clock health, operator metadata, compromise/revocation notifications

### 4.8. TSS Responsibility Boundaries

The TSS **MUST NOT**:
- interpret the legal, identification, or registry meaning of the hash
- act as a central root authority for a TSS federation
