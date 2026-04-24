---
title: Glossary
weight: 70
---

# Glossary

Definitions of key terms used across the W3DS and MetaState documentation.

---

## Access

The ability to retrieve or interact with data or services based on permissions and authentication. In W3DS, access to eVault data is governed by ACLs and resolution of identities.

---

## Authentication

The process of verifying the identity of a user or identifier. In W3DS, users authenticate using their W3ID via the `w3ds://auth` protocol.

---

## Authorized Agent

A user or software that has been authorized by another user to act on their behalf. Examples: Bob allows Alice to act on his behalf to file taxes; Bob authorizes a social network app to perform certain operations on his behalf, which may require the app to sign for him.

---

## Charter

A cryptographically signed document outlining group governance, rules, and regulations. Groups in MetaState can have charters that define how the group operates and who can perform which actions.

---

## Credential

A set of data relating to an identifier that is signed by an issuing party (e.g. a school diploma). See Verifiable Credential (VC).

---

## eID (ePassport)

A document, similar to X.509, which binds a user's W3ID and the user's Public Key. It is signed by a digital notary participating in PKI.

---

## eIDAS 2.0

An updated European Union regulation aimed at enhancing the security and reliability of electronic identification and trust services.

---

## Entity

A non-human object within the MetaState such as an organization, a platform, a business, a Group, a device, etc. that has its own eVault and can hold or verify credentials.

---

## Envelope

The smallest unit of data in an eVault, addressable by its unique identifier and ontology reference. Each envelope has an attached ontological definition and ACL that defines who is allowed to access it.

---

## eVault

A secure storage location or server for the management of data and credentials of a User, Post-Platform, or Group. In W3DS, each user or group has their own eVault identified by a W3ID.

---

## Group

An Entity: a reference to a number of users within the MetaState that holds its own W3ID and eVault. It is often seen as a "group" in social networks.

---

## Identity

The continuity of a subject (a person, group, organization, service, or thing) across time and contexts. An identity is what makes a subject recognizable and distinct across different moments and services.

In W3DS, identity is carried by the subject itself, not held by any service. It is realized through a persistent identifier (the W3ID), the cryptographic keys bound to it, and the body of data, credentials, and relationships anchored to that identifier.

An account at a service is not an identity; it is a service-local record that references one.

---

## Interoperability Profile

A set of technical standards and protocols that enable different systems to work together seamlessly. W3DS uses a shared ontology and Web 3 Protocol as part of its interoperability profile.

---

## Key Rotation

The practice of changing cryptographic keys to enhance security on a regular basis or in an emergency (e.g. when an eID or key is revoked and a new one must be issued).

---

## MetaState

The name of the current social-oriented project that uses the Web 3.0 Data Space (W3DS) architecture. It uses eVaults and unique identifiers, allowing Post-Platforms to access data from a user's eVault and interact with it based on authorization and permissions.

---

## MetaState Prototype

The current project to deliver the Prototype of the MetaState at TRL6, with the minimum required features and developments to produce the required functionality.

---

## MetaEnvelope

To group related pieces of data, Envelopes are linked together using a MetaEnvelope. Each MetaEnvelope has its own ID and points to the individual envelopes it includes. This makes it easy to bundle related info, track changes over time, and move or reference sets of data as one unit. It also helps fix duplicates by pointing everything to the correct version of an entity.

---

## Ontology Dictionary

A service that maps keys as defined in a Post-Platform to the schema definition of how data is referenced when stored in an eVault. Example: a post platform might refer to a name as `"firstName"` but it is stored as `"given_name"` in the vault; the Web3 Adapter stores the reference so that when the platform refers to `"firstName"`, it means write/update/get `"given_name"` on the vault.

---

## Physical Binding Documents

Signed documents by the Root CA in the system that state that an eName belongs to a user who owns a certain physical identity document.

---

## Post-Platform (and/or Service)

A service or platform that can operate with eVaults in the W3DS (dataless) mode. Post-platforms sync data with eVaults via the Web3 Adapter and Awareness Protocol.

---

## Private Key

A cryptographic key kept secret (e.g. in the crypto-engine of the phone) by its owner, used for digital signatures and decryption.

---

## Provisioner

Software that creates, controls, and updates eVaults. In the prototype, it is the "cloud service provider" software that creates eVaults.

---

## Public Key

A cryptographic key derived from the corresponding Private Key. It can be freely shared in the form of an e-passport (public key certificate) and is used for encryption and signature verification.

---

## Registry

A Post-Platform service for discovering, storing, and retrieving information about identifiers and their eVaults/IP. In W3DS, the Registry resolves eNames to eVault URLs and provides entropy and key binding certificates.

---

## Resolution

The process of retrieving and verifying keys or identifiers using their specific syntax to query the corresponding network or Registry.

---

## Search Platform

A platform that provides search/discovery service to other platforms that need to find certain users, groups, files, etc.

---

## Selective Disclosure

The ability to share only specific parts of data from within a Credential or information set.

---

## User

An individual who owns and controls their digital identity and eVault, interacting within the MetaState via any service of their choice.

---

## User Flow

The sequence of steps a user takes to complete a task or interaction within a system.

---

## User Journey

The overall experience and path a user follows when interacting within an interactive MetaState experience.

---

## Verification

The process of confirming the authenticity and validity of an identity, Credential, action, or claim. In W3DS, platforms verify signatures using public keys resolved via the Registry and eVault.

---

## Verifiable Credential (VC)

A standardized data model format for credentials that can be cryptographically signed and verified.

---

## W3DS

**Web 3.0 Data Space**: the basic architecture scheme used in the MetaState Prototype. It is a protocol that enables data synchronization across multiple platforms while ensuring users own and control their data in eVaults.

---

## Web 3.0 Identifier (W3ID / eName)

A globally or locally unique, persistent identifier for people, eVaults, Groups, Post-Platforms, docs, or triples. It enables verifiable, persistent, and decentralized digital identity.

The Web 3.0 Identifier format is `@<UUID>` for global identifiers (eNames), e.g. `@50e8400-e29b-41d4-a716-446655440000`. Local identifiers are plain UUIDs.

---

## Web3 Adapter

Software that runs next to a Post-Platform and is responsible for syncing the platform's DB with all relevant eVaults. It maps local schema to the global ontology and handles Awareness Protocol webhooks.

---

## Web 3 Protocol

The query language that a Web3 Adapter (or a Post-Platform without a Web3 Adapter) uses to talk to an eVault. In the prototype, this is implemented as the eVault GraphQL API.

---

## Web Triad

A **Digital Self** triad: (1) eName (W3ID), (2) ePassport (eID), (3) eVault.
