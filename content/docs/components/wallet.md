---
title: eID Wallet
weight: 1
---

# eID Wallet Component

## Purpose

The eID Wallet is the user-side control component that drives eVault creation, generates the user's initial key material, creates the initial certificate, and verifies the provider's confirmation.

## Responsibilities

- Present available cloud providers to the user
- Let the user select a provider for eVault hosting
- Establish protected communication with the provider
- Create and sign provisioning requests
- Generate the initial asymmetric key pair
- Create a self-signed certificate containing the allocated eName
- Send the certificate binding request to the provider
- Verify signed provider responses and receipts
- Persist the local association between eName and locally controlled private key
- Inform the user about success or failure
- Support delegated creation contexts where the initiator and subject are different entities

## Internal Structure

| Internal Part | Purpose |
|---|---|
| Provider Catalog Client | Obtains and displays provider list |
| Provisioning Orchestrator | Executes the create-and-bind workflow |
| Secure Key Manager | Generates and protects private key material |
| Certificate Builder | Creates self-signed certificate for the initial public key |
| Request Signer | Signs wallet protocol messages |
| Receipt Verifier | Verifies signed provider confirmations |
| Local Identity Store | Persists eName, key references, and local state |
| User Interaction Layer | Presents status and obtains user intent |

## Key Generation

### Requirements

- Key generation must use a cryptographically secure random source.
- Private keys must remain under wallet control.
- Keys should be generated inside secure hardware when such support exists.
- Generated key material must be associated locally with the allocated eName only after provider response validation.

### Expected Behavior

1. Wait until the provider returns the allocated eName.
2. Generate a new asymmetric key pair.
3. Retain the private key inside wallet-controlled secure storage.
4. Export or reference the public key for certificate creation.

For delegated variants:

- Child eVault creation may skip subject key generation during initial creation.
- Organization eVault creation may omit subject key generation entirely.

## Signing

The wallet participates in two different signing activities during this flow:

### Request Signing

- Wallet signs the provisioning request.
- Wallet signs the certificate-binding request.
- Request signatures provide integrity, origin authentication, and replay protection when combined with timestamps and correlation data.

### Certificate Self-Signature

- Wallet uses the generated user private key to self-sign the initial certificate.
- The self-signature proves possession of the corresponding private key at creation time.

## Certificate Creation

### Mandatory Certificate Content

- Subject eName
- Public key
- Issue time
- Expiration time or validity model
- Self-signature

### Certificate Rules

- The eName embedded in the certificate must exactly match the provider-allocated eName.
- The wallet must not create the certificate before receiving and validating the provider allocation response.
- The wallet must treat certificate format as a protocol contract, not as a UI artifact.

For delegated variants:

- A parent creating a child eVault may complete creation without producing the child's certificate.
- An organization eVault may have no subject certificate at all.

## Provider Interaction

### Outbound Interactions

- `CreateEvaultRequest`
- `BindInitialCertificateRequest`

### Inbound Interactions

- `CreateEvaultResponse`
- `BindInitialCertificateResponse`
- `ProtocolError`

### Verification Duties

- Verify provider response signature
- Verify correlation with the active provisioning session
- Verify that returned eName is stable across provider responses
- Verify signed receipt before marking the flow successful

## Local Data Stored by Wallet

| Data | Purpose |
|---|---|
| Allocated eName | Persistent identity reference |
| Key reference | Points to secure private key storage |
| Public key metadata | Supports local validation and display |
| Provider identifier | Records chosen hosting provider |
| Provisioning receipt | Audit and troubleshooting |
| Local lifecycle status | Tracks whether provisioning succeeded |
| Subject key state | Records whether subject key is bound, deferred, or absent |

## Failure Handling

- If allocation fails, wallet must not generate user-facing success state.
- If key generation succeeds but certificate binding fails, wallet should retain or discard the key according to local policy, but must not present the eVault as active.
- If receipt verification fails, wallet should surface an integrity warning and require explicit recovery or retry behavior.

## Security Notes

- eName must not be treated as a secret.
- The wallet must not accept unsigned or unverifiable provider success responses.
- The wallet should prefer hardware-backed key storage where available.
- Recovery setup is outside this flow and must not be simulated as complete if it has not been explicitly configured.
- The wallet must distinguish "created for another subject" from "self-controlled by current wallet holder".
