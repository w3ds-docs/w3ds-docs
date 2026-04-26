---
title: Create eVault for Self
weight: 2
---

# Use Case: Create eVault for Self

## Goal

Allow a subject to create their own eVault, bind their initial cryptographic key material, and finish with an active self-controlled eVault.

## Actors

- Subject
- eID Wallet
- Cloud Provider
- eVault
- Registry (optional)

## Preconditions

- The subject has a functioning eID Wallet instance.
- The wallet can establish a protected transport channel to available cloud providers.
- The wallet has access to cryptographically secure randomness.
- The selected provider accepts self-provisioning requests.
- Recovery configuration is not required for initial success.

## Trigger

The subject chooses to create their own Digital Self / eVault from the eID Wallet.

## Main Flow

1. The subject opens the eID Wallet.
2. The wallet displays the list of available cloud providers.
3. The subject selects a cloud provider.
4. The wallet creates a provisioning request and signs it.
5. The wallet sends the signed provisioning request to the provider.
6. The provider validates the request.
7. The provider generates a unique eName.
8. The provider creates an empty hosted eVault.
9. The provider returns a signed allocation response containing the eName and provider-side eVault reference.
10. The wallet validates the provider response and binds the returned eName to the active provisioning session.
11. The wallet generates the subject's initial asymmetric key pair.
12. The wallet creates a self-signed certificate that contains the allocated eName and the generated public key.
13. The wallet creates and signs a key-binding request.
14. The wallet sends the key-binding request to the provider.
15. The provider validates the request, verifies the certificate, and verifies that the eName matches the allocated provisioning session.
16. The provider stores the certificate in the eVault.
17. The provider records hosting metadata and marks the eVault as `active`.
18. The provider records that recovery is not configured.
19. The provider optionally publishes hosting information to the Registry.
20. The provider returns a signed initialization confirmation.
21. The wallet verifies the confirmation and persists the local association between the eName and the subject's key material.
22. The wallet informs the subject that the eVault has been created successfully.

## Alternative Flows by Main Flow Step

### A-4a: Provider Selection Fails

Branch from step 4.

1. The wallet cannot obtain valid provider metadata or trust information.
2. The wallet does not send a provisioning request.
3. The wallet informs the subject and returns to provider selection.

### A-6a: Provisioning Request Rejected

Branch from step 6.

1. The provider determines that the provisioning request is invalid, expired, duplicated, unauthorized, or incorrectly signed.
2. The provider returns a signed error response.
3. No eVault is created and no eName is allocated.

### A-9a: Allocation Response Cannot Be Trusted

Branch from step 9.

1. The provider returns the allocation response.
2. The wallet cannot verify the provider signature or session integrity.
3. The wallet does not continue to key generation.
4. The subject is informed that the allocation state is uncertain.

### A-11a: Key Generation Fails

Branch from step 11.

1. The wallet attempts to generate the initial key pair.
2. Key generation fails.
3. The wallet does not create the certificate and does not continue.
4. The eVault remains in `pending_key_binding`.

### A-15a: Provider Rejects Key Binding

Branch from step 15.

1. The provider receives the key-binding request.
2. The provider rejects it because the certificate is malformed, the eName does not match, or freshness or signature checks fail.
3. The provider does not store the certificate.
4. The eVault remains in `pending_key_binding`.

### A-20a: Confirmation Cannot Be Verified

Branch from step 20.

1. The wallet receives the initialization confirmation.
2. The wallet cannot verify the signature or receipt content.
3. The wallet must not report full success.
4. The provider may already consider the eVault active.

## Postconditions

### Success

- A unique eName has been allocated.
- A hosted eVault exists for that eName.
- The subject's initial certificate is stored in the eVault.
- The eVault is `active`.
- The eVault is `recovery_not_configured`.
- The wallet has persisted the local eName-to-key association.

### Failure

- No false success is presented to the subject.
- If allocation succeeded but key binding failed, the eVault remains non-self-controlled.

## Resulting Control Model

- `active`
- self-controlled
- subject key bound
- `recovery_not_configured`

## Open Questions

- What exact certificate format should be standardized for TRL7?
- What request-signing material should the wallet use before the subject key pair exists?
- What is the cleanup policy for eVaults left in `pending_key_binding`?
