# Phase 3A — Requirements Analysis

## 1. Purpose and interpretation

This document specifies prospective requirements for the hypothetical industrial-motor predictive-maintenance system. It is a design artifact, not evidence of a deployed system. The fixed task is binary classification: predict whether a motor is at high risk of failure within the next 24 hours and, for a valid inference, return `Normal` or `High Risk` together with a failure-risk probability.

The classifications below use the repository convention:

- **FACT** — a statement supported by an authoritative project input or an external source.
- **DESIGN ASSUMPTION** — a provisional condition or target introduced because no plant-specific evidence is available.
- **DESIGN DECISION** — an engineering choice made for this proposed system.

Numeric design assumptions are prospective acceptance or capacity-planning values. They are not measured results. They must be confirmed with plant stakeholders and replaced or revised after telemetry profiling, workflow observation, and load testing. Source IDs refer to `research/sources.md`.

## 2. Requirement-stage planning assumptions

The assignment and scenario provide no fleet size, cadence, latency, availability, sampling, retention, or volume values. The following assumptions make the requirements measurable and internally consistent.

| ID | Provisional assumption and justification |
|---|---|
| PA-01 | **100 instrumented motors initially, with capacity validation at 500 motors.** This gives a concrete initial load and a fivefold growth test without claiming a real fleet size. |
| PA-02 | **Scheduled scoring every 15 minutes per motor, plus an authorized on-demand score.** A 15-minute cadence produces four scores per motor-hour and is operationally timely relative to the fixed 24-hour prediction horizon without implying hard-real-time control. |
| PA-03 | **A provisional six-hour lookback window.** Six hours provides recent trend context while bounding online feature computation. Model development must compare alternative windows and approve the final value before production. |
| PA-04 | **Central sizing uses one approximately 256-byte normalized scalar record per motor-second and one approximately 4 KiB vibration feature packet per motor-minute.** These are engineering envelopes, including identifiers and quality metadata, not sensor specifications. Raw high-frequency vibration retention and edge processing remain architecture decisions and must be sized from an actual sensor survey. |
| PA-05 | **Ninety days of hot telemetry and two years of curated cold telemetry, labels, predictions, and audit records.** This provisional split supports recent investigation and longer-horizon model development while allowing older high-volume data to use a lower-cost tier. Plant policy may require a different retention period. |

## 3. Functional requirements

### FR-01 — Produce the fixed failure-risk prediction

- **Requirement statement:** For every valid scoring window, the system shall use one primary predictive model to estimate whether the identified industrial motor is at high risk of failure within the next 24 hours.
- **Rationale:** This is the fixed ML task and prevents scope drift into remaining-useful-life prediction, generic anomaly detection, multi-class diagnosis, or a multi-model solution.
- **Measurable criterion:** An acceptance test submits a valid window and verifies that exactly one versioned primary model is invoked and that the prediction horizon recorded with the result is 24 hours.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** `scenario.md`; DESIGN DECISION-01 and DESIGN DECISION-03 in `research/assumptions.md`; assignment PDF, Functional Requirements.

### FR-02 — Accept a versioned, validated inference input

- **Requirement statement:** The inference pipeline shall accept a versioned input contract containing motor and sensor identifiers, event and ingestion timestamps, units, quality flags, and recent vibration, temperature, electrical-current, and RPM measurements. RPM shall be available as operating context for speed-aware processing. The provisional online lookback is six hours (PA-03).
- **Rationale:** The model must associate measurements with the correct asset and time interval. Speed changes affect interpretation of rotating-equipment signals, while schema and unit metadata prevent silent incompatibility.
- **Measurable criterion:** Contract tests reject unknown schema versions, missing asset identity, invalid timestamps or units, and malformed values; accepted records preserve source and quality metadata through feature generation.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** `scenario.md`; S02, S03, S05; FACT-02, FACT-03, DESIGN ASSUMPTION-03, and DESIGN ASSUMPTION-04.

### FR-03 — Return an actionable and traceable output

