# Project Requirements

## Related notes

- [[Dissertation Home]]
- [[Performance KPI]]
- [[Browser Storage Threat Model]]
- [[AES-GCM Key Lifecycle]]
- [[Security Tests]]
- [[Diagrams/README]]

## Status

2026-10-12 draft. Architecture and sharing scope confirmed; requirements are proposed acceptance objectives, not completed implementation or passed tests. Clinician timing tolerance and the precise eight-image/500 ms event remain unvalidated.

## Scope

Main design: shared encrypted OPFS plus bounded native-frame RAM and coordinated Cornerstone decoded cache. Use `dicoogle-next/webapp` and Next backend; V3 is reference only. Test branch is inspiration, not mandatory code to merge.

Reuse is between successive authorised application sessions on the same machine/origin/browser profile/storage partition. No cross-device/profile shared cache or offline key acquisition is promised. Multi-tab tests assess consistency. Trust Dicoogle with study access decisions and protected DEK custody; do not claim backend-independent E2EE.

RAM-only remains a comparison. Persistent ciphertext is deliberate and relaxes strict physical zero-footprint. Security checks must not be bypassed for speed.

## Functional requirements

| ID | Proposed requirement | Verification |
|---|---|---|
| FR-01 | Display/navigation in Next webapp with Cornerstone | Integration test |
| FR-02 | Retrieve native selected frames with necessary metadata; retain separate full-instance baseline | Fixture/representation tests |
| FR-03 | Preserve pixel meaning, frame identity/order, orientation and transforms | Reference pixel/metadata comparison; PNG previews excluded from diagnostic cache path |
| FR-04 | Bound native-frame RAM and decoded Cornerstone cache with explicit ownership | Occupancy/copy/eviction measurements |
| FR-05 | Support sequential/reverse scrolling, pause, revisit, jump and series switching | Reproducible viewing tasks |
| FR-06 | Reuse encrypted cache after fresh User B study/key check | S03/S04 |
| FR-07 | OPFS/quota failure falls back to RAM/network, not plaintext persistence | S15 |
| FR-08 | Prioritise visible demand over bounded prefetch | Queue and latency traces |

## Security requirements

| ID | Proposed requirement | Verification |
|---|---|---|
| SEC-01 | Encrypt payloads and identifying index content before persistence | Storage-operation audit and S02 |
| SEC-02 | Validate expected context and AEAD before decode | S05/S06; reject invalid candidate |
| SEC-03 | Define nonce policy covering writes, retries, sessions, tabs and DEK reuse | S08; policy/use bounds must be selected |
| SEC-04 | No intentionally persisted browser unwrapped DEKs; protect server DEKs/KEK separately | Key/storage/log audit |
| SEC-05 | Enforce study/key permissions and reject missing/default key states | S04/S12 and malformed-key tests |
| SEC-06 | Validate current source/key/cache format independently of AEAD | S07; freshness authority required |
| SEC-07 | Publish consistent complete entries/index across cooperating tabs | S08/S09 |
| SEC-08 | Cancel work and discard old session-generation results on end/switch | S10/S11 across fetch/decrypt/decode/write/render |
| SEC-09 | Audit metadata/frame/preview HTTP cache, script, origin and embedding policy | S01/S13 and deployment audit |
| SEC-10 | State contributor/source provenance limits of shared-key AEAD | TM-07; backend-authenticated content-origin mechanism is a separate design choice |

## Privacy and compliance requirements

| ID | Proposed requirement | Verification |
|---|---|---|
| PRIV-01 | Minimise plaintext copies/prefetch and define retention | Occupancy/TTL and timed exposure tests |
| PRIV-02 | Minimise identifying filenames, headers, URLs and exports | Profile/metadata/log scan |
| PRIV-03 | Do not equate encrypted personal data with anonymity or cleanup with sanitisation | Review claims against primary GDPR/EDPB guidance |
| PRIV-04 | Use synthetic/de-identified studies and controlled memory evidence | Dataset/evidence-access/retention records |

## Performance and clinician-experience requirements

| ID | Proposed requirement | Verification / threshold status |
|---|---|---|
| PERF-01 | Measure first correct render and requested visible-set completion T_visible(N) | Eight-image/500 ms project target is provisional, not per-frame or clinical standard |
| PERF-02 | Measure input-to-correct-frame latency, stalls and output cadence | Median/p95; threshold from clinical evaluation where feasible |
| PERF-03 | Stratify by study/task rather than modality label alone | Anatomy, representation, frame dimensions, action and active viewport count |
| PERF-04 | Separately measure user-rated acceptable delay for opening, scrolling, jumping and switching | Controlled clinician tasks, observations/interviews and perceived-lag ratings; no numeric limit assumed |
| PERF-05 | Use cine/FPS papers only as technical workload evidence | Do not label 1000/FPS an acceptable user-interface latency |
| PERF-06 | Time key/network/crypto/decode/render/write stages without assuming independent execution | End-to-end tracing |
| PERF-07 | Optional cache population must not unnecessarily block cold display | Background-write and visible-demand measurements |

## Evaluation requirements

| ID | Proposed requirement | Verification |
|---|---|---|
| EVAL-01 | Compare P0-P4b with fixed datasets, representation, decoder, Cornerstone and network | Preset manifest |
| EVAL-02 | Distinguish cold/RAM-warm/OPFS-warm/cross-session/restart states | Reset/repeated-trial protocol |
| EVAL-03 | Record hardware/browser/OS/codecs and study descriptors | Reproduction manifest; Windows excluded from current test scope |
| EVAL-04 | Trace requirements to threats/tests and report residual risks | Result matrix |
| EVAL-05 | Assess representative clinician tasks where feasible; record participant modality expertise | Supervisor/ethics-approved small pilot; do not claim a universal diagnostic threshold |
| EVAL-06 | If clinicians unavailable, report measured latency and workload support, not proven clinical acceptability | Explicit evaluation limitation |

## Evidence

Radiology user requirements: Moise & Atkins DOI `10.1007/s10278-004-1003-9`. Real clinical usability: Jorritsma et al. DOI `10.1016/j.ijmedinf.2015.11.001`. Context-specific radiology UX: Iannessi et al. DOI `10.1007/s13244-018-0620-7`. Technical workload sources and their limits: [[Performance KPI]].

Security rationale: Nextcloud DOI `10.1109/EUROSP60621.2024.00051`; encrypted storage failures DOI `10.1145/3658644.3690309`; formal storage DOI `10.1007/978-3-031-68379-4_2`; browser keys/memory DOI `10.1016/j.fsidi.2024.301763` and `10.1016/j.fsidi.2024.301809`. No paper proves OPFS always faster/necessary or establishes universal modality-specific acceptable delays.

## Open decisions

DEK/KEK scope/protection, nonce strategy, policy lease, version authority, manifest bootstrap, contributor trust, retention, concurrency, compressed fixtures, target browsers, exact KPI, participant access and user-acceptability protocol.
