# Prefetching, RAM Caching, and OPFS

## Related notes

- [[Literature - Cache Prefetch Security]] — papers and technical sources supporting the caching and prefetching design.
- [[Zero-Footprint and OPFS]] — why RAM-only caching is the primary design and why OPFS changes the zero-footprint claim.
- [[DICOMweb and Dicoogle]] — how the backend retrieves complete instances or individual frames.
- [[Cornerstone3D Architecture]] — where the cache fits before decoding and rendering.
- [[Performance KPI]] — metrics for evaluating the sliding-window approach.
- [[Cache Benchmark]] — experimental versions and test procedure.

## Core idea

**Prefetching does not automatically mean writing data to disk.**

Prefetching means requesting data before the user asks for it. The prefetched data can be stored temporarily in RAM and evicted when it is no longer useful.

```text
Prefetching  = request data early
Caching      = retain data for possible reuse
Persistence  = retain data beyond the current runtime/session
Disk cache   = write data to browser-managed storage
```

Therefore:

```text
Prefetching != disk persistence
Caching != disk persistence
```

A zero-footprint viewer can prefetch images into a bounded RAM cache without using OPFS, IndexedDB, or Cache Storage for patient data.

## RAM-only prefetching

A RAM-only design keeps only the working set needed for the current viewing operation.

```text
Current frame: 100
RAM cache:     95–105
Prefetch:      106–112
Evict:         frames far from frame 100
```

The complete study does not need to be stored in RAM. The viewer can keep a small window around the current position and request additional frames when the user navigates.

```text
Dicoogle/PACS
      ↓
Fetch nearby frames
      ↓
Decode in a Worker
      ↓
Store working set in RAM
      ↓
Render current frames
      ↓
Prefetch likely next frames
      ↓
Evict distant frames
```

### Example with a large study

Suppose a study contains 10,000 frames:

```text
Total study:       10,000 frames
RAM working set:   50 frames
Current viewport:  5 frames
Prefetch window:   20 frames
```

The other frames remain on the backend. If the user moves outside the current working set, the viewer:

1. Evicts distant frames.
2. Requests newly needed frames.
3. Decodes them.
4. Adds them to the RAM cache.
5. Renders them.
6. Prefetches the next likely frames.

This is a sliding-window strategy.

```text
[user navigates →]

[server only] [evicted] [RAM working set] [prefetch] [server only]
```

For the complete terminology and threat-model discussion, see [[Zero-Footprint and OPFS]].

## Why use a bounded RAM cache?

A bounded cache prevents the viewer from consuming all available browser memory.

Possible memory budgets include:

```text
64 MiB
128 MiB
256 MiB
512 MiB
1 GiB
```

The exact value should be determined experimentally.

The cache can track:

- Frame identifier.
- Encoded byte size.
- Decoded pixel-buffer size.
- Last access time.
- Distance from the current viewport.
- Whether the frame was prefetched.
- Whether the frame is currently visible.

A simple first eviction policy is LRU:

> Evict the least recently used frame when the memory budget is exceeded.

A better medical-image policy can combine LRU with position:

> Keep visible and nearby frames, prefer frames in the current navigation direction, and evict distant frames first.

## Prefetching strategies

### No prefetching

Only fetch a frame after the user requests it.

```text
User requests frame → fetch → decode → render
```

This is the simplest baseline but may cause visible delays during scrolling.

### Fixed-window prefetching

Always prefetch a fixed number of frames around the current position.

```text
Current frame: 100
Keep:          95–105
Prefetch:      106–110
```

### Direction-aware prefetching

Prefetch more frames in the direction in which the user is moving.

```text
User scrolling forward:
  Keep previous frames: 3
  Prefetch next frames: 10

User scrolling backward:
  Keep previous frames: 10
  Prefetch next frames: 3
```

### Position-aware prefetching

Use viewport position, navigation direction, and recent interaction history to decide what to retain and prefetch.

This is likely more appropriate for medical image stacks than a simple FIFO cache.

## RAM-only security properties

The RAM-only strategy is closest to the strict zero-footprint requirement because the application does not intentionally persist patient images through browser storage APIs.