- **Requirement statement:** A valid inference shall return the class (`Normal` or `High Risk`), a failure-risk probability in `[0,1]`, motor ID, prediction timestamp, 24-hour horizon, source-window interval, model version, feature/preprocessing version, and data-quality status. Invalid or stale input shall return no risk class or probability and shall instead expose `Insufficient Data`; it shall never be converted silently to `Normal`.
- **Rationale:** The fixed outputs support prioritization, while identity, time, version, and quality context make the result auditable. Separating inference failure from a normal prediction avoids false reassurance.
- **Measurable criterion:** Schema tests verify every field, probability bounds, allowed labels, version traceability, and mutually exclusive valid-prediction versus `Insufficient Data` responses.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** `scenario.md`; DESIGN DECISION-02; S09 supports production checks beyond offline accuracy.

### FR-04 — Support scheduled and on-demand scoring

- **Requirement statement:** The system shall score each eligible motor every 15 minutes (PA-02) and shall support an authenticated on-demand request from the maintenance application. Repeated requests for the same motor, window, and model version shall be idempotent.
- **Rationale:** Scheduled scoring provides consistent coverage; on-demand scoring supports investigation without creating a different ML task. Idempotency prevents retry-driven duplicate results.
- **Measurable criterion:** Under PA-01, the scheduler creates 400 initial and 2,000 growth-case scheduled predictions per hour with no duplicate prediction keys; an authorized on-demand request produces or retrieves the corresponding traceable result.
- **Classification:** DESIGN ASSUMPTION.
- **Supporting source/basis:** PA-01 and PA-02; the cadence and load are not specified by external evidence.

### FR-05 — Integrate ingestion, storage, serving, and maintenance systems

- **Requirement statement:** The system shall provide versioned interfaces between (a) sensors/edge gateways and ingestion, (b) ingestion and centralized telemetry storage, (c) storage and the training/feature pipeline, (d) the feature pipeline and inference service, and (e) prediction storage and the maintenance dashboard/alert service. A CMMS/EAM or equivalent work-order/failure-event interface is assumed for outcomes and maintenance feedback.
- **Rationale:** Predictive maintenance requires acquisition, processing, and maintenance decision-making rather than a standalone classifier. The outcome interface is necessary to create and later evaluate 24-hour labels.
- **Measurable criterion:** End-to-end integration tests trace a uniquely identified telemetry event to stored data, a prediction, a displayed dashboard record, and—when available—a linked maintenance outcome; interface failures are observable and retryable.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** `scenario.md`; S01, S07; DESIGN ASSUMPTION-02 and OPEN QUESTION-07. The named plant products remain to be confirmed.

### FR-06 — Present a maintenance-engineer workflow

- **Requirement statement:** The dashboard shall let a maintenance engineer view fleet risk ordered by operational priority, filter by motor, site/area, risk state, data status, and time, inspect probability and recent sensor trends, see model/data freshness, and open the associated maintenance record. The prediction shall be advisory and shall not autonomously shut down equipment.
- **Rationale:** The prediction only creates value when a user can assess and act on it. Keeping the output advisory preserves a human decision boundary while safety authority is unspecified.
- **Measurable criterion:** A role-based user-acceptance test demonstrates discovery of a high-risk motor, inspection of its context and freshness, navigation to the maintenance record, and denial of autonomous control actions.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** `scenario.md`; S01; DESIGN ASSUMPTION-07.

### FR-07 — Manage alert lifecycle without repeated alert storms

- **Requirement statement:** On a transition into `High Risk`, the alert service shall create or update one active alert per motor and prediction horizon rather than emitting a new alert at every scoring cycle. The dashboard shall support acknowledgement, assignment, comment, investigation status, closure reason, and escalation according to configurable plant policy. Risk thresholds, hysteresis, and escalation rules shall be configuration controlled and finalized in Phase 3B rather than embedded in code.
- **Rationale:** Persistent lifecycle state supports accountable action and limits repeated notifications. Configurable thresholds avoid prematurely inventing a model operating point or plant escalation policy.
- **Measurable criterion:** Repeated high-risk predictions for the same active episode update one alert; state changes are timestamped with actor identity; threshold/configuration changes are versioned and audited.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** `scenario.md` dashboard/alert concept; S09; OPEN QUESTION-08. No achieved alert threshold is claimed.

