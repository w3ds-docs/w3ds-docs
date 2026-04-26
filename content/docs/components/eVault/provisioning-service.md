---
title: Provisioning Service
weight: 5
---

# eVault Provisioning Service

## Purpose

The Provisioning Service is a provider-side component of the eVault management system. It orchestrates eVault creation flows, including self-provisioning, delegated creation, organization creation, and creation of key-owning non-human entities.

It is not a separate trust domain from the eVault hosting system. It is part of the provider-operated software boundary and should be secured, audited, and deployed as part of the eVault management plane.

## Why It Belongs Inside the eVault System

- It simplifies the trust model by keeping provisioning inside the provider's security boundary.
- It makes responsibility for `eName` allocation explicit and local to the eVault management plane.
- It reduces the number of externally trusted actors.
- It allows evidence handling, control-model commitment, and eVault activation to be audited within one system boundary.

## Responsibilities

- Expose creation workflows for all supported eVault subject types
- Authenticate initiators, including wallet-based login for delegated flows
- Create and track provisioning sessions
- Generate or reserve unique `eName` values
- Orchestrate creation of hosted eVault records
- Collect, validate, store, or reference authority evidence
- Commit the chosen control model for the new eVault
- Coordinate subject key binding when required
- Record trusted-controller relationships where applicable
- Return signed provisioning receipts and error responses

## Supported Subject Types

| Subject Type | Example | Key Model |
|---|---|---|
| Human self | person creating eVault for self | subject key bound during creation |
| Non-subject human | child, incapacitated person | subject key deferred |
| Organization | firm, group, community | no subject key, governance-controlled |
| Key-owning non-human subject | device, wallet, service, application | subject key bound, recovery governed by trusted controller |

## Internal Structure

| Internal Part | Purpose |
|---|---|
| Session Manager | Tracks provisioning sessions, timestamps, nonces, and correlation IDs |
| Auth Gateway | Handles wallet-based login and maps authenticated initiators to creation rights |
| eName Allocator | Generates or reserves unique `eName` values |
| Subject Type Router | Chooses the creation workflow based on subject type and control model |
| Evidence Intake Module | Accepts uploaded evidence or evidence references |
| Evidence Validation Engine | Validates evidence structure, integrity, and chain-of-authority rules |
| Trusted Controller Manager | Records and validates trusted-person or trusted-organization linkage |
| Key Binding Coordinator | Coordinates certificate or public-key binding when the control model requires it |
| eVault Activation Coordinator | Commits the final control model and transitions the eVault to the correct state |
| Receipt Signer | Signs success and failure responses |

## Interfaces

| Interface | Consumer | Purpose |
|---|---|---|
| Self-provisioning API | eID Wallet | Create eVault for the authenticated human subject |
| Delegated provisioning API | Wallet-authenticated initiator | Create eVault for dependent human subject, organization subject, or non-human subject |
| Evidence API | Initiator-side clients | Upload or reference authority evidence |
| Key-binding API | eID Wallet or bootstrap environment | Bind subject key material when required |
| Receipt API | Initiator-side clients | Receive signed completion or error responses |

## eName Generation

### Requirements

- `eName` generation must happen inside the provider's trusted provisioning boundary.
- Generated `eName` values must be globally unique in the W3DS naming domain.
- `eName` allocation must be bound to the provisioning session and subject record.

### Security Rationale

Keeping `eName` generation inside the provisioning service:

- avoids split responsibility between orchestration and allocation
- simplifies auditing
- prevents ambiguity about which system committed the identity

## Evidence Handling

The provisioning service must support:

- parenthood or guardianship evidence
- power-of-attorney and delegated authority evidence
- corporate authorization packages and authority chains
- trusted-controller linkage evidence

Evidence may be:

- stored directly
- stored in a controlled evidence store and referenced
- attested after validation by the provisioning service

## Trust Model

The provisioning service is trusted to:

- correctly authenticate initiators
- allocate `eName` values
- validate authority evidence according to configured policy
- commit the correct control model

It is not trusted to:

- replace subject key material without auditability
- infer authority without evidence
- silently downgrade governance or controller metadata

## Security Considerations

- Wallet-based login must be resistant to replay and session fixation.
- Evidence validation results must be bound to the created eVault.
- `eName` generation and control-model commitment should be atomic with eVault record creation.
- Signed receipts should identify the committed control model and evidence-validation state.

## Relationship to Other Components

- The Provisioning Service is part of the eVault management system.
- It works with the eVault Host Manager to create hosted eVault records.
- It works with certificate and key-binding components to activate subjects that require keys.
- It works with Registry publication logic after local state is committed.
