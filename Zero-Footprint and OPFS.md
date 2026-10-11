# Zero-Footprint and OPFS

## Related notes

- [[Literature - Cache Prefetch Security]] — literature supporting the browser-viewer, cache, prefetching, and security discussion.
- [[Prefetching RAM OPFS]] — concrete RAM sliding-window and encrypted-OPFS architectures.
- [[DICOMweb and Dicoogle]] — current retrieval backend and implications for frame-level access.
- [[OHIF and Zero-Footprint Viewers]] — examples of the deployment/no-installation meaning of ZFV.
- [[Browser Storage Threat Model]] — detailed security threats and test cases.
- [[AES-GCM Key Lifecycle]] — encryption and session-key design.

## Why this note exists

The term **zero-footprint viewer (ZFV)** is ambiguous. Different systems use it to mean different things.

For this dissertation, distinguish between:

1. Zero-installation.
2. Application-level zero-footprint.
3. Cryptographic zero-footprint.
4. Strict physical zero-footprint.

This distinction is necessary because using OPFS may be compatible with zero-installation, but it does not automatically satisfy a strict no-disk-trace definition.

## Four meanings of zero-footprint

### Zero-installation viewer

A zero-installation viewer:

- Runs in a normal web browser.
- Requires no native application installation.
- Requires no plugins.
- Does not require a workstation-specific thick client.
- Retrieves studies from a remote PACS/VNA/DICOMweb server.

Under this definition, OPFS can be used and the viewer can still be called zero-footprint.

```text
No installed software
+ browser-based viewer
= zero-installation
```

This is the meaning commonly associated with browser-based viewers such as OHIF and older projects such as LesionTracker.

However, this definition says little about whether patient data remains in browser storage.

### Application-level zero-footprint

Under this definition, the application does not intentionally leave recoverable plaintext patient data in persistent browser storage.

The viewer may use browser-managed storage only if:

- Patient data is encrypted before persistence.
- Plaintext is not written intentionally.
- Persistent identifiers are minimized or protected.
- The cache is deleted at logout or session termination.
- Session keys are not persisted unnecessarily.
- Browser HTTP caching is controlled.

```text
Application-level ZFV
  = no intentionally persistent plaintext patient data
```

Encrypted OPFS may be compatible with this definition.

### Cryptographic zero-footprint

Under this definition, ciphertext may remain on the device, but it becomes unusable without the active session key.

```text
Encrypted cache remains
        ↓
Session key is destroyed
        ↓
Ciphertext cannot be decrypted
```

This is not the same as physical deletion. It is a cryptographic data-lifetime guarantee.

```text
Session starts
  → session key exists in memory
  → encrypted OPFS cache is usable

Session ends
  → key is destroyed or invalidated
  → cache is deleted where possible
  → remaining ciphertext is not decryptable
```

This can provide strong protection against an attacker who obtains only persistent cache files.

### Strict physical zero-footprint

Under the strictest definition:

> No patient data, traces, temporary files, or recoverable copies remain on the local machine after the viewer session ends.

This would include:

- No OPFS data.
- No IndexedDB data.
- No Cache Storage data.
- No browser HTTP cache containing patient data.
- No downloads.
- No plaintext temporary files.
- No patient identifiers in logs or URLs.
- No recoverable plaintext in RAM.
- No swap/pagefile copies.
- No crash-dump copies.
- No GPU-memory residue.
- No recoverable SSD blocks.

A normal browser application cannot reliably guarantee all of these conditions.

```text
Strict physical ZFV
  = no recoverable patient data anywhere on the client
```

OPFS is not compatible with this definition because OPFS is persistent browser storage.

## Why OPFS may not count as strict ZFV

OPFS stands for **Origin Private File System**.

It is:

- Private to the web origin.
- Not normally visible as a user-facing filesystem.
- Useful for large file-like data.
- Suitable for random reads and writes.
- Browser-managed.
- Persistent rather than volatile RAM.

The word “private” in OPFS means origin-private, not cryptographically private.

```text
Origin-private != encrypted
Origin-private != volatile
Origin-private != physically erased
```

OPFS data may survive:

- Tab closure.
- Page reload.
- Browser restart.
- Session termination.

The exact behavior depends on browser policies, eviction, storage clearing, and application cleanup.

Even after an application deletes an OPFS file, it generally cannot prove that all physical storage blocks have been overwritten. Browser storage is managed by the browser, filesystem, operating system, and storage hardware.

Therefore:

```text
OPFS can support zero-installation
but does not automatically support strict physical zero-footprint
```

## What if OPFS stores encrypted data?

Encryption changes the security claim but not the physical-storage fact.

```text
Plaintext DICOM
      ↓
AES-GCM encryption
      ↓
Ciphertext in OPFS
```

Encrypted OPFS provides protection against some attackers:

- A user inspecting browser storage.
- A filesystem-level reader who lacks the key.
- Accidental exposure of raw cache files.
- Unauthorized reuse of stale cache files after key invalidation.

However, encryption does not protect against every threat:

- Malicious JavaScript running in the same origin.
- XSS.
- A compromised dependency.
- An attacker who obtains the active session key.
- Plaintext frames in RAM during decoding.
- Plaintext pixels in GPU resources during rendering.
- Swap or crash dumps containing plaintext.
- Screenshots or screen recording.

Therefore, encrypted OPFS supports an application-level or cryptographic zero-footprint claim, not an absolute physical no-residue claim.

## Recommended terminology

Avoid:

> The viewer is zero-footprint because it uses OPFS.

Avoid:

> All patient data is securely erased from the machine.

Prefer:

