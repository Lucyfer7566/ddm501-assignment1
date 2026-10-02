# Phase 3B — Goals and Metrics

## 1. Purpose and status of targets

This document defines the prospective goal hierarchy and acceptance criteria for the hypothetical industrial-motor predictive-maintenance system. It does not report trained-model results, production measurements, cost savings, or deployment outcomes.

The fixed task remains binary classification: for each valid motor scoring window, estimate the probability that the motor will fail within the next 24 hours and return `Normal` or `High Risk`. The metrics below evaluate whether that prediction can support the business objective without creating an unreliable pipeline or excessive maintenance burden.

Every numeric threshold in this document is a **PROPOSED DESIGN TARGET**, not an externally established fact. The assignment, scenario, and verified source set provide no plant-specific business baseline or acceptable performance threshold. Targets must therefore be reviewed with maintenance, operations, and ML/platform stakeholders and revised if pilot evidence shows that the assumed operating envelope or error costs are unsuitable.

Numeric baseline windows and observation periods (including the 12-month historical baseline and minimum six-month pilot) are **DESIGN ASSUMPTIONS** selected to make evaluation prospective and reproducible. They are not claims about available plant history or adequate statistical power.

Source IDs refer to `research/sources.md`. Requirement IDs refer to `design/requirements.md`.

## 2. Measurement and baseline policy

### 2.1 Baseline periods and comparators

- **Business baseline:** use the same motors’ most recent 12 complete months before pilot deployment, normalized by motor operating hours. Planned shutdowns and periods with incomplete maintenance records shall be identified consistently in baseline and pilot periods. If 12 usable months are unavailable, the plant must approve a shorter documented period; no value may be invented.
- **Business evaluation:** use a minimum six-month pilot and continue observation if the number of failures or interventions is insufficient for a stable comparison. Where practical, compare pilot motors with matched non-pilot motors or use a staggered rollout to reduce the risk of attributing seasonal, workload, or maintenance-policy changes to the ML system.
- **System baseline:** no deployed-system baseline exists. Pre-production load, fault-injection, and acceptance tests establish the first measured values. The Phase 3A requirements are the acceptance comparators.
- **Model baselines:** evaluate the candidate on the same time- and asset-separated data against (1) an `Always Normal` policy, (2) an `Always High Risk` policy, (3) a maintenance-approved deterministic sensor-threshold rule, and (4) an interpretable logistic-regression baseline. Development baselines do not violate the one-primary-model deployment constraint.
- **No public benchmark substitution:** Paderborn and CWRU results cannot serve as achieved baselines for the 24-hour target because neither dataset provides the required production population and label [S05, S06; DR-08].

### 2.2 Evaluation units

The 15-minute scoring cadence creates overlapping 24-hour labels, so treating every window as an independent operational alert would overstate evidence and alert burden. Evaluation shall use the following units:

- A **scoring window** is one valid motor prediction opportunity. Window-level scores support threshold-independent metrics such as PR-AUC and diagnostic slice analysis.
- A **high-risk alert episode** follows the frozen FR-07 state machine: entry at `T_enter`, updates while active, closure after two consecutive valid scores at or below `T_exit`, no state change on `Insufficient Data`, and re-entry only after closure.
- A **failure event** is an approved event under DR-02 and DR-07. Failures are processed chronologically and matched one-to-one to the earliest unmatched episode beginning in the preceding 24 hours. A continuously active earlier episode is not moved retrospectively into the target window.
- A **negative motor-day** has adequate observation, no approved failure in the following 24 hours, and no censoring condition. `FP_day = 1` when at least one new unmatched episode starts that day; a continuing episode is not counted again. Otherwise `TN_day = 1`.
- An alert episode followed by preventive intervention is **censored for post-deployment failure-event precision/recall unless the approved label policy supplies a valid observed outcome**. It shall not automatically become a false positive merely because the intervention may have prevented failure. Its maintenance disposition remains part of BM-04.

All model acceptance metrics shall be calculated on untouched, time- and asset-separated test data, with uncertainty intervals and results by relevant motor/operating-regime slices. Performance from overlapping windows, random leakage-prone splits, or auxiliary public datasets shall not be used for promotion [DR-07, DR-08, DR-11].