Patient image data is not intentionally written to:

- OPFS.
- IndexedDB.
- Cache Storage.
- `localStorage`.
- `sessionStorage`.
- User-visible files.

However, RAM-only does not mean that plaintext can never exist outside application memory. During rendering, data may also exist in:

- JavaScript memory.
- Web Worker memory.
- WebAssembly memory.
- Browser-managed buffers.
- GPU textures.
- Operating-system swap.
- Crash dumps.

Therefore, the defensible security claim is:

> The application performs no intentional persistent storage of plaintext patient data and keeps the active working set in volatile memory, subject to browser-, operating-system-, and hardware-level limitations.

## Why OPFS may be considered

OPFS stands for **Origin Private File System**. It provides browser-managed file-like storage and is useful for large binary data and random access.

OPFS could be useful when:

- The study is too large for the available RAM budget.
- The network is slow or unreliable.
- The same frames are repeatedly revisited.
- Decoding is expensive and repeated downloads are costly.
- A large encrypted working set is useful for performance.

However, OPFS is persistent browser storage. It is private to the web origin, but it is not the same as volatile RAM and does not automatically provide encryption or secure deletion.

```text
OPFS improves persistence and potentially performance
but weakens the strict no-disk-footprint property
```

For the complete definition of this distinction, see [[Zero-Footprint and OPFS]].

## Encrypted OPFS as an alternative

If OPFS is used for patient data, the data should be encrypted before it is written.

```text
DICOM frame
    ↓
AES-GCM encryption
    ↓
Ciphertext
    ↓
OPFS
```

When the viewer needs the frame:

```text
OPFS ciphertext
    ↓
Read chunk
    ↓
AES-GCM decrypt and authenticate
    ↓
Decode
    ↓
Store active pixels in RAM
    ↓
Render
```

The OPFS cache should contain:

- Ciphertext.
- Random encryption IV/nonce.
- Authentication tag, as part of the AES-GCM result.
- Opaque frame/chunk identifier.
- Plaintext and ciphertext lengths.
- Key version or key identifier.

It should not contain:

- Plaintext DICOM pixel data.
- Patient names in filenames.
- Patient IDs in filenames.
- Human-readable study descriptions.
- Persistent decryption keys.

The terminology distinction between zero-installation, application-level, cryptographic, and strict physical zero-footprint is defined in [[Zero-Footprint and OPFS]].

## Session-bound encryption keys

A strong initial design is to keep the encryption key only for the active session.

```text
Session starts
      ↓
Generate or obtain session key
      ↓
Encrypt frames before writing to OPFS
      ↓
Decrypt frames when needed
      ↓
Logout or timeout
      ↓
Delete OPFS cache
      ↓
Destroy key references
```

If the key is not persisted, an attacker who only obtains the OPFS files should see ciphertext without the ability to decrypt it.

After logout or session expiration:

```text
ciphertext remains temporarily → no usable key → cache is not decryptable
```

The application should still attempt to delete the cache, but cryptographic invalidation is an additional protection because browser-level deletion cannot guarantee physical overwriting of every storage block.

## RAM cache versus OPFS cache

| Property | RAM-only cache | Encrypted OPFS cache |
|---|---|---|
| Patient data persistence | Not intentionally persistent | Ciphertext persists temporarily |
| Performance | Very fast | Slower due to storage and encryption I/O |
| Maximum size | Limited by available memory | Usually larger, subject to browser quota |
| Strict zero-footprint compatibility | Closest match | Not strict physical zero-footprint |
| Offline/revisit support | Poor | Better during the valid key/session lifetime |
| Cleanup | Clear references and terminate workers | Delete files and invalidate keys |
| Main risk | RAM, swap, GPU, crash dumps | Ciphertext exposure, key management, residual storage |
| Implementation complexity | Lower | Higher |
| Best role | Primary proposal-compliant design | Alternative or fallback architecture |

## Recommended architecture

The primary design should remain RAM-only until experiments show that it cannot meet the requirements.

