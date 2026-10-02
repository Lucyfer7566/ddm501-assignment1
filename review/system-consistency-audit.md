# Independent ML System Consistency Audit

Scope: `scenario.md`, `research/assumptions.md`, `research/rubric-checklist.md`, all four `design/` documents, `drafts/report-v1.md`, and the Mermaid source used by the report figure. This is a design consistency review, not a claim that the hypothetical system has been built or measured. Line numbers refer to the current Markdown/Mermaid sources.

## CRITICAL Findings

None. The report preserves the required binary motor-failure-risk target, next-24-hour horizon, four primary sensor modalities, and one deployed primary predictive model. The findings below must still be resolved before the design can be treated as internally testable.

## MAJOR Findings

### M-01 — Alert-episode lifecycle is too undefined to reproduce the promotion metrics

`drafts/report-v1.md:71,149,153-160,240` requires one active episode per motor/horizon and matches a failure only when the *episode begins* within the preceding 24 hours. `design/goals-metrics.md:29-33,130-135` uses that episode for recall, precision, F1, false-positive rate, and the `Always High Risk` comparator. `design/architecture.md:54-55,104-105` implements hysteresis and deduplication, but none of these sources specifies when an episode closes, when a new one may start, how hysteresis resolves a temporary dip, or what happens if a continuously active warning started more than 24 hours before a failure. As written, the same probability sequence could produce different TP/FP/FN counts and different `Always High Risk` precision depending on an unstated episode policy. Define a versioned start, close, expiry, and re-arm state machine; specify event matching and motor-day assignment at boundaries; and freeze it before threshold selection and test evaluation.

### M-02 — Telemetry reconciliation mixes broker acknowledgement, durable landing, and validation

`drafts/report-v1.md:144` and `design/goals-metrics.md:113` require reconciliation of gateway-acknowledged message IDs to validated/persisted records with zero unexplained loss. Yet `design/architecture.md:77-79` says MQTT broker acknowledgement is not proof of storage, while `drafts/report-v1.md:182,203,221` says ingestion lands raw records before acknowledgement without identifying which acknowledgement reaches the gateway. In addition, `drafts/report-v1.md:101` and `design/requirements.md:203-205` intentionally quarantine invalid messages, so they cannot all become *validated* records. Define the acknowledgement boundary and an application-level durable-receipt mechanism if that is the intended guarantee. Reconcile every accepted message ID to exactly one durable raw record and a valid, quarantined, or explicitly pending disposition after deduplication; measure validated availability separately.

### M-03 — Diagram sends `Insufficient Data` through the risk-alert path

The report embeds `diagrams/architecture.png` at `drafts/report-v1.md:172`. Its source, `diagrams/architecture.mmd:55-61`, routes invalid/stale data to a combined `Normal or High Risk or Insufficient Data` node and then unconditionally to `Immutable prediction and durable alert episode`. This conflicts with the separate non-risk response in `drafts/report-v1.md:67,188`, the data-health behavior in `design/architecture.md:52`, and the rule that a high-risk transition creates the alert in `design/requirements.md:79-81`. Split the diagram so `Insufficient Data` persists a reason/data-health event and reaches the dashboard without creating or updating a high-risk episode; the valid class path alone should reach risk-episode policy.

### M-04 — TO-04 and its matrix use incorrect component IDs

`design/tradeoffs.md:166` calls C07 the shared feature package, C10 the evaluation gate, C12 the registry, and C13/C14 the promotion path; `design/tradeoffs.md:179` calls C09 the primary model and C10 evaluation. The actual catalogue and report define C07 as the operational store, C09 as the shared feature package, C10 as orchestration, C11 as training, C12 as evaluation, C13 as registry, and C14 as deployment (`design/architecture.md:86,88,94-98`; `drafts/report-v1.md:206-214`). These references direct a reviewer to the wrong controls and undermine trade-off-to-architecture traceability. Correct the IDs throughout TO-04 and the matrix, then check the other trade-off references against the component catalogue.

