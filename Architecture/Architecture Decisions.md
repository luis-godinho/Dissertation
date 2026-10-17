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
- [[Research/Reading List|Reading List]]
- [[diagrams/README|Diagrams]]

## Status

Proposed architecture baseline for dissertation work. These are design decisions and assumptions, not completed implementation or proof of compliance. The working design is shared encrypted OPFS plus bounded RAM; unresolved implementation choices are listed separately below.

## Scope

Frontend target: `dicoogle-next/webapp` with Cornerstone. Backend target: `dicoogle-next`. Dicoogle V3 is comparison/reference only.

Existing Dicoogle login/token handling is reused. Study-level permission checks, cache key service, and cache controls are proposed. No external identity provider or per-user asymmetric key infrastructure is assumed.

The experimental OPFS/frame branch is implementation context and inspiration, not mandatory code to merge and not proof that all controls are already complete.

## Working architecture

Shared encrypted OPFS is the proposed persistent tier, not merely a fallback after RAM-only fails. A bounded plaintext native-frame RAM cache serves active demand and prefetching. Decoder Worker/WASM memory and the bounded Cornerstone decoded-image cache are separate parts of the working set and must be measured together.

```text
Authorised native-frame network retrieval or authenticated OPFS hit
  → bounded plaintext native-frame RAM
  → decoder Worker/WASM
  → bounded Cornerstone decoded-image cache
  → WebGL rendering

Reusable native-frame bytes
  → authenticated encryption
  → shared OPFS ciphertext
```

RAM-only and session-only encrypted OPFS remain comparison modes. Their performance relative to shared persistence is an experimental question.

## Decisions

| Decision | Rationale | Status / verification |
|---|---|---|
| Native diagnostic frame bytes are the cache unit | Preserve diagnostic representation; `/dic2png` is rendered preview | Validate transfer syntax, frame extraction, and metadata association |
| Bounded plaintext native-frame RAM and bounded Cornerstone decoded cache are coordinated | Avoid duplicate long-lived decoded-pixel stores | Measure retained copies, decoder memory, and coordinated eviction |
| Shared OPFS stores authenticated ciphertext, not plaintext | Reuse storage while requiring authorised key access | Proposed persistent tier; offline/profile inspection and cross-user tests |
| Dicoogle keeps recoverable scoped DEKs under backend KEK protection | Later authorised user may reuse ciphertext | Proposed trusted backend custody; not E2EE against backend |
| Study/key authorisation precedes reuse | Cache presence does not prove permission | Online key-access baseline; lease semantics unresolved |
| Validate expected context and AEAD before decode | Prevent substitution and malformed metadata processing | Reject candidate on mismatch/failure; derive expected context from request/trusted backend |
| Nonce policy spans writes, retries, tabs, workers, users, and sessions | GCM nonce reuse breaks confidentiality/integrity | Random policy with collision/key-use bounds versus coordinated allocation remains unresolved |
| Freshness is independent of AEAD | Old valid ciphertext can be replayed | Trusted current source/key version check required |
| Cooperative locking coordinates publication across tabs | Avoid partial entries and inconsistent indexes | Coordination is not authorisation; crash-safe publication must be tested |
| Logout performs session cleanup, not global cryptographic erasure | Backend may later grant User B key access | Invalidate session generation, cancel tasks, and remove plaintext/key references; no physical-zeroisation claim |
| Unwrapped browser DEKs are not intentionally persisted | Keep local persistent cache from containing its own usable decryption key | Inspect OPFS, IndexedDB, localStorage, sessionStorage, URLs, logs, and exports |
| Security checks remain enabled during performance measurements | Measure the proposed protected system, not an insecure shortcut | Record authorisation, freshness, AEAD, and key-access costs |

## Shared-cache data flow

