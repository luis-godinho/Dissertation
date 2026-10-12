# Performance KPI

## Related notes

- [[Project Requirements]]
- [[Dissertation Home]]
- [[Security Tests]]
- [[Browser Storage Threat Model]]
- [[Prefetching RAM OPFS]]
- [[Cache Benchmark]]

## Status

2026-10-12 draft, refined after owner clarification: the central user question is what delay feels acceptable during a particular study-viewing task. Technical FPS/cine rates are supporting workloads, not substitutes for that question. No verified universal CT/MRI/DBT acceptable-ms table was found in the consulted evidence.

## Three separate meanings of timing

1. **Technical responsiveness:** time from study/frame request to correct display; stalls, frame age, visible-set completion and main-thread blocking.
2. **Clinician-perceived acceptability:** whether waiting/lag interrupts the task, breaks control, or makes the viewer unsuitable; depends on task, study, reader and repeated delays.
3. **Clinical interpretation duration:** time needed to inspect and interpret a study. This is not the acceptable software waiting time, nor the report turnaround time.

Do not infer an acceptable navigation delay from a paper's total reading time or chosen cine FPS.

## Original project target

> Open eight mammography/tomosynthesis images in less than 500 ms.

Preserve this project-provided engineering target without treating it as a paper-backed clinical standard. Define what eight means (frames, viewports, instances or another visible set) and whether completion means first render, all requested visible units correctly rendered, or interaction-ready. Do not replace it with 500 ms per frame or all slices of eight multiframe objects.

Proposed measurement for review: T_visible(N) from registered visible-set request to the last correct requested unit rendered. N=1,2,4,8 are engineering-selected concurrency cases. Pass statistic and cold/warm/network conditions remain to be agreed.

## User-centred timing evaluation

| Task | What the user experiences | Measurements | Acceptability status |
|---|---|---|---|
| Open study/series | Wait before beginning inspection | First correct image, useful visible-set readiness, perceived wait | No modality-specific numerical limit established |
| Sequential scroll | Lag between input and requested slice; repeated stalls | Input-to-correct-frame latency, frame age, stalls and actual displayed IDs | Clinician pilot needed; FPS is not latency tolerance |
| Reverse or revisit | Whether the viewer follows a changed direction promptly | Reverse/revisit latency, stale results, perceived control | Validate by task/modality |
| Jump to distant frame | Delay at an uncached location | Cold jump latency and disruption | Report separately from warm adjacent scroll |
| Change series/prior | Wait before continuing comparison | Series readiness and correctly associated images | Use realistic clinical comparison task |
| Multi-viewport update | Wait for a usable set, possible unsynchronised images | T_visible(N), viewport skew, identity correctness | N and task must be specified |

Candidate studies: CT and DBT initially; MRI only after relevant dataset/task and numeric evidence or clinician observations are available. Within each, record anatomy, series, transfer syntax, dimensions, viewport layout and reader experience.

Suggested pilot (proposal, not published validated procedure): compare natural cache presets and, if ethically/practically approved, controlled added latency conditions in randomised order with identical tasks; collect perceived lag, interruption/need-to-wait, acceptable-for-task rating and observations. Define delay levels/participants with supervisors rather than inventing clinical thresholds. Do not equate questionnaire satisfaction with diagnostic safety.

A small pilot yields local exploratory acceptability, not a universal medical standard. If clinician access is unavailable, restrict claims to measured responsiveness/workload support and disclose unvalidated clinical acceptability.

## Directly relevant usability evidence

| ID | Paper and DOI | Findings relevant to this project | Limits |
|---|---|---|---|
| UX-01 | Moise & Atkins, Design Requirements for Radiology Workstations (2004). DOI `10.1007/s10278-004-1003-9` | Eight radiologists interviewed in 1999; study organisation/loading and modality-specific layouts matter; recommends observation/interview and iterative clinician prototype evaluation | Old workstation study; not acceptable-ms thresholds |
| UX-02 | Jorritsma et al., Post-deployment usability evaluation of a radiology workstation (2016; online 2015). DOI `10.1016/j.ijmedinf.2015.11.001` | Interviews and observations of 12 users; 92 usability problems ranging from minor frustration to significant delays/task failure; pre-deployment findings had limited clinical generalisability | Supports evaluation in realistic use, not CT/MRI/DBT numerical latency limits |
| UX-03 | Iannessi et al., A review of existing and potential computer user interfaces for modern radiology (2018). DOI `10.1007/s13244-018-0620-7` | Usability combines effectiveness, efficiency and satisfaction in the specific user/context; 2D/3D tasks and interfaces differ | Review, not controlled modality-specific waiting-time study |
| UX-04 | Moise & Atkins, Designing Better Radiology Workstations: Impact of Two User Interfaces on Interpretation Errors and User Satisfaction (2005). DOI `10.1007/s10278-004-1906-5` | A simulated comparative task with 20 lay participants measured task time/errors and SUS; staged UI improved these outcomes | Lay/simulated task, not clinicians; 17.0 vs 19.7 s is task completion, NOT acceptable viewer delay |

