---
title: Authority Validation Service
weight: 3
---

# Authority Validation Service

## Purpose

The Authority Validation Service verifies whether a trusted controller is authorized to perform a requested action on behalf of a subject.

It is responsible for:

- retrieving authority documents or document references
- building candidate authority chains
- validating document signatures
- validating document time ranges
- evaluating granted powers against requested actions
- checking continuity and consistency of the authority chain
- returning a signed authorization decision

## Scope

This component applies to authority verification for:

- dependent human subjects
- organization subjects
- non-human subjects

It is typically invoked before allowing actions such as:

- adding keys
- replacing keys
- revoking keys
- signing documents on behalf of a subject
- approving governance changes
- initiating recovery

## Responsibilities

- Accept authority verification requests from requesting actors
- Interpret the request in the context of a specific subject, trusted controller, and action set
- Retrieve relevant authority documents from subject and controller eVaults or referenced stores
- Construct one or more candidate authority chains
- Verify signatures on all documents in the evaluated chain
- Verify validity periods of documents
- Extract granted powers from authority documents
- Decide whether the requested action is within scope
- Produce signed decisions and error responses
- Expose machine-readable reasons for denial or indeterminate outcomes

## Non-Responsibilities

- It does not execute the protected action itself
- It does not create or edit authority documents
- It does not replace governance policy with local ad hoc logic
- It does not override explicit denial from a valid authority chain

## Inputs

### Minimum Request Input

The requesting actor should provide:

- subject identifier
- trusted controller identifier
- requested action or action set
- optional seed document or reference if already known

### Supporting Inputs

The service may also need:

- access to subject eVault
- access to trusted controller eVault when relevant
- policy version to evaluate against
- decision time

## Outputs

### Success Output

A signed authority decision containing:

- subject
- trusted controller
- requested actions
- final decision
- authorized and denied actions where relevant
- chain summary
- reason codes
- policy version
- decision validity window

### Failure Output

A signed error response or an `indeterminate` / `pending_manual_review` decision.

## Internal Structure

| Internal Part | Purpose |
|---|---|
| Request Parser | Validates incoming authority verification requests |
| Subject Resolver | Locates subject eVault and related evidence sources |
| Evidence Retriever | Retrieves relevant authority documents and references |
| Chain Builder | Builds candidate authority chains |
| Signature Verifier | Verifies document signatures |
| Validity Evaluator | Checks `validFrom`, `validUntil`, and evaluation-time applicability |
| Scope Evaluator | Maps granted powers to requested actions |
| Identity Consistency Checker | Confirms subject and trusted-controller identity alignment across the chain |
| Policy Engine | Applies policy version and validation rules |
| Decision Builder | Produces final decision payload |
| Response Signer | Signs decisions and error responses |

## Authority Documents

The service should support documents such as:

- guardianship documents
- parenthood evidence
- power-of-attorney
- corporate authorization documents
- charter-derived authority grants
- controller assignment documents
- delegated authority documents

Each document used for decision-making should be evaluated for:

- signature validity
- issuer correctness
- temporal validity
- subject consistency
- trusted-controller consistency
- granted powers

## Authority Chain Construction

The service should build authority chains itself whenever possible.

### Principle

The requesting actor should not be required to assemble the full authority chain.

### Chain-Building Expectations

- start from the subject and trusted controller identifiers
- retrieve all relevant authority documents or references
- construct one or more candidate chains
- evaluate each candidate chain against signature, time, scope, and consistency rules
- select the best valid chain if one exists

## Decision Model

| Decision | Meaning |
|---|---|
| `authorized` | The trusted controller is currently authorized to perform the requested action |
| `denied` | The trusted controller is not authorized to perform the requested action |
| `indeterminate` | The service cannot determine authorization conclusively |
| `pending_manual_review` | Automated validation is insufficient and manual review is required |

## Security Considerations

- The service must not treat relationship claims as sufficient proof without valid authority documents.
- The service must reject invalid signatures in the selected authority chain.
- The service must evaluate documents relative to the decision time.
- The service must not authorize actions outside the granted powers.
- The service should sign all decisions and important error responses.
- Cached decisions, if supported, must respect document expiry and policy invalidation rules.

## Trust Model

The Authority Validation Service is trusted to:

- retrieve the correct evidence set
- apply the configured policy consistently
- verify signatures and time validity correctly
- produce auditable signed decisions

It is not trusted to:

- invent missing authority
- extend scope beyond what documents grant
- ignore invalid or expired documents

## Interfaces

| Interface | Consumer | Purpose |
|---|---|---|
| `AuthorityVerificationRequest` | Provider APIs, governance services, recovery services | Ask whether a trusted controller may perform one or more actions |
| `AuthorityVerificationDecision` | Provider APIs, governance services, recovery services | Return signed decision |
| `AuthorityVerificationError` | Provider APIs, governance services, recovery services | Return structured signed failure |

## Dependencies

- subject eVault data access
- optional trusted controller eVault data access
- signature verification infrastructure
- policy configuration
- audit logging
- secure signing key for decision responses

## Operational Notes

- Manual review may be required for complex legal or organizational chains.
- Decision TTL should be bounded and policy-aware.
- Policy versioning should be visible in every decision so consuming systems can reason about compatibility and revalidation.
