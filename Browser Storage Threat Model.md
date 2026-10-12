# Browser Storage Threat Model

## Related notes

- [[Project Requirements]]
- [[Performance KPI]]
- [[Dissertation Home]]
- [[Architecture Decisions 2026-10-11]]
- [[AES-GCM Key Lifecycle]]
- [[Zero-Footprint and OPFS]]
- [[Security Tests]]
- [[Diagrams/README]]

## Status and scope

2026-10-12 preliminary model. Main target: encrypted persistent OPFS plus bounded plaintext RAM. Sharing is between successive authorised application sessions on one machine/origin/browser profile/storage partition. Cross-device/profile sharing and offline key acquisition are excluded initially. Concurrent tabs are a robustness case. Proposed mitigations are not passed tests. [[Project Requirements]] supersedes historical RAM-first/fallback wording.

## Assets

| ID | Asset | Locations |
|---|---|---|
| A1 | Native frame bytes and patient/study metadata | Network, RAM, parser, cache metadata if improperly protected |
| A2 | Decoded/rendered pixels | Worker/WASM, Cornerstone, canvas/GPU/driver |
| A3 | DEKs, wrapped DEKs, backend KEK | Active browser and trusted registry |
| A4 | Identity/version/index context | OPFS, descriptor service and registry |
| A5 | Session permissions and pending work | Token/lease/generation/scheduler state |
| A6 | Research artifacts | Metrics, screenshots, logs and memory evidence |

## Assumptions

Trust Dicoogle with study/key policy and KEK protection; HTTPS covers metadata/frames/keys. Trust viewer code/deployment; malicious same-origin code can defeat active plaintext/key protection. Origin storage is not app-user isolation. Offline-storage confidentiality assumes attacker lacks DEK/KEK and a usable stolen session token; token theft may enable key acquisition.

Previously authorised users may retain plaintext/keys. A shared DEK lets a holder produce valid AEAD ciphertext; it does not establish contributor identity or backend source provenance. Browser/OS/GPU plaintext during viewing is necessary and not universally sanitised by browser code.

## Threat/control mapping

| ID | Scenario | Proposed control | Requirement/test and limit |
|---|---|---|---|
| TM-01 | Later app user lacks study permission | Fresh key permission check; no prior-user plaintext reuse | FR-06/SEC-04/05; S03/S04; copied keys cannot be recalled |
| TM-02 | Offline disk/profile inspection | Ciphertext/protected index; minimise identifiers; no local unwrapped DEK | SEC-01/04, PRIV-02; S02; sizes/limited metadata remain |
| TM-03 | Frame/ciphertext/manifest substitution | Expected-context validation plus AEAD before decode | SEC-02/06; S05/S06; plain hashes not authenticity |
| TM-04 | Replay old valid entry/version | Trusted current version/content identity | SEC-06; S07; AEAD alone not freshness |
| TM-05 | Reuse key/nonce across rewrite/retry/tab/session | Explicit nonce scheme and key-use bounds | SEC-03; S08 |
| TM-06 | Invalid key state produces default/zero key | Strict input/key validation; fail closed | SEC-05; malformed-key case |
| TM-07 | Authorised DEK holder forges cached content | State contributor trust; add backend-authenticated content-origin verification if required | SEC-10; residual risk unless designed; AEAD not per-user provenance |
| TM-08 | Same-origin XSS/dependency compromise | Script policy, dependency review, feasible origin separation | SEC-09; active keys/plaintext remain exposed to compromised code |
| TM-09 | Different-origin state/pixel inference | Embedding/opener/resource policy and endpoint tests | SEC-09; S13; browser-specific residual channels |
| TM-10 | SSD activity inferred through attacker's own OPFS | Bound legitimate I/O and document platform leakage | FROST residual; encryption does not hide all patterns |
| TM-11 | Privileged memory/OS/swap/hibernation/GPU inspection | Bound copies/lifetimes; managed resource disposal | FR-04/SEC-08; S14; no full host-compromise guarantee |
| TM-12 | Late async result after logout/switch | Cancel work; invalidate generation; discard results | SEC-08; S10/S11 |
| TM-13 | Concurrent/interrupted write yields incomplete entry | Cooperative locks; consistent publication/recovery | SEC-07; S08/S09; locks not malicious-script access control |
| TM-14 | HTTP cache/preview/download/log bypass | Audit all paths; explicit sensitive-response policy | SEC-09/PRIV-02; S01/S02 |
| TM-15 | Large/malformed inputs or prefetch exhaust resources | Input/allocation limits, bounded queue/cache and safe fallback | FR-04/07/08; resource-pressure tests |

## Session lifecycle

```text
A authorised → obtains DEK → populates ciphertext → logs out
  → cancel work / clear plaintext / remove active key references

B logs in using same browser profile
  → independent current study/key decision
  → allowed: use scoped DEK and validated encrypted entries
  → denied: no new decryption access
```

Key leases and permission-expiry windows need specification; no immediate recall of delivered keys is promised. Logout is not global cryptographic erasure. Browser restart may retain ciphertext but requires new authorised key access.

## Evidence and non-claims

Nextcloud DOI `10.1109/EUROSP60621.2024.00051`: key insertion/ghost key/nonce misuse; historical patched system. Ecosystem DOI `10.1145/3658644.3690309`: key and metadata integrity. Formal storage DOI `10.1007/978-3-031-68379-4_2`: different malicious-server model, selective construction, revocation/rollback limits. Gecko DOI `10.1016/j.fsidi.2024.301763`: active/hibernation key recovery, not every post-exit case. MIC DOI `10.1016/j.fsidi.2024.301809`: IndexedDB structures, not arbitrary medical pixels. FROST: own-file SSD timing. Pixel Thief: CPU SVG-filter attack under specific framing/hardware conditions. GPU DOI `10.1145/3634737.3656283`: WebGPU/native CUDA case, not Web Crypto compromise.

No strict physical zero-footprint, memory/media erasure, E2EE against Dicoogle, recall of copied keys, universal clinical timing standard or automatic GDPR compliance is claimed. Non-extractable keys/Workers/locks are not complete same-origin/host security boundaries. Windows studies do not add Windows to the current test scope.

Use synthetic/de-identified data and restricted evidence handling; distinguish technical workload success from clinician-perceived acceptability in [[Performance KPI]].
