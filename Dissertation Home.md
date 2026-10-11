# Dissertation Home

## Topic

Privacy-Preserving Caching Architectures for Zero-Footprint Medical Image Viewers

## Programme

MCS Cybersecurity Master’s dissertation, academic year 2026/27.

## Core question

Can a browser-based medical-image viewer achieve the required responsiveness for large DICOM studies using bounded RAM caching, tiling, and prefetching without intentionally persisting patient data to local storage?

## Secondary question

Can encrypted OPFS caching improve performance while protecting persistent patient data and allowing ciphertext reuse by other users currently authorised for the same study?

## Current architecture

The verified main frontend path uses full-instance retrieval rather than the proposed frame-cache pipeline.

```text
dicoogle-next/webapp
  → /legacy/file?uid=<SOPInstanceUID>
  → Cornerstone wadouri loader and decoder workers
  → Cornerstone decoded-image cache
  → StackViewport / WebGL
```

Repository findings are described in [[Architecture Decisions 2026-10-11]]. They are investigation results, not proof that the proposed architecture is implemented.

## Alternative architecture

```text
Dicoogle-Next backend
  → current study permission and version check
  → protected study/cache DEK registry
  → authorised key access over HTTPS

Native frame retrieval
  → bounded plaintext RAM working set
  → decoder Worker
  → Cornerstone decoded-image cache
  → WebGL renderer

Optional persistent path
  → AES-GCM encryption before write
  → shared OPFS ciphertext
  → expected-context and authentication checks
  → decrypt into bounded RAM
```

## Current decision

The primary proposal-compliant path remains RAM-only caching with bounded prefetching and eviction. Shared encrypted OPFS is a candidate persistent comparison/fallback architecture. Adopting it as the primary architecture requires agreement that strict no-disk persistence is being relaxed to protected encrypted persistence.

The working shared-cache design trusts Dicoogle with protected DEK custody. Browser sessions retain active keys only in volatile state; the backend can grant the same cache key to a later authorised user. This is not end-to-end encryption against Dicoogle.

The latest shared-cache decisions are in [[Architecture Decisions 2026-10-11]] and [[AES-GCM Key Lifecycle]]. Earlier session-only-key examples in [[Zero-Footprint and OPFS]] and [[Prefetching RAM OPFS]] describe a personal/session-only mode. Their claim that logout permanently makes remaining ciphertext unreadable does not apply to the shared mode. Logout is session cleanup, not global cryptographic erasure.

OPFS supports zero-installation but not strict physical zero-footprint.

## Main notes

- [[Architecture Decisions 2026-10-11]]
- [[Literature - Cache Prefetch Security]]
- [[Prefetching RAM OPFS]]
- [[Zero-Footprint and OPFS]]
- [[DICOMweb and Dicoogle]]
- [[OHIF and Zero-Footprint Viewers]]
- [[Cornerstone3D Architecture]]
- [[Browser Storage Threat Model]]
- [[AES-GCM Key Lifecycle]]
- [[Performance KPI]]
- [[Cache Benchmark]]
- [[Security Tests]]

Some linked notes remain planned and have not yet been created.

## Research layers

### Literature

- [[Literature - Cache Prefetch Security]]
- [[OHIF and Zero-Footprint Viewers]]
- [[Cornerstone3D Architecture]]

### Architecture

- [[Architecture Decisions 2026-10-11]]
- [[Zero-Footprint and OPFS]]
- [[Prefetching RAM OPFS]]
- [[DICOMweb and Dicoogle]]
- [[Diagrams/README]]

### Security

- [[Browser Storage Threat Model]]
- [[AES-GCM Key Lifecycle]]
- [[GPU Memory Residue]]
- [[Security Tests]]

### Experiments

- [[Performance KPI]]
- [[Cache Benchmark]]
- [[Security Tests]]

## Current backend context

The frontend is always `dicoogle-next/webapp`, including compatibility comparisons with V3. The target backend is `dicoogle-next`; `dicoogle` V3 is a reference/comparison, not a replacement frontend.

The investigation found WADO-RS RetrieveFrames in the experimental `feat/frontend-frame-retrieval` branch/build, but not in the investigated `dev` source. The branch is inspiration and proof-of-concept evidence, not an automatic merge target. Native uncompressed frame tests were successful; compressed-syntax support and end-to-end viewer integration require separate validation.

`/dic2png` decodes DICOM and re-encodes a rendered PNG. It is a preview/thumbnail path, not a substitute for native diagnostic frame retrieval. HTTP chunked transfer does not imply DICOM frame streaming.

Transport/cache headers depend on backend and deployment; do not apply an observed V3 response contract universally to Next. The tested Next API used Spring no-store defaults, which still need explicit verification and policy.

## KPI

> Open 8 mammography tomosynthesis images in less than 500 ms.

Confirm the event before benchmarking: first visible render, all eight images decoded/rendered, or viewer interactive. Separate cold start, warm cache and cross-user warm cache. Server-only frame latency is not the end-to-end KPI.

See [[Performance KPI]] and [[Security Tests]].

## Immediate tasks

1. Review and agree the architecture/trust assumptions before implementation.
2. Confirm realistic datasets, frame layout and compressed transfer syntaxes.
3. Define the exact 500 ms event and experimental controls.
4. Measure full-instance and frame-network-only baselines.
5. Design bounded native-frame RAM and Cornerstone decoded-cache ownership.
6. Add bounded prefetch, eviction and session-generation cleanup.
7. Evaluate plaintext OPFS only as a synthetic-data performance control.
8. Evaluate session-only encrypted OPFS separately from shared encrypted OPFS.
9. Specify shared DEK custody, nonce policy, context/freshness checks and multi-tab writes.
10. Validate authorised User B reuse, denied key access, tampering, stale entries, interrupted writes and late completion after logout.

## Vault structure

Notes remain flat Markdown with Obsidian wikilinks; diagram sources are in `Diagrams/`. Do not reorganise the vault or `.obsidian` configuration as part of the decision update.
