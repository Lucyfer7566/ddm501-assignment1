# Final Delivery Audit

Audit date: 3 October 2026. Re-audit after the Figure 1 readability fix.

**Submission readiness: PASS.** Figure 1 is readable at normal page size in the current 27-page PDF. The former MAJOR readability finding is resolved. All 37 checklist requirements are satisfied; no CRITICAL or MAJOR issue, missing rubric requirement, or unresolved citation/evidence issue was identified. Three MINOR presentation or tracking findings remain.

## Scope and method

This audit supersedes the previous 2 October 2026 delivery audit. Required inputs were read: the previous `review/final-delivery-audit.md`, `final/report.md`, `final/DDM501_Assignment1_25ms13292_NgoAnhDuc.pdf`, the three-page `assignment/INDIVIDUAL ASSIGNMENT 1.pdf`, and `research/rubric-checklist.md`. `AGENTS.md`, `scenario.md`, the source ledger, BibTeX, and final DOCX/diagram assets were inspected read-only as supplementary evidence. The assignment on disk uses spaces in its filename; that actual file was used without renaming it.

Fresh renders of all 27 PDF pages were produced at 96 dpi, corresponding to an 816 x 1,056 pixel view of a full 8.5 x 11 inch page. All pages were visually screened in contact sheets; Figure 1 on page 18 and the component table on page 23 were also inspected as individual normal-size page images. Figure readability was judged from the full page, without enlarging a crop or substituting the source diagram. Text extraction, page geometry, image placement, source-to-DOCX comparison, and PDF table-column comparisons supplemented visual inspection. Temporary evidence was kept outside the repository.

PDF page references count from the beginning of the file; the document does not print page numbers. The earlier audit's 26-page pagination and artifact hashes no longer identify the current delivery.

Severity meanings:

- **CRITICAL:** invalid submission, fundamental problem-scope corruption, or fabricated evidence.
- **MAJOR:** material rubric or delivery-quality failure requiring correction before submission.
- **MINOR:** presentation or tracking weakness without lost required content.
- **PASS:** the inspected requirement is satisfied.

## Findings and previous-finding disposition

| ID | Result | Location | Evidence and disposition |
|---|---|---|---|
| FD-01 | PASS; former MAJOR resolved | PDF p. 18, Figure 1; AR-01 | Four bands organize acquisition/storage, training, inference, and monitoring/retraining. Component labels, risk outputs, arrow directions, and transfer letters can be read at normal page size. The caption explains transfers A-F and solid/dotted paths. The embedded raster is 1,166 x 1,463 pixels, displayed at approximately 420 x 527 PDF points (5.83 x 7.32 inches), compared with approximately 420 x 330 points previously. Although raster downsampling remains, it no longer prevents reading the diagram. No label overlap or clipping was observed. |
| FD-02 | MINOR; remains | PDF pp. 3-9, 13-16, 20-23 | Some table rows continue over page boundaries, with repeated column headings but no repeated row identifier. Narrow columns break words, including `maximum` on p. 14, `persistence` on p. 20, and `notification` on p. 23. All affected cell text survives; these are navigation and word-wrapping issues. Optional formatting improvement only; no change performed. |
| FD-03 | PASS; former MINOR resolved | `final/report.md`, Figure 1 image reference | The reference now uses `../diagrams/architecture.png`, which resolves from `final/` to the existing root-level asset. The PDF also contains the figure. |
| FD-04 | MINOR; remains | `PLAN.md`, Phase 8; rubric checklist SUB-01/SUB-02 | Plan rendering, visual-inspection, and filename checks remain unchecked. The checklist's historical final-source audit still records SUB-01/SUB-02 as NOT COMPLETE. Current PDF inspection and filename evidence satisfy both delivery requirements. These records were left unchanged under the audit-only scope. |
| FD-05 | MINOR; newly observed after re-render | PDF pp. 17-19 | Section 4.1's introductory text occupies p. 17 with substantial blank space; Figure 1 is on p. 18. The numbered caption starts directly beneath the figure, but its explanatory final sentences continue at the top of p. 19. The full legend remains present and intelligible. Keeping the explanatory caption together would improve presentation; this does not prevent reading the figure or tracing its flows. |

