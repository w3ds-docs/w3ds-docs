---
title: Cloud Provider
weight: 2
---

# Cloud Provider Component

## Purpose

The Cloud Provider hosts eVault instances on behalf of users and executes the provider-side responsibilities of eVault creation and initialization.

## Responsibilities

- Expose provider endpoints for eVault provisioning and initialization
- Create the hosted eVault storage and logical metadata
- Maintain provisioning session state until key binding is completed or expires
- Validate signed wallet requests
- Persist the initial self-signed certificate into the eVault
- Transition the eVault from pre-initialized state to active state
- Record that the provider hosts the eVault
- Optionally publish hosting information to the Registry
- Return signed confirmations and error responses
- Support delegated creation with deferred key binding and organization eVaults without subject keys
- Host the internal Provisioning Service that performs eName allocation and creation orchestration

## Logical Role in the Architecture

The Cloud Provider is not the owner of the user's identity or data. It is a hosting operator that provides infrastructure services for a user-controlled eVault.

The provider is trusted to:

- execute the protocol correctly
- preserve integrity of stored certificate and metadata
- expose secure transport and signed responses

The provider is not trusted as a substitute for user key ownership.

For delegated and organization variants, the provider must preserve explicit control semantics instead of treating all eVaults as self-controlled by default.

## Internal Structure

| Internal Part | Purpose |
|---|---|
| Provider API | Receives provisioning and key-binding requests |
| Provisioning Service | Internal eVault management component that orchestrates creation workflows and allocates eNames |
| Provisioning Session Store | Tracks correlation IDs, nonces, timestamps, and session expiration |
| eVault Host Manager | Creates and maintains hosted eVault instances |
| Certificate Validator | Validates certificate structure, eName binding, and request consistency |
| Metadata Store | Persists hosting metadata and state transitions |
| Registry Publisher | Publishes hosting information when Registry integration is enabled |
| Signing Service | Signs receipts and provider responses |

## eName Generation

### Requirements

- eName must be globally unique in the W3DS naming domain.
- eName generation must not depend on secrecy.
- eName allocation must be collision-safe.
- eName generation must not require a centralized authority in the critical path unless explicitly configured for the deployment.

### Expected Behavior

- The provider-side provisioning service generates the eName during the allocation phase.
- The generated eName is bound to the current provisioning session.
- The provider must not allow reassignment of the same eName to another eVault.
- If allocation is rolled back, the provider should define whether the eName is retired or can be reused. Reuse is discouraged.

## eVault Hosting

The provider hosts:

- logical eVault identity and lifecycle state
- storage for certificates and future user data
- hosting metadata required for lookup, operation, and audit

The provider should support a two-phase lifecycle during creation:

1. Allocate storage and eName
2. Bind initial certificate and activate

For delegated variants, the provider must also support:

1. Allocate storage and eName
2. Record whether subject key state is `deferred` or `absent`
3. Keep the eVault pending or activate it according to the chosen control model

## Registry Interaction

Registry interaction is optional in this flow.

When enabled, the provider may:

- publish that it hosts the eVault for a given eName
- publish the service endpoint required for later resolution
- defer publication if local activation succeeded but external publication is temporarily unavailable

Provider implementation should treat Registry publication as:

- required for discoverability in deployments that depend on Registry resolution
- non-blocking only if the deployment model supports eventual consistency for hosting publication

## Trust Model

### Trusted Behaviors

- Correct execution of provider protocol logic
- Integrity of provider-signed receipts
- Correct persistence of eVault metadata and certificate
- Availability of hosted storage according to service guarantees

### Non-Trusted Behaviors

- The provider must not be treated as the owner of the user's private key
- The provider must not be able to silently replace the bound certificate without detection
- The provider must not be considered a recovery authority by default

## Main Risks

| Risk | Description | Mitigation |
|---|---|---|
| MITM during provisioning | Attacker tampers with provider communication | HTTPS or equivalent secure transport, signed messages, provider identity verification |
| Certificate substitution | Attacker or faulty provider binds wrong key to allocated eName | Verify eName inside certificate, bind request to provisioning session and provider nonce |
| Replay of wallet requests | Old signed request is resent to create or alter eVault state | Correlation IDs, timestamps, nonce tracking, expiry windows |
| Orphaned pending eVaults | Allocation succeeds but initialization never finishes | Session expiration and cleanup policy |
| False success to wallet | Wallet receives unverifiable or forged confirmation | Signed provider receipts and mandatory wallet-side verification |

## Interfaces

| Interface | Direction | Purpose |
|---|---|---|
| `CreateEvault` | Wallet -> Provider | Allocate eName and empty eVault |
| `BindInitialCertificate` | Wallet -> Provider | Store initial certificate and activate eVault |
| `RegistryHostingUpdate` | Provider -> Registry | Optional hosting publication |
| `InitializationReceipt` | Provider -> Wallet | Signed confirmation of successful initialization |

## Variant Handling

### Child eVault

- The provider may create the eVault without child key material.
- The provider should mark the eVault as `pending_key_binding`.
- If a parent or guardian acts as temporary controller, this must be explicit in metadata.
- The provider should store or reference parental authority evidence so later child key binding can rely on existing proof.

### Organization eVault

- The provider may activate the eVault without storing a subject self-signed certificate.
- The provider must record that subject key material is absent.
- APIs that require subject signature must not assume organization eVault support unless a governance layer defines it.
- The provider should store or reference authority documents and their validation outcome.

## Evidence and Trust Considerations

- Evidence validation should happen inside the provider-side provisioning service or another component within the same management boundary.
- For organization flows, the provider should be able to represent the authority chain, not just a single uploaded document.
- Evidence validation outcomes must be auditable and bound to the created eVault.
