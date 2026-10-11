# Browser Storage Threat Model

## Related notes

- [[Architecture Decisions 2026-10-11]]
- [[AES-GCM Key Lifecycle]]
- [[Zero-Footprint and OPFS]]
- [[Security Tests]]
- [[Diagrams/README]]

## Status and assets

Initial proposed threat/control mapping for the pre-thesis; capabilities and experiments must be refined. Assets include native frame bytes, decoded pixels, patient/study metadata, rendered pixels, active DEKs, protected key records, cache manifests and evaluation logs.

## Trust assumptions

- Dicoogle and its key registry are trusted to enforce current study access and protect server KEKs.
- HTTPS protects frames, metadata and key-access responses in transit.
- The viewer origin/deployed code is trusted.
- Shared OPFS means one origin and browser profile/storage partition.
- Browser RAM, Workers, WASM and GPU contain plaintext while viewing.
- No protection is claimed against a fully compromised host or copied keys from previously authorised users.

## Threat/control mapping

| Threat | Proposed control | Limitation |
|---|---|---|
| Next unauthorised application user finds OPFS | Encrypted payload/index; backend checks before key release; no prior-user plaintext reuse | Already copied keys cannot be recalled |
| Offline profile/disk inspection | No unwrapped persistent DEK; minimise identifying filenames/headers | Ciphertext length and some metadata remain observable |
| Ciphertext/manifest/frame substitution | AEAD expected-context binding; reject before decode | Freshness needs independent source/key-version checks |
| Malicious key/header input | Validate key identity, scope, structure and version; no default keys | Trusted-backend design is not malicious-server E2EE |
| AES-GCM nonce reuse | Cross-session/tab nonce policy and key-use bounds | Unspecified policy is not a security guarantee |
| Same-origin XSS/dependency compromise | Strict script policy, dependency review and origin separation where feasible | Can use active crypto capability or plaintext |
| Cross-origin state/pixel inference | Restrictive embedding/opener/resource policies; endpoint tests | Browser-specific side channels remain |
| OPFS SSD contention | Bound intentional I/O; document browser/hardware risk | Encryption and origin isolation do not eliminate timing leakage |
| Live memory, swap, hibernation, crash/GPU residue | Bound copies/lifetimes; dispose application resources; state-labelled tests | No browser-level proof of full sanitisation |
| Late result after logout | Cancel requests and invalidate session generation | Must test every asynchronous path |
| Concurrent/partial cache write | Cooperative locks and complete-entry publication/recovery | Locks are not security against malicious same-origin code |
| Logs/thumbnails/HTTP cache bypass encryption | Explicit response/storage policy and de-identified metrics | Preview routes must be evaluated too |

## Paper boundaries

- Gecko paper: active and hibernation key recovery, not successful recovery after every normal session close. DOI `10.1016/j.fsidi.2024.301763`.
- MIC: live LevelDB/IndexedDB records, not arbitrary DICOM pixels or OPFS keys. DOI `10.1016/j.fsidi.2024.301809`.
- FROST: attacker times its own OPFS file; does not directly access victim-origin files.
- Pixel Thief: CPU SVG-filter attack on embeddable rendered content under specific hardware/browser assumptions; not Cornerstone GPU memory extraction.
- GPU cache paper: WebGPU attacker/native CUDA AES target, not a Web Crypto break. DOI `10.1145/3634737.3656283`.
- Cloud storage attacks: key provenance and metadata/chunk integrity need explicit controls. DOI `10.1145/3658644.3690309`.

## Non-claims

RAM-only means no intentional application persistent pixel cache, not no operating-system writes. Encrypted OPFS is persistent storage. Logout is not physical sanitisation or global key destruction. Study authorisation, key release and retention are proposed rather than inherited automatically from login. Windows results in supporting papers do not add Windows to the dissertation test scope.