### M-05 — Monitoring-loss promotion rule differs between the report and architecture

The report's C18 row says monitoring loss blocks promotion **after** an authorized grace period (`drafts/report-v1.md:217`). The architecture says missing required monitoring evidence blocks deployment/promotion while only *current approved serving* may continue temporarily (`design/architecture.md:112,147`). The report's acceptance gate also requires monitoring readiness (`drafts/report-v1.md:164`). Assign the grace period solely to continued serving, or explicitly justify and specify a separate promotion exception. Until then, the same observability outage can yield opposite release decisions.

### M-06 — Alert delivery metric can pass while user notification is unavailable

`design/goals-metrics.md:82,110` and `drafts/report-v1.md:141` call SM-01 an alert-delivery goal, but define success only as persistence to the maintenance dashboard. The operating path persists first and then notifies (`drafts/report-v1.md:188,216,227`); a notifier outage can therefore satisfy SM-01 while no engineer receives the intended notification. Distinguish durable dashboard availability from notification dispatch/delivery, and define which event meets the stakeholder-facing delivery objective. Keep retries and one-active-episode counting separate from notification attempts.

## MINOR Findings

### m-01 — Threshold-selection steps differ on whether F1 is a constraint

`drafts/report-v1.md:160` and `design/goals-metrics.md:154-159` select among thresholds using recall, precision, and negative-motor-day FPR, with F1 only a secondary comparison. `drafts/report-v1.md:240` and `design/tradeoffs.md:43` say validation thresholds must also satisfy F1 ≥0.63. Even though the F1 bound is close to that implied by the recall and precision minima, the two procedures can pick different thresholds near the boundary. Make the algorithm and untouched-test gate state the same four mandatory conditions.

### m-02 — Goal hierarchy mislabels valid-coverage goal

`design/goals-metrics.md:41-46` says H-01's SG-03 preserves valid prediction coverage, whereas SG-03 is availability and SG-05 is valid coverage in `design/goals-metrics.md:84,86,112,114`. The report's hierarchy paraphrases the concepts without IDs (`drafts/report-v1.md:131`), so the error has not changed the target, but it weakens the source hierarchy and traceability. Change the H-01 link to SG-05 for coverage and retain SG-03 for availability.

### m-03 — Dashboard filters and closure reason lack explicit component acceptance mapping

FR-06 requires filters for site/area, risk state, data status, and time (`design/requirements.md:69-75`); FR-07 also requires a closure reason (`design/requirements.md:77-83`). The report summarizes rank/filter and closure (`drafts/report-v1.md:70-71`), but C17 in `design/architecture.md:106` and `drafts/report-v1.md:216` names general presentation/workflow without enumerating those query and audit fields. Add them to C17's input/output or a linked UI acceptance contract so every requested interaction has an implementable owner.

### m-04 — Raw-data retention needed for reconstruction is not explicit

The report promises prediction reconstruction from retained lineage (`drafts/report-v1.md:84`) and immutable raw landing (`drafts/report-v1.md:182,205`), while the two-year cold-retention statement explicitly lists curated telemetry, labels, predictions, and audit data (`drafts/report-v1.md:59`). `design/architecture.md:85` assigns lifecycle rules to the raw object store without a raw-record retention duration. State how long the exact raw inputs or an equivalent immutable, replayable feature snapshot remain available; otherwise the reproducibility claim has an undefined time limit. Keep any added capacity outside the current 2.80 GB/day logical envelope until measured.

## PASS Items

