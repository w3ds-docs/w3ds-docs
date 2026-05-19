---
title: Decentralised Registry
weight: 41
bookCollapseSection: true
---

# Decentralised Registry

This section proposes two candidate architectures for replacing the single
centralised [Registry](/docs/Infrastructure/Registry) with a decentralised
design, and compares them in concrete protocol terms.

> **In plain terms**
>
> The Registry's job is to answer two questions about any user: where is their
> [eVault](/docs/Infrastructure/eVault), and which key proves their identity. A
> client supplies an eName and gets those answers back. Today one organisation
> runs this service on its own infrastructure. If that organisation's servers
> fail, if it alters an entry, or if it refuses to answer a lookup, every user
> and platform is affected at once. Decentralised means the same service is run
> by many independent organisations at the same time, so no single one of them
> can break it, alter an entry, or block a lookup on its own.

It is split across four pages:

1. **Overview** (this page): the problem, design goals, and the shared eName
   record used by both solutions.
2. [Solution 1: Federated DHT](federated-dht): independently operated registry
   nodes over a distributed hash table, with worked examples.
3. [Solution 2: Ledger-anchored](ledger-anchored): eName records on an
   append-only verifiable ledger, with worked examples.
4. [Comparison and migration](comparison): a side-by-side comparison and a
   staged rollout plan.

## Background

The Registry today is one service that every client trusts. After dropping the
entropy endpoint, it performs three functions:

1. **Service discovery**: `GET /resolve` and `GET /list` map a
   [W3ID](/docs/W3DS%20Basics/W3ID) or eName to an eVault or platform endpoint,
   backed by a vault-entry database.
2. **Verification key discovery**: `GET /.well-known/jwks.json` publishes the
   public keys used to verify Registry-issued key binding certificates.
3. **Key binding certificates**: JWTs that bind an eName to a public key. These
   are a temporary shortcut and are intended to move to a Remote Notary later.

> **Note on entropy**
>
> The previous `GET /entropy` endpoint is removed from scope. Provisioning
> flows that needed entropy should generate it locally in the
> [eID Wallet](/docs/Infrastructure/eID-Wallet) using a platform CSPRNG.
> Neither solution issues entropy, and the JWKS exists only to verify key
> binding certificates.

A single Registry is a [single point of failure](https://en.wikipedia.org/wiki/Single_point_of_failure),
a single point of trust, and a single point of censorship. One operator can
withhold a resolution, forge a key binding, or disappear and break every eName
at once. That contradicts the decentralised goals of W3DS.

Put simply, the three functions above answer three questions: "where do I find
this user?" (service discovery), "is this signature really theirs?" (key
binding), and "which key proves that?" (verification key discovery). The rest
of this section keeps those three jobs but spreads the responsibility for them
across many parties.

## Design goals

Any decentralised replacement must preserve:

- **Persistent eName resolution** that survives the loss of any single
  operator.
- **Verifiable key binding** so platforms can trust an eName to public key
  mapping without trusting one issuer.
- **No unilateral forgery or withholding**: no single operator can fabricate a
  record or silently censor one.
- **Migration support** so an eName can move between eVaults using
  also-known-as redirect records.
- **Backward compatible reads**: existing clients call `GET /resolve`,
  `GET /list`, and `GET /.well-known/jwks.json` and should continue to work
  against a gateway, even if the data layer underneath changes.

Solution 1 favours availability and low latency. Solution 2 favours
verifiability and a tamper-evident history.

## The shared eName record

Both solutions store the same logical object, the **eName record**. The schema
below is reused by both designs.

> **In plain terms**
>
> An eName record is the stored information for one identity. It records who
> the entry is for (`ename`), where to reach them (`uri` and `evault`), which
> key proves their identity (`keyBinding`), and a
> [digital signature](https://en.wikipedia.org/wiki/Digital_signature) from the
> owner covering all of it (`proof`). A digital signature can only be produced
> by the owner of the key, but anyone can check it. Because each record is
> signed by its own owner, whichever organisations store the records cannot
> alter one without the change being detectable. The most they can do is
> refuse to show a record.

```json
{
  "ename": "@e4d909c2-5d2f-4a7d-9473-b34b6c0f1a5a",
  "version": 7,
  "uri": "https://evault.example.com/users/user-a",
  "evault": "evault-identifier",
  "alsoKnownAs": ["@old-uuid-of-previous-evault"],
  "keyBinding": {
    "publicKey": "zDnaerx9Cp5X2chPZ8n3wK7mN9pQrS7tUvW1...",
    "alg": "ES256",
    "rotatedAt": 1737730800
  },
  "updatedAt": 1737730800,
  "proof": {
    "type": "ecdsa-2019",
    "created": 1737730800,
    "verificationMethod": "@e4d909c2-...#key-1",
    "signature": "z3FXQj..."
  }
}
```

Key rules that hold in both designs:

- `version` is a monotonically increasing counter. A record update is accepted
  only if its `version` is exactly one greater than the current record.
- `proof` is a self-signature by the eName owner, made with
  [public-key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography).
  The owner signs the canonical serialization of every field except `proof`.
  The record is valid only if `proof` verifies against the
  `keyBinding.publicKey` of the **previous** version, or against the genesis
  key for `version` 1. In other words, each new entry has to be signed by
  whoever was the rightful owner just before it, so the entries form an
  unbroken chain of custody.
- Key rotation is a normal record update: the new record carries the new
  `publicKey` and is signed by the old key. This chains keys forward and is the
  same mechanism the `w3id` ID log already uses.
- `alsoKnownAs` carries redirect history so a resolver can follow an eName that
  has migrated eVaults.

### Worked example: building a record

Below is the canonical signing input for a `version` 1 registration. The owner
generates a P-256 key pair in the eID Wallet, fills in the record, and signs
everything except `proof`.

```text
Signing input (canonical JSON, proof field excluded):
{"alsoKnownAs":[],"ename":"@e4d909c2-5d2f-4a7d-9473-b34b6c0f1a5a",
 "evault":"evault-001","keyBinding":{"alg":"ES256",
 "publicKey":"zDnaerx9Cp5X2chPZ8n3wK7mN9pQrS7tUvW1...","rotatedAt":1737730800},
 "updatedAt":1737730800,"uri":"https://evault.example.com/users/user-a",
 "version":1}

ECDSA P-256 / SHA-256 signature over that input -> proof.signature
```

A later key rotation produces `version` 2. The new record carries the new
public key, but `proof` is still signed with the **version 1** key, which is
what proves the rotation was authorised by the previous holder.

Because every record is self-signed and version-chained, neither a registry
node nor a ledger validator can forge or silently rewrite a record. They can
only choose whether to serve it. The two solutions differ in how they prevent
withholding and how they agree on which version is current. Continue to
[Solution 1](federated-dht).

## References

- [Registry](/docs/Infrastructure/Registry) for the current centralised design.
- [W3ID](/docs/W3DS%20Basics/W3ID) for identifier format, the ID log, key
  rotation, and also-known-as migration records.
- [eVault](/docs/Infrastructure/eVault) for how resolution and key binding are
  consumed in access control and webhook delivery.
- [Signing](/docs/W3DS%20Protocol/Signing) for how platforms verify signatures
  using resolved keys.
