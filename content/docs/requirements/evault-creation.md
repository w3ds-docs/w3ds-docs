---
title: eVault Creation
weight: 3
---

# Requirements: eVault Creation and Initialization

## Scope

These requirements define the TRL7 implementation baseline for creating and initializing a user eVault. They cover allocation, key binding, activation, and confirmation. Recovery configuration is out of scope.

The normative definition of supported control models is provided in [eVault Control Models](/docs/architecture/evault-control-models/).

## Functional Requirements

| ID | Classification | Requirement |
|---|---|---|
| FR-EC-1 | MUST | The system must allow a user to initiate eVault creation from the eID Wallet. |
| FR-EC-2 | MUST | The wallet must present a selectable list of cloud providers. |
| FR-EC-3 | MUST | The provider must accept a signed provisioning request from the wallet. |
| FR-EC-4 | MUST | The provider must generate a unique eName for each newly allocated eVault. |
| FR-EC-5 | MUST | The provider must create an empty hosted eVault before key binding completes. |
| FR-EC-6 | MUST | The provider must return the allocated eName to the wallet. |
| FR-EC-7 | MUST | The wallet must generate an initial asymmetric key pair after receiving the allocated eName. |
| FR-EC-8 | MUST | The wallet must create a self-signed certificate containing the allocated eName and public key. |
| FR-EC-9 | MUST | The wallet must send the certificate to the provider in a signed key-binding request. |
| FR-EC-10 | MUST | The provider must validate that the certificate eName matches the allocated eName. |
| FR-EC-11 | MUST | The provider must store the certificate in the eVault. |
| FR-EC-12 | MUST | The provider must mark the eVault as initialized only after the control model is fully committed: either successful certificate storage or explicit recording that subject key material is absent. |
| FR-EC-13 | MUST | The provider must record that it hosts the eVault. |
| FR-EC-14 | MUST | The provider must return an initialization confirmation to the wallet. |
| FR-EC-15 | SHOULD | The provider should return the confirmation as a signed receipt. |
| FR-EC-16 | SHOULD | The provider should publish hosting information to the Registry when the deployment model requires external discoverability. |
| FR-EC-17 | MUST | The final eVault state after successful completion must be `active` and `recovery_not_configured`. |
| FR-EC-18 | MUST | Recovery configuration must be handled by a separate flow. |
| FR-EC-19 | MUST | The system must support delegated creation of a child eVault without immediate subject key binding. |
| FR-EC-20 | MUST | The system must explicitly represent whether subject key state is `bound`, `deferred`, or `absent`. |
| FR-EC-21 | MUST | The system must support creation of an organization eVault without subject signature capability when governance rules permit it. |
| FR-EC-22 | MUST | The system must distinguish self-controlled, delegated, and no-subject-key eVaults in metadata and behavior. |
| FR-EC-23 | MUST | The system must support delegated creation through a provisioning service authenticated by wallet-based login. |
| FR-EC-24 | MUST | Child eVault creation flows must support attaching evidence that the initiator is authorized to act for the child. |
| FR-EC-25 | MUST | Organization eVault creation flows must support attaching or referencing authority evidence such as power-of-attorney or corporate authorization packages. |
| FR-EC-26 | MUST | The system must record evidence validation outcome and bind it to the created eVault. |
| FR-EC-27 | MUST | The provisioning service must be part of the provider-side eVault management system, not a separate external trust domain. |
| FR-EC-28 | MUST | The provider-side provisioning service must generate or reserve the `eName` within the provider's trusted management boundary. |
| FR-EC-29 | MUST | The system must support creation of key-owning non-human entities whose operational keys are bound at creation time. |
| FR-EC-30 | MUST | The system must support linking a trusted controller for recovery and governance of key-owning non-human subjects. |

These requirements apply to the supported control models defined in [eVault Control Models](/docs/architecture/evault-control-models/).

## Security Requirements

