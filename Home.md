---
status: index
tags: [home, dissertation, navigation]
---

# Home

## Start here

This vault documents a cybersecurity master's dissertation on privacy-preserving browser caching for medical-image viewing. It is **not** an autonomous DICOM-agent project.

## Scope summary

- Target frontend: `dicoogle-next/webapp` with Cornerstone.
- Target backend: `dicoogle-next`.
- Main proposed architecture: shared encrypted OPFS plus bounded plaintext native-frame RAM, managed decoder workers, coordinated bounded Cornerstone decoded-image cache, then rendering.
- RAM-only is a comparison baseline, not the primary architecture.
- Shared cache reuse means successive authorised sessions on the same machine, same origin, same browser profile/storage partition.

## Topics

- [[Architecture/Architecture Overview|Architecture Overview]]
- [[Architecture/Architecture Decisions|Architecture Decisions]]
- [[Research/Implementation Progress and Decision Log|Implementation Progress and Decision Log]]
- [[Architecture/Caching and Prefetching|Caching and Prefetching]]
- [[Architecture/Zero-Footprint and OPFS|Zero-Footprint and OPFS]]
- [[Architecture/Dicoogle Integration|Dicoogle Integration]]
- [[Specification/Project Requirements|Project Requirements]]
- [[Specification/Threat Model|Threat Model]]
- [[Specification/Key Management|Key Management]]
- [[Research/Scope and Research Question|Scope and Research Question]]
- [[Research/Evaluation Protocol|Evaluation Protocol]]
- [[Research/Performance and Usability|Performance and Usability]]
- [[Research/Security Tests|Security Tests]]
- [[Research/Reading List|Reading List]]
- [[Research/Literature Comparison Table|Literature Comparison Table]]
- [[diagrams/README|Diagrams]]

## Notes on draft status

Requirements, controls, and test outcomes are documented as proposed/planned unless explicitly verified.
