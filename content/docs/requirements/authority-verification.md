---
title: Authority Verification
weight: 4
---

# Requirements: Authority Verification

## Scope

These requirements define the TRL7 implementation baseline for verifying whether a trusted controller is authorized to perform a requested action on behalf of a subject.

This applies to:

- dependent human subjects
- organization subjects
- non-human subjects

The normative use case is [Verify Trusted Controller Authority](/docs/use-cases/verify-trusted-controller-authority/). The normative protocol is [Authority Verification Protocol](/docs/protocols/authority-verification/).

## Functional Requirements

| ID | Classification | Requirement |
|---|---|---|
| FR-AV-1 | MUST | The system must support authority verification for dependent human subjects, organization subjects, and non-human subjects. |
| FR-AV-2 | MUST | The system must accept a verification request containing at least the subject, trusted controller, and requested action or action set. |
| FR-AV-3 | SHOULD | The requester should be allowed to provide an optional initial authority document or reference as a hint. |
| FR-AV-4 | MUST | The Authority Validation Service must retrieve relevant authority documents from the subject eVault and, where applicable, the trusted controller eVault. |
| FR-AV-5 | MUST | The Authority Validation Service must build one or more candidate authority chains from retrieved documents. |
| FR-AV-6 | MUST | The Authority Validation Service must verify the signatures on all documents used in the selected authority chain. |
| FR-AV-7 | MUST | The Authority Validation Service must verify the validity period of all documents used in evaluation. |
| FR-AV-8 | MUST | The Authority Validation Service must extract granted powers from the evaluated documents. |
| FR-AV-9 | MUST | The system must verify whether the requested action is included in the granted powers. |
| FR-AV-10 | MUST | The system must verify that the authority chain is complete and consistent from a valid source to the trusted controller. |
| FR-AV-11 | MUST | The system must return a signed authority decision. |
| FR-AV-12 | MUST | The authority decision must identify the subject, trusted controller, requested action or actions, and final result. |
| FR-AV-13 | SHOULD | The authority decision should include a summary of documents used in evaluation. |
| FR-AV-14 | MUST | The system must support the decision outcomes `authorized`, `denied`, `indeterminate`, and `pending_manual_review`. |
| FR-AV-15 | MUST | The system must support partial authorization when multiple actions are evaluated in one request. |
| FR-AV-16 | MUST | The system must bind the authority decision to a policy version. |
| FR-AV-17 | SHOULD | The authority decision should include a validity period or TTL. |

## Security Requirements

| ID | Classification | Requirement |
|---|---|---|
| SR-AV-1 | MUST | The system must not authorize an action unless the authority chain is valid for that action at evaluation time. |
| SR-AV-2 | MUST | The system must reject or deny any chain containing invalid or unverifiable document signatures. |
| SR-AV-3 | MUST | The system must reject or deny any chain containing expired or not-yet-valid required documents. |
| SR-AV-4 | MUST | The system must not infer action scope from relationship alone when the requested action is not explicitly or normatively granted. |
| SR-AV-5 | MUST | The Authority Validation Service must treat requester-supplied seed documents only as hints, not as sufficient proof by themselves. |
| SR-AV-6 | MUST | Signed authority decisions must include correlation data and timestamps to support auditability and replay resistance. |
| SR-AV-7 | SHOULD | Request messages should also be signed when the deployment model requires non-repudiation or stronger caller accountability. |
| SR-AV-8 | MUST | The requesting system must not treat `indeterminate` or `pending_manual_review` as authorization success. |
| SR-AV-9 | MUST | The authority decision must be bound to the evaluation time and policy version used. |
| SR-AV-10 | SHOULD | The system should support evidence retention or decision retention sufficient for later audit and dispute resolution. |

## Non-Functional Requirements

| ID | Classification | Requirement |
|---|---|---|
| NFR-AV-1 | MUST | The system must produce deterministic results for the same document set, policy version, and evaluation time. |
| NFR-AV-2 | SHOULD | The system should support policy-driven manual review when automated validation is insufficient. |
| NFR-AV-3 | SHOULD | The system should provide machine-readable reason codes in authority decisions and errors. |
| NFR-AV-4 | SHOULD | The system should support normalized action naming to avoid ambiguous scope interpretation. |
| NFR-AV-5 | SHOULD | The system should support auditable logging of document retrieval, chain construction, and decision generation steps. |
| NFR-AV-6 | MAY | The system may cache decisions for a bounded TTL if invalidation rules are clearly defined. |
| NFR-AV-7 | SHOULD | Cached decisions should be invalidated when relevant authority documents change, expire, or are revoked. |

## Required Decision Semantics

| Decision | Requirement |
|---|---|
| `authorized` | Must mean that the requested action is currently within the trusted controller's validated powers. |
| `denied` | Must mean that the requested action is not authorized under the evaluated document set and policy. |
| `indeterminate` | Must mean that the system cannot determine authorization conclusively. |
| `pending_manual_review` | Must mean that automated processing has deferred the case to manual validation. |

## Required Validation Dimensions

| Dimension | Requirement |
|---|---|
| Signature validity | MUST be checked for every document used in the selected chain. |
| Time validity | MUST be checked relative to the evaluation time. |
| Scope validity | MUST be checked for each requested action. |
| Chain continuity | MUST be checked for missing or unsupported links. |
| Identity consistency | MUST be checked for subject and trusted-controller identity alignment. |

## Failure Handling

- If no relevant documents are found, the decision must be `denied` or `indeterminate` according to policy.
- If document retrieval fails due to temporary service issues, the decision should be `indeterminate`.
- If manual review is required by policy, the decision must be `pending_manual_review`.
- The system must not allow protected actions to proceed on the basis of incomplete or inconclusive validation.

## Acceptance Criteria

- A developer can implement the validation service and consuming systems directly from these requirements and the protocol.
- An architect can verify that authority is action-specific and chain-based.
- A reviewer can confirm that invalid signatures, expired documents, broken chains, and out-of-scope actions do not result in false authorization.