**CRITICAL:** none. **MAJOR:** none. **MINOR:** three (FD-02, FD-04, FD-05). Findings are counted once, even where mentioned again below.

## Figure 1 and architecture

**PASS.** The original normal-page readability failure is resolved on the delivered PDF itself. The visual bands, short labels, repeated transfer letters, and directional arrows expose the main architecture without requiring high zoom. Fine dotted lines are lighter than the component text but remain visible; the caption identifies their meaning.

The diagram and Sections 4.2-4.4 retain all required lifecycle paths:

- Training: governed sensor history and approved outcomes -> validation/curation -> shared preprocessing/features -> XGBoost training -> independent evaluation -> MLflow registry -> human-approved shadow/deployment or rollback.
- Inference: validated recent sensor data -> six-hour window and shared features/validity gate -> cached approved bundle -> 24-hour failure probability -> threshold/hysteresis policy -> Normal/High Risk -> persisted prediction and dashboard; High Risk also produces a durable alert episode. Invalid data follows the Insufficient Data path.
- Feedback: data health, service/model signals, and matured outcomes -> monitoring -> drift/quality/performance investigation -> human-approved Airflow retraining -> batch validation, evaluation, and controlled promotion. Monthly review and validated triggers remain stated.

Transfers A-F are explicitly explained across the caption on pp. 18-19. The registry/deployment boundary, failure rejection, human approval, and monitoring/retraining feedback agree with the narrative and C01-C20 descriptions. Matching transfer letters are intentional connectors between bands, rather than omitted system flows. No inference-to-actuator path appears.

## Rubric completeness

The authoritative assignment weights remain Problem Definition 20%, Requirements Analysis 20%, Goals and Metrics 20%, High-Level Architecture Design 25%, and Trade-offs Analysis 15%. The checklist contains 28 weighted report criteria and nine scenario/submission constraints. **All 37 receive PASS.** Checklist bookkeeping status was assessed separately from delivered content.

