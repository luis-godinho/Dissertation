# Revised shared-cache architecture diagrams

These five diagrams revise the supplied Mermaid sources. They describe a proposed architecture, not completed repository code. Dicoogle login/token validation is existing; study-level permission enforcement, key-access service, protected key registry, native-frame caching, nonce coordination, and version checks must be designed and verified.

## Selected working key model

Use a trusted-Dicoogle cache-key service with durable, protected DEK custody keyed by repository/study/cache scope and key version. A server KEK protects stored DEKs. Authorised sessions receive the scoped DEK over HTTPS and import/use it only in volatile browser state. This protects local persistent ciphertext, not against Dicoogle itself. It is not end-to-end encryption against the server. Per-recipient public-key wrapping is a possible alternative, not assumed simultaneously implemented.

## Required design constraints

- Shared OPFS means same origin and browser profile/storage partition, not all devices or OS/browser profiles. Multiple app users may reuse ciphertext, never the prior user's plaintext or active key handle.
- Request current permission and expected frame/source/key context before encrypted-cache reuse. Choose an explicit online/policy-lease model; offline sharing is a separate requirement.
- Fresh nonce for every encryption under a DEK, including rewrites, retries, tabs and sessions. Define generation/collision bounds or coordinated allocation; do not derive a nonce solely from the frame number.
- AEAD authenticates expected frame/cache identity and format/key/source version. Derive expected AAD from trusted request/server context, not solely from a mutable OPFS header. Authentication is not freshness: independently check current versions or immutable content identity.
- Protect identifying manifest content with authenticated encryption. Minimise plaintext bootstrap fields. A plain SHA-256 hash is not an authenticity mechanism if attacker can replace both bytes and hash. Shared-key AEAD does not identify which authorised user created an entry.
- Treat cache entries as untrusted: authentication or structural failure means no decoding of candidate bytes; invalidate/refetch or report controlled failure. Never use default/all-zero keys.
- Publish entries/index changes consistently across tabs and workers. Web Locks can coordinate cooperating same-origin contexts; they are not access control against malicious same-origin code.
- Cold display does not need to wait for OPFS encryption/write. Persistence is optional background work bounded by the session generation; late results cannot repopulate plaintext after logout.
- Cleanup cancels requests/prefetch, invalidates the session generation, disposes viewport/decoded cache, clears plaintext references and resets managed workers. Tab-exit/crash handlers are not reliable sanitisation mechanisms. Browser/OS/GPU/hibernation residues remain risks.
- User A logout is session cleanup, not global cryptographic erasure, because the backend can give User B key access. Revocation prevents new access; it cannot recall copies of keys/plaintext already obtained. Strong shredding/puncturable wrapping is future work.
- Apply frame-embedding/script/origin protections and explicit sensitive-response cache policy to metadata, frames and previews, not only OPFS.
- Record de-identified timing, occupancy, hits, evictions and integrity failures without patient names, URLs/UIDs, raw frames or keys.

## Evidence behind the revisions

- Share with Care (2024), DOI 10.1109/EUROSP60621.2024.00051: key insertion, ghost key and GCM nonce reuse; secure primitives do not imply a secure protocol.
- A Broken Ecosystem (2024), DOI 10.1145/3658644.3690309: key authenticity, chunk/metadata/context binding and integrity failures.
- Formal E2EE Cloud Storage (2024), DOI 10.1007/978-3-031-68379-4_2: shared ciphertext/per-recipient key headers; selective proof and revocation/rollback scope limits.
- PKW (2022), DOI 10.1007/978-3-031-22966-4_22: cryptographic shredding needs stronger key-state mechanisms than dropping a session reference.
- Gecko storage (2024), DOI 10.1016/j.fsidi.2024.301763; MIC (2024), DOI 10.1016/j.fsidi.2024.301809; GE Venue (2026), DOI 10.1016/j.fsidi.2026.302121: active key/plaintext memory and workflow-dependent exposure; no universal post-exit recovery claim.
- FROST (2026): OPFS SSD contention; malicious origin measures its own files. Encryption and ordinary origin isolation are not complete side-channel mitigations.
- Pixel Thief (USENIX Security 2024): CPU SVG filter side channel under particular embedding/browser/hardware assumptions; not direct Cornerstone GPU-texture extraction.
- NIST SP 800-38D: AEAD/nonce requirements and replay limits.
- W3C Web Locks: same-origin browser-context coordination.

## Experimental fairness

P1-P4b keep decoder configuration, Cornerstone policy, frame representation, network conditions and navigation traces fixed; quantify key-access/setup separately from OPFS crypto/I/O. P1 is not literally memory-free: rendering and Cornerstone still hold transient decoded data. P4a cannot reuse ciphertext after losing its sole session key. P4b needs backend access to obtain a key in a fresh session; OPFS is not an offline-authorisation mechanism.

Add cross-user warm hits, denied key access, corrupt/swapped entries, stale key/source versions, multi-tab writes and interrupted writes to the privacy/performance protocol. ABE and puncturable wrapping remain design alternatives, not required extra browser modules.