## 3. Explicit goal hierarchy

### H-01 — Reduce unplanned motor downtime

**Business Goal BG-01** reduce unplanned motor downtime  
→ **System Goals SG-01/SG-02/SG-03/SG-05** deliver timely alerts, preserve prediction-path availability, and maintain valid prediction coverage
→ **Model Goal MG-01** detect as many genuine 24-hour failure events as practicable  
→ **Metrics BM-01, SM-01–SM-03, MM-01**  
→ **Baselines** historical downtime rate; no deployed-system baseline; `Always Normal`, deterministic-rule, and logistic baselines  
→ **Proposed targets** ≥15% relative downtime-rate reduction; system service targets from Phase 3A; failure-event recall ≥85%.

### H-02 — Reduce emergency maintenance burden without increasing total cost materially

**Business Goal BG-02** reduce emergency corrective work and associated burden/cost  
→ **System Goals SG-01/SG-04** provide actionable alerts and reliable data delivery  
→ **Model Goals MG-01/MG-02** find impending failures while keeping alerts sufficiently precise  
→ **Metrics BM-02, BM-03, SM-01, SM-04, MM-01, MM-02**  
→ **Baselines** historical emergency work-hours/cost; pre-production reconciliation tests; deterministic-rule and logistic baselines  
→ **Proposed targets** ≥10% relative reduction in emergency corrective labor-hours per 1,000 motor operating hours, with total motor-maintenance cost per 1,000 hours no more than 5% above baseline; recall ≥85% and precision ≥50%.

### H-03 — Avoid unnecessary maintenance and alert fatigue

**Business Goal BG-03** avoid unnecessary interventions and excessive false alarms  
→ **System Goals SG-01/SG-05** deduplicate alert episodes and monitor user-visible alert burden  
→ **Model Goals MG-02/MG-03** control false alert episodes while preserving recall  
→ **Metrics BM-04, SM-05, MM-02–MM-04**  
→ **Baselines** historical condition-triggered “no actionable defect” rate where available; `Always High Risk`, deterministic-rule, and logistic baselines  
→ **Proposed targets** ≤20% of ML-initiated completed interventions find no actionable condition; precision ≥50%; F1 ≥0.63; negative-motor-day false-positive rate ≤1%.

### H-04 — Operate a maintainable, evidence-based ML service

**Business Goals BG-01–BG-03** depend on trustworthy prospective evaluation  
→ **System Goals SG-02–SG-04** meet latency, availability, and data-pipeline objectives  
→ **Model Goals MG-04/MG-05** demonstrate prevalence-aware discrimination and useful probability estimates  
→ **Metrics SM-02–SM-04, MM-05, MM-06**  
→ **Baselines** Phase 3A service targets; held-out prevalence; deterministic-rule/logistic comparators; constant-prevalence probability predictor  
→ **Proposed targets** Phase 3A service thresholds; PR-AUC at least twice the held-out positive-window prevalence and above both learned/rule baselines; Brier score lower than the constant-prevalence baseline.

## 4. Goal-to-metric-and-target map

