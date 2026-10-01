# Phase 5 — Trade-off Analysis

## Scope and decision status

This document analyzes exactly four engineering trade-offs for the fixed task: predicting whether an industrial motor is at high risk of failure within the next 24 hours. It does not report achieved model, system, or business performance. Numeric acceptance criteria are the **PROPOSED DESIGN TARGETS** already defined in `design/goals-metrics.md`; plant scale, sampling, retention, and timing values are the **DESIGN ASSUMPTIONS** recorded in `research/assumptions.md` and `design/requirements.md`.

Source identifiers refer to the verified ledger in `research/sources.md`. Requirement, goal, metric, and component identifiers refer respectively to `design/requirements.md`, `design/goals-metrics.md`, and `design/architecture.md`.

## TO-01 — False negatives versus false positives

**Engineering question.** Where should the operating threshold sit between missing genuine 24-hour failure risks and creating false alerts, unnecessary inspections, and alert fatigue?

**Option A.** Use a recall-first, lower decision threshold so that more genuine failure events are warned about, accepting more false-positive alert episodes.

**Option B.** Use a precision-first, higher decision threshold so that alerts are less frequent and more likely to be actionable, accepting more missed failures.

**Advantages of A.**

- Better supports BG-01 and BG-02 because fewer impending failures should pass without an ML warning.
- Gives the Maintenance Engineer more opportunities to inspect a motor before the fixed 24-hour horizon expires.
- Aligns model selection with the scenario's primary purpose rather than with overall accuracy, which is not an acceptance metric.

**Disadvantages of A.**

- Increases unmatched alert episodes, investigation workload, and the chance of unnecessary maintenance.
- Can erode engineer trust and cause alert fatigue, undermining BG-03 even if offline recall is high.
- May shift cost from emergency work to excessive preventive work, violating the BM-03 cost guardrail.

**Advantages of B.**

- Produces a smaller, more concentrated alert queue for the Maintenance Engineer.
- Reduces false-alarm-driven inspections and protects the Plant Operations Manager's maintenance-cost constraint.
- Makes each alert easier to treat as an operational priority.

**Disadvantages of B.**

- More failure events can occur without an alert, directly weakening BG-01 and MG-01.
- Avoidable missed failures may preserve emergency maintenance burden and unplanned downtime.
- A very high threshold could make the service operationally quiet but ineffective.

**Scenario-specific consequences.** The Plant Operations Manager bears the downtime consequence of a false negative, while the Maintenance Engineer bears much of the repeated-alert and inspection burden created by false positives. The ML/Platform Engineer must therefore expose both error types by alert episode and motor-day rather than optimize an abstract accuracy score. Because a score is produced every 15 minutes, an ungrouped policy could turn one persistent condition into many notifications; FR-07 and architecture component C16 must apply hysteresis and deduplication before user-visible alerts. The advisory-only workflow in FR-06 limits direct safety or control consequences, but it does not eliminate wasted work or missed-warning risk.

**Selected design decision.** **DESIGN DECISION:** adopt Option A only within explicit false-positive constraints. Select the highest-recall validation threshold that achieves MM-01 failure-event recall of at least 85%, while also achieving MM-02 alert-episode precision of at least 50%, MM-04 negative-motor-day false-positive rate of at most 1%, and MM-03 F1 of at least 0.63. Freeze the threshold and alert-episode policy before untouched testing. If no threshold passes every gate, do not deploy the candidate. C16 shall add hysteresis and one-active-episode deduplication, and C17 shall present the result for human review rather than initiate autonomous control.

**Justification.** This constrained recall-first policy connects the primary downtime goal (BG-01) and emergency-work goal (BG-02) to a measurable missed-failure limit while protecting BG-03 and BM-03 from an unconstrained `Always High Risk` behavior. It implements FR-03, FR-06, and FR-07; uses the probability and threshold outputs of C15 and C16; and makes the Maintenance Engineer, Plant Operations Manager, and ML/Platform Engineer responsible for different observable parts of the same decision. Condition-based maintenance is an end-to-end decision process rather than a model score alone [S01], and production readiness requires monitoring behavior beyond offline model accuracy [S09].

**Residual risk.** The proposed gates may be unsuitable once plant prevalence, failure cost, and team capacity are measured. Rare failures may make estimates uncertain; motor or operating-regime slices may have different error rates; and an alert followed by preventive action can censor the counterfactual outcome. These risks remain visible through uncertainty intervals, slice reporting, BM-04 disposition review, and the intervention-censoring policy. Any threshold change requires controlled validation and approval rather than retrospective tuning on the test set.

