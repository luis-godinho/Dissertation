---
status: evolving
tags: [research, implementation, decisions, progress]
updated: 2026-10-24
---

# Implementation Progress and Decision Log

## Purpose and related notes

Record what has been implemented, the decisions taken and their rationale, the evidence available, and what is deliberately deferred. This is a dated progress record, not a replacement for the target architecture.

- [[Home]]
- [[Architecture/Architecture Decisions|Architecture Decisions]]
- [[Architecture/Dicoogle Integration|Dicoogle Integration]]
- [[Architecture/Caching and Prefetching|Caching and Prefetching]]
- [[Specification/Project Requirements|Project Requirements]]
- [[Specification/Key Management|Key Management]]
- [[Specification/Threat Model|Threat Model]]
- [[Research/Evaluation Protocol|Evaluation Protocol]]
- [[Research/Performance and Usability|Performance and Usability]]
- [[Research/Security Tests|Security Tests]]
- [[diagrams/README|Diagrams]]

## Evidence and status conventions

- **Decided:** selected scope or design constraint, not proof of implementation.
- **Implemented / focused tests reported passing:** recorded implementation results with the verification scope stated.
- **Resolved within tested scope:** a specific correction and executed regression result have been reported; broader coverage is not implied.
- **Runtime not verified:** no live service/browser demonstration establishes the behaviour yet.
- **Planned:** intended next work, not completed work.
- **Deferred:** excluded from the current milestone, not abandoned.

This snapshot is based on implementation reports and the subsequent correction report supplied by the project owner on 2026-10-24. Tests were not independently rerun for this research note. A documentation commit does not publish the implementation code or establish deployment/runtime success.

## Current snapshot — 2026-10-24

| Area | Recorded state | Evidence / limit |
|---|---|---|
| Implementation vehicle | Incremental work from `dicoogle-next` dev on `feat/dissertation-frame-baseline` | Initial base `75427c8a75d6ca7cb9cc65cb536473c1c67d4813` |
| Implementation revision | Local commit `b6476c0`, `feat(backend): add native-frame retrieval endpoint` | Reported committed non-Markdown changes; not pushed; detailed Markdown reports remain local/untracked |
| Synthetic DICOM fixture | Implemented; nine focused tests initially reported passing | Included again in backend regressions; structural validation, not external IOD certification |
| Native-frame endpoint | Implemented; latest focused run reports 44 tests passing | 18 protocol + 26 application; zero failures/errors/skips; full backend suite not run |
| Encapsulated/fragmented PixelData rejection | Corrected and service-level regression reported passing | Fragments returns 415; missing/invalid supported native PixelData remains 422; fragment-specific HTTP multipart-body behaviour not verified |
| Isolated runtime setup and API routing | Planned | No completion/runtime evidence supplied |
| Selected-study series/instance enumeration | Planned for the demonstration path | Reuse existing QIDO APIs; no study-search semantics rewrite |
| Native-frame browser transport, decode and viewport | Planned | No browser/Cornerstone integration result recorded for the new endpoint |
| P0/P1 demonstration and timing | Not yet verified | No live latency, eight-viewport, or passing 500 ms result |
| Bounded native RAM, encrypted OPFS and shared-key access | Deferred | Final architectural target remains unchanged |

The base revision identifies the starting code. The latest local implementation commit is recorded separately. Remote availability of that implementation commit has not been established; no remote link is asserted. Future evidence should record the tested commit and any additional uncommitted diff/file snapshot.

## Decisions and rationale

### ID-01 — Incremental development from the selected dev baseline

Decision: continue on `feat/dissertation-frame-baseline`, created from the selected dev revision. `feat/frontend-frame-retrieval` is a read-only implementation reference, not the development vehicle or an automatic bulk import.

Reason: preserve the chosen current codebase and introduce small changes with controlled dependencies and verification. Existing reference work is not treated as nonexistent, but is not imported wholesale.

Consequence: retain current dev fixes, existing full-instance viewing and unrelated user changes. A branch merge, rebase, broad dependency import or deployment replacement is not implied.

### ID-02 — Demonstrate retrieval before reusable cache and encryption layers

Decision: first establish a limited native-frame P1-like path in one Cornerstone viewport, retaining the full-instance viewer as local baseline P0.

Reason: fix frame identity, representation, transport, errors and measurement boundaries before adding cache reuse and cryptographic overhead.

Consequence: this milestone does not demonstrate P2, P4a, P4b or clinical suitability. The final proposed design remains shared encrypted OPFS plus bounded plaintext native-frame RAM and separately managed decoded/worker/GPU state.

### ID-03 — Restrict the initial supported pixel profile

Decision: Explicit VR Little Endian (`1.2.840.10008.1.2.1`), native uncompressed PixelData, SamplesPerPixel=1, MONOCHROME2, BitsAllocated=16, BitsStored=16, HighBit=15 and unsigned PixelRepresentation=0, for the selected Secondary Capture SOP classes.

