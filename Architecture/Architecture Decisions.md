---
status: proposed
tags: [architecture, decisions]
---

# Architecture Decisions

## Related notes

- [[Home]]
- [[Architecture/Zero-Footprint and OPFS|Zero-Footprint and OPFS]]
- [[Architecture/Caching and Prefetching|Caching and Prefetching]]
- [[Specification/Key Management|Key Management]]
- [[Specification/Threat Model|Threat Model]]
- [[Research/Security Tests|Security Tests]]
- [[diagrams/README|Diagrams]]

## Status

Proposed architecture baseline for dissertation work. These are design decisions and assumptions, not completed implementation or proof of compliance.

## Scope

Frontend target: `dicoogle-next/webapp` with Cornerstone. Backend target: `dicoogle-next`. Dicoogle V3 is comparison/reference only.

Existing Dicoogle login/token handling is reused. Study-level permission checks, cache key service, and cache controls are proposed.

The experimental OPFS/frame branch is implementation context and inspiration, not mandatory code to merge and not proof that all controls are already complete.

## Decisions

| Decision | Rationale | Status / verification |
|---|---|---|
| Native diagnostic frame bytes are the cache unit | Preserve diagnostic representation; `/dic2png` is rendered preview | Validate transfer syntax, frame extraction, and metadata association |
| Bounded plaintext native-frame RAM and bounded Cornerstone decoded cache are coordinated | Avoid duplicate long-lived decoded-pixel stores | Measure retained copies and coordinated eviction |
| Shared OPFS stores authenticated ciphertext, not plaintext | Reuse storage while requiring authorised key access | Offline/profile inspection and cross-user tests |
| Dicoogle keeps recoverable scoped DEKs under backend KEK protection | Later authorised user may reuse ciphertext | Proposed trusted backend custody; not E2EE against backend |
| Study/key authorisation precedes reuse | Cache presence does not prove permission | Online key-access baseline; lease semantics unresolved |
| Validate expected context and AEAD before decode | Prevent substitution and malformed metadata processing | Reject candidate on mismatch/failure |
| Nonce policy spans writes, retries, tabs, and sessions | GCM nonce reuse breaks confidentiality/integrity | Selection unresolved; must be specified/tested |
| Freshness is independent of AEAD | Old valid ciphertext can be replayed | Trusted current source/key version check required |
| Cooperative locking coordinates publication across tabs | Avoid partial entries and inconsistent indexes | Coordination is not authorisation |
| Logout performs session cleanup, not global cryptographic erasure | Backend may later grant User B key access | Revocation blocks new access only |

## Shared-cache data flow

```text
User/session + requested study
  → Dicoogle permission and version check
  → scoped DEK available in active browser state

RAM hit
  → current-session plaintext

OPFS hit
  → validate expected scope/version
  → verify AEAD
  → plaintext native frame

Network miss
  → authorised native frame retrieval
  → display through bounded RAM/decoder/Cornerstone
  → optionally encrypt and publish OPFS entry in background
```

Cold display need not wait for persistent write.

## Boundaries and non-claims

Shared means same origin and browser profile/storage partition, not cross-device/profile sharing. Browser cleanup is not proof of complete RAM/swap/GPU/media sanitisation. This architecture does not claim strict physical zero-footprint, universal clinical timing thresholds, or automatic GDPR compliance.
