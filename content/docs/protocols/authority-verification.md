---
title: Authority Verification Protocol
weight: 2
---

# Authority Verification Protocol

## Purpose

This protocol defines how a requesting actor asks the Authority Validation Service to determine whether a trusted controller is authorized to perform one or more actions on behalf of a subject.

The protocol is designed to minimize logic on the requesting side. The requesting actor provides a minimal input set, and the Authority Validation Service performs document retrieval, authority-chain construction, signature verification, validity checks, and scope evaluation.

## Scope

This protocol applies to authority verification for:

- dependent human subjects
- organization subjects
- non-human subjects

This protocol may be used before allowing actions such as:

- add key
- replace key
- revoke key
- sign document on behalf of subject
- approve governance change
- initiate recovery

## Related Documents

- [Verify Trusted Controller Authority](/docs/use-cases/verify-trusted-controller-authority/)
- [eVault Control Models](/docs/architecture/evault-control-models/)

## Roles

| Role | Description |
|---|---|
| Requesting Actor | A component or client that needs an authority decision |
| Authority Validation Service | External service that retrieves documents, builds chains, validates evidence, and returns a signed decision |
| Subject eVault | Source of authority documents or references for the subject |
| Trusted Controller eVault | Optional source of additional authority documents or references |

## Design Principle

The requesting actor should provide the minimum possible input:

- subject identifier
- trusted controller identifier
- requested action or action set
- optional first authority document or reference if already known

The Authority Validation Service should do the rest.

## Message Envelope

All messages should use a common envelope format.

```json
{
  "messageType": "string",
  "protocolVersion": "1.0",
  "correlationId": "uuid",
  "timestamp": "2026-04-24T09:15:00Z",
  "sender": {
    "type": "client|authority_validation_service",
    "id": "string"
  },
  "payload": {},
  "signature": {
    "algorithm": "ECDSA_P256_SHA256",
    "keyId": "string",
    "value": "base64url"
  }
}
```

## Common Field Requirements

| Field | Required | Notes |
|---|---|---|
| `messageType` | MUST | Defines message semantics |
| `protocolVersion` | MUST | Defines protocol compatibility |
| `correlationId` | MUST | Ties request and decision together |
| `timestamp` | MUST | Used for freshness and auditability |
| `signature` | SHOULD | Strongly recommended for both request and response |

## AuthorityVerificationRequest

Sent by a requesting actor to the Authority Validation Service.

```json
{
  "messageType": "AuthorityVerificationRequest",
  "protocolVersion": "1.0",
  "correlationId": "95db3b0e-f6c1-465d-87c4-39a9f8d5a268",
  "timestamp": "2026-04-24T09:15:00Z",
  "sender": {
    "type": "client",
    "id": "evault-provider-api-1"
  },
  "payload": {
    "subject": {
      "eName": "@11111111-1111-1111-1111-111111111111",
      "subjectType": "dependent_human|organization|non_human"
    },
    "trustedController": {
      "eName": "@22222222-2222-2222-2222-222222222222",
      "controllerType": "human|organization"
    },
    "requestedActions": [
      "add_key",
      "replace_key"
    ],
    "knownAuthoritySeed": {
      "documentId": "uuid-optional",
      "reference": "optional-reference",
      "digest": "base64url-optional"
    },
    "evaluationContext": {
      "decisionTime": "2026-04-24T09:15:00Z",
      "policyVersion": "authority-policy-v1"
    }
  },
  "signature": {
    "algorithm": "ECDSA_P256_SHA256",
    "keyId": "client-signing-key-1",
    "value": "base64url"
  }
}
```

## Request Semantics

### Required Inputs

- `subject.eName`
- `trustedController.eName`
- at least one entry in `requestedActions`

### Optional Inputs

- `knownAuthoritySeed`
  Used when the requester already knows one relevant authority document or reference. The service should treat this as a hint, not as the full chain.

### Action Names

Requested actions should use a normalized action vocabulary.

Example action names:

- `add_key`
- `replace_key`
- `revoke_key`
- `sign_document`
- `approve_governance_change`
- `initiate_recovery`

## AuthorityDocumentReference

Used by the Authority Validation Service to identify a document used in evaluation.

```json
{
  "documentId": "uuid",
  "source": {
    "eVault": "@11111111-1111-1111-1111-111111111111",
    "path": "/authority/chain/0001"
  },
  "digest": "base64url",
  "documentType": "guardianship|charter|power_of_attorney|delegation|controller_assignment",
  "validFrom": "2026-01-01T00:00:00Z",
  "validUntil": "2027-01-01T00:00:00Z"
}
```

