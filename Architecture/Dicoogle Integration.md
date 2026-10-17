---
status: needs-review
tags: [architecture, dicoogle]
---

# Dicoogle Integration

This note summarizes already documented repository findings and working assumptions. It is investigated context that still needs implementation revalidation.

## Current integration context

- Frontend target: `dicoogle-next/webapp` with Cornerstone.
- Backend target: `dicoogle-next`.
- Existing login/token handling remains in Dicoogle.
- Dicoogle V3 is comparison/reference context, not another target frontend.

## Proposed controls requiring revalidation

- Study-level permission checks for cache reuse.
- Trusted key-access service for recoverable scoped DEKs.
- Shared encrypted OPFS metadata/context validation before decode.
- Session-generation cleanup that rejects late async completions.

## Explicit non-claims

- No external identity provider is introduced here.
- This note does not claim completed controls or completed branch merges.
