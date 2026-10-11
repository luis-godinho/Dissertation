# Dissertation Home

## Topic

Privacy-Preserving Caching Architectures for Zero-Footprint Medical Image Viewers

## Programme

MCS Cybersecurity Master’s dissertation, academic year 2026/27.

## Core question

Can a browser-based medical-image viewer achieve the required responsiveness for large DICOM studies using bounded RAM caching, tiling, and prefetching without intentionally persisting patient data to local storage?

## Secondary question

If RAM-only caching is insufficient, can session-bound encrypted OPFS caching improve performance while preventing unauthorized recovery of persistent patient data?

## Current architecture

```text
Dicoogle/PACS
  → DICOM request
  → Web Worker
  → RAM sliding-window cache
  → Cornerstone3D
  → renderer
```

## Alternative architecture

```text
Dicoogle/PACS
  → Web Worker
  → AES-GCM
  → encrypted OPFS
  → decrypt
  → RAM working set
  → Cornerstone3D
```

## Current decision

The primary architecture is RAM-only caching with tiling, prefetching, and eviction. Encrypted OPFS is an alternative/fallback to evaluate if RAM-only operation cannot meet the performance requirements.

OPFS supports a zero-installation viewer, but it does not satisfy strict physical zero-footprint by default because it is persistent browser storage.

## Main notes

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

## Research layers

### Literature

- [[Literature - Cache Prefetch Security]]
- [[OHIF and Zero-Footprint Viewers]]
- [[Cornerstone3D Architecture]]

### Architecture

- [[Zero-Footprint and OPFS]]
- [[Prefetching RAM OPFS]]
- [[DICOMweb and Dicoogle]]

### Security

- [[Browser Storage Threat Model]]
- [[AES-GCM Key Lifecycle]]
- [[GPU Memory Residue]]
- [[Security Tests]]

### Experiments

- [[Performance KPI]]
- [[Cache Benchmark]]

## Current backend context

The current Dicoogle frontend uses a UID-based file URL similar to:

```text
/legacy/file?uid=<SOPInstanceUID>
```

The observed response is:

```http
Content-Type: application/dicom
Transfer-Encoding: chunked
Content-Disposition: attachment; filename=...
Accept-Ranges: bytes
```

This appears to return a complete DICOM instance. The sliding-window design is straightforward if each image has a separate SOP Instance UID. If one SOP Instance contains many frames, frame-level retrieval must be investigated.

See [[DICOMweb and Dicoogle]].

## KPI

Initial target from the dissertation proposal:

> Open 8 mammography tomosynthesis images in less than 500 ms.

The exact event still needs to be confirmed:

- First visible render.
- All eight images decoded.
- All eight images rendered.
- Viewer interactive.

See [[Performance KPI]].

## Immediate tasks

1. Confirm the exact Dicoogle image retrieval path.
2. Determine whether the study uses one SOP Instance per image or multiframe objects.
3. Measure the current viewer without an application cache.
4. Implement a bounded RAM cache.
5. Add small-window prefetching.
6. Measure cache hit ratio and navigation latency.
7. Test RAM pressure with large studies.
8. Investigate Dicoogle frame-level retrieval if required.
9. Add encrypted OPFS only as a comparison or fallback.
10. Add persistent-storage and cleanup security tests.

## Vault structure

```text
Dissertation/
├── Dissertation Home.md
├── 00 Overview/
├── 01 Literature/
├── 02 Architecture/
├── 03 Threat Model/
└── 04 Experiments/
```

The current files can remain in one folder initially. Move them into folders only after the links work correctly.