### FR-08 — Capture human and outcome feedback

- **Requirement statement:** The system shall record alert acknowledgement and disposition, maintenance actions, confirmed failure type and event time when available, and false-alarm feedback without allowing a user to overwrite the original prediction. Feedback shall be linked by motor, prediction, and work-order identifiers and shall enter a controlled label-validation process.
- **Rationale:** Delayed outcomes are required for performance monitoring and future retraining, while immutable predictions preserve auditability and prevent label leakage or retrospective alteration.
- **Measurable criterion:** An integration test links a closed alert and work order to the original immutable prediction; only authorized label stewards can approve an outcome for training use.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** DESIGN ASSUMPTION-02 and DESIGN ASSUMPTION-05; S09.

## 4. Non-functional requirements

### NFR-01 — End-to-end response latency

- **Requirement statement:** Under the initial 100-motor load, at least 95% of scheduled predictions shall be visible in the dashboard within two minutes of the scheduled window close; no normally processed prediction shall arrive later than the next 15-minute scoring cycle.
- **Rationale:** The target concerns the next 24 hours, so minute-scale—not subsecond—delivery is sufficient for an advisory maintenance workflow and avoids unjustified hard-real-time infrastructure.
- **Measurable criterion:** Load tests measure event time/window close to successful dashboard persistence and show p95 ≤ 2 minutes and maximum normal-path latency < 15 minutes at the initial load.
- **Classification:** DESIGN ASSUMPTION.
- **Supporting source/basis:** PA-01 and PA-02; no external source or scenario fact specifies a latency target.

### NFR-02 — Expected load and throughput

- **Requirement statement:** The production design shall sustain the PA-01 initial load of 100 scalar records per second, 100 vibration feature packets per minute, and 400 scheduled predictions per hour, and shall pass capacity tests at the 500-motor growth case: 500 scalar records per second, 500 vibration packets per minute, and 2,000 predictions per hour.
- **Rationale:** Explicit event and scoring rates turn “scalable” into a testable capacity requirement. The fivefold case provides headroom for fleet growth without asserting actual demand.
- **Measurable criterion:** A representative one-hour load test at each envelope completes without data loss, unbounded queue growth, or violation of NFR-01; resource saturation and consumer lag are recorded.
- **Classification:** DESIGN ASSUMPTION.
- **Supporting source/basis:** PA-01, PA-02, and PA-04; S07 supports decoupled messaging but not these load values.

### NFR-03 — Scaling strategy

- **Requirement statement:** Stateless ingestion consumers, feature jobs, and inference workers shall scale horizontally by partitioned motor ID; stateful stores shall support capacity expansion and partition-aware retention. Autoscaling or operator scaling shall use queue lag, processing latency, and resource saturation, not request count alone.
- **Rationale:** Motor-key partitioning preserves per-asset ordering while enabling parallel processing. Multiple operational signals reduce the risk of scaling after backlogs have already breached freshness targets.
- **Measurable criterion:** Capacity testing demonstrates scale-out from the initial to growth envelope without schema changes or loss of per-motor ordering; removing a worker redistributes work and clears the backlog within the latency objective.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** PA-01; S07 publish/subscribe decoupling; S09 production infrastructure monitoring.

### NFR-04 — Service reliability and recoverability

- **Requirement statement:** The prediction and dashboard path shall target 99.5% monthly availability, excluding approved maintenance. Components shall use health checks, retry with bounded backoff, idempotent writes, durable queues, and retention of the last approved model and preprocessing package for rollback.
- **Rationale:** The system is operationally important but advisory rather than an autonomous safety control, making 99.5% a provisional balance between continuity and design complexity. Retries and rollback address transient and release-related failures.
- **Measurable criterion:** Monthly availability is measured at the prediction-consumption boundary; fault-injection tests verify retry/idempotency and successful rollback to the previous approved bundle. The target is a design assumption, not an achieved result.
- **Classification:** DESIGN ASSUMPTION.
- **Supporting source/basis:** DESIGN ASSUMPTION-07 and DESIGN ASSUMPTION-09; S08 and S09 support reliability controls but not the numerical target.

