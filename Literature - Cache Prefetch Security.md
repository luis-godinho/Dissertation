# Literature Notes: Caching, Prefetching, and Security

## Related notes

- [[Zero-Footprint and OPFS]] — definitions of zero-footprint and implications of persistent browser storage.
- [[Prefetching RAM OPFS]] — sliding-window caching, prefetching, eviction, and OPFS alternatives.
- [[DICOMweb and Dicoogle]] — backend retrieval path and DICOM object/frame considerations.
- [[OHIF and Zero-Footprint Viewers]] — reference viewer architecture and deployment meaning of zero-footprint.
- [[Cornerstone3D Architecture]] — decoding, runtime caching, and rendering pipeline.
- [[Browser Storage Threat Model]] — storage, XSS, key, and residual-data threats.
- [[Performance KPI]] — benchmark definitions and the 500 ms target.

## Purpose

This note identifies which papers and technical references are relevant to:

> Privacy-preserving caching architectures for zero-footprint medical image viewers.

Focus areas:

- RAM-only caching.
- Prefetching.
- Large DICOM studies.
- Browser-based medical-image viewers.
- Persistent browser storage.
- Encryption at rest.
- Memory and GPU residue.
- Performance/security trade-offs.

## Dissertation problem

Web-based medical-image viewers need to retrieve and render large DICOM studies in a browser. Caching and prefetching improve responsiveness, but persistent browser storage can leave patient data on the client.

The dissertation therefore studies whether a bounded volatile cache can provide adequate performance without intentionally persisting patient images to disk.

If RAM-only caching is insufficient for very large studies, encrypted persistent caching such as OPFS may be evaluated as an alternative.

```text
Performance requirement:
  cache + prefetch + progressive loading

Privacy requirement:
  no intentional plaintext patient-data persistence

Security requirement:
  threat model browser storage, RAM, GPU, swap, crash dumps, and XSS
```

## Literature map

| Source | Main contribution | Importance |
|---|---|---:|
| Caching and Prefetching Strategy for Web-Based Medical Image Visualization System | Caching, prefetch prediction, navigation performance | Very high |
| JavaScript Access to DICOM Network and Objects in Web Browser | Browser DICOM retrieval, RAM, IndexedDB, browser storage | Very high |
| Web-Based DICOM Viewers: A Survey and a Performance Classification | Viewer architectures and evaluation methodology | High |
| OHIF Viewer paper | Reference browser-based medical viewer architecture | High |
| Cornerstone3D documentation | Loading, decoding, rendering, and runtime cache behavior | High |
| LesionTracker paper | Older zero-installation browser viewer and clinical workflow | High for background |
| OPFS documentation | Browser file-like persistent storage | High for implementation |
| Storage quota/eviction documentation | Browser persistence limitations | Medium/high |
| Keys to the Cloud | Encrypted web storage and key/XSS threats | Very high for security |
| OWASP Browser Storage Testing | Practical client-side storage security tests | High |
| Vulnerable GPU Memory Management | GPU-memory residue threat | High for limitations/threat model |
| Web Crypto API documentation | AES-GCM implementation reference | High for implementation |

## Caching and prefetching paper

### Why it matters

The paper *Caching and Prefetching Strategy for Web-Based Medical Image Visualization System* is the most directly relevant performance paper. It focuses on the mechanisms needed to make medical-image navigation responsive:

- Caching.
- Prefetching.
- Navigation behavior.
- Image access patterns.
- Cache organization.
- Performance evaluation.

### What to extract

Record:

- What is cached: frames, images, tiles, instances, or volumes.
- Whether data is encoded or decoded.
- Cache size and eviction policy.
- How the next images are predicted.
- Whether navigation direction is used.
- How many images are prefetched.
- Whether the cache is RAM or disk-backed.
- Dataset modality and size.
- Network conditions.
- Evaluation metrics.
- Cold-cache and warm-cache behavior.

### Relevance

This paper supports the performance side of the dissertation.

```text
Existing work:
  cache + prefetch for medical-image performance

This dissertation:
  cache + prefetch + privacy/security analysis
```

The dissertation extension should investigate:

- Plaintext patient data at rest.
- Encrypted persistent browser caches.
- Session-bound keys.
- Cache cleanup.
- Browser storage inspection.
- GPU and memory residue.

### Experiments inspired by this paper

Compare:

```text
No cache
RAM cache
RAM cache + prefetch
Position-aware RAM cache
Encrypted persistent cache
Encrypted persistent cache + RAM working set
```

Measure:

- Time to first render.
- Time to complete render.
- Adjacent navigation latency.
- Distant navigation latency.
- Cache hit ratio.
- Network bytes transferred.
- Peak RAM.
- Prefetch usefulness.

## JavaScript and DICOM in the browser

### Why it matters

*JavaScript Access to DICOM Network and Objects in Web Browser* connects DICOM retrieval with browser execution. It is useful for understanding:

- Retrieving DICOM objects.
- Managing binary data in the browser.
- Using RAM for image data.
- Using browser-side storage.
- Handling large studies.
- Improving access through pre-caching and concurrent requests.

### What to extract

Focus on:

- How the browser requests DICOM data.
- Whether complete objects or selected images are retrieved.
- How binary data is represented.
- Whether data is decoded immediately.
- How browser memory is used.
- How IndexedDB or browser filesystem storage is used.
- Performance impact of concurrent requests.
- Dataset sizes and image modalities.

### Relevance

It helps justify:

```text
DICOM backend
  → browser request
  → binary response
  → decode
  → RAM/rendering
```

It also helps explain why persistent storage becomes attractive when the study is larger than the available RAM working set.

This is older literature. Use it for architectural history and concepts, not as the source for current OPFS implementation details.

## Web-based DICOM viewer survey

### Why it matters

*Web-Based DICOM Viewers: A Survey and a Performance Classification* helps position the dissertation among existing viewers and provides guidance for evaluation methodology.

### Variables to control

- Browser and version.
- Operating system.
- CPU and RAM.
- GPU.
- Network bandwidth and latency.
- Study modality.
- Image dimensions.
- Transfer syntax.
- Number of frames.
- Cold versus warm cache.

Report median and p95 latency rather than only averages.

### Dissertation improvement

Add security measurements that normal viewer surveys often omit:

- Plaintext in persistent storage.
- Cache deletion behavior.
- Key lifecycle.
- Encryption overhead.
- Storage metadata leakage.
- Network-cache behavior.
- RAM working-set size.
- Cleanup after logout.

## OHIF

OHIF is a practical reference architecture for browser-based medical imaging. Study:

- DICOMweb integration.
- Metadata retrieval.
- Pixel-data retrieval.
- Cornerstone integration.
- Browser rendering.
- Clinical workflows.

OHIF’s “zero-footprint” terminology mainly means:

- No native installation.
- No thick-client workstation application.
- No plugin requirement.
- Viewer delivered through the browser.
- Studies retrieved from external image archives.

It should not automatically be interpreted as proof that no browser-managed data ever touches disk.

Map this pipeline:

```text
QIDO-RS metadata
  → study/series/instance selection
  → WADO-RS or image request
  → Cornerstone image loader
  → decoder
  → image/volume cache
  → viewport
  → WebGL/GPU
```

See [[OHIF and Zero-Footprint Viewers]].

## Cornerstone3D

Cornerstone3D handles much of the image loading, decoding, caching, and rendering work in modern web medical viewers.

Study:

- Image IDs.
- DICOM image loader.
- `loadImage` and `loadAndCacheImage` behavior.
- Image cache size and eviction.
- Volume cache.
- Progressive loading.
- Worker-based decoding.
- Encoded versus decoded data retention.
- Rendering and GPU resource lifecycle.

The most useful cache boundary is before decoding:

```text
DICOMweb/Dicoogle response
  → secure cache
  → encoded DICOM/image bytes
  → Cornerstone decoder
  → decoded pixels
  → rendering
```

See [[Cornerstone3D Architecture]].

## LesionTracker

LesionTracker is an early open-source browser-based clinical-trial imaging viewer. It demonstrates:

- Browser-only deployment.
- HTML5 architecture.
- Cornerstone integration.
- DICOM server connectivity.
- DICOMweb support.
- Clinical workflows.
- Measurements and annotations.
- Study switching.
- Synchronized navigation.

Its most important lesson is that “zero-footprint” mainly means browser delivery instead of an installed desktop application.

It does not provide a modern solution for:

- Encrypted OPFS.
- AES-GCM cache encryption.
- Session-bound keys.
- Browser storage inspection.
- Secure cache deletion.
- GPU memory residue.
- Swap/crash-dump threats.

## OPFS

OPFS is relevant for a large persistent binary cache. It provides:

- Origin-private file-like storage.
- Random-access file operations.
- Large binary data handling.
- Worker-compatible access patterns.

OPFS is not automatically encrypted and is not volatile RAM.

```text
OPFS:
  private to the origin
  browser-managed
  persistent
  not automatically confidential
  not guaranteed to be forensically erased
```

If used for patient data:

```text
DICOM bytes
  → AES-GCM
  → OPFS ciphertext
```

See [[Zero-Footprint and OPFS]] and [[Prefetching RAM OPFS]].

## Encrypted web storage

*Keys to the Cloud: Formal Analysis and Concrete Attacks on Encrypted Web Storage* is important because it demonstrates that encrypted client-side storage does not automatically make a web application secure.

Encryption protects stored ciphertext from a storage inspector, but not necessarily from malicious JavaScript executing in the same origin while the user is authenticated.

```text
Attacker only reads OPFS:
  encryption can help

Attacker executes same-origin JavaScript:
  may request decryption or read plaintext
```

Evaluate encryption together with:

- CSP.
- Trusted Types where possible.
- Dependency minimization.
- Authentication.
- Key lifecycle.
- Worker boundaries.
- Session expiration.

