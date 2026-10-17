---
status: proposed
tags: [architecture, privacy, zero-footprint]
---

# Zero-Footprint and OPFS

## Related notes

- [[Architecture/Caching and Prefetching|Caching and Prefetching]]
- [[Architecture/Architecture Decisions|Architecture Decisions]]
- [[Specification/Threat Model|Threat Model]]
- [[Specification/Key Management|Key Management]]
- [[Research/Security Tests|Security Tests]]

## Terminology used in this vault

"Zero-footprint" is used carefully and not as a formal certification label.

1. **Zero-installation:** no local thick-client install, browser-delivered viewer.
2. **Application-level no intentional persistent plaintext:** plaintext patient data is not intentionally persisted.
3. **Cryptographic ephemerality:** ciphertext may persist, but without active key access it is not decryptable.
4. **Strict physical zero-footprint:** no recoverable traces anywhere on client media/memory.

The dissertation uses (1)-(3) where justified and explicitly avoids claiming (4).

## OPFS distinction

OPFS is origin-private persistent storage. "Private" means origin isolation, not automatic encryption or guaranteed physical erasure.

Therefore:

```text
OPFS can support zero-installation
but does not by itself prove strict no-residue privacy
```

## Working claim boundaries

Use this style of claim:

> Zero-installation browser viewer with no intentional persistent plaintext patient data; persistent cache entries are authenticated ciphertext with session-bound key access controls. Browser/OS/hardware residue remains an explicit limitation.

Avoid claims of perfect privacy, guaranteed secure physical erase, universal legal compliance, or global cryptographic erasure on logout.

## Relation to current architecture

The working architecture uses bounded plaintext RAM plus shared encrypted OPFS. RAM-only remains a comparison baseline.

```text
Authorised frame request or authenticated OPFS hit
  → bounded plaintext RAM
  → decode/render pipeline
  → optional background encrypted OPFS publication
```

Session cleanup cancels work and clears active references, but revocation cannot recall plaintext/keys already copied outside cooperative controls.