| Goal ID | Goal | Supporting downstream goal | Metric ID(s) | Baseline/comparator | Target / acceptable threshold | Status |
|---|---|---|---|---|---|---|
| BG-01 | Reduce unplanned motor downtime. | SG-01, SG-02, SG-03; MG-01 | BM-01 | Measured pre-pilot 12-month downtime rate for the same motor population. | ≥15% relative reduction after the pilot measurement period. | PROPOSED DESIGN TARGET |
| BG-02 | Reduce emergency maintenance burden/cost without shifting excessive cost elsewhere. | SG-01, SG-04; MG-01, MG-02 | BM-02, BM-03 | Measured pre-pilot emergency labor-hours and total motor-maintenance cost per 1,000 operating hours. | Emergency labor-hours reduced ≥10%; total maintenance cost no more than 5% above baseline. | PROPOSED DESIGN TARGET |
| BG-03 | Avoid unnecessary maintenance and false-alarm-driven work. | SG-01, SG-05; MG-02, MG-03 | BM-04, SM-05, MM-02–MM-04 | Use a historical comparator only if condition-triggered interventions use the same disposition definition; otherwise report no comparable baseline and measure the pilot guardrail. | No-actionable-condition interventions ≤20%; precision ≥50%; F1 ≥0.63; negative-motor-day FPR ≤1%. | PROPOSED DESIGN TARGET |
| SG-01 | Deliver each valid new/updated high-risk alert promptly and once per episode. | MG-01, MG-02 | SM-01 | No deployed baseline; test against FR-07 and NFR-01. | ≥99% delivered within 5 minutes; duplicates do not create new alert episodes. | PROPOSED DESIGN TARGET |
| SG-02 | Keep scoring and dashboard response timely. | MG-01 | SM-02 | First pre-production load test; NFR-01 is the comparator. | p95 ≤2 minutes from window close to dashboard visibility; normal-path maximum <15 minutes. | PROPOSED DESIGN TARGET, inherited from NFR-01 |
| SG-03 | Keep the prediction/dashboard path available. | MG-01 | SM-03 | First measured pre-production and pilot month; no current service exists. | ≥99.5% monthly availability, excluding approved maintenance. | PROPOSED DESIGN TARGET, inherited from NFR-04 |
| SG-04 | Preserve timely, reconcilable telemetry through the data pipeline. | MG-01–MG-05 | SM-04 | Gateway message manifest in acceptance tests; first measured pilot month. | 100% reconciliation in one-hour initial/growth load tests; ≥99.5% of acknowledged normal-operation messages available for processing within 5 minutes; zero unexplained loss. | PROPOSED DESIGN TARGET |
| SG-05 | Produce valid scores for an adequate share of scheduled windows and expose data gaps. | MG-01–MG-05 | SM-05 | First shadow-mode month, with ineligible/censored windows reported separately. | ≥95% of eligible scheduled motor-windows yield a valid prediction; all others expose a reason code rather than `Normal`. | PROPOSED DESIGN TARGET |
| MG-01 | Detect genuine failures within the fixed 24-hour horizon. | BG-01, BG-02 | MM-01 | `Always Normal` recall = 0; also compare deterministic-rule and logistic baselines on the same test set. | Failure-event recall ≥85%. | PROPOSED DESIGN TARGET |
| MG-02 | Ensure high-risk episodes are sufficiently actionable. | BG-02, BG-03 | MM-02 | `Always High Risk` precision equals evaluation prevalence; compare deterministic-rule and logistic baselines. | Alert-episode precision ≥50%. | PROPOSED DESIGN TARGET |
| MG-03 | Balance recall and precision while directly limiting false alerts. | BG-03 | MM-03, MM-04 | `Always Normal`: F1 = 0 and FPR = 0; simulate `Always High Risk` and practical baselines through the frozen episode state machine. | F1 ≥0.63 and negative-motor-day FPR ≤1%, while MM-01 and MM-02 also pass. | PROPOSED DESIGN TARGET |
| MG-04 | Demonstrate useful threshold-independent ranking on imbalanced data. | H-04 | MM-05 | Held-out positive-window prevalence is the no-skill PR-AUC baseline; also compare rule and logistic baselines. | PR-AUC ≥2× held-out prevalence, above both practical baselines, with the 95% bootstrap interval reported. | PROPOSED DESIGN TARGET |
| MG-05 | Make the reported failure-risk probability useful for prioritization. | H-04 | MM-06 | Constant probability equal to training prevalence; compare its Brier score on the same test data. | Candidate Brier score lower than the constant-prevalence baseline, with calibration plots reviewed by operating-regime slice. | PROPOSED DESIGN TARGET |

## 5. Business goals and metrics

Business results shall be attributed cautiously. A reduction observed after deployment is not automatically caused by the ML system: production volume, motor mix, planned shutdowns, staffing, maintenance policy, and sensor coverage may change. Report numerator, denominator, baseline period, pilot period, and exclusions.