### NFR-05 — Graceful degradation under missing or stale data

- **Requirement statement:** A missing, corrupt, or stale required channel shall never be interpreted as evidence of normal operation. The serving contract shall either use a model version explicitly validated for the available-channel pattern or return `Insufficient Data`. Provisionally, input is stale when no valid update for a required low-rate channel has arrived for two scoring intervals (30 minutes).
- **Rationale:** Silent imputation or default-normal behavior can conceal a sensor, gateway, or ingestion failure. Two scoring intervals allow one delayed batch while bounding stale operation.
- **Measurable criterion:** Fault tests remove, corrupt, and delay each channel and verify the approved degraded-model path or `Insufficient Data`, a visible data-health alert, and no `Normal` result produced solely from absent data.
- **Classification:** DESIGN ASSUMPTION.
- **Supporting source/basis:** DESIGN ASSUMPTION-06 and DESIGN ASSUMPTION-09; research notes, Cross-sensor implications; S08 and S09.

### NFR-06 — Connectivity interruption and backlog recovery

- **Requirement statement:** The gateway/ingestion boundary shall buffer at least 24 hours of the PA-04 central-transmission envelope during connectivity loss, expose buffer utilization, and replay with original event timestamps when connectivity returns. Replayed events shall be deduplicated and shall not produce misleading current alerts from expired windows.
- **Rationale:** Temporary disconnection is assumed, and missing telemetry must remain observable. A buffer equal to the prediction horizon is a provisional continuity target rather than a claim about actual outage duration.
- **Measurable criterion:** A 24-hour simulated disconnection retains the planned envelope, raises a connectivity/data-freshness alert, replays without duplicate storage, and marks late predictions so they cannot masquerade as current risk.
- **Classification:** DESIGN ASSUMPTION.
- **Supporting source/basis:** DESIGN ASSUMPTION-06; S07 delivery semantics; S08 OT availability concerns. The 24-hour duration is not externally sourced.

### NFR-07 — Maintainability and reproducibility

- **Requirement statement:** Code, configuration, schemas, preprocessing logic, feature definitions, models, and alert rules shall be version controlled. Every deployed prediction shall be reproducible from immutable version identifiers and retained input/feature lineage. Changes shall pass automated unit, contract, data-validation, integration, and rollback tests before controlled deployment.
- **Rationale:** Training-serving consistency and auditable change control reduce technical debt and allow diagnosis of prediction changes.
- **Measurable criterion:** A release gate blocks an unversioned or failing artifact; an audit exercise reconstructs the artifact versions and source window for a sampled prediction.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** S09; FACT-07.

### NFR-08 — Layered operational and model monitoring

- **Requirement statement:** Monitoring shall cover sensor/data health, pipeline/service health, feature and input distributions, prediction and alert behavior, and delayed outcome performance. Dashboards and alerts shall include missingness, staleness, unit/range violations, clock skew, duplicates, queue lag, feature freshness, inference failures/latency, prediction distribution, alert rate, drift indicators, and performance/calibration when labels mature.
- **Rationale:** Offline model scores alone do not establish production readiness; failures can originate in sensors, data, infrastructure, transformations, or changing input–target relationships.
- **Measurable criterion:** Each listed signal has an owner, queryable history, runbook, and testable alert route before production readiness approval; monitoring tests inject at least one fault in each layer and verify detection.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** S08, S09, S10; FACT-07 and FACT-08.

### NFR-09 — Controlled retraining and promotion

- **Requirement statement:** The ML/platform owner shall review retraining readiness at least monthly and additionally when validated data-quality, input-drift, accumulated-label, or outcome-performance triggers fire. A trigger shall initiate investigation, not automatic promotion. A candidate may be registered and deployed only after data validation, time/asset-aware evaluation, comparison with the current approved model and baseline, documented approval, and rollback readiness.
- **Rationale:** Distribution change does not by itself prove concept drift or justify replacement. Human-gated promotion limits the risk of training on corrupt or unrepresentative delayed labels.
- **Measurable criterion:** A simulated trigger opens a review record; deployment controls reject candidates lacking required evidence and approval; rollback restores the previous bundle. Monthly review cadence is audited.
- **Classification:** DESIGN ASSUMPTION.
- **Supporting source/basis:** S09, S10; OPEN QUESTION-13. The monthly review interval is provisional; the evidence supports trigger review, not that cadence.