## TO-02 — Edge inference versus centralized/cloud inference

**Engineering question.** Should complete failure-risk inference run independently at each plant gateway, or should gateways forward validated telemetry/features to one centrally governed inference service?

**Option A.** Deploy the preprocessing package and primary model to every edge gateway, producing probability and risk class locally.

**Option B.** Keep acquisition, durable buffering, and versioned vibration feature extraction at the gateways, but perform final feature assembly and primary-model inference in a centralized service. For this design, the central service is on-premises rather than in a public cloud.

**Advantages of A.**

- Allows local scoring during loss of the central network path, provided the gateway and required inputs remain healthy.
- Avoids sending every raw high-rate vibration sample beyond the gateway.
- Shortens the network portion of the scoring path.

**Disadvantages of A.**

- Multiplies deployment, rollback, model-version, and health-monitoring responsibilities across gateways.
- Makes consistent feature/model promotion and fleet-wide troubleshooting harder for the ML/Platform Engineer.
- Requires gateway resources and lifecycle controls for full inference, increasing the risk of version drift between motors or production lines.

**Advantages of B.**

- Provides one governed deployment point for the approved model, threshold, shared feature contract, monitoring, and rollback.
- Fits the assumed 100-motor initial and 500-motor growth load without introducing a separate inference runtime at every gateway.
- Simplifies integration of historian/work-order context and makes fleet-level dashboards and drift slices consistent.
- Retains bandwidth control by calculating high-rate vibration features at C02 rather than forwarding unrestricted raw streams.

**Disadvantages of B.**

- Depends on the gateway-to-central path and the availability/capacity of the central broker, feature, and inference services.
- A central fault can affect many motors at once.
- Requires controlled transport and storage of operational telemetry; moving the service to a public cloud would add an unapproved OT boundary and connectivity dependency.

**Scenario-specific consequences.** The 24-hour prediction horizon and 15-minute cadence do not require millisecond local control, while the architecture must still tolerate temporary network interruption. Maintenance users need a consistent fleet view; Plant Operations needs continued data capture; and the ML/Platform Engineer needs auditable model promotion and rollback. MQTT's delivery semantics require explicit application-level deduplication and reconciliation rather than an assumption of exactly-once end-to-end storage [S07]. OT systems also have distinctive performance, reliability, safety, topology, and security constraints that must shape connectivity choices [S08]; S08 does not establish a plant-specific cloud prohibition or availability target.

**Selected design decision.** **DESIGN DECISION:** choose Option B as the initial production architecture: centralized on-premises inference in C15, with C02 performing sensor acquisition, a 24-hour durable buffer, and versioned vibration feature extraction. Do not deploy the primary risk model to gateways and do not use public-cloud inference at this stage. C03, C04, and C05 provide brokered delivery, validation, idempotent persistence, and replay; missing or stale windows return `Insufficient Data` rather than a fabricated `Normal` result.

**Justification.** Centralized inference best supports NFR-07 maintainability, NFR-08 monitoring, NFR-09 controlled promotion, and the C12/C14 registry-and-deployment controls. Gateway buffering satisfies NFR-06, while the cached last-approved central model and rollback behavior support NFR-04. The decision also respects the assumed load in NFR-02 and the advisory workflow in FR-06 without paying the operational cost of a distributed model fleet. It gives the Maintenance Engineer one consistent dashboard, the Plant Operations Manager a bounded OT deployment, and the ML/Platform Engineer one governed inference release path.

**Residual risk.** A central outage or an interruption longer than the gateway buffer can suspend current scores, and a central capacity defect has a wider blast radius than one edge failure. The system therefore exposes data freshness, queue lag, availability, and `Insufficient Data` state; buffers and replays recoverable messages; and rolls back to the last approved model. A site connectivity and security assessment may later justify edge failover, but that would be a separately evaluated architecture change, not a silent second deployment path.

## TO-03 — Prediction/data freshness versus infrastructure cost

**Engineering question.** How much sensor and scoring freshness is justified for a 24-hour advisory prediction before bandwidth, storage, compute, and alert-processing cost become disproportionate?

**Option A.** Stream and retain high-rate raw telemetry centrally and score continuously or at sub-minute intervals.

