---
title: Create eVault
weight: 1
---

# eVault Creation Use Cases

eVault creation is not a single scenario. In W3DS, the creation flow depends on who initiates the process and what kind of subject the new eVault represents.

This use case family is split into four separate scenarios:

1. [Create eVault for Self](/docs/use-cases/create-evault-self/)
2. [Create eVault for Non-Subject Entity](/docs/use-cases/create-evault-dependent/)
3. [Create eVault for Organization](/docs/use-cases/create-evault-organization/)
4. [Create eVault for Key-Owning Non-Human Entity](/docs/use-cases/create-evault-agent/)

## Why the Split Is Necessary

These scenarios differ in fundamental ways:

- who authenticates and who the subject is
- whether subject-controlled key material exists at creation time
- whether a self-signed certificate is required
- whether delegated authority evidence must be collected and validated
- whether the resulting eVault becomes self-controlled, delegated, or governance-controlled

The fourth scenario adds a different model:

- the subject is not a human, but it still owns operational key material
- recovery and higher-level control are delegated to a trusted controller

## Shared Invariants

All four creation flows share the following principles:

- the provider allocates a unique eName
- the provider creates a hosted eVault
- recovery setup is out of scope for initial creation
- eName is an identifier, not a secret
- the resulting control model must be explicit in metadata and state

## Shared State Vocabulary

| State or Marker | Meaning |
|---|---|
| `created` | eVault record exists but initialization is not complete |
| `pending_key_binding` | eVault exists but subject key binding has not been completed |
| `active` | eVault is operational under its declared control model |
| `recovery_not_configured` | Recovery setup is still missing |
| `delegated_controlled` | eVault is controlled by an actor other than the subject |
| `subject_key_absent` | eVault has no subject-controlled key material |
| `trusted_controller_linked` | A trusted controller is recorded as the recovery and governance authority for the subject |

## Scope Note

Detailed message-level protocol behavior is specified in [eVault Creation Protocol](/docs/protocols/evault-creation/).