### NFR-10 — OT-aligned security and access control

- **Requirement statement:** Telemetry, predictions, model artifacts, credentials, and maintenance outcomes shall be protected with authenticated service identities, least-privilege role-based access, encryption in transit and at rest, secret rotation, audit logging, and network segmentation at the OT/IT boundary. The advisory system shall not introduce a control path to motor actuation.
- **Rationale:** Plant telemetry and maintenance records can expose sensitive operational information, and security controls must respect OT performance, reliability, and safety constraints.
- **Measurable criterion:** Security review verifies documented data flows and trust boundaries, denied unauthorized access, encrypted interfaces/storage, secret rotation, auditable privileged actions, and absence of an inference-to-actuator command path.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** S08; `scenario.md`; DESIGN ASSUMPTION-07. No plant-specific regulatory claim is made.

## 5. Data requirements

### DR-01 — Collect primary plant telemetry

- **Requirement statement:** The primary production data source shall be time-series telemetry for vibration, temperature, electrical current, and RPM from instrumented motors through an IoT/edge gateway. Additional variables such as load, torque, ambient conditions, motor type, or duty state may be collected only when their provenance is documented and their inclusion is supported by research or explicitly approved as a design assumption.
- **Rationale:** The four core modalities are fixed by the scenario; operating context may be necessary to separate load/speed effects from degradation.
- **Measurable criterion:** The data catalogue identifies each field’s source, asset/channel mapping, unit, sampling/aggregation policy, owner, quality status, and permitted use; unregistered features cannot enter training or serving.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** `scenario.md`; S02, S03, S05; FACT-02 through FACT-04.

### DR-02 — Obtain authoritative maintenance outcomes

- **Requirement statement:** Training and evaluation shall use plant telemetry joined to a governed maintenance/work-order or failure-event source containing stable motor identity, event time, event type, confirmation status, and planned/unplanned status. The operational definition of “failure” and the authoritative system of record must be approved by maintenance and operations stakeholders before target labels are generated.
- **Rationale:** No selected public dataset supplies the required 24-hour target, and a precise timestamped outcome is necessary to distinguish positive, negative, and censored examples.
- **Measurable criterion:** Label-generation jobs reject events without approved identity/time semantics; the approved failure definition and label source are versioned and traceable for every training dataset.
- **Classification:** DESIGN ASSUMPTION.
- **Supporting source/basis:** DESIGN ASSUMPTION-02; OPEN QUESTION-01 and OPEN QUESTION-02; S05 and S06 limitations.

### DR-03 — Preserve a complete, versioned observation schema

- **Requirement statement:** Every telemetry observation shall carry motor ID, sensor/channel ID, event timestamp, ingestion timestamp, value or payload reference, unit, schema version, source/gateway ID, and quality status. Clock basis and timestamp precision shall be documented.
- **Rationale:** These fields enable alignment, deduplication, freshness measurement, unit checking, provenance, and replay across training and inference.
- **Measurable criterion:** Schema validation reports completeness for every required field and quarantines nonconforming records; no quarantined record enters production feature generation until corrected or explicitly waived.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** research notes, IoT and industrial data ingestion; DESIGN ASSUMPTION-03; S07 and S09.

### DR-04 — Validate quality before feature generation

- **Requirement statement:** Data validation shall check schema, type, unit, physical/configured range, timestamp plausibility, clock skew, duplicate key, ordering, sampling continuity, and cross-channel motor/time alignment. Quality results shall be stored with the data and aggregated by channel, motor, and time.
- **Rationale:** Invalid units, timestamps, or asset mappings can create plausible but incorrect features and training-serving skew.
- **Measurable criterion:** Automated validation executes on every batch/stream partition; invalid records are counted and quarantined; quality dashboards show pass/fail rates and the responsible data owner can trace a failure to source.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** S09; FACT-07; research notes, Monitoring layers.

