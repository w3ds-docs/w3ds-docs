---
title: Creation and Initialization
weight: 4
---

# eVault Component: Creation and Initialization

## Purpose

This document describes the eVault component as it participates in the initial user creation flow. It focuses on the logical eVault structure, stored initialization data, lifecycle state, and interfaces relevant to first-time activation.

## Responsibilities

- Represent the user's hosted personal data space
- Hold the initial certificate that binds the first public key to the eName
- Maintain metadata required to determine whether the eVault is usable
- Expose or support interfaces needed for later read, write, and verification operations
- Preserve initialization state independently from recovery configuration state

## Logical Structure

At initialization time, the eVault should be modeled as two coupled layers:

| Layer | Purpose |
|---|---|
| Logical identity layer | Associates the eVault with eName, lifecycle state, and hosting metadata |
| Storage layer | Stores certificate, metadata records, and future user-controlled content |

## Internal Structure

| Internal Part | Purpose |
|---|---|
| Identity Metadata Record | Stores eName and lifecycle state |
| Certificate Store | Stores the initial self-signed certificate and later certificate history if supported |
| Hosting Metadata Record | Records provider hosting information |
| State Manager | Controls transitions between `created`, `pending_key_binding`, and `active` |
| Recovery Metadata Record | Indicates whether recovery is configured |

## Stored Data

### Mandatory Data

| Data | Description |
|---|---|
| `eName` | Persistent identifier bound to this eVault |
| `evaultId` | Provider-local or globally unique eVault reference |
| `state` | Current lifecycle state |
| `initialCertificate` | Self-signed certificate containing eName and first public key |
| `hostingMetadata` | Information identifying the provider that hosts the eVault |
| `createdAt` | Timestamp of initial allocation |
| `initializedAt` | Timestamp of successful key binding |
| `recoveryState` | Indicates recovery configuration status |
| `subjectKeyState` | Indicates whether subject key material is bound, deferred, or absent |
| `controlMode` | Indicates whether the eVault is self-controlled, delegated, or governed without subject key |
| `authorityEvidence` | Stored evidence or references proving delegated creation authority |
| `authorityValidationState` | Outcome of evidence validation |

### Certificate Content Expectations

The stored certificate must contain:

- the bound `eName`
- the initial public key
- validity metadata
- self-signature

### Metadata Expectations

Hosting metadata should include:

- provider identifier
- provider service endpoint or hosting reference
- last initialization status update

For delegated or organization creation, metadata should also include:

- evidence identifiers or references
- evidence digests
- validation outcome
- who validated the evidence

## States

| State | Meaning |
|---|---|
| `created` | The eVault record exists but is not yet ready for certificate binding |
| `pending_key_binding` | The eVault and eName have been allocated, but no valid initial certificate is stored yet |
| `active` | The eVault is in an operational state under its declared control model |
| `recovery_not_configured` | Additional state marker indicating recovery has not yet been configured |
| `delegated_controlled` | Additional state marker indicating control by an actor other than the subject |
| `subject_key_absent` | Additional state marker indicating no subject key material exists |

## State Transition Rules

### `created` -> `pending_key_binding`

- Entered after provider allocates eName and creates hosted storage
- The eVault must not be treated as user-operable yet

### `pending_key_binding` -> `active`

- Requires successful validation and storage of the initial certificate
- Requires metadata update confirming hosting and initialization completion

For organization variants, activation may occur without certificate storage only when the control model explicitly records that subject key material is absent.

### `active` + `recovery_not_configured`

- This is the expected result of the initial creation flow
- Recovery remains explicitly absent until a separate process configures it

### `pending_key_binding` + `delegated_controlled`

- Valid for a child eVault created by a parent or guardian
- The eVault exists, but the subject does not yet control it with their own keys

### `active` + `subject_key_absent`

- Valid for an organization eVault that has no subject signature capability
- Normal operations must rely on governance metadata rather than subject self-signature

## Interfaces

| Interface | Consumer | Purpose |
|---|---|---|
| Allocation interface | Cloud Provider | Create logical eVault record and move to pending state |
| Certificate binding interface | Cloud Provider | Persist initial certificate and activate eVault |
| Metadata query interface | Wallet, Provider, Registry integration | Confirm state and hosting information |
| Future data access interfaces | Platforms and user-controlled services | Out of scope for this flow but dependent on active state |

## Constraints

- The eVault must not become `active` before certificate storage succeeds.
- Exception: organization eVaults may become `active` without certificate storage only if `subjectKeyState=absent` is explicitly recorded.
- The eVault must not rely on secrecy of eName for protection.
- Recovery configuration must not be implied by successful activation.
- Certificate replacement must be handled by a separate governed flow.
- Child eVaults created without subject keys must remain distinguishable from fully self-controlled eVaults.
- Evidence used to justify delegated creation must be integrity-protected and traceable.

## Implementation Notes

- The certificate should be stored as immutable initialization evidence, even if later certificates are added.
- State transitions should be atomic with metadata persistence.
- If Registry publication is required by deployment policy, the provider should define whether activation waits for publication or records a deferred synchronization obligation.
