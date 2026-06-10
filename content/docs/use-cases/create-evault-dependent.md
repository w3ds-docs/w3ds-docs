---
title: Create eVault for Dependent Human Subject
weight: 3
---

# Use Case: Create eVault for Dependent Human Subject

## Goal

Allow an initiator to create an eVault for a dependent human subject, such as a child or an incapacitated person, before that subject has subject-controlled key material.

## Actors

- Initiator
- Initiator's eID Wallet
- Provider-side Provisioning Service
- Cloud Provider
- eVault
- Registry (optional)

## Preconditions

- The initiator is authorized to act for the dependent human subject.
- The initiator can authenticate to the provider-side provisioning service using wallet-based login.
- The provider-side provisioning service supports delegated creation flows.
- The provider supports delegated creation for dependent human subjects.
- The flow supports collecting delegated-authority evidence.
- Subject key binding may be deferred.

## Trigger

The initiator chooses to create an eVault for a child or another dependent human subject through the provider-side provisioning service.

## Main Flow

1. The initiator opens the provider-side provisioning service.
2. The service requests wallet-based authentication.
3. The initiator authenticates to the service using their wallet.
4. The service creates an authenticated delegated-creation session.
5. The service displays the list of available cloud providers.
6. The initiator selects a provider.
7. The service collects the identity data of the dependent human subject.
8. The service requests evidence that the initiator is authorized to act for that dependent human subject.
9. The initiator uploads or submits the evidence package.
10. The service validates the evidence package format and required metadata.
11. The service creates a delegated provisioning request and signs it with service-controlled credentials.
12. The service sends the delegated provisioning request to the provider.
13. The provider validates the request and verifies that delegated creation is allowed.
14. The provider generates a unique eName for the new eVault.
15. The provider creates an empty hosted eVault for the dependent human subject.
16. The provider records that the eVault was created through a delegated flow.
17. The provider stores or references the submitted evidence and its validation result.
18. The provider optionally records temporary controller metadata for the initiating subject.
19. The provider marks the eVault as `pending_key_binding`.
20. The provider records that recovery is not configured.
21. The provider optionally publishes hosting information to the Registry.
22. The provider returns a signed delegated-creation response containing the eName, eVault reference, and current control state.
23. The service verifies the provider response.
24. The service informs the initiator that the eVault has been created and that subject key binding must be completed later.

## Alternative Flows by Main Flow Step

### A-3a: Wallet Login Fails

Branch from step 3.

1. The initiator cannot complete wallet-based login.
2. The provider-side provisioning service does not create the delegated-creation session.
3. The flow ends before provider interaction begins.

### A-10a: Evidence Package Rejected by Service

Branch from step 10.

1. The service determines that the evidence is incomplete, malformed, expired, or fails preliminary checks.
2. The service rejects the evidence package.
3. The service does not call the provider until valid evidence is provided.

### A-13a: Delegated Creation Request Rejected by Provider

Branch from step 13.

1. The provider rejects the delegated provisioning request because the service is not trusted, the request is invalid, or delegated creation is not allowed.
2. The provider returns a signed error response.
3. No eVault is created.

### A-17a: Evidence Cannot Be Stored or Referenced

Branch from step 17.

1. The provider accepts the delegated flow but cannot persist or reference the authority evidence.
2. The provider must either reject completion or keep the eVault in an evidentially incomplete state according to deployment policy.
3. The response must make that state explicit.

## Postconditions

### Success

- A unique eName has been allocated.
- A hosted eVault exists for the dependent human subject.
- Delegated creation metadata is stored.
- Authority evidence is stored or referenced.
- The eVault remains `pending_key_binding`.
- The eVault is `recovery_not_configured`.

### Failure

- No delegated eVault is created without explicit authority handling.
- The eVault must not be presented as self-controlled by the dependent human subject.

## Resulting Control Model

- `pending_key_binding`
- optionally `delegated_controlled`
- subject key deferred
- `recovery_not_configured`

## Notes

- This use case includes children and other dependent human subjects who cannot yet control their own eVault.
- A later flow must define how subject key material is attached and how delegated control is reduced or removed.
