---
status: needs-review
tags: [research, reading-list]
---

# Reading List

## Related notes

- [[Zero-Footprint and OPFS]] — definitions of zero-footprint and implications of persistent browser storage.
- [[Caching and Prefetching]] — bounded RAM, prefetching, eviction, and shared encrypted OPFS.
- [[Architecture/Architecture Decisions|Architecture Decisions]] — working design and unresolved implementation choices.
- [[Specification/Key Management|Key Management]] — backend-held keys, active browser keys, and shared-cache reuse.
- DICOMweb and Dicoogle (pending note) — backend retrieval path and DICOM object/frame considerations.
- OHIF and Zero-Footprint Viewers (pending note) — reference viewer architecture and deployment meaning of zero-footprint.
- Cornerstone3D Architecture (pending note) — decoding, runtime caching, and rendering pipeline.
- [[Threat Model]] — storage, XSS, key, and residual-data threats.
- [[Performance and Usability]] — benchmark definitions and the working 500 ms target, not a universal clinical threshold.

## Purpose

This note identifies which papers and technical references are relevant to:

> Privacy-preserving caching architectures for zero-footprint medical image viewers.

Focus areas:

- Shared encrypted OPFS caching.
- Bounded plaintext native-frame RAM working sets.
- RAM-only and session-only encrypted caching as comparison modes.
- Prefetching.
- Large DICOM studies.
- Browser-based medical-image viewers.
- Persistent browser storage.
- Encryption at rest.
- Backend key custody and authorised cross-session reuse.
- Memory and GPU residue.
- Performance/security trade-offs.

## Dissertation problem

Web-based medical-image viewers need to retrieve and render large DICOM studies in a browser. Caching and prefetching improve responsiveness, but persistent browser storage can leave patient data on the client.

The proposed architecture uses shared encrypted OPFS as the persistent tier and bounded plaintext native-frame RAM as the active working set. Decoder Worker/WASM memory and the bounded Cornerstone decoded-image cache must also be managed and measured. RAM-only is an experimental baseline, not the primary architecture, and OPFS is not merely a fallback after RAM-only fails.

Dicoogle retains recoverable scoped DEKs under backend KEK protection. After current study permission and version checks, an authorised session obtains a key and trusted cache descriptor. A later authorised user on the same origin and browser profile/storage partition may reuse compatible ciphertext. The backend is trusted with keys; this is not E2EE against Dicoogle.

```text
Performance requirement:
  shared encrypted OPFS + bounded RAM + prefetch + progressive loading

Privacy requirement:
  no intentional plaintext patient-data persistence

Security requirement:
  authorisation + context/freshness checks + authenticated encryption
  + key lifecycle + browser/RAM/GPU/swap/crash-dump/XSS threat analysis
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

The paper *Caching and Prefetching Strategy for Web-Based Medical Image Visualization System* is a directly relevant performance reference. It focuses on the mechanisms needed to make medical-image navigation responsive:

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
- Shared encrypted persistent browser caches.
- Backend-held recoverable keys and current-session browser key access.
- Authorisation, trusted context, and freshness before cache reuse.
- Session cleanup versus global cryptographic erasure.
- Browser storage inspection.
- GPU and memory residue.

### Experiments inspired by this paper

Use the comparison presets in [[Caching and Prefetching]]:

```text
P1: Native-frame network-only baseline
P2: Bounded RAM baseline
P3: Plaintext OPFS control (synthetic data only)
P4a: Session-only encrypted OPFS
P4b: Shared encrypted OPFS + bounded RAM working set
```

P1 still includes runtime/Cornerstone decoded memory for rendering. Compare no prefetching, fixed prefetching, and direction/position-aware prefetching where applicable. Keep authorisation, freshness, and AEAD checks enabled during protected-mode timings.

Measure:

- Time to first render.
- Time to complete render.
- Adjacent navigation latency.
- Distant navigation latency.
- Cache hit ratio.
- Network bytes transferred.
- Peak RAM, including decoder and decoded-image retention.
- Prefetch usefulness.
- Encryption/decryption, key-access, and validation overhead.
- Reload, logout, restart, and later authorised cache reuse.

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
  → native-frame binary response
  → bounded RAM
  → decode
  → rendering
```