| Metric ID | Business metric | Definition and direction | Baseline | Proposed target | Measurement window / owner |
|---|---|---|---|---|---|
| BM-01 | Unplanned motor downtime rate | `unplanned motor downtime hours / motor operating hours × 1,000`; lower is better. Only approved motor-related unplanned stops are included. | Same motors’ measured pre-pilot 12-month rate; not currently available. | ≥15% relative reduction. | Minimum six-month pilot, then rolling 12 months; Plant Operations Manager. |
| BM-02 | Emergency corrective maintenance burden | `emergency corrective motor-maintenance labor-hours / motor operating hours × 1,000`; lower is better. | Measured pre-pilot 12-month rate; not currently available. | ≥10% relative reduction. | Minimum six-month pilot and rolling 12 months; Maintenance Manager/Engineer. |
| BM-03 | Total motor-maintenance cost guardrail | `(labor + parts + contracted motor-maintenance cost) / motor operating hours × 1,000`; lower is preferable and material increases are unacceptable. Downtime loss is reported separately to avoid double counting. | Measured pre-pilot 12-month cost in a consistent currency and accounting scope. | No more than 5% above baseline while BM-01 improves; otherwise the business case fails review. | Minimum six-month pilot and rolling 12 months; Plant Operations/Finance owner. |
| BM-04 | Unnecessary ML-initiated intervention rate | `ML-initiated completed inspections/work orders closed with no actionable condition / all ML-initiated completed inspections/work orders with a recorded disposition`. Lower is better. An intervention that finds a documented actionable condition is not called unnecessary merely because failure was prevented. | Use a historical rate only if existing condition-triggered interventions use the same definition; otherwise report no comparable baseline and treat the first pilot measurement against the absolute guardrail without claiming improvement. | ≤20%. | Monthly with rolling six-month view; Maintenance Engineer. |

The numeric business targets are deliberately treated as pilot exit criteria, not promises. If the baseline records cannot distinguish motor downtime, planned work, emergency work, and intervention disposition reliably, the business metrics are not yet evaluable and data remediation precedes any claim of benefit.

## 6. System goals and metrics

| Metric ID | System metric | Definition and measurement point | Baseline | Proposed target | Requirement consistency / owner |
|---|---|---|---|---|---|
| SM-01 | Alert delivery success | Percentage of valid new/updated `High Risk` episodes both durably persisted to the dashboard and handed successfully to the configured notification connector within 5 minutes. Human reading is not claimed; retries update the same episode. | No deployed baseline; establish in end-to-end acceptance test. | ≥99% within 5 minutes and zero duplicate active episodes for the same motor/horizon. | FR-07, NFR-01; ML/Platform Engineer. |
| SM-02 | End-to-end inference latency | Time from scheduled feature-window close or accepted on-demand request to successful prediction visibility, measured at p50, p95, p99, and maximum. | First load-test distribution; no current service exists. | p95 ≤2 minutes and normal-path maximum <15 minutes at the 440/2,200 total hourly prediction envelopes. | NFR-01, NFR-02; ML/Platform Engineer. |
| SM-03 | Prediction-path availability | `eligible scheduled opportunities persisted and retrievable before the next cycle / all eligible scheduled opportunities`; approved maintenance is removed from numerator and denominator. | First measured month; no current service exists. | ≥99.5% per calendar month. | NFR-04; ML/Platform Engineer. |
| SM-04 | Data-pipeline reliability | Reconcile every application message ID to one durable raw record and terminal valid, quarantined, or pending disposition after deduplication; separately measure valid-data processing delay. MQTT acknowledgement is not storage proof. | Acceptance-test manifest and first pilot month. | 100% terminal reconciliation in the one-hour 100/500-motor load tests; ≥99.5% of valid records available for processing within 5 minutes; zero unexplained loss. | NFR-02, NFR-03, NFR-06, DR-03–DR-05; Platform/Data owner. |
| SM-05 | Valid scheduled-prediction coverage | `valid predictions / eligible scheduled motor-windows`; `Insufficient Data`, censored, and ineligible windows are reported separately rather than counted as `Normal`. | First shadow-mode month. | ≥95% coverage, with 100% of non-valid attempts carrying a reason code and data-health visibility. | FR-03, NFR-05, DR-06; ML/Platform Engineer. |

The 5-minute alert-delivery objective does not replace the stricter p95 two-minute inference target: it provides a tail allowance for alert persistence/notification while retaining the NFR-01 latency gate. Availability also does not conceal data quality; a responsive service returning `Insufficient Data` remains available but reduces SM-05 coverage.

