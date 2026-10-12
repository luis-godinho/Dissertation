# Dissertation Home

## Topic

Privacy-Preserving Caching Architectures for Zero-Footprint Medical Image Viewers

## Programme

MCS Cybersecurity Master’s dissertation, academic year 2026/27.

## Core question

Can encrypted persistent browser caching and a bounded plaintext RAM working set provide responsive medical-image viewing while protecting client-side patient data?

## Secondary question

How do performance, clinician-perceived responsiveness, integrity, key lifecycle and residual-data exposure differ between network-only, RAM-only, plaintext OPFS and encrypted OPFS modes?

## Current implementation context

The investigated main frontend uses full-instance `/legacy/file?uid=<SOPInstanceUID>` retrieval, Cornerstone wadouri loading/decoder workers, decoded-image caching and StackViewport/WebGL rendering. The experimental frame/OPFS branch is inspiration only; it is not automatically the target implementation. Native uncompressed frame retrieval worked there, while compressed-syntax support and end-to-end integration require validation. `/dic2png` supplies rendered PNG previews, not diagnostic native-frame bytes.

## Current decision — 2026-10-12

Confirmed by the project owner: shared encrypted OPFS plus bounded RAM is the main proposed architecture. RAM-only is an experimental comparison baseline, not the default design with persistence reserved only for failure. Whether persistence improves each workload remains an experimental question.

Sharing is between successive Dicoogle application sessions on the same machine, origin and browser profile/storage partition. Cross-device/profile sharing is excluded. Concurrent tabs are a consistency case, not a promise of simultaneous independent users.

```text
Trusted Dicoogle backend
  → current study permission/version check
  → protected cache DEK registry
  → authorised key access over HTTPS

Native frame or authenticated OPFS cache hit
  → bounded plaintext frame RAM
  → decoder Worker
  → controlled Cornerstone decoded cache
  → WebGL rendering

Persistent path
  → encrypt before write
  → shared OPFS ciphertext/protected index
```

Dicoogle is trusted with protected DEK custody; this is not E2EE against Dicoogle. Logout removes active plaintext/key references but is not global cryptographic erasure because later authorised users may recover key access.

Encrypted persistence relaxes the original strict no-disk interpretation. Use zero-installation plus privacy-preserving authenticated persistent caching terminology, not a physical zero-residue guarantee. The dated 2026-10-11 RAM-first/fallback wording is historical where it conflicts with [[Project Requirements]].

## Performance and usability

Original project target: open eight mammography/tomosynthesis images in less than 500 ms. Its operational meaning remains provisional; it is not a published clinical timing standard or 500 ms per frame.

Distinguish technical frame-demand/cine profiles from what clinicians perceive as acceptable delay. Evaluate study opening, sequential scroll, reverse, jump, series/study switch and multi-viewport changes with modality/task context. No unsupported CT/MRI/DBT acceptable-ms limits will be invented. See [[Performance KPI]] for evidence and DOI references.

## Main notes

- [[Project Requirements]]
- [[Performance KPI]]
- [[Browser Storage Threat Model]]
- [[AES-GCM Key Lifecycle]]
- [[Security Tests]]
- [[Architecture Decisions 2026-10-11]]
- [[Literature - Cache Prefetch Security]]
- [[Prefetching RAM OPFS]]
- [[Zero-Footprint and OPFS]]
- [[Diagrams/README]]
- [[DICOMweb and Dicoogle]]
- [[Cornerstone3D Architecture]]
- [[Cache Benchmark]]

Some linked notes remain planned. Existing numbered diagrams, literature tables, flat Markdown structure and `.obsidian` configuration remain unchanged.

## Immediate tasks

1. Review requirements/trust assumptions with supervisors.
2. Ratify eight-image display-unit meaning, timing event and conditions.
3. Select datasets/transfer syntaxes and clinical viewing tasks.
4. Plan clinician evaluation of perceived delay separately from paper-derived FPS workloads.
5. Instrument input-to-correct-render latency, visible-set readiness, stalls and frame identity.
6. Define key scope, nonce policy, freshness, retention and multi-tab publication.
7. Compare cold/warm and successive authorised-user cache reuse.
8. Validate tampering, stale entries, interrupted writes and late completion after logout.