```text
Dicoogle/PACS
      ↓
DICOMweb or Dicoogle image request
      ↓
Web Worker
      ├── Fetch
      ├── Decode
      └── Manage cache
      ↓
Bounded RAM cache
      ├── Visible frames
      ├── Nearby frames
      └── Prefetched frames
      ↓
Cornerstone3D / image renderer
      ↓
Canvas/WebGL/WebGPU
```

An optional persistent comparison architecture can then be evaluated:

```text
Dicoogle/PACS
      ↓
Web Worker
      ↓
Fetch image/chunk
      ↓
AES-GCM encryption
      ↓
Encrypted OPFS cache
      ↓
Decrypt on cache hit
      ↓
Decode
      ↓
Bounded RAM working set
      ↓
Cornerstone3D / renderer
```

## Experimental comparison

Compare:

| Version | Description | Purpose |
|---|---|---|
| V0 | No application cache | Worst-case baseline |
| V1 | RAM-only LRU cache | Volatile-cache baseline |
| V2 | RAM cache with prefetching | Measure prefetch benefit |
| V3 | Plain OPFS cache | Persistent performance baseline |
| V4 | Encrypted OPFS cache | Measure secure persistence overhead |
| V5 | Encrypted OPFS plus RAM cache | Hybrid candidate architecture |

Test different values for:

- RAM budget.
- Number of prefetched frames.
- Number of retained frames behind the viewport.
- Number of retained frames ahead of the viewport.
- Encoded versus decoded cache contents.
- Frame-level versus fixed-size chunking.

Measure:

- Time to first render.
- Time to complete render.
- Adjacent-frame navigation latency.
- Distant-frame navigation latency.
- Cache hit ratio.
- Network bytes transferred.
- Encryption time.
- Decryption time.
- DICOM decoding time.
- Peak RAM usage.
- OPFS usage.
- Cleanup time.
- Main-thread blocking.

See [[Cache Benchmark]] and [[Performance KPI]].

## Important issue with complete DICOM responses

The current Dicoogle endpoint returns:

```http
Content-Type: application/dicom
Transfer-Encoding: chunked
```

This means the complete DICOM object is transferred using HTTP chunks. It does not necessarily mean that the viewer can decode individual frames while the response is arriving.

### One DICOM instance per image

```text
Frame/image 1 → one DICOM request
Frame/image 2 → one DICOM request
Frame/image 3 → one DICOM request
```

This works well with RAM prefetching because the viewer can prefetch complete instances individually.

### One large multiframe DICOM instance

```text
One request → many frames inside one large DICOM object
```

This may require:

- DICOMweb frame retrieval.
- Byte-range requests.
- Server-side frame extraction.
- Streaming-capable decoders.
- Tiling or rendered-frame endpoints.

HTTP chunking alone does not solve this problem.

See [[DICOMweb and Dicoogle]].

## Recommended decision process

1. Implement and benchmark RAM-only caching.
2. Add small-window prefetching.
3. Test whether the KPI can be met.
4. Increase study size and measure memory pressure.
5. Add eviction and position-aware caching.
6. Test complete DICOM instances versus individual frames.
7. Only if RAM-only operation is insufficient, implement encrypted OPFS.
8. Compare the performance and security trade-off.

The research question is not:

> Must all prefetched data be written to disk?

The better question is:

> Can a bounded RAM working set, combined with intelligent prefetching and eviction, provide the required responsiveness for large medical-image studies without persistent patient-data storage?

If the answer is no, then the next question is:

> What performance benefit does encrypted persistent caching provide, and what security properties are lost or retained?

## Short conclusion

For the dissertation’s primary architecture:

```text
Prefetch into RAM
→ retain a bounded working set
→ evict distant frames
→ refetch from Dicoogle when necessary
```

Do not assume that every prefetched image must be written to OPFS.

Use OPFS only as an experimental alternative or fallback for data that cannot be handled efficiently by the RAM working set. If OPFS stores patient data, encrypt it before persistence and use session-bound keys.

## Literature connection

The caching and prefetching assumptions should be checked against [[Literature - Cache Prefetch Security]].
