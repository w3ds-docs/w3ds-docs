---
title: Glossary
weight: 1
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

## Authority Validation Service

An external service that validates whether a trusted controller has authority to perform a requested action for a subject by checking document signatures, validity periods, granted powers, and chain continuity.

---

## Authorized Agent

A user or software that has been authorized by another user to act on their behalf. Examples: Bob allows Alice to act on his behalf to file taxes; Bob authorizes a social network app to perform certain operations on his behalf, which may require the app to sign for him.

---

## Charter

A cryptographically signed document outlining group governance, rules, and regulations. Groups in MetaState can have charters that define how the group operates and who can perform which actions.

---

## Control Model

The explicit model that defines how an eVault is governed, whether the subject has key material, who may authorize structural actions, and how recovery is expected to work. See [eVault Control Models](/docs/architecture/evault-control-models/).

---

## Credential

A set of data relating to an identifier that is signed by an issuing party (e.g. a school diploma). See Verifiable Credential (VC).

---

## Delegation Chain

A sequence of signed documents that directly or indirectly proves that a trusted controller has authority to act for a subject.

---

## Dependent Human Subject

A human subject who does not yet control the eVault directly, such as a child or an incapacitated person.

---

## eID (ePassport)

A document, similar to X.509, which binds a user's W3ID and the user's Public Key. It is signed by a digital notary participating in PKI.

---

## eIDAS 2.0

An updated European Union regulation aimed at enhancing the security and reliability of electronic identification and trust services.

---

## Entity

A subject or object within MetaState or W3DS, such as a user, organization, platform, group, device, or service.

---

## Envelope

The smallest unit of data in an eVault, addressable by its unique identifier and ontology reference. Each envelope has an attached ontological definition and ACL that defines who is allowed to access it.

---

## eVault

A secure storage location or server for the management of data and credentials of a subject. In W3DS, each human subject, organization subject, group, device, or service may have its own eVault identified by a W3ID.

---

## Governance Authority

The authority model that permits structural actions for an eVault, such as delegated control, organizational governance, or trusted-controller authority.

---

## Group

An organization subject that represents a number of users within MetaState or W3DS and may hold its own W3ID, eVault, and governance documents.

---

## Human Subject

A natural person represented by an eVault.

---

## Initiator

The authenticated actor who starts an eVault creation or lifecycle flow.

---

## Interoperability Profile

A set of technical standards and protocols that enable different systems to work together seamlessly. W3DS uses a shared ontology and Web 3 Protocol as part of its interoperability profile.

---

## Key Rotation

The practice of changing cryptographic keys to enhance security on a regular basis or in an emergency.

---

## MetaState

The name of the current social-oriented project that uses the Web 3.0 Data Space (W3DS) architecture. It uses eVaults and unique identifiers, allowing Post-Platforms to access data from a subject's eVault and interact with it based on authorization and permissions.

---

## MetaState Prototype

The current project to deliver the Prototype of the MetaState at TRL6, with the minimum required features and developments to produce the required functionality.

---

## MetaEnvelope

To group related pieces of data, Envelopes are linked together using a MetaEnvelope. Each MetaEnvelope has its own ID and points to the individual envelopes it includes. This makes it easy to bundle related info, track changes over time, and move or reference sets of data as one unit. It also helps fix duplicates by pointing everything to the correct version of an entity.

---

## Non-Human Subject

A device, application, wallet, service, or other non-human entity represented by an eVault.

---

## Ontology Dictionary

A service that maps keys as defined in a Post-Platform to the schema definition of how data is referenced when stored in an eVault.

---

## Organization Subject

A company, group, community, or other collective entity represented by an eVault.

---

## Physical Binding Documents

Signed documents by the Root CA in the system that state that an eName belongs to a user who owns a certain physical identity document.

---

## Post-Platform (and/or Service)

A service or platform that can operate with eVaults in the W3DS mode. Post-platforms sync data with eVaults via the Web3 Adapter and Awareness Protocol.

---

## Private Key

A cryptographic key kept secret by its owner, used for digital signatures and decryption.

---

## Provisioner

Software that creates, controls, and updates eVaults. In the current architecture this responsibility is realized by the provider-side provisioning service within the eVault management system.

---

## Provisioning Service

A provider-side component of the eVault management system that authenticates initiators, allocates `eName`, validates evidence, orchestrates eVault creation, and commits the chosen control model.

---

## Public Key

A cryptographic key derived from the corresponding Private Key. It can be freely shared in the form of a public key certificate and is used for encryption and signature verification.

---

## Registry

A W3DS service for discovering, storing, and retrieving information about identifiers and their eVaults or service endpoints. In W3DS, the Registry resolves eNames to eVault URLs and may provide other infrastructure services.

---

## Resolution

The process of retrieving and verifying keys or identifiers using their specific syntax to query the corresponding network or Registry.

---

## Search Platform

A platform that provides search and discovery services to other platforms that need to find users, groups, files, or other entities.

---

## Selective Disclosure

The ability to share only specific parts of data from within a Credential or information set.

---

## Subject

The entity represented by the eVault.

---

## Trusted Controller

A trusted person or trusted organization explicitly linked to a subject for governance or recovery purposes.

---

## User

An individual who owns and controls their digital identity and eVault, interacting within MetaState or W3DS via services of their choice.

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

**Web 3.0 Data Space**: the architecture used in MetaState and related systems. It enables interoperable, subject-controlled data and identity through eVaults, identifiers, protocols, and governance models.

---

## Web 3.0 Identifier (W3ID / eName)

A globally or locally unique, persistent identifier for subjects, eVaults, organizations, post-platforms, documents, or other entities. It enables verifiable, persistent, and decentralized digital identity.

The Web 3.0 Identifier format is `@<UUID>` for global identifiers (eNames), e.g. `@50e8400-e29b-41d4-a716-446655440000`. Local identifiers are plain UUIDs.

---

## Web3 Adapter

Software that runs next to a Post-Platform and is responsible for syncing the platform's database with relevant eVaults. It maps local schema to the global ontology and handles Awareness Protocol webhooks.

---

## Web 3 Protocol

The query language or interaction protocol that a Web3 Adapter or Post-Platform uses to talk to an eVault. In the prototype, this is implemented as the eVault GraphQL API.

---

## Web Triad

A **Digital Self** triad: (1) eName (W3ID), (2) ePassport (eID), (3) eVault.