It also helps explain why persistent storage becomes attractive when the study is larger than the available RAM working set. Cross-session reuse is a separate motivation to evaluate, not a guaranteed performance benefit.

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
- Authorisation/key state and source/cache version.

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
- Substitution, replay, tampering, and unauthorised reuse attempts.

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

See OHIF and Zero-Footprint Viewers (pending note).

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

The proposed cache boundary is before decoding:

```text
Authorised network response or authenticated OPFS hit
  → bounded plaintext native-frame RAM
  → decoder Worker/WASM
  → bounded Cornerstone decoded-image cache
  → WebGL rendering
```

Preserve the native diagnostic representation and its metadata association. `/dic2png` is a rendered preview, not the native diagnostic cache unit. Coordinate eviction across native-frame RAM, decoder buffers, and Cornerstone rather than retaining duplicate long-lived decoded stores.

See Cornerstone3D Architecture (pending note).

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
- Shared-cache backend key custody and authorised reuse.
- Browser storage inspection.
- Secure cache deletion.
- GPU memory residue.
- Swap/crash-dump threats.

## OPFS

OPFS is relevant for the proposed persistent encrypted binary-cache tier. It provides:

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

Proposed patient-data path:

```text
Native diagnostic frame bytes
  → AES-GCM with expected-context binding
  → shared OPFS ciphertext + protected entry metadata
```

On reuse, validate expected scope and current versions, then verify AEAD before any plaintext reaches the decoder. Shared means same origin and browser profile/storage partition, not cross-device/profile sharing.

See [[Zero-Footprint and OPFS]] and [[Caching and Prefetching]].

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

The working shared-cache model trusts Dicoogle with recoverable scoped DEKs protected by a backend KEK. Current permission and version checks precede key release. Session-only encryption is a separate comparison mode; the shared DEK is not assumed to be irrecoverably destroyed on logout. Revocation cannot make a user forget copied keys or plaintext.

See [[Threat Model]] and [[Key Management]].

## OWASP browser-storage testing

Use OWASP browser-storage guidance to derive tests:

```text
Search IndexedDB for plaintext DICOM signatures
Search OPFS for plaintext patient identifiers
Inspect Cache Storage
Inspect localStorage/sessionStorage
Check extractable CryptoKeys and privileged key use
Check logout cleanup and late-task cancellation
Check browser-cache headers
Check sensitive data in URLs and logs
Check unauthorised and later authorised shared-cache reuse
Check tampering, substitution, replay, and version mismatch
```

Plaintext OPFS experiments use synthetic data only. Treat negative findings as bounded by acquisition method and test conditions, not universal proof of absence.

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

This motivates bounded working sets, explicit resource cleanup, and documenting browser/OS limitations. WebGL is the working rendering path; this wider threat map is not a claim that WebGPU is implemented. Session cleanup is not proof of physical RAM/GPU/swap sanitisation.

See GPU Memory Residue (pending note).

## Web Crypto and AES-GCM

Use Web Crypto as the browser-native implementation reference for encrypted persistent caching.

```text
Native frame + expected associated data
  → AES-GCM encryption
  → ciphertext + nonce/authentication tag
  → shared OPFS
```

Rules:

- Use authenticated encryption and a full authentication tag.
- Never reuse a nonce under the same DEK, including retries, rewrites, frames, tabs, workers, users, and sessions.
- Select random nonces with documented collision/key-use bounds or coordinated allocation; the mechanism is unresolved. Do not use frame number alone.
- Bind expected namespace, study/frame identity, transfer syntax, cache-format version, DEK version, and source version where applicable using unambiguous associated data.
- Establish expected context from the request/trusted backend, not a mutable cache header alone.
- Associated data is authenticated, not encrypted; minimise exposed identifying values.
- Keep keys out of filenames and plaintext metadata. Do not intentionally persist unwrapped browser keys in browser storage, URLs, logs, or metric exports.
- Keep browser key access scoped to authorised active work; backend recoverability supports later authorised shared-cache reuse.
- A non-extractable key does not prevent malicious privileged code from using it.
- Test tampering and authentication failure. Verify freshness separately; AEAD alone does not prevent replay.

