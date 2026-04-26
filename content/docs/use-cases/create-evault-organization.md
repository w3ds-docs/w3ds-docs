---
title: Create eVault for Organization
weight: 4
---

# Use Case: Create eVault for Organization

## Goal

Allow a subject to create an eVault for an organization such as a company, group, or community, under a governance-based control model that may not include subject-controlled key material.

## Actors

- Initiator
- Initiator's eID Wallet
- Provider-side Provisioning Service
- Cloud Provider
- eVault
- Registry (optional)

## Preconditions

- The initiator is authorized to act for the organization subject.
- The initiator can authenticate to the provider-side provisioning service using wallet-based login.
- The provider-side provisioning service supports organization creation flows.
- The provider supports organization eVault creation.
- The flow supports collection and validation of organizational authority evidence.
- The organization control model does not require a subject self-signed certificate at creation time.

## Trigger

The initiator chooses to create an eVault for an organization through the provider-side provisioning service.

## Main Flow

1. The initiator opens the provider-side provisioning service.
2. The service requests wallet-based authentication.
3. The initiator authenticates to the service using their wallet.
4. The service creates an authenticated organization-creation session.
5. The service displays the list of available cloud providers.
6. The initiator selects a provider.
7. The service collects the organization identity data required for eVault creation.
8. The service requests authority evidence, such as power-of-attorney, corporate authorization documents, or another valid governance package.
9. The initiator uploads or submits the evidence package.
10. The service validates the evidence package structure and required metadata.
11. The service verifies the authority chain as far as its trust model allows.
12. The service creates an organization provisioning request and signs it with service-controlled credentials.
13. The service sends the signed provisioning request to the provider.
14. The provider validates the request and verifies that organization creation is allowed for this service and session.
15. The provider generates a unique eName for the organization eVault.
16. The provider creates an empty hosted eVault for the organization.
17. The provider stores metadata indicating that the subject is an organization subject.
18. The provider stores the chosen governance or control mode.
19. The provider stores or references the authority evidence and its validation outcome.
20. The provider explicitly records that subject key material is absent.
21. The provider marks the eVault as `active` under the governance-based control model.
22. The provider records that recovery is not configured.
23. The provider optionally publishes hosting information to the Registry.
24. The provider returns a signed response containing the eName, eVault reference, and control metadata.
25. The service verifies the provider response.
26. The service informs the initiator that the organization eVault has been created successfully under a governance-based control model.

## Alternative Flows by Main Flow Step

### A-3a: Wallet Login Fails

Branch from step 3.

1. The initiator cannot complete wallet-based login.
2. The provider-side provisioning service does not create the organization-creation session.
3. The flow ends before provider interaction begins.

### A-11a: Authority Chain Cannot Be Verified

Branch from step 11.

1. The service cannot verify that the submitted evidence is valid for this action.
2. The service places the request into manual review or rejects it.
3. The provider is not called until the authority chain is accepted.

### A-14a: Provider Rejects Organization Creation

Branch from step 14.

1. The provider rejects the organization provisioning request because the service is not trusted, the request is invalid, or the requested control model is unsupported.
2. The provider returns a signed error response.
3. No organization eVault is created.

### A-20a: Subject Key Absence Not Recorded Correctly

Branch from step 20.

1. The provider creates the organization eVault but fails to commit the `subject_key_absent` control state.
2. The provider must not finalize the eVault as active under an ambiguous control model.
3. The flow must fail or remain incomplete until the control metadata is consistent.

## Postconditions

### Success

- A unique eName has been allocated for the organization.
- A hosted eVault exists for the organization.
- Governance metadata is stored.
- Authority evidence is stored or referenced.
- The eVault is `active`.
- The eVault is `subject_key_absent`.
- The eVault is `recovery_not_configured`.

### Failure

- No organization eVault is created without explicit governance metadata.
- No organization eVault is presented as self-controlled by a natural-person subject unless that is defined by another control model.

## Resulting Control Model

- `active`
- governance-controlled
- `subject_key_absent`
- `recovery_not_configured`

## Notes

- This use case covers firms, groups, and communities when their eVault is controlled by organizational governance rather than by a single subject key.
- The later operational model must define how writes, governance changes, and recovery are authorized.