```text
User/session + requested study
  → Dicoogle permission and version check
  → backend KEK unwraps scoped stored DEK
  → DEK + trusted cache descriptor delivered over HTTPS
  → scoped key available in active browser state

RAM hit
  → valid current-session plaintext
  → decoder/Cornerstone/rendering

OPFS hit
  → validate expected scope/source/key/cache-format version
  → verify AEAD with expected associated data
  → plaintext native frame
  → bounded RAM/decoder/Cornerstone/rendering

Network miss
  → authorised native frame retrieval
  → display through bounded RAM/decoder/Cornerstone
  → optionally encrypt and publish OPFS entry in background

Study close / logout / timeout
  → invalidate session generation and cancel outstanding work
  → remove study plaintext and active key references
  → retain shared ciphertext under retention policy

Later authorised User B
  → new permission/version check and authorised key access
  → reuse compatible shared ciphertext
```

Cold display need not wait for persistent write. A cached entry is a candidate, not proof of permission, integrity, or freshness.

## Comparison presets

- P1: native-frame network-only baseline; runtime/Cornerstone decoded memory still exists for rendering.
- P2: bounded RAM baseline.
- P3: plaintext OPFS control, using synthetic data only.
- P4a: session-only encrypted OPFS.
- P4b: shared encrypted OPFS plus bounded RAM, the proposed architecture.

Evaluate demand-first scheduling, prefetching, cold/warm access, reload, logout, browser restart, and later authorised reuse. Do not assume persistence is always faster.

## Boundaries and non-claims

Shared means same origin and browser profile/storage partition, not cross-device/profile sharing. Dicoogle is trusted with recoverable DEKs; the design is not E2EE against the backend. Session-only encryption and backend-recoverable shared-cache encryption are different modes.

Browser cleanup is not proof of complete RAM/swap/GPU/media sanitisation. Shared ciphertext can remain legitimately recoverable after User A logs out. Revocation prevents new authorised key access but cannot make a user forget copied keys or viewed plaintext. KEK rotation alone does not revoke an already disclosed DEK.

AEAD authenticates an entry against expected context but does not alone prevent replay or identify which shared-key holder wrote it. AAD is authenticated, not encrypted; identifying values must not be exposed unnecessarily. A non-extractable browser key can still be used by malicious same-origin code.

This architecture does not claim strict physical zero-footprint, universal clinical timing thresholds, or automatic GDPR compliance.

## Evidence and traceability

These are existing repository references, not a claim that the implementation is complete or that every publication has been independently revalidated here.

| Reference | Decision traced |
|---|---|
| [[Architecture/Caching and Prefetching|Caching and Prefetching]] | Shared encrypted OPFS plus bounded RAM; native-frame pipeline; P1–P4b comparison presets |
| [[Specification/Key Management|Key Management]] | Backend KEK/DEK custody, trusted context, nonce policy, cleanup versus erasure, and unresolved key choices |
| [[Specification/Threat Model|Threat Model]] | Threat boundaries and exclusions to retain during implementation |
| [[Research/Security Tests|Security Tests]] | Validation plan; passing results must be reported separately |
| Share with Care; DOI `10.1109/EUROSP60621.2024.00051` | Existing key-management evidence reference |
| Formal encrypted cloud storage; DOI `10.1007/978-3-031-68379-4_2` | Existing key-management evidence reference |
| Puncturable key wrapping; DOI `10.1007/978-3-031-22966-4_22` | Existing reference for stronger deletion as future work, not logout behaviour |
| NIST SP 800-38D; NIST SP 800-57 Part 1 Rev. 5; W3C Web Cryptography API | Existing cryptographic and key-lifecycle implementation references |

## Open implementation decisions

- DEK scope, key-registry schema, and backend KEK deployment protection.
- Trusted source-version authority, key versioning, and migration/invalidation policy.
- Random nonces with documented collision/key-use limits versus coordinated nonce allocation.
- Encrypted-manifest format and bootstrap, metadata minimisation, and crash-safe publication.
- Permission/key lease duration, multiple active users/tabs, and session-expiry coordination.
- Retention, quota/eviction behaviour, supported browsers, and OPFS-unavailable fallback.
- Whether any offline mode is required; it is not assumed by the online key-access baseline.
- Exact workload-specific performance KPI and measurement procedure.

Whether shared OPFS is the primary proposed tier is not an open question in this baseline. These remaining choices concern how to implement and evaluate it.
