# AES-GCM Key Lifecycle

## Related notes

- [[Architecture Decisions 2026-10-11]]
- [[Zero-Footprint and OPFS]]
- [[Prefetching RAM OPFS]]
- [[Browser Storage Threat Model]]
- [[Security Tests]]

## Status

Proposed design, not implementation. Personal/session-only encryption and shared-cache encryption are different modes.

## Working shared-key model

```text
Protected backend KEK
  → wraps stored study/cache DEK
  → Dicoogle checks current study permission
  → DEK and trusted cache descriptor sent over HTTPS
  → Web Crypto key handle in current browser session
  → encrypt/decrypt OPFS entries
```

Dicoogle is trusted with DEK custody and can release the same key to a later authorised user. This protects local ciphertext against readers lacking the key, not against the Dicoogle backend itself. No external identity provider or per-user asymmetric key infrastructure is assumed.

## Key states

| State | Behaviour |
|---|---|
| Study closed / no valid access | No active DEK or plaintext working set for that study |
| Authorised access | Obtain scoped DEK and current source/key descriptor |
| Viewing | Use key only for permitted current-session work |
| Study close / logout / timeout | Invalidate session generation; cancel tasks; remove plaintext and key references |
| Later authorised User B | Retrieve same valid key version and reuse OPFS ciphertext |
| Permission revoked | Backend denies new access; cooperative client stops after policy/session expiry |
| Cache/key version retired | Refetch/re-encrypt or invalidate under documented migration policy |

## Encryption envelope

Payload: original native frame bytes. Use AES-GCM with fresh nonce and a full authentication tag. Do not persist unwrapped DEKs in OPFS, IndexedDB, localStorage, sessionStorage, filenames, URLs, logs or metric exports.

Associated data must use unambiguous encoding and bind expected repository/cache namespace, study/frame identity, frame number, transfer syntax, cache-format version, DEK version and source version where applicable. Establish expected values from the request/trusted backend context rather than trusting a mutable cache header alone. AAD is authenticated, not encrypted; do not expose identifying values unnecessarily.

```text
Candidate entry
  → validate structure, scope and version
  → verify AEAD with expected AAD
  → only verified plaintext reaches decoder
```

A plain SHA-256 hash is not a substitute for authentication. AEAD does not identify which authorised user created a shared-key entry and does not by itself prevent replay.

## Nonce policy

Never reuse a nonce under the same key, including rewrites, retries, multiple frames, tabs, workers and users. Do not use frame number alone. Select either a random nonce policy with collision/key-use bounds or coordinated allocation; document and test the choice before implementation.

A non-extractable key can still be used by malicious code executing with the application's privileges. If export/wrapping is required during key provisioning, specify that separate provisioning stage; do not promise arbitrary wrapping of a permanently non-extractable browser key.

## Cleanup versus erasure

Removing a key reference or assigning null is not proof of physical zeroisation. Shared ciphertext is still legitimately recoverable through backend-controlled key access after User A logs out. Therefore logout is session cleanup, not global cryptographic erasure.

Revocation cannot make users forget copied keys or viewed plaintext. KEK rotation alone does not revoke an already disclosed DEK. Stronger deletion must address all recoverable keys/wrappings/backups; puncturable wrapping is future work.

## Evidence

- Share with Care: DOI `10.1109/EUROSP60621.2024.00051`.
- Formal encrypted cloud storage: DOI `10.1007/978-3-031-68379-4_2`.
- Puncturable key wrapping: DOI `10.1007/978-3-031-22966-4_22`.
- NIST SP 800-38D: AES-GCM/IV requirements.
- NIST SP 800-57 Part 1 Rev. 5: general key management.
- W3C Web Cryptography API: key usage/export boundaries.

## Unresolved

Key registry schema and KEK deployment protection; key scope and version migration; permission lease; nonce coordination; crash-safe publication; encrypted-manifest bootstrap; multiple active users/tabs; whether any offline mode is required.