## AuthorityVerificationDecision

Sent by the Authority Validation Service after evaluation.

```json
{
  "messageType": "AuthorityVerificationDecision",
  "protocolVersion": "1.0",
  "correlationId": "95db3b0e-f6c1-465d-87c4-39a9f8d5a268",
  "timestamp": "2026-04-24T09:15:02Z",
  "sender": {
    "type": "authority_validation_service",
    "id": "authority-service-1"
  },
  "payload": {
    "subject": {
      "eName": "@11111111-1111-1111-1111-111111111111"
    },
    "trustedController": {
      "eName": "@22222222-2222-2222-2222-222222222222"
    },
    "requestedActions": [
      "add_key",
      "replace_key"
    ],
    "decision": "authorized|denied|indeterminate|pending_manual_review",
    "authorizedActions": [
      "add_key"
    ],
    "deniedActions": [
      "replace_key"
    ],
    "policyVersion": "authority-policy-v1",
    "decisionValidUntil": "2026-04-24T09:25:02Z",
    "chainSummary": {
      "chainStatus": "valid|invalid|incomplete",
      "documentsUsed": [
        {
          "documentId": "uuid-1",
          "documentType": "guardianship",
          "signatureValid": true,
          "timeValid": true,
          "scopeValid": true
        },
        {
          "documentId": "uuid-2",
          "documentType": "delegation",
          "signatureValid": true,
          "timeValid": true,
          "scopeValid": false
        }
      ]
    },
    "reasons": [
      {
        "code": "ACTION_OUT_OF_SCOPE",
        "message": "replace_key is not included in the granted powers"
      }
    ]
  },
  "signature": {
    "algorithm": "ECDSA_P256_SHA256",
    "keyId": "authority-service-signing-key-1",
    "value": "base64url"
  }
}
```

## Decision Semantics

| Decision | Meaning |
|---|---|
| `authorized` | All requested actions are authorized |
| `denied` | None of the requested actions are authorized |
| `indeterminate` | The service cannot reach a reliable decision |
| `pending_manual_review` | Automated checks are insufficient and manual review is required |

## Partial Authorization

The service may return mixed results when multiple actions are evaluated in one request.

Example:

- `add_key` is authorized
- `replace_key` is denied

In such cases:

- `authorizedActions` must list granted actions
- `deniedActions` must list denied actions
- `decision` may still be `authorized` only if all requested actions are granted

## Service-Side Processing Model

The Authority Validation Service should perform the following internally:

1. Identify the subject eVault and, when relevant, the trusted controller eVault.
2. Retrieve all potentially relevant authority documents and references.
3. Build one or more candidate authority chains.
4. Verify document signatures.
5. Verify document time validity.
6. Extract granted powers.
7. Check requested action scope.
8. Check chain continuity and identity consistency.
9. Select the best valid chain, if any.
10. Return a signed decision.

## Error Response

```json
{
  "messageType": "AuthorityVerificationError",
  "protocolVersion": "1.0",
  "correlationId": "95db3b0e-f6c1-465d-87c4-39a9f8d5a268",
  "timestamp": "2026-04-24T09:15:01Z",
  "sender": {
    "type": "authority_validation_service",
    "id": "authority-service-1"
  },
  "payload": {
    "code": "DOCUMENT_RETRIEVAL_FAILED",
    "message": "Required authority documents could not be retrieved",
    "retryable": true
  },
  "signature": {
    "algorithm": "ECDSA_P256_SHA256",
    "keyId": "authority-service-signing-key-1",
    "value": "base64url"
  }
}
```

## Security Considerations

### Signature Integrity

- The service must verify document signatures before trusting any authority claim.
- The requesting actor should verify the signature on the returned decision.

### Time Validity

- The service must evaluate validity windows relative to `decisionTime`.
- A decision should not outlive the relevance of the underlying documents.

### Scope of Powers

- Authorization is action-specific.
- Broad relationship claims must not be interpreted as universal permission unless the policy explicitly allows it.

### Chain Construction

- The service should construct the chain itself whenever possible.
- Requester-supplied seed references are hints, not proof.

### Replay and Auditability

- Requests and decisions should carry correlation IDs and timestamps.
- Signed decisions should be auditable and retain the policy version used.

## Open Questions

- Should action vocabulary be globally standardized or deployment-specific?
- Should the decision include a normalized proof graph instead of a compact chain summary?
- Should the service cache decisions, and if so, under what invalidation rules?