Reason: make correctness deterministic without simultaneously implementing compressed-frame parsing, transcoding or broad modality support.

Consequence: unsupported layouts, bit depths, signed pixels, colour, SOP classes and transfer syntaxes are explicitly rejected. Syntax must not be inferred merely from payload size. Encapsulated support remains a future, named-transfer-syntax extension.

### ID-04 — Keep representations distinct

| Representation | Role |
|---|---|
| Complete Part-10 instance | Existing P0 retrieval baseline |
| `native-frame-v1` | Selected native frame payload; intended future cache unit |
| `wrapped-single-frame-v1` | Possible transient decode envelope; not the network payload or persistent cache unit |
| Decoded pixels | Decoder/Cornerstone memory, separate from native-byte RAM |
| Rendered PNG preview | Existing preview path; excluded from native-frame claims |

Decision: no `/dic2png`, silent frame substitution or legacy fallback to manufacture apparent success. A future transient wrapper must preserve the requested frame's metadata and representation association.

### ID-05 — Use explicit frame-list and resource identity semantics

```text
GET /api/dicom-web/studies/{studyInstanceUID}/series/{seriesInstanceUID}/instances/{sopInstanceUID}/frames/{frameList}
```

The application context supplies `/api`; the controller is mapped under `/dicom-web`.

- API frames are one-based; convert internal zero-based indices explicitly.
- Reject malformed, duplicate, nonpositive and out-of-range values; no silent deduplication or frame-zero/frame-one substitution.
- Preserve valid unique request order, including `8,1,3`.
- Limit the demonstration to eight unique requested frames. This is an engineering bound, not a universal protocol limit.
- Match the complete study/series/SOP tuple against the stored and parsed instance. Unknown/mismatched tuples use the same not-found outcome; no SOP-only fallback.

### ID-06 — Return binary multipart frames and an explicit cache policy

Decision: `multipart/related` with `application/octet-stream` parts, per-part frame association, provenance and `Cache-Control: no-store`. Invalid requests do not produce successful partial multipart output.

Reason: preserve selected-frame identity and make the demand-only baseline observable.

Consequence: this limited experimental profile is not comprehensive DICOMweb conformance. No-store expresses HTTP cache policy, not proof of absence of process/browser/GPU residue.

### ID-07 — Bound complete-instance reads; do not claim selective disk I/O

Decision: read/parse the complete stored instance within an enforced bound, then extract the selected native frames.

| Engineering limit | Value |
|---|---:|
| Stored-instance input | 64 MiB |
| Native frame | 16 MiB |
| Sum of selected response frame payloads | 64 MiB |
| Unique requested frames | 8 |

The input cap is enforced during reading rather than after an unbounded read. Dimensions/counts/sizes/offsets use checked arithmetic before allocation/extraction. Multipart framing is additional to the payload cap and bounded separately by the request/part limits.

Consequence: `frameSelectiveResponse=true`, `frameSelectiveBackendIo=false`. Backend instance-read bytes/counts and selected response payload bytes are separate measurements. Parsing and response construction can create additional copies; the input cap is not a peak-memory claim. Ranged reads and large-instance frame indexing remain deferred.

### ID-08 — Preserve a genuinely demand-only P1 path

Decision: no reusable native-byte cache or adjacent prefetch in the new demonstration path; revisits perform fresh frame-body retrieval. Do not invoke existing load-all prefetch behaviour for this path.

Reason: obtain a measurable baseline before P2/persistent-cache comparisons.

Consequence: distinguish actual HTTP requests, cache policy, native hits and decoded-cache reuse. Cornerstone transient memory is not native-byte caching. A bypass flag alone does not prove requests occurred.

### ID-09 — Reject stale display work during frontend integration

Decision: use request identity/generation checks and cancellation where supported so an earlier completion cannot overwrite the current frame or update a closed viewport.

Reason: asynchronous frame-selection correctness is required before broader cache/session lifecycle work.

Consequence: examine internal loader/render/cache side effects; a generation check after stale publication is insufficient. Full ownership cleanup and key lifecycle remain separate milestones.

### ID-10 — Keep the first runtime demonstration isolated and synthetic

Decision: synthetic-only storage/index roots; no real patient files, broad home/root mounts, production data or deployment claim.

Reason: frame-endpoint authentication does not resolve study-policy or broader deployment concerns.

Consequence: startup, fixture registration/indexing and browser routing require explicit configuration review and observation. Isolation is a prerequisite, not proof of production security or user separation.

### ID-11 — Keep timing claims narrower than the project target

Decision: distinguish request/headers/body completion, decode, display-call, animation-frame and supported render-event boundaries.

Reason: post-fetch display timing, playback FPS and clinical interpretation duration are not interchangeable with end-to-end viewer delay.

