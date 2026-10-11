# Literature Comparison Table

## Common theme

**Web-based medical-image viewers: efficient retrieval, browser-side storage, performance, and security.**

| Paper | Medical-imaging context | Viewer/deployment model | Retrieval approach | Caching/storage approach | Performance aspect evaluated | Security/privacy aspect evaluated | Main contribution | Relevant limitation or unaddressed aspect | Relevance to this dissertation |
|---|---|---|---|---|---|---|---|---|---|
| Godinho et al. (2017), *An efficient architecture to support digital pathology in standard medical imaging repositories* | Digital pathology and whole-slide imaging | Zero-footprint HTML5/JavaScript viewer connected to a Dicoogle PACS | DICOMweb using QIDO-RS and WADO-RS; selected frames representing tiles; image pyramids and multiresolution retrieval | Viewer maintains required image portions in memory; no systematic comparison of browser storage technologies | Evaluates opening, zooming, panning, tile retrieval, storage size, and comparison with a JPEG2000/JPIP viewer; reports approximately 20 ms average tile retrieval and approximately 18× workflow speed-up over the comparison viewer | Security and persistent browser-side storage are not evaluated | Demonstrates that a Dicoogle-based, standards-compliant, zero-footprint viewer can efficiently retrieve and display only relevant image regions | Does not compare RAM, browser cache, and OPFS; does not evaluate encryption or residual medical-image data on the client; does not study the performance/security trade-off of persistent browser storage | Provides the architectural and retrieval background for the dissertation; supports the need to study client-side caching and storage separately |
| [Add paper] |  |  |  |  |  |  |  |  |  |
| [Add paper] |  |  |  |  |  |  |  |  |  |
| [Add paper] |  |  |  |  |  |  |  |  |  |
| [Add paper] |  |  |  |  |  |  |  |  |  |
| [Add paper] |  |  |  |  |  |  |  |  |  |

## Column guidance

Use the same questions for every paper:

- **Medical-imaging context:** What modality or imaging workflow is studied?
- **Viewer/deployment model:** Is it browser-based, zero-footprint, desktop, mobile, or cloud-based?
- **Retrieval approach:** Does it retrieve complete instances, frames, tiles, regions, or progressive quality layers?
- **Caching/storage approach:** Where is data kept, and is storage temporary or persistent?
- **Performance aspect evaluated:** What metrics, workflows, hardware, and cache conditions are tested?
- **Security/privacy aspect evaluated:** Does the paper address authentication, encryption, residual data, or privacy?
- **Main contribution:** What does the paper demonstrate or propose?
- **Relevant limitation or unaddressed aspect:** What does it not evaluate that matters to this dissertation?
- **Relevance to this dissertation:** How does it support the research background, method, or gap?

## Working research gap

The table should help establish a specific gap rather than claim that previous work ignored web viewers or caching entirely:

> Existing work demonstrates browser-based medical-image visualization using techniques such as DICOMweb, frame or tile retrieval, image pyramids, caching, and client-side processing. However, the combined performance and security implications of choosing different browser-side storage strategies for cached DICOM data remain insufficiently characterized. This dissertation addresses that gap by comparing RAM, browser cache, and OPFS under controlled viewer workflows and security requirements.

## Suggested paper categories

To keep the rows comparable, select papers that belong to at least one of these related categories:

- Zero-footprint or browser-based DICOM viewers.
- DICOMweb retrieval and medical-image interoperability.
- DICOM frame or tile retrieval.
- Medical-image caching and prefetching.
- Web-based viewer performance evaluation.
- Browser storage or OPFS for sensitive data.
- Security and privacy of client-side medical-image storage.

## How to fill each row

For each new paper, do not write a general summary. Extract only information that answers the table columns. The most important columns for identifying the dissertation gap are:

1. Caching/storage approach.
2. Performance aspect evaluated.
3. Security/privacy aspect evaluated.
4. Relevant limitation or unaddressed aspect.
5. Relevance to this dissertation.

A paper may be strong in retrieval performance but not address secure browser persistence. Another may study browser storage security but not DICOM viewers. Together, the rows can show why the dissertation's combination of medical imaging, browser storage, performance, and security is necessary.