| ID | Classification | Requirement |
|---|---|---|
| SR-EC-1 | MUST | All transport for this flow must be protected by HTTPS or equivalent secure transport. |
| SR-EC-2 | MUST | Wallet requests must be signed. |
| SR-EC-3 | MUST | Provider responses that indicate allocation success or initialization success must be signed. |
| SR-EC-4 | MUST | Every protocol message must include a timestamp and correlationId. |
| SR-EC-5 | MUST | The system must implement replay protection for signed requests. |
| SR-EC-6 | MUST | The certificate used in this flow must be self-signed. |
| SR-EC-7 | MUST | The certificate must include the allocated eName. |
| SR-EC-8 | MUST | The provider must reject certificate substitution attempts. |
| SR-EC-9 | MUST | The provider must reject initialization requests whose session binding, timestamp, or nonce validation fails. |
| SR-EC-10 | MUST | The system must not rely on secrecy of eName as an authentication or authorization control. |
| SR-EC-11 | SHOULD | The wallet should verify the provider's signed confirmation before presenting success to the user. |
| SR-EC-12 | SHOULD | The wallet should use hardware-backed secure key storage where available. |
| SR-EC-13 | SHOULD | The provider should retain signed initialization receipts for audit and dispute resolution. |
| SR-EC-14 | MUST | The system must not infer subject authorization merely from the existence of an eVault. |
| SR-EC-15 | MUST | Delegated and organization variants must rely on explicit controller or governance metadata for authorization. |
| SR-EC-16 | MUST | Authority evidence used in delegated creation must be integrity-protected and auditable. |
| SR-EC-17 | SHOULD | Organization evidence validation should verify the authorization chain back to persons who have signing authority for the relevant action. |
| SR-EC-18 | MUST | The provider-side provisioning service must be secured as part of the eVault management plane and must not expose weaker trust assumptions than the hosting system itself. |
| SR-EC-19 | MUST | `eName` allocation must be auditable and bound to the same provisioning session that creates the eVault. |

## Non-Functional Requirements

| ID | Classification | Requirement |
|---|---|---|
| NFR-EC-1 | MUST | The creation flow must be implemented as a deterministic allocation phase followed by explicit control-model commitment. |
| NFR-EC-2 | MUST | State transitions for certificate persistence and activation must be atomic from the provider's perspective. |
| NFR-EC-3 | MUST | Failed flows must not result in user-visible false success. |
| NFR-EC-4 | SHOULD | Providers should define timeout and cleanup rules for eVaults left in `pending_key_binding`. |
| NFR-EC-5 | SHOULD | Protocol messages should be versioned to support backward-compatible evolution. |
| NFR-EC-6 | SHOULD | The system should produce sufficient logs and receipts to support operational diagnostics. |
| NFR-EC-7 | MAY | Registry publication may be asynchronous if the deployment model tolerates eventual consistency for hosting discovery. |
| NFR-EC-8 | SHOULD | Implementations should use one explicit state model across self, child, and organization variants rather than relying on implicit interpretation. |
| NFR-EC-9 | SHOULD | Evidence validation should support both automated checks and manual review where legal or organizational complexity requires it. |
| NFR-EC-10 | SHOULD | The provider-side provisioning service should be deployable and monitorable as part of the eVault management system rather than as a separately governed subsystem. |

## Required State Semantics

The control-model interpretation of these states and markers is defined in [eVault Control Models](/docs/architecture/evault-control-models/).

| State | Requirement |
|---|---|
| `created` | The system may use this as an internal pre-binding allocation state. |
| `pending_key_binding` | The system must represent an allocated but not yet activated eVault. |
| `active` | The system must use this only after either certificate validation and storage succeed or a no-subject-key control model is explicitly committed. |
| `recovery_not_configured` | The system must explicitly distinguish activation from recovery readiness. |
| `delegated_controlled` | The system should use this to mark eVaults controlled by an actor other than the subject. |
| `subject_key_absent` | The system must use this or equivalent metadata to indicate absence of subject key material. |
| `trusted_controller_linked` | The system should use this to mark subjects whose recovery and governance are linked to a trusted controller. |

## Acceptance Criteria

- A developer can implement provider endpoints and wallet flows directly from these requirements and the protocol specification.
- An architect can verify that identity allocation, key binding, and activation are separated.
- A reviewer can confirm that replay protection, signed messages, and certificate-to-eName binding are mandatory implementation concerns.