See [[Browser Storage Threat Model]] and [[AES-GCM Key Lifecycle]].

## OWASP browser-storage testing

Use OWASP browser-storage guidance to derive tests:

```text
Search IndexedDB for plaintext DICOM signatures
Search OPFS for plaintext patient identifiers
Inspect Cache Storage
Inspect localStorage/sessionStorage
Check extractable CryptoKeys
Check logout cleanup
Check browser-cache headers
Check sensitive data in URLs and logs
```

## GPU memory residue

*Vulnerable GPU Memory Management: Towards Recovering Raw Data from GPU* demonstrates that image data can remain recoverable in GPU memory after applications stop using it.

Medical image data may pass through:

```text
Network buffer
  → JavaScript/Worker memory
  → WebAssembly memory
  → decoded pixel buffer
  → WebGL/WebGPU texture
  → GPU memory
```

This motivates bounded working sets, explicit resource cleanup, and documenting browser/OS limitations.

See [[GPU Memory Residue]].

## Web Crypto and AES-GCM

Use Web Crypto as the browser-native implementation reference for encrypted persistent caching.

```text
Frame/chunk
  → AES-GCM encryption
  → ciphertext + IV/authentication tag
  → OPFS
```

Rules:

- Use authenticated encryption.
- Use a fresh random IV for every encryption operation with a key.
- Do not reuse an IV/key pair.
- Keep keys out of filenames and plaintext metadata.
- Prefer session-bound keys for the initial design.
- Do not persist the decryption key in the same cache.
- Test tampering and authentication failure.

## Literature-based architecture

```text
DICOMweb/Dicoogle
      ↓
Metadata retrieval
      ↓
Selected frame or instance retrieval
      ↓
Web Worker
      ├── Fetch
      ├── Decrypt if persistent cache hit
      ├── Encrypt before persistence
      ├── DICOM parsing
      └── Decoding
      ↓
Bounded RAM cache
      ├── visible images
      ├── nearby images
      └── prefetched images
      ↓
Cornerstone3D
      ↓
Canvas/WebGL/WebGPU
```

Primary architecture:

```text
RAM-only cache
+ tiling
+ prefetching
+ eviction
+ DICOMweb/Dicoogle progressive retrieval
```

Alternative architecture:

```text
Encrypted OPFS cache
+ session-bound keys
+ RAM decoded working set
+ cleanup and invalidation
```

See [[Prefetching RAM OPFS]].

## Literature-based research gap

> Existing medical-image viewer research studies browser deployment, DICOM retrieval, rendering, caching, and prefetching, while client-side security research studies encrypted browser storage and key-management threats. There is limited integration of these areas for large DICOM studies, particularly regarding the performance cost and security lifecycle of encrypted browser-side caching.

The dissertation connects these areas by evaluating:

```text
DICOM retrieval
+ medical-image prefetching
+ bounded RAM caching
+ optional encrypted persistent caching
+ session-bound key management
+ browser storage testing
+ RAM/GPU residual-data limitations
```

## Reading template

```text
# Source title

## Citation

Authors:
Year:
Venue:
DOI/URL:

## Research problem

## Architecture

## Data flow

## Cache design

## Prefetch strategy

## Security model

## Encryption/key management

## Dataset/workload

## Metrics

## Main results

## Limitations

## Relevance to my dissertation

## Useful quotation/paraphrase

## Follow-up questions
```

## Priority reading order

1. Caching and Prefetching Strategy for Web-Based Medical Image Visualization System.
2. JavaScript Access to DICOM Network and Objects in Web Browser.
3. OHIF Viewer paper and architecture documentation.
4. Cornerstone3D progressive-loading and cache documentation.
5. Web-Based DICOM Viewers survey.
6. LesionTracker paper.
7. OPFS documentation.
8. Browser storage quota and eviction documentation.
9. Keys to the Cloud.
10. OWASP Browser Storage Testing.
11. Vulnerable GPU Memory Management.
12. Web Crypto AES-GCM documentation.

## Final interpretation

The literature does not imply that all prefetched data must be written to disk.

Primary path:

```text
Fetch into RAM
  → bounded working set
  → prefetch intelligently
  → evict distant data
  → refetch when needed
```

OPFS is initially:

- A persistent performance baseline.
- An optional fallback for studies larger than the RAM budget.
- A design requiring encryption and key management.
- A relaxation of strict no-disk-persistence.

Main question:

> Can intelligent RAM-only caching, tiling, and prefetching achieve the required responsiveness for realistic large DICOM workloads without persistent patient-data storage?

Secondary question:

> If RAM-only caching is insufficient, what performance benefit does encrypted OPFS provide, and what additional security risks does it introduce?

## Connected dissertation concepts

- [[Prefetching RAM OPFS]]
- [[Zero-Footprint and OPFS]]
- [[Performance KPI]]
- [[Threat Model]]
- [[Cache Benchmark]]
