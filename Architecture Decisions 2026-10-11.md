# Architecture Decisions — 2026-10-11

## Related notes

- [[Dissertation Home]]
- [[Zero-Footprint and OPFS]]
- [[Prefetching RAM OPFS]]
- [[AES-GCM Key Lifecycle]]
- [[Browser Storage Threat Model]]
- [[Security Tests]]
- [[Diagrams/README]]

## Status

Proposed architecture baseline for pre-thesis discussion. These are design decisions and assumptions, not completed implementation or proof of compliance. RAM-only remains the proposal-compliant primary path; encrypted shared OPFS is a comparison/fallback candidate.

## Scope

Frontend: `dicoogle-next/webapp`, using Cornerstone. Target backend: `dicoogle-next`. V3 is comparison/reference only. Existing Dicoogle login/token validation is used; no external identity provider is assumed. Authentication redesign is outside the main contribution, but study/key authorisation must be defined for shared-cache use.

The experimental OPFS/frame branch is inspiration only. The repository investigation found working native uncompressed frame retrieval there, not a completed secure-cache architecture on dev.

## Decisions

| Decision | Rationale | Status / verification |
|---|---|---|
| Native frame bytes are the cache unit | Preserve diagnostic representation; `/dic2png` is a rendered preview | Validate transfer syntax, frame extraction and metadata association |
| Custom RAM caches pre-decode bytes; Cornerstone owns decoded objects | Avoid an unnecessary second decoded-pixel cache | Measure all retained copies and coordinated eviction |
| Shared OPFS stores ciphertext, not plaintext frames | Reuse storage without granting decryption merely by cache presence | Offline/profile inspection and cross-user tests |
| Dicoogle keeps recoverable DEKs in a protected registry | User B must recover the same key after User A logout | Proposed server custody; not E2EE against backend |
| Study/key authorisation precedes reuse | Cache presence does not establish current permission | Online key-access baseline; policy-lease/offline semantics unresolved |
| Validate entries before decode | Encryption alone does not prevent substitution or malformed metadata | AEAD plus expected identity/version checks; reject on failure |
| Explicit nonce policy spans writes, retries, users and tabs | GCM nonce reuse breaks confidentiality and integrity | Random policy with key-use bounds or coordinated allocation; unresolved selection |
| Authenticate sensitive manifests and minimise exposed headers | Plain hashes are not authenticity controls | Identity/context binding and metadata scans |
| Freshness is independent of AEAD | Old valid ciphertext can be replayed | Trusted current source/key version or immutable content identity |
| Coordinate writes/index changes across tabs | Avoid partial entries and inconsistent publication | Locks for cooperative contexts plus interrupted-write recovery |
| Cleanup cancels asynchronous work and invalidates session generation | Late results must not repopulate cleared plaintext | Delayed fetch/decode tests |
| Metrics are de-identified | Evaluation must not become another patient-data leak | No raw frames, keys, patient names or unnecessary UIDs/URLs |

## Shared-cache data flow

```text
User/session + requested study
  → Dicoogle permission and version check
  → scoped DEK available only in active browser state

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

Cold display need not wait for persistent encryption/write. Both branches are bounded and must reject late results from ended sessions.

## Boundaries and non-claims

Shared means the same origin and browser profile/storage partition, not all machines or profiles. Logout removes the user's active plaintext/key references but leaves a potentially reusable encrypted cache. It is not global cryptographic erasure if Dicoogle can release the key again. Revocation denies new key access; it cannot recall previously obtained keys/plaintext.

Application cleanup does not prove RAM, swap, crash-dump, GPU or SSD sanitisation. Non-extractable Web Crypto keys do not stop privileged same-origin code from invoking allowed operations. Workers are processing/lifecycle boundaries, not a complete security boundary against the host or trusted application scripts.

## Alternatives not selected for the initial prototype

- Per-recipient public-key wrapping: needs authentic recipient keys and a browser private-key provisioning/recovery design.
- Attribute-based encryption: useful healthcare policy alternative, but not required for the initial browser cache.
- Puncturable wrapping: stronger deletion/forward-security future work, not achieved by dropping a JavaScript reference.

## Evidence

| Source | Identifier | Supports / limits |
|---|---|---|
| Share with Care (2024) | DOI `10.1109/EUROSP60621.2024.00051` | Key insertion, ghost key and nonce reuse; authentic keys and strict input validation |
| A Broken Ecosystem (2024) | DOI `10.1145/3658644.3690309` | Object/metadata integrity, key authenticity and chunk binding |
| Formal E2EE Cloud Storage (2024) | DOI `10.1007/978-3-031-68379-4_2` | Shared ciphertext and user-specific key protection; selective proof; rollback/revocation excluded |
| Puncturable Key Wrapping (2022) | DOI `10.1007/978-3-031-22966-4_22` | Strong shredding requires key-state updates, not session cleanup alone |
| Gecko private storage (2024) | DOI `10.1016/j.fsidi.2024.301763` | Active-memory/hibernation key recovery; not universal post-exit recovery |
| MIC (2024) | DOI `10.1016/j.fsidi.2024.301809` | Structured live-memory residue; not Cornerstone/OPFS extraction |
| GE Venue study (2026) | DOI `10.1016/j.fsidi.2026.302121` | Clinical-state/time-labelled PHI/image exposure |
| Browser encryption performance (2022) | DOI `10.1109/SmartCloud55982.2022.00037` | Microbenchmark method; Local Storage/1 kB results do not transfer directly to OPFS |
| FROST (2026) | DIMVA; DOI not verified | OPFS SSD timing via attacker's own storage; no direct victim-file read |
| Pixel Thief (2024) | USENIX Security | CPU SVG-filter leakage under specific framing/browser/hardware conditions |
| Healthcare ABE comparison (2024) | DOI `10.1038/s41598-024-57692-w` | Alternative field-level EHR policy architecture, not DICOM-cache latency |
| Healthcare PbD review (2020) | DOI `10.3390/electronics9030452` | Minimisation/lifecycle/assessment rationale; not legal compliance proof |
| ESR GDPR statement (2017) | DOI `10.1007/s13244-017-0552-7` | Radiology context; use current GDPR/EDPB for specific obligations |

## Open decisions

Cache key scope; backend KEK custody; source-version authority; random versus allocated nonce scheme; encrypted manifest format; key lease duration; multi-tab session policy; retention; OPFS-unavailable fallback; supported browsers; exact KPI; whether shared persistence becomes a primary or fallback architecture.