| Checklist ID | Current PDF/source evidence | Result |
|---|---|---|
| SC-01 | Section 1.2, p. 2: ML task, target, horizon, outputs, and decision context. | PASS |
| SC-02 | Section 1.5, pp. 3-4: three primary and two supporting stakeholders with distinct concerns. | PASS |
| SC-03 | Sections 2.3 and 4.2-4.4, pp. 7-9 and 19-24: production operations, scaling, reliability, and maintenance. | PASS |
| SC-04 | Section 2.4, pp. 10-11: sources, collection/provenance, quality, and privacy. | PASS |
| SC-05 | Section 3.1, pp. 11-13: measurable downtime, emergency-work, cost, and intervention outcomes. | PASS |
| SC-06 | Sections 4.2-4.4 and 5, pp. 19-26: complete lifecycle and meaningful production decisions. | PASS |
| SC-07 | Sections 1.2 and 4.1, pp. 2 and 17: one bounded primary classifier; comparators remain baselines. | PASS |
| PD-01 | Section 1.1, p. 1: sourced background and explicitly assumed organization. | PASS |
| PD-02 | Section 1.2, p. 2: measurable motor population, inputs, binary target, horizon, and cadence. | PASS |
| PD-03 | Section 1.3, p. 2: assumed as-is maintenance workflow and limitations. | PASS |
| PD-04 | Section 1.4, pp. 2-3: conditional ML justification, rule/logistic baselines, data limitations. | PASS |
| PD-05 | Section 1.5, pp. 3-4: stakeholder roles, actions, and concerns. | PASS |
| RA-F01 | FR-01/FR-04, pp. 5-6: core prediction, model scope, cadence, and demand triggers. | PASS |
| RA-F02 | FR-02/FR-03, p. 5: input contract and output class/probability/context. | PASS |
| RA-F03 | FR-05/FR-08, pp. 6-7: integration interfaces and outcome linkage. | PASS |
| RA-F04 | FR-06-FR-08, pp. 6-7: dashboard use, action/episode workflow, and feedback. | PASS |
| RA-N01 | NFR-01/NFR-02, pp. 7-8: latency, response time, throughput, units, and load assumptions. | PASS |
| RA-N02 | NFR-02/NFR-03, p. 8; Section 4.4, p. 24: initial/growth load and scaling. | PASS |
| RA-N03 | NFR-04-NFR-06, pp. 8-9; Section 4.4, p. 24: availability, failure handling, buffering, and degradation. | PASS |
| RA-N04 | NFR-07-NFR-09, p. 9; Sections 4.2.3/4.4, pp. 19-20 and 24: updates, traceability, monitoring, controlled retraining. | PASS |
| RA-D01 | Section 2.4.1, p. 10: motor telemetry and authoritative maintenance/outcome sources. | PASS |
| RA-D02 | Section 2.4.2, p. 10: quality gates, freshness, missingness, imputation, and partitions. | PASS |
| RA-D03 | Section 2.4.3, p. 11: classification, access, minimization, retention, and protection. | PASS |
| RA-D04 | Section 2.4.4, p. 11: rate/volume/retention calculations and excluded overhead. | PASS |
| GM-01 | Section 3.1, pp. 11-13: business/system/model hierarchy, owners, windows, and dependencies. | PASS |
| GM-02 | Business-goal table, p. 12: downtime, emergency work, cost, unnecessary interventions. | PASS |
| GM-03 | Section 3.2, pp. 13-15: SM-01-SM-05 service, delivery, availability, reconciliation, and coverage metrics. | PASS |
| GM-04 | Section 3.3, pp. 15-16: MM-01-MM-06, units/denominators, episode matching, threshold selection. | PASS |
| GM-05 | Sections 3.1-3.4, pp. 11-16: proposed justified thresholds, comparator baselines, and promotion gates. | PASS |
| AR-01 | Figure 1, p. 18, and caption pp. 18-19: clear labelled architecture readable at normal page size. | PASS |
| AR-02 | Figure 1 and Section 4.2, pp. 18-20: directional data, model-bundle, prediction, and feedback flows. | PASS |
| AR-03 | Sections 4.2.1-4.2.3, pp. 19-20: ingestion, preprocessing, training/evaluation/registry, inference, monitoring/retraining. | PASS |
| AR-04 | Section 4.3, pp. 20-23: C01-C20 purpose, responsibilities, inputs/outputs, failure behavior. | PASS |
| AR-05 | Section 4.3, pp. 20-23: candidate technologies and scenario-specific rationale. | PASS |
| TO-01 | Sections 5.1-5.4, pp. 24-26: four meaningful engineering trade-offs. | PASS |
| SUB-01 | Current 27-page PDF opened, text-checked, freshly rendered, and visually inspected. | PASS |
| SUB-02 | `DDM501_Assignment1_25ms13292_NgoAnhDuc.pdf` matches the required populated filename pattern. | PASS |

Filename inspection verifies the assignment naming convention, not independent authentication of enrollment records. No additional page-count, paper-size, or mandatory example-trade-off restriction appears in the inspected assignment.

## Document integrity after re-render

**PASS.** The PDF opens without repair in MuPDF, is unencrypted, and all 27 pages render successfully. Every page has extractable text. Text-span geometry checks found no spans outside page boundaries, and visual screening found no clipped or overlapping report text, blank pages, or damaged figure placement.

