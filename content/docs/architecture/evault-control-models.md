---
title: eVault Control Models
weight: 2
---

# eVault Control Models

## Purpose

This document defines the control models used by eVaults during and after creation. It is the reference point for interpreting who may act for an eVault, whether the subject has key material, and how recovery and governance are expected to work.

Control models are necessary because eVault creation is not limited to self-controlled human subjects. W3DS must also support dependent human subjects, organization subjects, and non-human subjects that operate with their own keys but remain governed by a trusted controller.

## Canonical Terms

Use the following terms consistently across architecture, requirements, protocols, and use cases.

| Term | Meaning |
|---|---|
| `subject` | The entity represented by the eVault |
| `human subject` | A natural person represented by the eVault |
| `dependent human subject` | A human subject who does not yet control the eVault directly, such as a child or an incapacitated person |
| `organization subject` | A company, group, community, or other collective entity represented by the eVault |
| `non-human subject` | A device, application, wallet, service, or other non-human entity represented by the eVault |
| `initiator` | The authenticated actor who starts the creation flow |
| `trusted controller` | A trusted person or trusted organization explicitly linked for governance or recovery |
| `governance authority` | The authority model that permits structural actions for the eVault |

## Why This Document Exists

Without explicit control models, the system would make unsafe assumptions such as:

- every eVault has a human subject with self-controlled keys
- every active eVault can sign on its own behalf
- delegated creation automatically implies indefinite delegated control
- organizations can be represented like people

This document prevents those assumptions by defining the allowed models explicitly.

## Control Model Dimensions

Every eVault control model is defined by the combination of:

| Dimension | Question |
|---|---|
| Subject type | What kind of entity does the eVault represent? |
| Operational key ownership | Does the subject itself own active key material? |
| Governance authority | Who may authorize structural or recovery actions? |
| Recovery authority | Who can restore or replace control after failure? |
| Evidence requirements | What proof is needed to create and maintain this control model? |

## Supported Control Models

| Control Model | Subject Type | Subject Key State | Governance Authority | Typical Use Case |
|---|---|---|---|---|
| `self` | human | `bound` | subject | person creates eVault for self |
| `delegated` | dependent human subject | `deferred` | initiator acting under delegated authority | parent creates eVault for child |
| `governed_without_subject_key` | organization subject | `absent` | governance authority | company, group, community |
| `trusted_controller_key_owner` | non-human subject | `bound` | trusted controller | device, wallet, application, service |

## Model 1: Self

### Description

The subject creates and controls their own eVault directly. Subject key material is created and bound during the creation flow.

### Characteristics

- subject is a human
- subject key material is present from creation time
- subject authenticates and signs for their own eVault
- recovery is later configured by the subject through supported recovery flows

### Required State

- `active`
- subject key bound
- `recovery_not_configured` immediately after creation

### Typical Evidence Model

- no delegated authority evidence required
- wallet-based self-authentication is sufficient for creation

## Model 2: Delegated

### Description

An initiator creates an eVault for a dependent human subject who cannot yet control it directly.

### Characteristics

- subject is a child or another dependent human subject
- subject key material is not yet present
- the initiator acts temporarily or structurally on behalf of the dependent human subject
- the system must record evidence of delegated authority

### Required State

- `pending_key_binding`
- optionally `delegated_controlled`
- `recovery_not_configured`

### Typical Evidence Model

- parenthood evidence
- guardianship evidence
- legal or medical authority evidence for dependent adults

### Constraints

- the dependent subject must not be treated as self-controlled before key binding is completed
- delegated creation does not by itself define the full long-term governance model

## Model 3: Governed Without Subject Key

### Description

The subject is an organization or collective entity that does not have a single subject-owned signing key.

### Characteristics

- subject is not a natural person
- subject key material is absent
- authority comes from governance documents and authorized representatives
- operational permissions depend on governance interpretation, not on subject self-signature

### Required State

- `active`
- `subject_key_absent`
- `recovery_not_configured`

### Typical Evidence Model

- articles of incorporation
- charter
- corporate registry extract
- power of attorney
- delegated authorization chain

### Constraints

- no system component may assume that this eVault can produce subject self-signatures
- governance state must be explicit and auditable

## Model 4: Trusted Controller Key Owner

### Description

The subject is a non-human subject with its own operational keys. Recovery and higher-level governance are assigned to a trusted controller.

### Characteristics

- subject may be a device, application, wallet, or service
- subject key material is present and operational
- the subject may authenticate and sign on its own behalf
- trusted-controller linkage governs recovery and structural changes

### Required State

- `active`
- subject key bound
- `trusted_controller_linked`

### Typical Evidence Model

- proof that the initiator may create the non-human subject
- proof of trusted-controller relationship
- ownership or operational assignment records

### Constraints

- operational autonomy does not imply governance autonomy
- the trusted controller must be recorded explicitly

## Mapping to eVault Creation Use Cases

| Use Case | Control Model |
|---|---|
| [Create eVault for Self](/docs/use-cases/create-evault-self/) | `self` |
| [Create eVault for Non-Subject Entity](/docs/use-cases/create-evault-dependent/) | `delegated` |
| [Create eVault for Organization](/docs/use-cases/create-evault-organization/) | `governed_without_subject_key` |
| [Create eVault for Key-Owning Non-Human Entity](/docs/use-cases/create-evault-agent/) | `trusted_controller_key_owner` |

## Mapping to Protocol Fields

| Protocol Field | Meaning |
|---|---|
| `requestedSubjectType` | High-level subject classification |
| `controlMode` | Declared control model |
| `subjectKeyState` | Whether subject key material is `bound`, `deferred`, or `absent` |
| `evidenceContext` | Evidence expectations for delegated or governed creation |

## Mapping to eVault State Markers

| Marker | Meaning |
|---|---|
| `delegated_controlled` | A dependent human subject is currently controlled through another actor's delegated authority |
| `subject_key_absent` | The subject has no subject-owned key material |
| `trusted_controller_linked` | A trusted controller is recorded as recovery and governance authority |

## Security Implications

### General

- eVault existence must not be treated as proof of subject autonomy
- key presence must not be inferred from `active` alone
- governance authority must not be inferred from operational key ownership alone

### Self

- security depends on the subject's own key protection and later recovery setup

### Delegated

- security depends on strong delegated-authority evidence and clear termination of delegated control

### Governed Without Subject Key

- security depends on governance-document integrity and authority-chain validation

### Trusted Controller Key Owner

- security depends on separation between operational keys and trusted-controller powers

## Architectural Constraints

- every created eVault must have exactly one declared control model at creation completion
- control-model transitions must be explicit lifecycle events
- evidence requirements must follow the control model, not ad hoc UI decisions
- recovery flows must be compatible with the control model

## Open Questions

- Can a control model change after creation without creating a new governance artifact?
- How should mixed models be represented, for example a non-human subject temporarily governed by an organization with delegated human operators?
- Should `delegated_controlled` and `trusted_controller_linked` be treated as state markers, metadata markers, or both?