## Technical workload evidence — not user-acceptability limits

| ID | Paper and DOI | Workload use | Limits |
|---|---|---|---|
| W-01 | Investigation of viewing procedures for interpretation of breast tomosynthesis image volumes: a detection-task study with eye tracking (2013; online 2012). DOI `10.1007/s00330-012-2675-z` | DBT experiment used initial cine at 9,14,25 fps plus free scrolling | Rates specify presentation in that task, not acceptable input lag |
| W-02 | Sequential viewing of abdominal CT images at varying rates (1994). DOI `10.1148/radiology.191.1.8134556` | Five radiologists/29 cases; 0.5,1,2,4,7,21 images/s and reader-selectable viewing; selectable was superior | Authors favoured no more than 1-2 fps for fixed-cine primary diagnosis of subtle masses in their task; not universal CT rate or browser latency limit |
| W-03 | Drew et al., Scanners and drillers: Characterizing expert visual search through volumetric images (2013). DOI `10.1167/13.10.3` | 24 radiologists/chest CT; faster drilling and slower scanning strategies | Supports burst/pause/reverse traces, not one correct FPS |
| W-04 | Pillai et al., Reading, Fast and Slow: Characterizing Radiologists' Visual Search Through Abdominal CT for Detecting Hepatic Metastases (2026). DOI `10.1016/j.acra.2026.02.018` | Recent task-specific search study; scrolling rate not associated with sensitivity, longer interpretation time was | Not a latency-tolerance experiment; measurement windows are not playback FPS |
| W-05 | Williams & Drew, What do we know about volumetric medical image interpretation?: a review of the basic science and medical image perception literatures (2019). DOI `10.1186/s41235-019-0171-6` | Task/reader/modality-specific search context including MRI, CT and DBT | No universal acceptable-ms threshold or numerical MRI profile derived |

## Metrics

```text
T_first = first correct render - accepted viewing request
T_visible(N) = last correct render in requested set - accepted request
L_frame = correct requested-frame render - navigation request
```

Record event definition and render-proxy limitations; network/decode completion alone is not display. Include permission/key setup when required. Report median/p95, long stalls and tails, not only average FPS.

For a controlled cine demand r, nominal inter-frame interval is 1000/r ms: 9 fps ≈111.11 ms, 14 fps ≈71.43 ms, 25 fps=40 ms. This is scheduling arithmetic, not a clinician-validated latency target. Prefetch can meet deadlines even if an individual frame's retrieval lasts longer than the interval.

Record on-time presentation, missed frames/deadlines, stalls and actual cadence. Interactive input coalescing must be explicit and logged; controlled cine must not silently drop frames to claim success. Concurrent demand equals the sum of active viewport rates, not necessarily N times the rate of one viewport.

## Benchmark controls

Use P0-P4b with fixed data/representation/codec/Cornerstone/network/layout and comparable traces. Distinguish cold, RAM-warm, OPFS-warm, cross-session warm and restart. Time permission/key, network, decrypt, decode, render and background persistence separately while measuring end-to-end results. Never disable integrity/authorisation to meet a timing goal. MRI numeric tolerances remain TBD.

## Recent contextual evidence not adopted as a clinical threshold

Alhasan & Alhasan (2025), Technical requirements and optimization strategies for home-based teleradiology workstations: a review article, DOI `10.1186/s13244-025-02081-8`, discusses a suggested network round-trip latency below 50 ms. That is network RTT, not measured acceptable input-to-image delay, and not modality-specific. It also reports standardisation gaps. Do not turn its 50 ms statement into a proven clinician tolerance.

## Pending agreement

Exact eight-image event and conditions; usable diagnostic-image definition; rendering proxy; clinician access/ethics and tasks; acceptability rating/protocol; permitted coalescing/skips; whether numerical pass criteria can be established locally.