Consequence: render-call return, render events and rAF are operational proxies, not physical presentation proof. Eight frames in one object or sequentially viewed in one viewport are not eight simultaneous displays. The eight-image/500 ms target still requires a defined workload/event/conditions and an actual measured result.

## Dated implementation and verification records

### 2026-10-24 — Synthetic fixture completed

- One synthetic study, two series, four instances.
- Series A: eight distinct 64x64 unsigned 16-bit MONOCHROME2 frames in a Multi-frame Grayscale Word Secondary Capture object (`1.2.840.10008.5.1.4.1.1.7.3`).
- Series B: three single-frame Secondary Capture companions (`1.2.840.10008.5.1.4.1.1.7`).
- Explicit VR Little Endian; 8192 native bytes/frame, 65536 total native pixel bytes in A. Full Part-10 file sizes include metadata and are larger.
- Deterministic patterns, fixed synthetic identifiers/dates and frame/instance hashes in a manifest. The manifest generation timestamp is not a stable fixture-content identifier.
- Test-scope generator/tests: `backend/dicoogle-next/src/test/java/org/dicoogle/app/testing/fixtures/SyntheticDicomFixtureGenerator.java` and `SyntheticDicomFixtureGeneratorTest.java`.
- Ignored generated output: `backend/dicoogle-next/target/synthetic-dicom-fixtures/synthetic-study-v1/`.

Evidence supplied: nine focused tests passing with zero failures/errors/skips. Checks included fresh re-read, frame values/distinctness, manifest consistency, deterministic regeneration and controlled reset/overwrite refusal. These tests were included again in the backend regression selections.

Limits: structural/module-attribute validation, not external IOD certification, clinical validity, runtime ingestion or performance evidence.

### 2026-10-24 — Native-frame endpoint implemented; initial focused run

Recorded components:

- `NativeFrameService`, `NativeFrameLimits`, `NativeFrameResponse` in the DICOMweb protocol module.
- Frame route in `DicomwebController`.
- Existing locator/storage abstractions, authentication and synthetic fixture reused.
- Service, controller and controller-limit tests; existing full-instance and fixture regressions retained.
- Selected-frame binary multipart output, request-order preservation, bounds/profile/error checks and no preview/legacy fallback.
- Global `HttpMediaTypeNotAcceptableException` mapping in `ApiExceptionHandler`, correcting unacceptable media negotiation from generic 500 to 406. This is a cross-cutting change and was treated as such in review.
- Metrics for requests/success/failure, instance-read bytes, selected-frame response bytes and backend reads; not browser timing evidence.

Initial reported focused run: 17 protocol + 26 application = 43 tests, zero failures/errors/skips, build success. This result preceded the fragment-specific review finding and is retained as historical evidence, not the latest count.

### 2026-10-24 — Fragment-specific review finding

The supported-profile validator classified every non-byte-array PixelData value as structural inconsistency (422). The fragment-specific `Fragments` representation needed the contract's unsupported-organisation outcome (415). Existing profile tests did not exercise that branch.

Decision: make the smallest explicit distinction rather than adding compressed support. Missing/invalid supported native PixelData keeps 422. Encapsulated/fragmented data is rejected with 415. This is the current experimental contract, not a statement that encapsulated DICOM is inherently corrupt.

### 2026-10-24 — Correction completed and focused regressions rerun

Owner-supplied correction record:

- `NativeFrameService.java:19,252-258`: explicit `Fragments` rejection with 415; missing/invalid PixelData retains 422.
- `NativeFrameServiceTest.java:191-201,415-456`: new regression using Explicit VR Little Endian and valid preceding metadata so it reaches the fragment-specific branch, rather than succeeding only through earlier syntax rejection.
- Local implementation commit: `b6476c0`, `feat(backend): add native-frame retrieval endpoint`; not pushed.
- Detailed implementation report: `docs/dissertation/implementation-reports/01b-native-frame-endpoint.md`, section 10 appended and read back.
- Markdown documentation and local context files remained excluded/untracked according to the supplied record.

Reported executed verification, from the implementation backend directory:

```bash
./mvnw -Dspotless.apply.skip=true -Dspotless.check.skip=true -pl dicoogle-next-protocol-dicomweb,dicoogle-next -am -Dtest=NativeFrameServiceTest,DicomwebRetrieveServiceTest,SyntheticDicomFixtureGeneratorTest,NativeFrameControllerTest,NativeFrameControllerLimitTest,DicomwebControllerTest -Dsurefire.failIfNoSpecifiedTests=false test
```

| Latest focused run | Tests | Failures | Errors | Skipped |
|---|---:|---:|---:|---:|
| Protocol | 18 | 0 | 0 | 0 |
| Application | 26 | 0 | 0 | 0 |
| Total | 44 | 0 | 0 | 0 |

