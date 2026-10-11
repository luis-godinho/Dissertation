# Security Tests

## Related notes

- [[Architecture Decisions 2026-10-11]]
- [[Browser Storage Threat Model]]
- [[AES-GCM Key Lifecycle]]
- [[Performance KPI]]
- [[Cache Benchmark]]
- [[Diagrams/README]]

## Status

Planned evaluation protocol. These are acceptance objectives, not passed tests. Use synthetic/de-identified studies; control access to memory dumps and delete research evidence under a retention policy.

## Experiment presets

| Preset | Difference | Purpose |
|---|---|---|
| P0 | Full-instance legacy path and current viewer behaviour | Historical reference; separate retrieval-granularity comparison |
| P1 | Native frames, no additional reusable frame cache | Frame-network baseline; rendering still uses RAM/Cornerstone |
| P2 | Native frames plus bounded plaintext RAM | Proposal-compliant volatile caching |
| P3 | Same RAM plus plaintext OPFS | Insecure performance control; synthetic data only |
| P4a | Same RAM plus session-only encrypted OPFS | Personal/session-lifetime comparison; no reuse after losing sole key |
| P4b | Same RAM plus shared encrypted OPFS and Dicoogle key access | Cross-user warm reuse under trusted backend |

Keep frame representation, datasets, transfer syntax, decoder/Cornerstone policy, navigation traces and network conditions fixed. Measure prefetch separately. Quantify backend permission/key setup, encryption, OPFS I/O, decode and render separately.

## Security cases

| ID | Scenario | Expected result / evidence |
|---|---|---|
| S01 | Inspect P2 persistent storage | No intentional DICOM pixel/identifier writes; examine browser HTTP cache and previews too |
| S02 | Inspect P4b OPFS and exports | Ciphertext/protected metadata; no plaintext frames or unwrapped keys |
| S03 | User A populates, logs out, authorised B opens same study | Reuse ciphertext after fresh B permission/key check; no reuse of A plaintext state |
| S04 | Unauthorised user requests study DEK | Denied new access; candidate ciphertext not decrypted/rendered |
| S05 | Corrupt ciphertext/tag | Authentication failure; no decoder receives candidate bytes |
| S06 | Swap entries or modify frame/study/key identity | Expected-context check fails or AEAD rejects |
| S07 | Replay stale source/key version | Reject/refetch under explicit freshness policy |
| S08 | Rewrite/retry concurrently under one DEK | No key/nonce reuse; consistent publication |
| S09 | Interrupt entry or index write | Partial entry not treated as valid hit; recover/refetch |
| S10 | Logout during delayed fetch/decode/write | Cancel and reject old-generation completions; no plaintext repopulation |
| S11 | Study switch, timeout, reload, browser restart/crash | Apply documented cleanup/key-access policy; do not rely on guaranteed unload handling |
| S12 | Revoke study access | Deny new key release; measure policy-lease window; do not claim recall of copied DEKs |
| S13 | Cross-origin embed/opener/resource requests | Test intended framing/resource policy against protected endpoints |
| S14 | Inspect runtime buffers after controlled workflow | State/time-labelled exposure results; do not equate no finding with absence of all residue |
| S15 | OPFS unavailable/quota exceeded/cache lost | Controlled RAM/network fallback; no plaintext persistence fallback |

## Metrics

Report median/p95 latency, transferred bytes, network/RAM/OPFS hits, cache occupancy, evictions, crypto and decode/render time, cleanup outcome, integrity failures and cross-user warm-cache benefit. Record no patient names, raw frames, keys or unnecessary identifiers.

The 500 ms KPI requires a defined end-to-end event. A server-only transfer measurement does not establish that eight diagnostic images are decoded/rendered and interactive within the target.

## Evidence and limits

GE Venue supports clinical-state/timed acquisition (DOI `10.1016/j.fsidi.2026.302121`); browser encryption benchmarks support separating latency/throughput but used 1 kB Local Storage values (DOI `10.1109/SmartCloud55982.2022.00037`). Nextcloud attacks support nonce/input-validation cases (DOI `10.1109/EUROSP60621.2024.00051`). XSinator supports cross-origin test design, not automatic proof that a website is secure (DOI `10.1145/3460120.3484739`).

Searching for DICM/PNG magic alone is not sufficient: native frame bytes may contain neither. Compare known synthetic pixel patterns, metadata/identifiers, instrumented storage operations and authenticated decryption expectations. A plain hash and absence of readable strings do not prove encryption or key secrecy.