| Integrity check | Fresh verification | Result |
|---|---|---|
| Source-to-DOCX content | All 349 normalized source units are present in the DOCX. Units include headings, prose, lists, table headings/cells, captions, and references; Markdown syntax, whitespace, and typographic quote conversion are normalized. | PASS |
| PDF body/caption/reference preservation | 333 of 349 units match the normalized PDF text contiguously. The remaining 16 are table cells interrupted by page breaks and repeated column headings, rather than lost content. | PASS |
| PDF tables | All eight tables and all 215 table body cells match their corresponding PDF column text streams after normalizing whitespace/quotes/line-break hyphens and removing repeated headers. | PASS |
| Sections | Sections 1-5 and their subsections occur in the intended order; Section 6 References follows. Architecture moves to pp. 17-24, trade-offs to pp. 24-26, and references to pp. 26-27. | PASS |
| Identifiers | FR-01-FR-08, NFR-01-NFR-10, BG-01-BG-03, BM-01-BM-04, SM-01-SM-05, MM-01-MM-06, and C01-C20 sets agree between source and PDF. | PASS |
| Citations | The source and PDF contain the same ordered sequence of 32 numeric citation-marker occurrences, including the ten reference labels. | PASS |
| Characters and markup | No U+FFFD replacement character, literal Markdown artifacts, TODO, TBD, DRAFT, or FIXME found in report/PDF text. Mathematical symbols and accented names survive; no en/em dash occurs in extracted report/PDF text. Figure labels were checked visually because they are raster content. | PASS |
| Image portability/presence | Source-relative image path resolves; the embedded figure is present on p. 18. Source PNG dimensions are 1,913 x 2,400; embedded raster dimensions are 1,166 x 1,463. | PASS |

The evidence supports preservation of the current source through delivery. It does not claim byte identity with the superseded PDF or certify a printer/viewer that was not tested. Remaining table and caption pagination weaknesses are the MINOR findings above.

## Problem, metrics, and trade-off consistency

**PASS.** The fixed task remains binary classification of industrial-motor failure risk within the next 24 hours. `Normal` and `High Risk` remain prediction classes; `Insufficient Data` is a service/data-quality state. The six-hour feature lookback is distinct from the `(t, t + 24 hours]` outcome horizon. One XGBoost primary model is proposed, with deterministic and logistic comparators outside production serving. No RUL, generic-anomaly, multi-class, or multi-model scope change was introduced.

Business goals remain distinct from system and model goals, with stated owners, measurement windows, baselines, denominators, and proposed gates. Downtime reduction of at least 15%, emergency labor reduction of at least 10%, the 5% cost guardrail, and the 20% non-actionable-intervention limit remain proposed targets. The p95 two-minute visibility, 99.5% opportunity availability, 95% valid coverage, and 440/2,200 hourly prediction envelopes remain tied to explicit planning assumptions.

Model gates retain event recall >=85%, episode precision >=50%, F1 >=0.63, negative-motor-day FPR <=1%, PR-AUC at least twice held-out prevalence and above practical baselines, and Brier score below the constant-prevalence baseline. Threshold freezing, untouched evaluation, uncertainty/slice reporting, intervention censoring, and human approval remain intact. Volume arithmetic and the rounded F1 floor are internally consistent. No model score, latency, uptime, deployment, or business saving is presented as achieved.

Four trade-offs remain complete: false negatives versus false positives; edge versus central/cloud inference; freshness versus infrastructure cost; and model complexity/performance versus latency/maintainability. Each includes alternatives, benefits, disadvantages/risks, scenario-specific stakeholder reasoning, an explicit design decision, and residual risks. The decisions align with the proposed architecture and acceptance gates.

## Citation and evidence integrity

**PASS.** All body citation numbers [1]-[10] resolve to one matching final reference. Source identities and claim scope agree with the verified ledger and BibTeX. Fixed scenario inputs, externally supported facts, design assumptions, design decisions, and proposed targets remain distinguishable. No unsupported factual performance number or fabricated result was found.

Fresh checks of accessible publisher, university, standards, government, and author-organization records support the existing reference set. Direct DOI/IEEE/ACM/Elsevier access was restricted for some records; available publisher search records, institutional metadata, and the prior verified ledger were used rather than claiming newly read full text. Access restrictions alone are not citation defects.