### DR-05 — Handle missing and corrupt sensor readings explicitly

- **Requirement statement:** Missing, corrupt, out-of-range, duplicate, and out-of-order readings shall be represented explicitly. Corrections, interpolation, imputation, and dropped values shall create method indicators and retain the original observation. Imputation shall be fitted on training data only and applied identically in training and serving; no method may convert absence into an apparently healthy measurement.
- **Rationale:** Explicit treatment prevents concealed sensor failure, leakage, and inconsistent transformations.
- **Measurable criterion:** Data tests inject each defect class and verify quarantine or documented transformation, preservation of the raw value, a quality indicator in derived features, and identical transformation code/version in training and serving.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** S09; research notes, Cross-sensor implications; NFR-05.

### DR-06 — Enforce inference-window completeness and freshness

- **Requirement statement:** A provisionally valid six-hour scoring window shall contain at least 95% of expected scalar observations for each required low-rate channel and at least one valid vibration feature packet in each of the most recent 15 minutes. Data shall also satisfy the 30-minute staleness rule in NFR-05. A model explicitly validated for a documented missing-channel pattern may define a different contract; otherwise the system returns `Insufficient Data`.
- **Rationale:** A concrete gate is needed to prevent scoring on severely incomplete windows. The 95% and packet requirements are provisional and must be recalibrated after observing sensor reliability and model sensitivity.
- **Measurable criterion:** Boundary tests at 94.9% and 95.0% completeness, missing recent vibration, and 30-minute staleness produce the specified accept/reject outcomes and visible reason codes.
- **Classification:** DESIGN ASSUMPTION.
- **Supporting source/basis:** PA-02 and PA-03; DESIGN ASSUMPTION-09; research notes do not provide a universal completeness threshold.

### DR-07 — Generate leakage-resistant 24-hour labels

- **Requirement statement:** For a prediction time `t`, a positive label shall indicate an approved failure event in `(t, t + 24 hours]`. A negative label may be assigned only when the motor remains observable for the full horizon with no approved failure. Planned shutdowns, ambiguous or duplicate events, already-failed states, and post-maintenance stabilization periods shall be excluded or explicitly censored under a versioned policy. Features shall use data available at or before `t` only.
- **Rationale:** This preserves the fixed horizon, prevents future information from leaking into features, and avoids treating unobserved outcomes as normal operation.
- **Measurable criterion:** Label tests cover horizon boundaries, duplicate work orders, planned shutdowns, censored windows, and feature timestamps; any feature timestamp later than `t` fails dataset validation.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** `scenario.md`; DESIGN ASSUMPTION-02 and DESIGN ASSUMPTION-05; OPEN QUESTION-01 and OPEN QUESTION-02.

### DR-08 — Separate public benchmark data from production evidence

- **Requirement statement:** Paderborn and CWRU data may be used for preprocessing, ingestion, or method prototypes with provenance and licence/terms retained, but they shall not be treated as the production training population, native 24-hour labels, or evidence of achieved project performance. Unaligned datasets shall not be merged into a synthetic multimodal record.
- **Rationale:** Both datasets are useful but differ materially from continuous plant histories and the required target.
- **Measurable criterion:** Dataset metadata records origin and permitted purpose; review checks find no production-performance claim or cross-dataset record lacking same-asset/time alignment.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** S05 and S06; DESIGN ASSUMPTION-08 and DESIGN DECISION-04.

### DR-09 — Protect operational data and minimize personal data

- **Requirement statement:** Motor telemetry and predictions shall be classified as operationally sensitive. User identity shall be retained only where needed for acknowledgement, approval, assignment, and audit. Access and export shall follow role and purpose; encryption, retention enforcement, deletion, and access logging shall apply across hot, cold, backup, and derived datasets. Personal notes not needed for the maintenance purpose shall not be collected.
- **Rationale:** The primary data concern is industrial confidentiality and integrity, while user workflow records can introduce limited personal data. Data minimization reduces exposure without inventing a plant-specific legal regime.
- **Measurable criterion:** The data inventory maps classification, owner, purpose, role access, retention, and deletion behavior; access tests deny unauthorized roles and retention tests cover replicas and derived data.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** S08; OPEN QUESTION-14. No organization-specific privacy or regulatory obligation is asserted.

