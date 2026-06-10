---
title: Create eVault for Key-Owning Non-Human Entity
weight: 5
---

# Use Case: Create eVault for Key-Owning Non-Human Entity

## Goal

Allow an initiator to create an eVault for a non-human subject that has its own operational key material, such as an autonomous device, application, wallet, or service, while linking that subject to a trusted controller that governs recovery and higher-level control.

## Examples

- autonomous machine or vehicle
- IoT device
- software wallet
- backend service
- autonomous application agent

## Actors

- Initiator
- Initiator's eID Wallet
- Provider-side Provisioning Service
- Non-human subject key manager or bootstrap environment
- Cloud Provider
- eVault
- Registry (optional)

## Preconditions

- The initiator is authorized to create the eVault for the non-human subject.
- The initiator can authenticate to the provider-side provisioning service using wallet-based login.
- The provider-side provisioning service supports creation of key-owning non-human subjects.
- The subject can generate or receive operational key material during creation.
- The flow supports linking a trusted controller as recovery and governance authority.
- The provider supports storing both subject key material and trusted-controller metadata.

## Trigger

The initiator chooses to create an eVault for a device, application, wallet, or service that will operate using its own keys.

## Main Flow

1. The initiator opens the provider-side provisioning service.
2. The service requests wallet-based authentication.
3. The initiator authenticates to the service using their wallet.
4. The service creates an authenticated agent-creation session.
5. The service displays the list of available cloud providers.
6. The initiator selects a provider.
7. The service collects identity and classification data for the new non-human subject.
8. The service collects information about the trusted controller that will govern recovery and higher-level control.
9. The service collects authority evidence showing that the initiator may create and manage this subject.
10. The initiator submits the required evidence package.
11. The service validates the evidence package and the declared trusted-controller relationship.
12. The service requests generation or registration of the subject's operational key pair.
13. The subject key manager or bootstrap environment generates the operational key pair or provides the public key to the service.
14. The service creates a provisioning request for a key-owning non-human subject and signs it with service-controlled credentials.
15. The service sends the provisioning request to the provider.
16. The provider validates the request and verifies that this creation mode is allowed.
17. The provider generates a unique eName for the non-human subject.
18. The provider creates an empty hosted eVault.
19. The provider stores metadata indicating that the subject is a key-owning non-human entity.
20. The provider stores or references the authority evidence and validation outcome.
21. The provider stores trusted-controller metadata identifying the trusted controller responsible for recovery and governance.
22. The service constructs or submits the subject's initial certificate or public-key binding material.
23. The provider validates the subject key-binding material and stores it in the eVault.
24. The provider marks the eVault as `active`.
25. The provider records that recovery is governed through the trusted-controller relationship rather than through human-subject recovery flows such as friends or notaries.
26. The provider optionally publishes hosting information to the Registry.
27. The provider returns a signed creation response containing the eName, eVault reference, control metadata, and trusted-controller linkage.
28. The service verifies the provider response.
29. The service informs the initiator that the eVault has been created and that the non-human subject may now operate using its own keys.

## Alternative Flows by Main Flow Step

### A-3a: Wallet Login Fails

Branch from step 3.

1. The initiator cannot complete wallet-based login.
2. The provider-side provisioning service does not create the agent-creation session.
3. The flow ends before provider interaction begins.

### A-11a: Authority or Trusted-Controller Relationship Cannot Be Validated

Branch from step 11.

1. The service cannot validate the submitted authority evidence or the claimed trusted-controller relationship.
2. The request is rejected or moved to manual review.
3. The provider is not called until validation succeeds.

### A-13a: Subject Key Generation or Registration Fails

Branch from step 13.

1. The subject key manager cannot generate the key pair, or the bootstrap environment cannot produce valid key registration data.
2. The service cannot prepare a complete creation request with subject key material.
3. The flow stops before provider-side activation.

### A-23a: Provider Rejects Subject Key Binding

Branch from step 23.

1. The provider accepts subject creation but rejects the submitted key-binding material.
2. The provider does not activate the eVault.
3. The eVault remains non-operational or in `pending_key_binding`, depending on provider policy.

### A-25a: Trusted-Controller Metadata Is Not Committed

Branch from step 25.

1. The provider stores the subject key but fails to commit trusted-controller metadata.
2. The provider must not finalize the eVault as correctly governed.
3. The flow must fail or remain incomplete until trusted-controller linkage is consistent.

## Postconditions

### Success

- A unique eName has been allocated for the non-human subject.
- A hosted eVault exists for that subject.
- Subject operational key material is bound to the eVault.
- Trusted-controller metadata is stored.
- Authority evidence is stored or referenced.
- The eVault is `active`.
- The eVault is linked to a trusted controller for recovery and governance.

### Failure

- No operationally active subject is created without explicit trusted-controller linkage.
- The eVault must not be presented as an independent human-controlled identity.

## Resulting Control Model

- `active`
- subject key bound
- non-human operational subject
- `trusted_controller_linked`
- recovery delegated to trusted controller

## Notes

- This use case differs from organization eVault creation because the subject itself has active key material and can sign or authenticate.
- This use case differs from self-provisioned human creation because recovery and higher-level governance are not handled by the subject directly.
- The later lifecycle model must define key rotation, trusted-controller replacement, and emergency recovery procedures for the non-human subject.
