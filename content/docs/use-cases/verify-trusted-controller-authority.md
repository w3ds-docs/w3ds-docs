---
title: Verify Trusted Controller Authority
weight: 6
---

# Use Case: Verify Trusted Controller Authority

## Goal

Verify whether a `trusted controller` is authorized to represent a subject and perform a requested action on behalf of that subject.

This use case applies to:

- dependent human subjects
- organization subjects
- non-human subjects

The result of verification is used before permitting actions such as:

- changing keys
- adding keys
- signing documents on behalf of the subject
- performing other governance or recovery operations

## Actors

- Requesting actor
- eVault
- Authority Validation Service
- Trusted controller
- Subject

## Supporting Artifacts

The subject's eVault must contain, or reference, signed authority documents that prove the trusted controller's powers either directly or through a chain of delegation.

Examples:

- parenthood or guardianship documents for a dependent human subject
- charter, corporate authorization documents, and power-of-attorney for an organization subject
- ownership, assignment, or governance documents for a non-human subject

## Preconditions

- A subject eVault exists.
- The subject eVault and, where applicable, the trusted controller eVault contain or reference the authority documents relevant to the requested action.
- The requesting actor presents the identity of the subject, the identity of the trusted controller, and the requested action or action set.
- The Authority Validation Service is available to validate signatures, validity periods, and authority chains.

## Trigger

A system component needs to decide whether a trusted controller may perform a specific action on behalf of a subject.

## Minimum Input from the Requesting Actor

The requesting actor should provide only the minimum necessary input:

- who trusts: the subject
- whom the subject trusts: the trusted controller
- which action or action set is being requested
- optionally, an initial authority document or reference if the caller already has one

The requesting actor should not be responsible for building the full authority chain. That work belongs to the Authority Validation Service.

## Main Flow

1. A requesting actor initiates an authority verification request for a specific subject, trusted controller, and requested action.
2. The requesting actor sends the minimum input set to the Authority Validation Service.
3. The Authority Validation Service identifies the subject eVault and, where relevant, the trusted controller eVault.
4. The Authority Validation Service retrieves from those eVaults all documents and references that may be relevant to the requested action.
5. The Authority Validation Service builds one or more candidate authority chains from the subject to the trusted controller.
6. The Authority Validation Service validates the integrity of the retrieved documents or references.
7. The Authority Validation Service verifies the signatures on each document in each candidate chain.
8. The Authority Validation Service checks the validity period of each document.
9. The Authority Validation Service extracts the powers granted by each document.
10. The Authority Validation Service verifies that the requested action is included in the granted powers.
11. The Authority Validation Service verifies that the chain of authority is complete, consistent, and uninterrupted from a valid source to the trusted controller.
12. The Authority Validation Service selects the valid chain, if any, that authorizes the requested action.
13. The Authority Validation Service determines whether the trusted controller is authorized for the requested action at the current time.
14. The Authority Validation Service returns a signed decision with supporting evidence references, validity findings, and scope findings.
15. The requesting actor records or consumes the decision.
16. The system either permits or denies the requested action based on the returned decision.

## Authority Evaluation Rules

The Authority Validation Service must evaluate at least the following:

| Rule | Meaning |
|---|---|
| Signature validity | Every document in the chain must carry a valid signature from the person or authority that issued it |
| Time validity | Each document must be valid at the time of evaluation |
| Scope of powers | The requested action must be explicitly included in the granted powers or be derivable from a higher-level grant |
| Chain continuity | The authority chain must connect the subject to the trusted controller without unsupported gaps |
| Subject consistency | All documents in the chain must refer to the same subject or a valid transformation of subject identity |
| Controller identity consistency | The trusted controller being checked must match the controller identity described in the chain |

## Alternative Flows by Main Flow Step

### A-3a: No Relevant Authority Documents Found

Branch from step 3.

1. The Authority Validation Service queries the subject eVault and, where relevant, the trusted controller eVault for relevant authority documents.
2. No matching documents or references are found.
3. The verification request returns a denial result.

### A-7a: A Signature in the Chain Is Invalid

Branch from step 7.

1. The Authority Validation Service verifies the signature of one of the documents.
2. The signature is invalid, unverifiable, or bound to the wrong signer.
3. The chain is rejected.
4. The final decision is denial.

### A-8a: A Document Is Expired or Not Yet Valid

Branch from step 8.

1. The Authority Validation Service checks time validity.
2. One or more required documents are expired or not yet valid.
3. The chain is rejected unless another still-valid chain exists.

### A-10a: Requested Action Is Outside Granted Powers

Branch from step 10.

1. The Authority Validation Service extracts granted powers from the document chain.
2. The requested action is not included in the granted powers.
3. The decision is denial for that action even if the trusted controller may hold other powers.

### A-11a: Chain of Authority Is Incomplete

Branch from step 11.

1. The Authority Validation Service evaluates the authority chain.
2. The chain contains a missing delegation step, an unsupported signer, or a broken link to the subject.
3. The decision is denial.

### A-13a: Validation Service Cannot Reach a Final Decision

Branch from step 13.

1. The Authority Validation Service cannot determine the result conclusively because required documents are unavailable, retrieval fails, parsing fails, or validation policy requires manual review.
2. The service returns `indeterminate` or `pending_manual_review`.
3. The requesting actor must not treat the authority as confirmed.

## Postconditions

### Success

- The system has a signed verification decision for the requested action.
- The decision explicitly identifies:
  - the subject
  - the trusted controller
  - the requested action
  - the result
  - the evidence chain used for evaluation
- The action may proceed only if the decision is positive.

### Failure

- The system has a signed denial or indeterminate result.
- The requested action must not proceed unless a subsequent valid decision is obtained.

## Output Decision Model

| Decision | Meaning |
|---|---|
| `authorized` | The trusted controller is currently authorized to perform the requested action |
| `denied` | The trusted controller is not authorized to perform the requested action |
| `indeterminate` | The system cannot determine authorization conclusively |
| `pending_manual_review` | Automated checks are insufficient and manual validation is required |

## Notes

- Authority is action-specific. A trusted controller may be authorized to add keys but not to sign documents, or vice versa.
- A valid relationship to the subject is not sufficient on its own; the action must also be within scope.
- The same trusted controller may have different authority bases for different subjects.
- The Authority Validation Service should do as much of the retrieval and chain-construction work as possible so the requesting actor remains simple and less error-prone.

## Open Questions

- Should revocation lists or revocation registries be mandatory for authority documents?
- Should the Authority Validation Service return a normalized machine-readable proof graph?
- Which actions require direct grants, and which may be derived from broader governance powers?
