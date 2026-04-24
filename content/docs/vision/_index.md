---
title: Vision
weight: 10
---

# Vision

## 1. Purpose

The system aims to enable a new digital environment where data, identity, and interactions are not controlled by platforms but remain under the control of users and entities themselves.

It provides an infrastructure in which multiple independent services can operate on shared user-controlled data without creating silos, lock-in, or monopolistic control.

---

## 2. Core Idea

The core idea is the separation of:

- data
- identity
- services (platforms)

Users, organizations, and other entities maintain their own persistent data and identity, while services compete to provide functionality on top of that data.

---

## 3. Target State

The system should make it possible that:

- A user can use multiple services without duplicating data
- A user can switch services without losing history, identity, or data
- Services do not own user data and cannot lock users in
- Identity is persistent, portable, and independent of any single identity provider
- Interactions between users happen through shared data, not through isolated platforms

---

## 4. Key Properties

### Data Independence

Data is stored outside of services and remains accessible regardless of the specific application.

### Service Replaceability

Any service can be replaced without loss of data or identity.

### Interoperability by Design

Different services can work with the same data without direct integration between each other.

### Persistent Identity

Each entity has a long-lived identifier that is not tied to any platform.

### Verifiable Interactions

Any action involving another entity, such as one user sending a file to another, can be verified and traced when needed.

---

## 5. Scope of the System

The system is not a single application.

It is:

- an infrastructure layer
- a set of protocols and services
- a foundation for building multiple competing applications

It supports use cases such as:

- social interaction
- group coordination and governance
- document signing
- reputation and trust
- data sharing between services

---

## 6. What This System Is Not

The system is not:

- a single platform or super-app
- a centralized database
- a closed ecosystem
- a fixed set of services

It does not aim to replace all applications, but to change how they interact with data and users.

---

## 7. Design Direction

The system prioritizes:

- user control over data and identity
- freedom to choose and switch between services, and competition among them
- minimizing central points of control
- clear separation of responsibilities between components

Trade-offs may be made between strict decentralization and practical usability, but central control points must be explicit and justified.

---

## 8. Success Criteria

The system is considered successful if:

- users can freely switch between services without data loss
- multiple independent services can operate on the same user data
- no single service can unilaterally block or control user interactions
- identity and data remain usable even if individual services disappear