> The viewer follows a zero-installation browser architecture and implements application-level zero-footprint handling for patient data. Large image objects may be persisted only as authenticated ciphertext in an origin-private cache. Session-scoped decryption keys are not persisted, and the cache is removed at session termination where browser APIs permit. Complete forensic elimination of browser-, operating-system-, and hardware-level remnants is outside the application’s control.

Other suitable names:

- Privacy-preserving persistent cache.
- Encrypted zero-installation viewer.
- Session-bound encrypted cache.
- Cryptographically ephemeral cache.
- Application-level zero-data-residue architecture.
- Secure persistent cache for a zero-installation viewer.

## Relation to the dissertation proposal

The written proposal emphasizes:

- In-memory caching.
- Tiling.
- Prefetching.
- No local disk persistence.
- A browser-only read/write architecture.
- Threat modelling of browser storage, swap, crash dumps, and memory residue.

Therefore, the safest primary architecture is:

```text
DICOMweb/Dicoogle
      ↓
Web Worker
      ↓
Decode and process
      ↓
Bounded RAM cache
      ↓
Tiling and prefetching
      ↓
Cornerstone/rendering
```

OPFS should initially be treated as:

- A persistent performance baseline.
- An alternative architecture.
- A fallback when the RAM-only design cannot meet the KPI.
- A security-performance trade-off.

If OPFS becomes part of the main architecture, the dissertation should state explicitly that the requirement has been refined from strict no-disk persistence to encrypted application-level persistence.

The concrete cache and prefetch design is described in [[Prefetching RAM OPFS]].

## Architecture comparison

| Architecture | Installation-free | Patient plaintext on disk | Ciphertext on disk | Strict physical ZFV | Suitable claim |
|---|---:|---:|---:|---:|---|
| RAM-only cache | Yes | Not intentionally | No | Closest | Application-level volatile ZFV |
| Plain OPFS | Yes | Potentially | No | No | Zero-installation only |
| Encrypted OPFS with persistent key | Yes | Not intentionally | Yes | No | Encrypted persistent cache |
| Encrypted OPFS with session-only key | Yes | Not intentionally | Yes temporarily | No | Cryptographically ephemeral cache |
| RAM + encrypted OPFS fallback | Yes | Not intentionally | Yes temporarily | No | Hybrid privacy-preserving viewer |
| Server-side rendering/paging | Yes | Not intentionally on client | No client cache required | Closest client-side model | Browser zero-installation with server-side data management |

## Threat-model distinction

### Storage inspector

Can inspect OPFS/IndexedDB files but cannot execute application JavaScript.

Mitigation:

- AES-GCM encryption.
- Session-only key.
- Opaque filenames.
- No plaintext metadata.

### Same-origin attacker

Can execute JavaScript in the viewer origin during an active session.

Mitigation:

- CSP.
- Trusted Types where possible.
- Dependency minimization.
- Secure authentication.
- Short-lived sessions.
- Worker isolation.

Encryption at rest alone is not sufficient against this attacker because the application must decrypt data for viewing.

### Privileged local attacker

Can inspect process memory, swap, crash dumps, or GPU resources.

Mitigation:

- Bound plaintext working set.
- Explicit buffer release.
- Worker termination.
- GPU resource cleanup where possible.
- Document browser/OS limitations.

A browser application cannot fully guarantee protection against this attacker.

The full threat model should be maintained in [[Browser Storage Threat Model]].

## Tests for the zero-footprint claim

### Persistent-storage tests

- Search OPFS for known plaintext or DICOM signatures.
- Search IndexedDB for pixel data and patient identifiers.
- Inspect Cache Storage.
- Inspect browser HTTP cache behavior.
- Inspect service-worker caches.
- Search filenames and keys for patient identifiers.
- Test before and after logout.
- Test after browser restart.

### Encryption tests

- Verify OPFS contains ciphertext rather than plaintext.
- Attempt decryption without the session key.
- Modify one ciphertext byte.
- Verify AES-GCM authentication fails.
- Confirm keys are not unnecessarily extractable.
- Confirm session keys are not persisted.

### Lifecycle tests

- Logout.
- Session timeout.
- Page refresh.
- Tab close.
- Worker termination.
- Network interruption.
- Partial cache writes.
- Browser crash, where feasible.

### Runtime-residue tests

- Measure decoded buffers.
- Clear references to pixel data.
- Terminate Workers.
- Revoke object URLs.
- Destroy WebGL/WebGPU resources where possible.
- Document that swap, crash dumps, and GPU memory cannot be fully verified from browser JavaScript.

## Suggested presentation wording

> Zero-footprint has two interpretations. In the conventional viewer sense, it means no software is installed on the workstation. In the stricter privacy sense, it means patient data does not remain on the client after the session. OPFS supports the first definition but not the second by default because it is persistent browser storage. Our primary design therefore evaluates RAM-only caching, tiling, and prefetching. Encrypted OPFS is evaluated as an alternative for workloads that exceed the RAM budget.

## Suggested research questions

> Can a browser-based medical-image viewer achieve the required responsiveness for large DICOM studies using bounded RAM caching, tiling, and prefetching without intentionally persisting patient data to local storage?

> If RAM-only caching is insufficient, can session-bound encrypted OPFS caching improve performance while preventing unauthorized recovery of persistent patient data?

## Planned related notes

- [[DICOMweb and Dicoogle]]
- [[OHIF and Zero-Footprint Viewers]]
- [[Cornerstone3D Architecture]]
- [[Browser Storage Threat Model]]
- [[AES-GCM Key Lifecycle]]
- [[Performance KPI]]
- [[Threat Model]]
- [[Cache Benchmark]]
- [[Security Tests]]