**Option B.** Use bounded acquisition and aggregation rates, edge-derived vibration features, scheduled 15-minute scoring plus on-demand scoring, a six-hour feature window, and tiered retention.

**Advantages of A.**

- Can expose sensor changes sooner and preserves richer raw signals for later feature development and diagnosis.
- Provides more frequent opportunities to update the risk estimate.
- Reduces the scheduling interval between a material signal change and the next score.

**Disadvantages of A.**

- Increases network, storage, feature-computation, and monitoring demand for every motor.
- More frequent correlated scores can create alert churn unless lifecycle logic becomes more complex.
- The project has no evidence that sub-minute central scoring improves the fixed 24-hour decision enough to justify its additional operational burden.

**Advantages of B.**

- Bounds capacity using the PA-04/DR-10 planning envelope and makes initial and growth-load testing reproducible.
- Supports stable rolling features and a manageable alert lifecycle for maintenance users.
- Allows selective raw-vibration capture for investigation without making unrestricted raw streaming a normal-path dependency.
- Tiered retention keeps recent operational data readily accessible while controlling long-term hot-storage growth.

**Disadvantages of B.**

- A rapidly developing condition may not be reflected until the next scheduled score.
- Aggregated vibration features can discard signatures that a future raw-signal model might use.
- The assumed sampling, lookback, and retention policy may prove too coarse or too expensive after actual device and workload measurement.

**Scenario-specific consequences.** The prediction supports maintenance planning within 24 hours, not real-time motor protection. The Maintenance Engineer needs timely, stable episodes more than a flood of near-identical scores; the Plant Operations Manager needs a defensible infrastructure cost; and the ML/Platform Engineer needs bounded throughput at both 100 and 500 motors. A freshness policy must still meet NFR-01 and SM-01/SM-02 after each window closes, while DR-06 must suppress invalid predictions when the necessary data are incomplete or stale.

**Selected design decision.** **DESIGN DECISION:** choose Option B. Under the existing **DESIGN ASSUMPTIONS**, acquire scalar readings at one record per motor-second, generate one vibration feature packet per motor-minute at C02, build a six-hour feature window at C07, and score every 15 minutes plus on demand through C15. Retain 90 days in hot operational storage and two years in cold storage, with raw vibration retained locally or uploaded selectively under policy. Preserve the **PROPOSED DESIGN TARGET** of p95 two minutes from window close to dashboard visibility and a normal-path maximum below 15 minutes.

**Justification.** The decision supports BG-02/BG-03 by limiting infrastructure and alert-handling burden without changing the fixed 24-hour model target. It implements FR-04, NFR-01, NFR-02, DR-05, and DR-10 and corresponds to C02, C05, C06, C07, and C15. The explicit assumptions make capacity tests repeatable and prevent an unsupported claim that the selected rates are optimal. Operations retains a cost-bounded platform, maintenance receives a predictable alert cadence, and the platform owner receives measurable load and latency envelopes.

**Residual risk.** Fast-onset failures could develop inside a 15-minute interval, and the chosen six-hour history may omit useful longer-term degradation. Actual payload sizes, compression, sensor capabilities, network limits, and retention obligations may invalidate the planning estimates. The pilot must measure event lead time, valid-prediction coverage, queue lag, storage growth, and missed-event timing; rates or windows may then be revised through versioned requirements and validation without changing the 24-hour target silently.

## TO-04 — Model complexity/performance versus latency and maintainability

**Engineering question.** Should the primary model learn directly from raw or lightly processed temporal signals, or use a governed set of engineered features with a tree-based classifier?

**Option A.** Train a more complex temporal/deep-learning model using raw or lightly processed multivariate sequences.

**Option B.** Train one XGBoost binary classifier on versioned engineered sensor, trend, operating-state, and data-quality features; retain deterministic rules and logistic regression only as evaluation baselines.

**Advantages of A.**

- May learn temporal representations and cross-sensor patterns not captured by manually selected summary features.
- Preserves an avenue for higher predictive performance if sufficient representative labeled histories and compute become available.
- Reduces dependence on a fixed hand-designed feature set.

**Disadvantages of A.**

- Requires a more demanding raw-sequence ingestion, training, serving, and monitoring contract.
- Adds compute, tuning, versioning, and diagnostic complexity without evidence that it improves this plant's exact 24-hour task.
- Makes feature attribution and failure investigation harder for maintenance and platform stakeholders.

**Advantages of B.**