### DR-10 — Size, retain, and monitor the data estate

- **Requirement statement:** Capacity planning shall use PA-04 and PA-05 until replaced by a sensor survey. For 100 motors, scalar telemetry is approximately `100 × 86,400 × 256 bytes = 2.21 GB/day`, vibration feature packets are approximately `100 × 1,440 × 4 KiB = 0.59 GB/day`, and the combined planning rate is approximately **2.80 GB/day** before replication, indexes, compression, or filesystem overhead. This is approximately **252 GB for 90 hot days** and **2.05 TB for two cold years**. At 500 motors, the corresponding pre-overhead envelope is approximately **14.0 GB/day**, **1.26 TB hot**, and **10.2 TB cold**.
- **Rationale:** A transparent formula provides a reviewable storage and throughput basis while exposing which assumptions must be replaced. Separating logical volume from overhead avoids false precision.
- **Measurable criterion:** Architecture capacity estimates reproduce the formula, add measured overhead and replication factors explicitly, and alert at agreed utilization thresholds; quarterly review compares actual volume with the envelope and revises capacity before exhaustion.
- **Classification:** DESIGN ASSUMPTION.
- **Supporting source/basis:** PA-01, PA-04, and PA-05; no external source provides scenario-specific volume or retention.

### DR-11 — Preserve lineage and training/evaluation integrity

- **Requirement statement:** Every curated dataset shall record source partitions, validation results, transformation and label-policy versions, extraction time, and dataset version. Training, validation, and test partitions shall be separated by time and motor or motor group where feasible; overlapping windows from the same underlying recording shall not cross partitions.
- **Rationale:** Lineage enables reproduction, while time/asset-aware separation reduces leakage and overly optimistic evaluation from correlated windows.
- **Measurable criterion:** A dataset manifest reconstructs every row/window from governed sources and automated tests detect overlapping source intervals or disallowed motor/time overlap across partitions.
- **Classification:** DESIGN DECISION.
- **Supporting source/basis:** S09; `research/research-notes.md`, Evaluation cautions.

## 6. Traceability to the requirements-analysis rubric

| Rubric item | Covered by |
|---|---|
| RA-F01 — core ML functionality | FR-01, FR-04 |
| RA-F02 — inputs and outputs | FR-02, FR-03 |
| RA-F03 — integrations | FR-05, FR-08 |
| RA-F04 — user interaction | FR-06, FR-07, FR-08 |
| RA-N01 — latency, throughput, response time | NFR-01, NFR-02 |
| RA-N02 — load and scalability | NFR-02, NFR-03 |
| RA-N03 — uptime, failure handling, graceful degradation | NFR-04, NFR-05, NFR-06 |
| RA-N04 — maintainability, monitoring, retraining | NFR-07, NFR-08, NFR-09 |
| RA-D01 — sources | DR-01, DR-02, DR-08 |
| RA-D02 — quality | DR-03 through DR-07, DR-11 |
| RA-D03 — privacy | NFR-10, DR-09 |
| RA-D04 — volume | DR-10 |

## 7. Deferred decisions and dependencies

The following are intentionally not claimed as resolved in Phase 3A:

- the plant-approved definition of failure and authoritative work-order/event system;
- the final lookback window, sensor sampling rates, raw-vibration retention, and edge-versus-central feature processing;
- specific plant products, protocols, network zones, escalation recipients, and safety/regulatory obligations;
- the model family, probability threshold, hysteresis values, and business/model acceptance thresholds;
- achieved latency, availability, throughput, data quality, model performance, or business impact.

Phase 3B must define the business, system, and model goal hierarchy and acceptance thresholds. The architecture and trade-off phases must either satisfy these requirements or record and justify an explicit revision.
