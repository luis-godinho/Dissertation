---
status: proposed
tags: [architecture, cache, prefetch]
---

# Caching and Prefetching

## Related notes

- [[Architecture/Architecture Decisions|Architecture Decisions]]
- [[Architecture/Zero-Footprint and OPFS|Zero-Footprint and OPFS]]
- [[Specification/Project Requirements|Project Requirements]]
- [[Research/Performance and Usability|Performance and Usability]]
- [[Research/Security Tests|Security Tests]]
- [[Research/Reading List|Reading List]]

## Core idea

Prefetching requests data early. Caching retains data for reuse. Neither requires plaintext disk persistence.

```text
Prefetching  = request data early
Caching      = retain for reuse
Persistence  = retain beyond current session/runtime
```

## Working architecture position

Main proposed architecture:

```text
Network frame or authenticated OPFS hit
  → bounded plaintext native-frame RAM
  → decoder Worker/WASM
  → bounded Cornerstone decoded-image cache
  → WebGL rendering
```

Encrypted OPFS is part of the proposed shared-cache design, while RAM-only remains a baseline for comparison.

## Bounded RAM behavior

Use a bounded working set around active viewport demand and navigation direction. Keep visible/nearby frames and evict distant frames first.

Possible controls to measure:

- RAM budget (MiB/GiB).
- Prefetch window size and direction bias.
- Queue priority (visible demand over prefetch).
- Eviction policy (e.g., LRU + distance weighting).

## Encrypted OPFS behavior

If persistence is used, store authenticated ciphertext only.

```text
Native frame bytes
  → AES-GCM encrypt (fresh nonce per DEK use)
  → OPFS ciphertext + protected entry metadata
```

On read:

```text
OPFS candidate
  → expected-context/freshness checks
  → AEAD verification
  → only verified plaintext reaches decoder
```

Do not persist unwrapped browser keys intentionally.

## Comparison presets

- P1 native-frame network-only baseline.
- P2 bounded RAM baseline.
- P3 plaintext OPFS control (synthetic data only).
- P4a session-only encrypted OPFS.
- P4b shared encrypted OPFS + RAM.

P1 still includes runtime/Cornerstone decoded memory as part of rendering.

## Performance/security interpretation

Persistence benefits are an experimental question, not proof that RAM-only is always slower. Security checks (authorisation, freshness, AEAD validation) must stay enabled during timing measurements.