Result supplied: BUILD SUCCESS, 29.290 seconds. The initial 43-test run and latest 44-test run are repeated verification selections, not 87 distinct tests.

Status: the identified fragment-status blocker is resolved within the reported service-level test scope.

Still not verified: the full backend suite, frontend/browser/live runtime, fragment-specific HTTP multipart-body behaviour, external IOD conformance, or performance. A service exception assertion does not independently establish HTTP body behaviour for that same input. No native-cache, encryption, study-policy or shared-user success is implied.

## Deferred features and re-entry conditions

| Item | Current boundary / rationale | Re-entry condition |
|---|---|---|
| Bounded native-byte RAM (P2) | First establish correct P1 transport/display | Verified baseline, explicit budget and ownership/eviction tests |
| Full cache/session cleanup | Minimal stale-display handling first | Reusable caches with delayed-work/logout/user-switch tests |
| AES-GCM envelope | Outside retrieval milestone | Approved context/key-use rules and isolated round-trip/negative tests |
| Ciphertext OPFS | Storage should not hide representation defects | Tested crypto, bounded namespace/metadata/publication design |
| Session-only P4a | Optional intermediate/comparison mode | Explicit scope approval and tested same-session key lifecycle |
| Study-level authorisation | Authentication alone is insufficient | Approved server-enforced policy and allow/deny/bypass tests |
| Backend KEK/DEK custody | Required for final P4b, not initial P1 | Approved provisioning, durable protection, scope/version/retirement and release tests |
| Authorised User B reuse (P4b) | Depends on real policy and recoverable scoped keys | Fresh permission/key access, ciphertext reuse and denied-user/cleanup evidence |
| Compressed/encapsulated formats | Current fixed-size extraction is uncompressed only | Relevant dataset and named syntax; correct fragment/frame reconstruction, response type and decoder tests; never assume one fragment equals one frame |
| Directional prefetch | Keep initial demand-only behaviour observable | Stable baseline, bounded scheduling and measured off/on comparison |
| Advanced prediction/adaptation | Not necessary for the first working parts | Specific hypothesis and measurable incremental benefit |
| Eight simultaneous images / 500 ms | One-viewport milestone; target not operationally fixed | Approved workload/event definition, relevant dataset and repeatable complete measurements |
| Multi-tab coordination | Concurrent publication and session/key ownership need design | Stable single-context operation, locking/recovery and concurrent tests |
| Selective backend disk I/O | Complete bounded instance is still read | Evidence justifies indexing/ranged reads and measured storage comparison |
| Broader modalities/clinical datasets | Narrow synthetic OT/SC fixture first | Dataset provenance, required syntax support and validated security/deployment prerequisites |

Deferred does not mean abandoned. The final target remains shared encrypted OPFS plus bounded RAM. No offline-access, cross-device sharing or new key hierarchy is selected by this milestone.

## Unresolved security and deployment concerns

Earlier inspections identified study-policy gaps, anonymous preview/filesystem routes, broad roots/mounts, token handling and lifecycle/header/logging concerns in the inspected baseline. Keep these tracked until specific corrections and evidence exist.

- Static findings are not automatically demonstrated runtime exposures.
- The new authenticated frame route does not secure every sensitive retrieval path or implement study-level policy.
- Synthetic data does not make a publicly exposed or broadly mounted deployment safe.
- No real-data, production-security or meaningful multi-user-isolation claim follows from the current endpoint.
- HTTP cache policy and ordinary cleanup are not proof of forensic sanitisation.

## Next checkpoint

With the fragment-specific correction and focused rerun now recorded, the next bounded work is isolated local configuration and API routing, followed by selected-study enumeration, binary-safe transport, one-viewport native-frame navigation, minimal stale-result handling and reproducible P0/P1 evidence.

No completion evidence for that live/frontend work has been supplied. Do not start reusable caching, encryption or shared-key work automatically. Freeze a stable partial demonstration according to the actual presentation deadline and observed progress.

## Maintenance

For each future entry record:

```text
Date:
Decision or implementation change:
Reason / alternatives not selected:
Implementation branch and tested commit or diff snapshot:
Files/components affected:
Verification actually executed and observed result:
Checks not run / limitations:
Current status:
Deferred work / next dependency:
```

Preserve dated history and explain supersession. Keep design decisions, implementation presence, focused test success, runtime results and performance claims separate. Detailed evidence remains in the implementation checkout's `docs/dissertation/implementation-reports/`; this note summarises it. Omit machine-specific absolute paths, credentials and patient data.

## Change history

- 2026-10-24: recorded the dev-based implementation vehicle, synthetic fixture and limited native-frame decisions; preserved the initial 43-test run and review finding, then recorded its correction, local unpushed commit `b6476c0`, and latest 44-test focused verification. Cache/security/performance extensions remain explicitly deferred.
