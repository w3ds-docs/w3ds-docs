---
title: eVault Creation
weight: 1
---

# eVault Creation Architecture

## High-Level Explanation

The eVault creation flow is a two-phase initialization process. First, the Cloud Provider allocates an eName and an empty hosted eVault. Second, the eID Wallet generates the user's initial key pair, creates a self-signed certificate containing that eName, and binds the certificate to the hosted eVault. After successful binding, the provider activates the eVault and records that recovery is not yet configured.

Two important variants relax the second phase:

- a child eVault may be created with deferred subject key binding
- an organization eVault may exist without subject signature capability

For delegated and advanced variants, the provider runs an internal Provisioning Service inside the eVault management system. This internal service authenticates initiators, collects and validates authority evidence, allocates `eName`, and orchestrates the final control-model commitment.

## Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Wallet as eID Wallet
    participant Prov as Provisioning Service
    participant Evault as eVault
    participant Registry as Registry (optional)

    User->>Wallet: Start eVault creation
    Wallet->>Wallet: Display provider list
    User->>Wallet: Select provider
    Wallet->>Prov: CreateEvaultRequest (signed)
    Prov->>Prov: Validate request and allocate eName
    Prov->>Evault: Create hosted eVault
    Prov-->>Wallet: CreateEvaultResponse (signed, eName, pending_key_binding)
    Wallet->>Wallet: Verify provider response
    Wallet->>Wallet: Generate key pair
    Wallet->>Wallet: Create self-signed certificate with eName
    Wallet->>Prov: BindInitialCertificateRequest (signed)
    Prov->>Prov: Validate request and certificate
    Prov->>Evault: Store certificate
    Prov->>Evault: Mark active + recovery_not_configured
    opt Registry publication enabled
        Prov->>Registry: RegistryHostingUpdate
    end
    Prov-->>Wallet: BindInitialCertificateResponse (signed receipt)
    Wallet->>Wallet: Verify receipt and persist local identity mapping
    Wallet-->>User: eVault created successfully
```

## Activity Diagram

```mermaid
flowchart TD
    A[User starts flow in wallet] --> B[Wallet loads provider list]
    B --> C[User selects provider]
    C --> D[Wallet sends signed create request]
    D --> E{Provisioning service validates request?}
    E -- No --> F[Return signed error]
    E -- Yes --> G[Provisioning service allocates eName and empty eVault]
    G --> H[Provider returns signed allocation response]
    H --> I{Wallet verifies response?}
    I -- No --> J[Stop and show integrity error]
    I -- Yes --> K[Wallet generates key pair]
    K --> L[Wallet creates self-signed certificate with eName]
    L --> M[Wallet sends signed certificate binding request]
    M --> N{Provisioning service validates certificate and session binding?}
    N -- No --> O[Keep eVault pending_key_binding and return error]
    N -- Yes --> P[Provisioning service stores certificate]
    P --> Q[Provisioning service marks eVault active]
    Q --> R[Provisioning service marks recovery_not_configured]
    R --> S{Registry publication enabled?}
    S -- Yes --> T[Publish or queue hosting update]
    S -- No --> U[Skip Registry step]
    T --> V[Return signed initialization receipt]
    U --> V
    V --> W{Wallet verifies receipt?}
    W -- No --> X[Show verification warning and require follow-up]
    W -- Yes --> Y[Persist local eName-key association]
    Y --> Z[Show success to user]
```

## Data Flow

| Step | Data Produced | Producer | Consumer |
|---|---|---|---|
| Provider selection | Provider identifier | Wallet UI | Wallet orchestrator |
| Create request | Signed provisioning request, correlationId, timestamp, nonce | Wallet or delegated client | Provisioning Service |
| Allocation response | eName, eVault reference, pending state, provider nonce, signed response | Provisioning Service | Wallet or delegated client |
| Key generation | Private key, public key | Wallet | Wallet secure storage / certificate builder |
| Certificate binding request | eName, certificate, signed request, correlation data | Wallet or bootstrap environment | Provisioning Service |
| Initialization commit | Stored certificate, evidence state, hosting metadata, active state | Provisioning Service | eVault |
| Optional publication | Hosting reference | Provisioning Service | Registry |
| Final receipt | Signed confirmation of activation | Provisioning Service | Wallet or delegated client |

## Variant-Specific Architectural Notes

### Child eVault Created by Parent

- The initiator and the dependent human subject are different actors.
- Allocation may complete before the child has wallet-managed keys.
- The architecture must preserve a visible difference between "eVault exists" and "child controls it".
- The architecture should allow parenthood or guardianship evidence to be attached at creation time.

### Organization eVault

- The subject may not be representable by a single signing key.
- The architecture must not assume every eVault can self-sign.
- Authorization must rely on an explicit governance model.
- The architecture should support validation of authorization chains and corporate evidence packages.

### Key-Owning Non-Human Entity

- The subject owns operational keys and may sign or authenticate on its own behalf.
- Recovery and higher-level governance are linked to a trusted controller.
- The architecture must preserve both subject operational autonomy and trusted-controller authority.

## Architectural Constraints

- Identity allocation and certificate binding must remain distinct operations.
- `eName` generation should occur inside the provider-side provisioning service.
- The wallet must not generate the certificate against an assumed eName.
- The provider must not activate the eVault before certificate persistence succeeds.
- Recovery setup must remain a separate lifecycle flow.
- Signed receipts are strongly recommended to support verifiable completion.