## Literature-based architecture

```text
User/session + requested study
      ↓
Dicoogle permission and current source/key-version check
      ↓
Scoped DEK + trusted cache descriptor over HTTPS
      ↓
RAM lookup / OPFS candidate / authorised network retrieval
      ↓
Worker-side cache and decode pipeline
      ├── Validate expected context and freshness
      ├── Verify AEAD before decoding OPFS data
      ├── Fetch native bytes on cache miss
      ├── Encrypt reusable entries before persistence
      └── Parse/decode verified or authorised native bytes
      ↓
Bounded native-frame RAM + bounded Cornerstone decoded cache
      ↓
WebGL rendering
```

Primary proposed architecture:

```text
Shared encrypted OPFS cache
+ bounded plaintext native-frame RAM working set
+ managed decoder Workers/WASM
+ coordinated bounded Cornerstone decoded-image cache
+ demand-first prefetching and eviction
+ frame/tile/progressive retrieval where supported
+ authorisation, context/freshness checks, and key lifecycle
```

Comparison modes:

```text
P1 network-only
P2 bounded RAM
P3 plaintext OPFS (synthetic data only)
P4a session-only encrypted OPFS
P4b shared encrypted OPFS + bounded RAM
```

Cold display need not wait for an OPFS write. Persistence benefits must be measured rather than assumed. See [[Caching and Prefetching]] and [[Architecture/Architecture Decisions|Architecture Decisions]].

## Literature-based research gap

> Working hypothesis: existing medical-image viewer research studies browser deployment, DICOM retrieval, rendering, caching, and prefetching, while client-side security research studies encrypted browser storage and key-management threats. Integrated evidence remains to be established for the performance cost, authorised reuse, and security lifecycle of shared encrypted browser-side DICOM caching.

This is a research hypothesis to validate through source review, not proof of exhaustive novelty. The dissertation connects these areas by evaluating:

```text
DICOM native-frame retrieval
+ medical-image prefetching
+ bounded RAM and decoded working sets
+ shared encrypted persistent OPFS caching
+ backend KEK/DEK custody and active-session key access
+ authorisation/context/freshness validation
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

Read these alongside the existing evidence references in [[Specification/Key Management|Key Management]]: Share with Care, formal encrypted cloud storage, puncturable key wrapping, NIST SP 800-38D, and NIST SP 800-57 Part 1 Rev. 5. Use [[Research/Literature Comparison Table|Literature Comparison Table]] for source-specific limitations and verification status.

## Final interpretation

The literature does not imply that all prefetched data must be written to disk. The proposed design nevertheless includes shared encrypted OPFS as a persistent tier, rather than waiting for a RAM-only design to fail.

Primary path:

```text
Current permission/version check + authorised key access
  → bounded plaintext RAM hit, authenticated OPFS hit, or network fetch
  → managed decoder and bounded Cornerstone cache
  → display
  → encrypt/publish reusable entries asynchronously where appropriate
```

On logout or study close, invalidate the session generation, cancel tasks, and remove plaintext/key references. Compatible shared ciphertext may remain under retention policy for later authorised reuse. This is session cleanup, not global cryptographic erasure or strict physical zero-footprint.

Main question:

> Can shared encrypted OPFS caching plus bounded RAM provide the required responsiveness for realistic large DICOM workloads while preserving the stated security and privacy boundaries?

Secondary question:

> How do network-only, bounded RAM, session-only encrypted OPFS, and shared encrypted OPFS plus bounded RAM differ in performance, reuse, privacy exposure, and operational limitations?

Remaining implementation questions include key scope, KEK protection, nonce coordination, source-version authority, leases, multi-tab behaviour, retention, fallback, and exact workload-specific KPIs. Shared persistence as the primary proposed tier is not an unresolved fallback decision.

## Connected dissertation concepts

- [[Caching and Prefetching]]
- [[Zero-Footprint and OPFS]]
- [[Performance and Usability]]
- [[Threat Model]]
- [[Architecture/Architecture Decisions|Architecture Decisions]]
- [[Specification/Key Management|Key Management]]
- Cache Benchmark (pending note; not linked in this vault yet)
