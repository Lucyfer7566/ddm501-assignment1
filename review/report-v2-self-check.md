# Report V2 Final Self-Check

## Scope

This self-check evaluates `drafts/report-v2.md` against the 37 IDs in `research/rubric-checklist.md`, the three Phase 7 audits, the fixed scenario, and the established design. “Satisfied” below means the report contains the required content; it does not claim achieved performance or completion of final PDF submission.

## Review-finding resolution

| Reviewed issue | Resolution in report/design | Status |
|---|---|---|
| Final PDF and required filename | These are Phase 8 submission constraints, not defects in the Markdown report. No PDF or identity value was fabricated. | DEFERRED TO FINALIZATION |
| Unmeasurable BM-04 baseline/denominator | Section 3.1 defines numerator, denominator, comparable-history condition, and no-baseline behavior. | RESOLVED |
| Undefined episode/FPR counting | FR-07 and Section 3.3 define entry, update, exit, re-arm, insufficient-data behavior, one-to-one matching, and new-episode motor-day counting. | RESOLVED |
| Inconsistent threshold gates / vague MM-06 | Section 3.3 applies recall, precision, FPR, and F1 during threshold search; MM-06 has one measurable Brier gate and mandatory diagnostic slice disposition. | RESOLVED |
| Weak target rationale | Section 3.1 identifies initial negotiation targets, explains their guardrail purpose, and requires stakeholder approval using prevalence, cost, capacity, and shadow alert volume before untouched evaluation. | RESOLVED |
| Unsupported plant-specific ML claim | Section 1.4 presents nonlinear/temporal signal value as a hypothesis and makes calibration an evaluation requirement. | RESOLVED |
| MQTT acknowledgement/reconciliation ambiguity | Sections 3.2, 4.2.1, and C04 distinguish MQTT transport acknowledgement, application durable receipt, raw persistence, and valid/quarantined/pending dispositions. | RESOLVED |
| `Insufficient Data` routed into risk episode | Revised Mermaid/PNG and Section 4.2.2 route it to a reason/data-health event and dashboard without a risk episode. | RESOLVED |
| Incorrect TO-04 component IDs | Report Section 5.4 and `design/tradeoffs.md` use C09 features, C11 training, C12 evaluation, C13 registry, C14 deployment, and C15 serving. | RESOLVED |
| Monitoring-loss promotion grace conflict | Section 3.4 and C18 block promotion immediately; grace applies only to continued serving of the current approved bundle. | RESOLVED |
| Alert persistence mistaken for delivery | SM-01 and C17 separately require durable dashboard persistence and connector handoff; human reading is not claimed. | RESOLVED |
| On-demand load omitted | Planning, FR-04, NFR-01/NFR-02, and SM-02 include labelled 10% headroom and prevent displacement of scheduled work. | RESOLVED |
| Availability denominator ambiguous | NFR-04 and SM-03 use eligible scheduled opportunities and define maintenance exclusion. | RESOLVED |
| Supporting stakeholders implicit | Section 1.5 names security/identity and label-steward goals and concerns. | RESOLVED |
| Inference preprocessing compressed in diagram | Revised figure separates recent validated data from shared preprocessing/features and completeness/freshness gating. | RESOLVED |
| Raw/feature retention for reconstruction unclear | Sections 2.1, 2.3, 2.4.4, C06, and NFR-07 state a two-year feature/source audit window and disclose excluded capacity. | RESOLVED |
| Site data classification stated as fact | Section 2.4.3 labels operational sensitivity as a design decision pending site classification. | RESOLVED |

## Rubric coverage

| Checklist IDs | Report V2 evidence | Status |
|---|---|---|
| SC-01–SC-07 | Sections 1.2, 1.5, 2.3–2.4, 3, 4, and 5 define the fixed ML task, stakeholders, production lifecycle, data, measurable value, and bounded one-model complexity. | SATISFIED |
| PD-01–PD-05 | Sections 1.1–1.5 cover context, measurable problem, assumed current workflow, conditional ML justification, and primary/supporting stakeholders. | SATISFIED |
| RA-F01–RA-F04 | Section 2.2 specifies prediction, input/output contracts, integrations, user workflow, feedback, and the deterministic episode policy. | SATISFIED |
| RA-N01–RA-N04 | Section 2.3 quantifies latency/load/availability, defines scaling/degradation, and specifies maintainability, monitoring, and controlled retraining. | SATISFIED |
| RA-D01–RA-D04 | Section 2.4 defines plant/public sources, labels, quality and missing-data handling, privacy/security, and volume calculations. | SATISFIED |
| GM-01–GM-05 | Sections 3.1–3.4 provide the hierarchy, owners, windows, business/system/model metrics, baselines, proposed target rationale, threshold selection, and promotion policy. | SATISFIED |
| AR-01–AR-05 | Sections 4.1–4.4 and the revised figure provide data flows, training/inference/feedback paths, 20 component responsibilities, candidate tools, and failure behavior. | SATISFIED |
| TO-01 | Sections 5.1–5.4 analyze exactly four scenario-specific alternatives, advantages, disadvantages, decisions, justifications, and residual risks. | SATISFIED |
| SUB-01 | Final PDF rendering and visual inspection. | PENDING FINALIZATION; not a report-content rubric defect |
| SUB-02 | Required filename using the student's real ID and name. | PENDING USER IDENTITY / FINALIZATION |

## Citation check

- All body citations `[1]`–`[10]` have matching reference entries and matching S01–S10 source-ledger records.
- The independent evidence audit verified source existence against publisher, standards-body, author-organization, or official dataset pages.
- Report V2 introduces no new external source, DOI, dataset, statistic, or achieved result.
- Plant-specific values remain labelled **DESIGN ASSUMPTION** or **PROPOSED DESIGN TARGET** and are not attributed to external literature.

## Remaining status

- **CRITICAL report-content issues:** 0
- **MAJOR report-content issues:** 0
- **Unsatisfied weighted/scenario rubric items:** 0
- **Citations needing verification:** 0
- **Submission-stage constraints pending:** SUB-01 and SUB-02 only