| Reference | Evidence role and verification record | Result |
|---|---|---|
| [1] Jardine et al., 2006 | Acquisition/processing/maintenance-decision lifecycle; [Elsevier record](https://www.sciencedirect.com/science/article/abs/pii/S0888327005001512). | PASS |
| [2] Nandi et al., 2005 | Motor monitoring modalities; ledger/DOI identity and [IEEE's discussion of the original review](https://ieeexplore.ieee.org/document/9875201/). Direct original-article access restricted. | PASS |
| [3] Randall and Antoni, 2011 | Bearing vibration diagnostics and speed-aware processing; [Elsevier record](https://www.sciencedirect.com/science/article/pii/S0888327010002530). | PASS |
| [4] Carvalho et al., 2019 | Data/problem-dependent predictive-maintenance method choice; [Elsevier record](https://www.sciencedirect.com/science/article/pii/S0360835219304838). | PASS |
| [5] Lessmeier et al., 2016 | Motor-current/bearing benchmark; [PHM publication](https://papers.phmsociety.org/index.php/phme/article/view/1577) and [official Paderborn dataset](https://mb.uni-paderborn.de/en/kat/research/bearing-datacenter/data-sets-and-download). | PASS |
| [6] CWRU Bearing Data Center | Motor-bearing vibration prototype data; [official university overview](https://engineering.case.edu/bearingdatacenter/welcome). | PASS |
| [7] MQTT Version 5.0, 2019 | Publish/subscribe transport and delivery semantics; [OASIS standard](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html). | PASS |
| [8] NIST SP 800-82r3, 2023 | OT security, reliability, and physical-process constraints; [NIST publication](https://www.nist.gov/publications/guide-operational-technology-ot-security). | PASS |
| [9] Breck et al., 2017 | Production ML testing/monitoring beyond offline scores; [Google Research publication](https://research.google/pubs/the-ml-test-score-a-rubric-for-ml-production-readiness-and-technical-debt-reduction/). | PASS |
| [10] Gama et al., 2014 | Concept drift and supervised input/target relationship; [author-institution metadata](https://repositorio.inesctec.pt/items/fa4e59eb-b0cf-44f0-994a-c6634999d435/full). | PASS |

Paderborn and CWRU remain limited to prototypes; neither is asserted to supply native plant-wide 24-hour failure labels or prove this system's performance. Design hypotheses remain subject to validation rather than being attributed as achieved literature results.

## Audit-only preservation

Only `review/final-delivery-audit.md` was overwritten. No report content, DOCX, PDF, architecture source, image, scenario, assignment, checklist, or plan was modified. SHA-256 checks before and after writing the audit confirm preservation of the protected delivery artifacts and authoritative inputs. Temporary renders and comparison evidence remain outside the repository.

| Audited artifact | SHA-256 |
|---|---|
| `final/report.md` | `cb9c8f025ff463776b29d251a6263c768c7e13759ac531987c99206d3eb0b7fd` |
| `final/DDM501_Assignment1_25ms13292_NgoAnhDuc.docx` | `301708159697976837bd06f575faeee7f1b665b435af55f6a4b10f6966cdfaed` |
| `final/DDM501_Assignment1_25ms13292_NgoAnhDuc.pdf` | `641f12d7b713cb967c7efb3c99fe58513011528db33d5684c027177af0777483` |
| `diagrams/architecture.mmd` | `db78fd942f7bcbfedeb0385adb0d0de568c8ef62edd4cd24ff636be5d74d62eb` |
| `diagrams/architecture.png` | `954b74c7a93e15dc797ded225289069a9b70fb6ac80b31e403ed5a6fc0ea6fb7` |
| `diagrams/architecture.svg` | `dd6cebb8483f53b3e1fef9b355aeb0f859845e36dcc0555384ed5f726e00781f` |
| `assignment/INDIVIDUAL ASSIGNMENT 1.pdf` | `ed13920ad8ef27a5213503a7d43d2b45101d4e1389aba592dfbc4c2ecfd87a54` |

CRITICAL issues remaining: 0
MAJOR issues remaining: 0
Missing rubric requirements: 0
Unresolved citation/evidence issues: 0
Submission readiness: PASS