- **Target/model scope:** `scenario.md:9-20,56-65`, `drafts/report-v1.md:17-19,170,186-188`, and `design/architecture.md:5-7,53-54` agree on binary Normal/High Risk plus probability, failure within the next 24 hours, and one deployed XGBoost primary model. Logistic/rule comparators are non-production baselines.
- **Labels and leakage:** `design/requirements.md:225-231` and `drafts/report-v1.md:93-105,149` agree on `(t, t+24 hours]`, full-horizon observability for negatives, censoring, and at-or-before-`t` features. MM-05/MM-06 use window-level binary outcomes; MM-01–MM-04 use operational event/episode units (`drafts/report-v1.md:149-158`).
- **Planning envelope:** 100/500 motors, one-second scalar envelopes, one-minute vibration packets, six-hour lookback, 15-minute scoring, 24-hour edge buffer, and 90-day hot/two-year cold retention align across `design/requirements.md:21-25,249-255`, `design/tradeoffs.md:123-129`, and `drafts/report-v1.md:53-59,78-83,113-119,258`. The stated 2.80 GB/day and fivefold 14.0 GB/day logical estimates follow the declared payload assumptions; they are planning values, not observations.
- **Core paths and failure behavior:** The report contains explicit training, inference, monitoring, and controlled retraining paths (`drafts/report-v1.md:178-194`). The prose preserves `Insufficient Data` for invalid windows, last-approved model serving during registry outage, and human-gated promotion (`drafts/report-v1.md:186-194,227`). The diagram exception is M-03.
- **Trade-off decisions:** TO-01 through TO-04 in the report select constrained recall, central on-premises inference with edge feature extraction, bounded cadence/retention, and one engineered-feature XGBoost model (`drafts/report-v1.md:233-267`). These choices match the architecture at the decision level; the component-ID defect is M-04.
- **Stakeholder boundary:** Maintenance reviews and acts on advisory alerts, Operations owns downtime/cost/pilot decisions, and ML/Platform owns evidence and release operation; joint approval and no actuator path appear in `drafts/report-v1.md:39-45,135,188` and `design/architecture.md:43,56`.

## Traceability Matrix

| Design chain | Requirement / assumption | Goal and metric | Architecture owner | Trade-off | Report evidence | Audit result |
|---|---|---|---|---|---|---|
| Fixed 24-hour binary target and one model | `scenario.md:9-20,56-65`; FR-01, DR-07 | MG-01–MG-05; MM-01–MM-06 | C08–C16 | TO-01, TO-04 | `drafts/report-v1.md:17-19,93-95,149-160,170,180-188` | PASS; episode semantics M-01 |
| User-visible action and decision rights | FR-03, FR-06–FR-08 | BG-01–BG-03; BM-01–BM-04; SM-01 | C16–C17, C20 | TO-01 | `drafts/report-v1.md:39-45,65-72,131-145,188,240` | PASS in principle; M-06, m-03 |
| Telemetry and data quality | FR-02, FR-05; DR-01–DR-06 | SG-04–SG-05; SM-04–SM-05 | C01–C09, C18 | TO-02, TO-03 | `drafts/report-v1.md:91-105,182-188,200-208` | M-02; diagram M-03 |
| Capacity, cadence, retention | PA-01–PA-05; NFR-01–NFR-03, NFR-06, DR-10 | SG-02–SG-04; SM-02–SM-04 | C02–C07, C15 | TO-02, TO-03 | `drafts/report-v1.md:53-59,78-83,111-119,225,249-258` | PASS on planning arithmetic; m-04 |
| Reliability and degradation | NFR-04–NFR-06 | SG-03, SG-05; SM-03, SM-05 | C02–C07, C13–C18 | TO-02 | `drafts/report-v1.md:81-86,188,223-229` | PASS in prose; M-03, M-05 |
| Training, evaluation, release | DR-07–DR-08, DR-11; NFR-07, NFR-09 | MG-01–MG-05; MM-01–MM-06 | C08–C14, C19 | TO-04 | `drafts/report-v1.md:95-105,162-164,178-182,207-214,260-267` | PASS in report; M-04 in design trace |
| Monitoring and retraining feedback | NFR-08–NFR-09 | SM-01–SM-05; delayed MM-01–MM-06 | C18–C19 | TO-01, TO-02, TO-04 | `drafts/report-v1.md:85-86,190-194,217-219` | PASS on loop; M-05 release rule |

The first revision priority is to formalize the episode policy and reconciliation contract, then correct the diagram and component references. Those changes make the existing acceptance metrics and release gates reviewable without changing the fixed ML task.