## 7. Model goals and metrics

### 7.1 Operating-threshold metrics

Using the one-to-one alert-episode/failure-event matching policy in Section 2.2:

- `TP` = matched high-risk alert episodes/failure events.
- `FP` = unmatched high-risk alert episodes.
- `FN` = unmatched approved failure events.
- `TN_day` = eligible negative motor-days with no new unmatched alert episode.
- `FP_day` = eligible negative motor-days with at least one new unmatched alert episode.

| Metric ID | Model metric | Formula / interpretation | Baseline | Proposed target |
|---|---|---|---|---|
| MM-01 | Failure-event recall | `TP / (TP + FN)`. Fraction of approved failures with a matched alert episode beginning in the preceding 24 hours. | `Always Normal` = 0; measure deterministic-rule and logistic baselines. | ≥85%. |
| MM-02 | Alert-episode precision | `TP / (TP + FP)`. Fraction of new high-risk episodes matched to an approved failure. | `Always High Risk` equals the event prevalence induced by the episode policy; measure practical baselines. | ≥50%. |
| MM-03 | F1 score | `2 × precision × recall / (precision + recall)`. Summary of the balance at the selected operating point; it does not replace the separate recall and precision gates. | `Always Normal` = 0; measure rule and logistic baselines. | ≥0.63. This is the approximate minimum implied when recall is 0.85 and precision is 0.50. |
| MM-04 | Negative-motor-day false-positive rate | `FP_day / (FP_day + TN_day)`. Fraction of eligible negative motor-days that start at least one new unmatched episode under the frozen state machine. | `Always Normal` = 0; simulate `Always High Risk`, rule, and logistic policies through the same state machine rather than asserting a universal value. | ≤1%. |

### 7.2 Threshold-independent and probability metrics

| Metric ID | Model metric | Definition / interpretation | Baseline | Proposed target |
|---|---|---|---|---|
| MM-05 | PR-AUC | Area under the window-level precision–recall curve on untouched test data. It is used because precision exposes alert purity and recall exposes missed positive windows under an imbalanced target. | Positive-window prevalence on the held-out set; deterministic-rule and logistic comparators. | ≥2× held-out prevalence and above both practical baselines; report the 95% bootstrap interval. |
| MM-06 | Brier score and calibration review | Mean squared error between predicted probability and the binary 24-hour outcome; lower is better. Reliability plots and counts are mandatory overall and by predeclared motor/operating-regime slice. | Constant probability equal to training prevalence, evaluated on the same test set. | Overall Brier score lower than the constant-prevalence baseline. Slice plots require written disposition but are not an unsupported numerical gate. |

Window-level confusion metrics shall also be reported for diagnostic transparency, but the alert-episode/failure-event metrics are the promotion gates because they better represent the user-visible workflow. All metrics require counts as well as ratios; a percentage based on too few failure events must be reported as uncertain, not presented as proof of readiness.

Offline promotion metrics shall use retrospective or shadow-mode outcomes that were not changed by model-triggered maintenance. After operational use begins, intervention-censored episodes are reported separately; they are neither silently discarded nor forced into the false-positive count.

## 8. Recall priority, false positives, and alert fatigue

Failure-risk detection prioritizes recall because a false negative leaves an impending failure without an ML warning and can contribute directly to unplanned downtime, the primary business objective. The proposed threshold therefore starts from the recall requirement rather than maximizing overall accuracy. Accuracy is not an acceptance metric because a rare-failure dataset can make an `Always Normal` policy appear strong while detecting no failures.

Recall cannot be maximized without constraints. An `Always High Risk` policy has perfect recall but creates unmanageable false alarms, weak precision, unnecessary inspection, loss of trust, and possible neglect of later alerts. The operating threshold shall therefore be selected on validation data as follows:

1. Apply the versioned alert deduplication/hysteresis policy from FR-07.
2. Identify thresholds meeting recall ≥85%.
3. Reject thresholds with alert-episode precision <50%, negative-motor-day FPR >1%, or F1 <0.63.
4. Among the remaining thresholds, select the one with the highest recall; use expected alert volume, lead-time distribution, and calibration as secondary comparisons.
5. Freeze the threshold and episode policy before evaluation on the untouched test set.
6. If no threshold passes all gates, do not promote the model; improve data, features, labels, or the modelling approach rather than silently relaxing the criteria.

This policy connects model errors to operational burden. MM-02 and MM-04 control false alerts; FR-07 prevents repeated 15-minute scores from becoming separate notifications; SM-01 checks delivery without duplication; and BM-04 measures whether human review still results in unnecessary work.

## 9. Acceptance and promotion gate

A candidate primary model and its threshold may proceed to a controlled shadow/pilot deployment only when all of the following are documented:

1. the label definition and failure-event source are approved under DR-02 and DR-07;
2. training, validation, and test separation satisfies DR-11;
3. MM-01 through MM-06 meet their proposed targets on untouched test data or an explicitly approved target revision is recorded before testing;
4. SM-01 through SM-05 pass representative load, integration, and fault-injection tests;
5. results include denominators, uncertainty intervals, motor/operating-regime slices, and comparison with all declared baselines;
6. the threshold, episode policy, model, preprocessing, dataset, and evaluation code are versioned;
7. rollback and monitoring requirements in NFR-07 through NFR-09 are ready; and
8. no metric is described as achieved production performance before a valid pilot measurement exists.

Business metrics BM-01 through BM-04 are pilot outcome criteria, not prerequisites for shadow deployment. Failure to meet them triggers business/design review rather than retrospective alteration of model metrics.

## 10. Rubric and requirements traceability

| Rubric item | Coverage in this document |
|---|---|
| GM-01 — clear hierarchy | Sections 3 and 4 map business → system → model → metric → baseline → target. |
| GM-02 — business goals | Section 5 defines downtime, emergency burden/cost, cost guardrail, and unnecessary intervention metrics. |
| GM-03 — system goals | Section 6 defines alert delivery, latency, availability, pipeline reliability, and prediction coverage. |
| GM-04 — model goals | Section 7 defines recall, precision, F1, false-positive rate, PR-AUC, and probability quality. |
| GM-05 — thresholds and baselines | Sections 2, 4, 7, 8, and 9 define comparators, proposed thresholds, selection policy, and acceptance gates. |

| Phase 3A requirement | Phase 3B consistency |
|---|---|
| FR-01 / DR-07 — binary failure within 24 hours | All model outcomes and event matching retain the fixed 24-hour horizon. |
| FR-04 / PA-02 — 15-minute scoring | Evaluation groups repeated 15-minute scores into alert episodes; cadence is unchanged. |
| FR-07 — deduplicated alert lifecycle | SM-01 and the model evaluation policy count an alert episode rather than every repeated score. |
| NFR-01 — p95 ≤2 minutes; normal maximum <15 minutes | SM-02 adopts both thresholds; SM-01 adds a compatible 5-minute alert-delivery objective. |
| NFR-02 — initial/growth load | SM-02 and SM-04 require acceptance at the 100- and 500-motor envelopes. |
| NFR-04 — 99.5% monthly availability | SM-03 adopts the same target and exclusion rule. |
| NFR-05 / DR-06 — missing/stale data | SM-05 separates `Insufficient Data` from `Normal` and measures valid coverage. |
| NFR-08 / NFR-09 — monitoring and controlled promotion | Sections 7–9 require uncertainty, slices, baselines, calibration, monitoring, and a gated promotion process. |
| DESIGN DECISION-04 / DR-08 — no benchmark result becomes a project result | Public data are excluded as achieved 24-hour performance baselines. |

## 11. Open dependencies

These proposed targets resolve OPEN QUESTION-08 and OPEN QUESTION-09 only provisionally. Before production claims are possible, the plant must still provide or approve:

- the exact operational definition and cost of motor failure;
- trustworthy operating-hour, downtime, labor, parts-cost, intervention, and work-order dispositions;
- failure prevalence and enough delayed outcomes to estimate uncertainty;
- the acceptable daily alert workload by maintenance team and motor criticality; and
- any stakeholder-authorized revision to the proposed targets before the untouched test set is examined.