- Fits the bounded central tabular inference path and proposed latency/capacity envelope.
- Allows training and inference to share one versioned feature package, reducing skew and simplifying lineage, tests, and rollback.
- Supports probability calibration, threshold evaluation, missingness indicators, and slice monitoring within a maintainable release process.
- Is compatible with the architecture's one-primary-model constraint while still permitting fair non-production baselines.

**Disadvantages of B.**

- Depends on feature engineering and may miss temporal information retained in raw waveforms.
- Can inherit bias or instability from aggregation choices, data-quality patterns, and motor-to-motor feature differences.
- Does not guarantee that the proposed recall, precision, calibration, or PR-AUC gates will be met.

**Scenario-specific consequences.** The project currently defines an operational target and data contracts but has no verified plant dataset or achieved model comparison. The Maintenance Engineer benefits from stable, inspectable feature trends; the Plant Operations Manager needs timely, dependable scoring; and the ML/Platform Engineer must reproduce, monitor, retrain, and roll back the entire preprocessing/model bundle. Published reviews support matching methods to the available data and operational problem rather than assuming one universally best model [S04]. Public bearing benchmarks can support pipeline development, but their damage types and training representativeness limit transfer to this plant's native 24-hour label [S05].

**Selected design decision.** **DESIGN DECISION:** choose Option B. Use one primary XGBoost classifier in C09/C15 with the shared, versioned C07 feature package. Evaluate it against the declared rules and logistic-regression baselines, calibrate and threshold it under C10's gates, register the complete bundle in C12, and promote it through C13/C14 only if MM-01 through MM-06 and relevant system tests pass. Do not deploy an ensemble or a second risk model.

**Justification.** This is the simplest candidate that satisfies FR-01's one-model behavior while supporting NFR-01 latency, NFR-07 maintainability, NFR-08 monitoring, and NFR-09 controlled retraining. It is consistent with the training and inference paths in C07–C15 and with the no-promotion rule when the target gates cannot be met. The decision values demonstrated end-to-end fitness over speculative algorithmic sophistication and retains evidence from baselines without turning those baselines into production models. Production-ML readiness also depends on reproducible pipelines, monitoring, and operational controls beyond a single offline score [S09].

**Residual risk.** Engineered features and XGBoost may fail the proposed model gates, calibrate poorly, or generalize unevenly across motors and regimes. Feature extraction can also hide useful waveform structure. Such failure does not justify relaxing gates after seeing the test result: it triggers data/label review and a new controlled comparison, which may include a temporal model if representative histories and infrastructure evidence support it.

## Cross-trade-off consistency and traceability

| Trade-off | Business goals | Stakeholder consequence | Requirements and metrics | Architecture alignment |
|---|---|---|---|---|
| TO-01 — error balance | BG-01, BG-02, BG-03; BM-03, BM-04 | Operations receives missed-failure protection; maintenance receives alert-burden constraints; platform owns threshold evidence. | FR-03, FR-06, FR-07; MM-01–MM-04; SM-01 | C15 probability, C16 threshold/hysteresis/deduplication, C17 advisory dashboard |
| TO-02 — inference location | BG-01–BG-03 | Maintenance gets one fleet view; operations retains local capture; platform gets central governance and rollback. | NFR-02, NFR-04, NFR-06–NFR-09; DR-04, DR-09 | C02 edge buffer/features; C03–C05 delivery; C12/C14/C15 central on-premises serving |
| TO-03 — freshness/cost | BG-02, BG-03 | Operations receives a bounded cost envelope; maintenance receives stable alert cadence; platform gets testable throughput. | FR-04; NFR-01, NFR-02; DR-05, DR-06, DR-10; SM-01, SM-02, SM-05 | C02 acquisition/features, C05/C06 storage, C07 windows, C15 scheduled/on-demand scores |
| TO-04 — model complexity | BG-01–BG-03 | Maintenance gets inspectable trends; operations gets bounded serving; platform gets reproducible retraining and rollback. | FR-01; NFR-01, NFR-07–NFR-09; MM-01–MM-06 | C07 shared features, C09 primary model, C10 evaluation, C12–C15 promotion and serving |

Together, the decisions preserve one coherent system: bounded edge processing feeds centralized on-premises inference; the single XGBoost model produces a probability every 15 minutes for valid windows; a constrained recall-first threshold produces deduplicated advisory episodes; and monitoring/retraining may change a released bundle only through the documented validation and approval path. None of the decisions changes the binary target or 24-hour horizon.
