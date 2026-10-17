---
status: proposed
tags: [architecture, overview]
---

# Architecture Overview

The working proposal combines shared encrypted OPFS with bounded plaintext native-frame RAM and bounded Cornerstone decoded-image caching.

## Diagram navigation

- [[Architecture/Components|Components]]
- [[Architecture/Sequence|Sequence]]
- [Logical container architecture](../diagrams/Logical_container_architecture.mmd)
- [Trust boundary diagram](../diagrams/Trust-boundary_diagram.mmd)
- [Experimental mode comparison](../diagrams/Experimental-mode_comparison.mmd)

## Summary

- Native diagnostic frame bytes are the cache unit.
- `/dic2png` remains rendered preview output, not diagnostic cache units.
- Authorised key access is provided by trusted Dicoogle backend context.
- Logout is coordinated session cleanup, not global cryptographic erasure.
