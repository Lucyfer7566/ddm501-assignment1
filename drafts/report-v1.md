# Design of an ML-Based Predictive Maintenance System for Industrial Motors Using IoT Sensor Data

This document specifies a proposed production-oriented ML system. It is not evidence of a trained or deployed system, and it reports no experimental or operational results. Throughout the document, **FACT** denotes a claim supported by a cited source or fixed project input, **DESIGN ASSUMPTION** denotes a provisional value introduced because plant-specific evidence is unavailable, and **DESIGN DECISION** denotes an engineering choice made for this design.

# 1. Problem Definition

## 1.1 Context and Background

Industrial motors support production processes whose interruption can create downtime and emergency maintenance work. Condition-based maintenance is not only a modelling problem: it connects data acquisition, data processing, and maintenance decision-making [1]. Motor condition can be observed through several signal families, including electrical current, vibration, speed, and thermal measurements [2]. In rotating bearings, vibration can contain fault-related impulsive content, while shaft speed affects characteristic frequencies and may require speed-aware preprocessing [3]. These findings justify a multimodal design, but they do not establish that every signal is equally useful for every motor or plant.

The organizational context is a **DESIGN ASSUMPTION**: a hypothetical industrial plant operates multiple instrumented motors and can associate sensor streams with stable asset identities. No real organization, fleet, baseline, or deployment is claimed. The proposed system covers the full lifecycle from OT data collection to a maintenance decision workflow, including ingestion, validation, training, evaluation, deployment, monitoring, feedback, and controlled retraining. It remains advisory and introduces no command path to motor actuation.

The design matters because a standalone fault classifier would not address whether data are current and trustworthy, whether alerts reach maintenance staff, whether repeated warnings cause unnecessary work, or whether a changed model can be deployed safely. The intended business value is therefore to reduce unplanned motor downtime and emergency maintenance burden while avoiding unnecessary interventions and excessive false alarms.

## 1.2 Measurable Problem Statement and Scope

For each eligible industrial motor and valid recent sensor window, the system shall use **one primary predictive model** to estimate the probability that the motor will fail within the **next 24 hours**. It shall return `Normal` or `High Risk` and the failure-risk probability. The primary inputs are recent vibration, temperature, electrical-current, and rotational-speed (RPM) measurements. RPM also provides operating context for speed-aware processing.

The target is binary classification, not remaining-useful-life regression, generic anomaly detection, or multi-class fault diagnosis. The probability supports prioritization; the class and alert episode support an operational maintenance decision. A valid result is measured against approved 24-hour failure events. System value is evaluated prospectively through downtime, maintenance, alert-delivery, latency, availability, data-pipeline, and model-quality metrics defined in Section 3. None of those metrics is claimed as achieved.

The **DESIGN ASSUMPTION** used to make the system testable is an initial population of 100 motors, with capacity validation at 500 motors. Eligible motors are scored every 15 minutes and on authenticated demand using a provisional six-hour lookback. These values are engineering planning inputs, not facts about an existing plant.

## 1.3 Current Situation

No current plant workflow was supplied. The following as-is description is therefore a **DESIGN ASSUMPTION**, not an observed organizational fact. Maintenance personnel currently combine planned maintenance, reactive work after alarms or failures, manual inspection, and deterministic sensor or equipment-protection thresholds. Work orders or failure records are assumed to capture at least motor identity and event time, although the authoritative system and the operational definition of failure remain to be approved.

This assumed workflow has three relevant limitations. First, fixed thresholds and manual inspection may not combine recent patterns across vibration, temperature, current, and RPM. Second, reactive records may be inconsistent or delayed, weakening label quality and post-event evaluation. Third, the absence of an integrated prediction lifecycle prevents systematic traceability from sensor evidence to prediction, alert disposition, maintenance action, and outcome. The proposed system supplements rather than removes deterministic equipment protection and expert judgment.

## 1.4 Justification for ML

ML is appropriate as a candidate approach because the decision depends on nonlinear, time-dependent relationships among multiple sensor and operating signals. Predictive-maintenance research uses several ML families and emphasizes selecting a method according to the available data and operational problem rather than assuming a universally best algorithm [4]. A learned classifier can combine engineered trends, time/frequency information, RPM context, and missingness indicators into a calibrated risk estimate that is difficult to express as one universal rule.

ML is not justified merely because sensor data exist. A deterministic maintenance-approved threshold rule and logistic regression remain mandatory baselines. The candidate is valuable only if it improves prevalence-aware ranking and probability quality while meeting recall, precision, false-positive, latency, reliability, and maintainability gates. If representative plant histories and trustworthy timestamped outcomes are unavailable, or if the candidate cannot outperform these baselines, deployment is not justified.

The Paderborn dataset contains synchronous vibration and motor-current measurements with supporting operating variables [5], and CWRU provides documented motor-bearing vibration data [6]. Both can support ingestion, signal-processing, and evaluation-code prototypes. Neither dataset provides the complete production population or a native “failure within the next 24 hours” target. Their reported results must not be presented as performance of this system, and unrelated records must not be merged into artificial multimodal examples.

## 1.5 Stakeholders and Concerns

| Stakeholder | Goals and decisions | Principal concerns |
|---|---|---|
| Maintenance Engineer | Review prioritized motors; inspect probability, trends, and data freshness; acknowledge, assign, investigate, close, and link alerts to work orders. | Missed failures, excessive alerts, insufficient diagnostic context, stale data shown as healthy, and inability to explain or audit a prediction. |
| Plant Operations Manager | Reduce unplanned downtime and emergency work while controlling total maintenance and infrastructure cost; approve operational policy and pilot continuation. | Production disruption, unnecessary preventive work, unreliable service, unsafe automation, weak business attribution, and uncontrolled OT connectivity. |
| ML / Platform Engineer | Operate ingestion, validation, training, serving, monitoring, registry, deployment, and rollback; produce reproducible evidence for promotion. | Data/label quality, training-serving skew, latency, availability, security, drift, delayed labels, model/version traceability, and maintainable infrastructure. |

Maintenance and operations must jointly approve the failure definition, alert policy, and intervention workflow. Security/identity administrators and label stewards are supporting control roles within the responsibilities above rather than additional business owners. The system remains human-in-the-loop because no actuator authority or safety case has been supplied.

# 2. Requirements Analysis

## 2.1 Requirement Status and Planning Envelope

All numerical values in this section are **DESIGN ASSUMPTIONS** or **PROPOSED DESIGN TARGETS** unless a citation states otherwise. They create measurable acceptance conditions and must be reviewed after a sensor/site survey and shadow-mode pilot.

| Planning item | Provisional value and justification |
|---|---|
| Fleet | 100 motors initially; capacity test at 500 motors to test fivefold growth. |
| Scoring | Every 15 minutes per eligible motor, plus authenticated on-demand scoring; timely relative to the 24-hour advisory horizon without implying real-time control. |
| Feature lookback | Six hours; enough to provide recent trends while bounding online computation, subject to validation against alternatives. |
| Central data envelope | Approximately 256 bytes per motor-second for normalized scalar data and 4 KiB per motor-minute for vibration feature packets; these are sizing envelopes, not sensor specifications. |
| Retention | 90 days hot and two years cold for curated telemetry, labels, predictions, and audit data; subject to plant policy. |

## 2.2 Functional Requirements

| ID | Requirement and measurable criterion | Classification / rationale |
|---|---|---|
| FR-01 | For every valid window, invoke exactly one versioned primary model and record the 24-hour horizon. | **DESIGN DECISION.** Preserves the fixed target and one-model scope. |
| FR-02 | Accept a versioned contract containing motor/sensor identity, event and ingestion timestamps, units, quality flags, and recent vibration, temperature, current, and RPM. Contract tests reject unknown schemas, identities, units, timestamps, or malformed values. | **DESIGN DECISION.** Ensures correct asset/time association and speed-aware processing [2], [3]. |
| FR-03 | A valid result contains class, probability in `[0,1]`, motor ID, prediction time, horizon, source-window interval, model/feature versions, and quality state. Invalid or stale input returns `Insufficient Data`, never `Normal`. | **DESIGN DECISION.** Makes results actionable and auditable. |
| FR-04 | Score every 15 minutes and on authenticated demand. Requests with the same motor, window, and model version are idempotent. Tests expect 400 predictions/hour initially and 2,000/hour at growth load. | **DESIGN ASSUMPTION.** Establishes a measurable cadence and load. |
| FR-05 | Provide versioned interfaces among gateway, ingestion, storage, feature/training pipeline, inference, dashboard/alerts, and a CMMS/EAM-equivalent outcome source. An integration test traces telemetry to storage, prediction, display, and linked outcome. | **DESIGN DECISION.** Connects acquisition, processing, and maintenance action [1]. |
| FR-06 | Let the Maintenance Engineer rank/filter motors, view probability, recent trends, freshness and versions, and open the maintenance record. No user or prediction may autonomously shut down a motor. | **DESIGN DECISION.** Retains a human decision boundary. |
| FR-07 | Create or update one active alert episode per motor/horizon; support acknowledgement, assignment, comment, investigation, closure, and configurable escalation. Threshold, hysteresis, and rule changes are versioned and audited. | **DESIGN DECISION.** Prevents a new alert every 15 minutes. |
| FR-08 | Link immutable predictions to alert disposition, maintenance action, confirmed failure time/type, and false-alarm feedback. Only authorized review can approve an outcome for training. | **DESIGN DECISION.** Supports delayed performance evaluation without retrospective prediction changes. |

## 2.3 Non-Functional Requirements

| ID / concern | Proposed requirement and acceptance criterion | Classification / failure behavior |
|---|---|---|
| NFR-01 Performance | At initial and growth load, p95 time from scheduled window close to dashboard visibility shall be ≤2 minutes; normal-path maximum shall be <15 minutes. | **PROPOSED DESIGN TARGET.** Minute-scale response is appropriate to an advisory 24-hour horizon. |
| NFR-02 Throughput | Sustain 100/500 scalar records per second, 100/500 vibration packets per minute, and 400/2,000 predictions per hour. A one-hour test shall show no unexplained loss, unbounded queue growth, or latency breach. | **DESIGN ASSUMPTION.** Derived from the stated planning envelope. |
| NFR-03 Scalability | Stateless ingestion, feature, and inference workers scale horizontally by motor-ID partition; stores expand through capacity and retention controls. Scaling uses queue lag, feature freshness, latency, and saturation. | **DESIGN DECISION.** Preserves per-motor ordering while enabling parallel work. |
| NFR-04 Reliability | Target ≥99.5% monthly availability excluding approved maintenance. Use health checks, bounded retry, idempotent writes, durable queues, backups, and the last approved bundle for rollback. | **PROPOSED DESIGN TARGET.** Advisory importance does not justify assuming safety-control availability. |
| NFR-05 Graceful degradation | Missing, corrupt, or stale required data shall use only a model validated for that pattern or return `Insufficient Data`. A low-rate channel is provisionally stale after two scoring intervals (30 minutes). | **DESIGN ASSUMPTION.** Missing telemetry cannot become healthy evidence. |
| NFR-06 Connectivity | Buffer the central-transmission envelope locally for 24 hours, expose utilization, replay original event time, deduplicate, and prevent expired replay from creating a current alert. | **DESIGN ASSUMPTION.** The duration matches the prediction horizon; it is not an observed outage requirement. |
| NFR-07 Maintainability | Version code, configuration, schema, features, models, and alert rules; reconstruct every prediction from retained lineage. Unit, contract, data, integration, smoke, and rollback checks gate releases. | **DESIGN DECISION.** Production ML requires controls beyond offline accuracy [9]. |
| NFR-08 Monitoring | Monitor sensor/data health, service/pipeline health, feature/input distributions, output/alert behavior, and delayed outcome performance; every alert has an owner, history, route, and runbook. | **DESIGN DECISION.** Covers the production layers identified in ML readiness work [9]. |
| NFR-09 Retraining | Review readiness monthly and when validated quality, drift, label-volume, or performance signals fire. A signal starts investigation, not automatic promotion; every candidate repeats validation and approval. | **DESIGN ASSUMPTION/DECISION.** Input change alone does not prove concept drift [10]. |
| NFR-10 Security | Apply authenticated service identities, least privilege, encryption in transit/at rest, secret rotation, audit logging, OT/IT segmentation, and no inference-to-actuator path. | **DESIGN DECISION.** OT controls must account for performance, reliability, safety, topology, and security [8]. |

## 2.4 Data Requirements

### 2.4.1 Sources, provenance, and labels

The primary production source is time-series vibration, temperature, current, and RPM from motors through an industrial gateway. Each observation carries motor and sensor IDs, event and ingestion timestamps, units, schema and source versions, payload/value reference, and quality status. Additional load, torque, ambient, motor-type, or duty variables require documented provenance and explicit approval.

The second essential source is an authoritative maintenance/work-order or failure-event system. This source is a **DESIGN ASSUMPTION** until the plant identifies it. Maintenance and operations must approve what constitutes failure, its timestamp, confirmation status, and planned/unplanned status before labels are generated. For prediction time `t`, a positive label means an approved failure occurs in `(t, t + 24 hours]`; a negative requires full observation through the horizon and no approved failure. Planned shutdowns, ambiguous/duplicate events, already-failed states, post-maintenance stabilization, and intervention-affected outcomes are excluded or explicitly censored. Features may use only information available at or before `t`.

Paderborn and CWRU data [5], [6] may support preprocessing and code prototypes with provenance retained, but shall not be used as evidence of native 24-hour labels, plant prevalence, or achieved performance.

### 2.4.2 Quality and missing/corrupt readings

Validation checks schema, type, identity, unit, configured/physical range, timestamp plausibility, clock skew, duplicate key, ordering, continuity, and cross-channel alignment. Invalid records are quarantined with reason codes; corrections never overwrite the raw observation. Interpolation or imputation records the method and quality indicator, is fitted on training data only, and is applied by the same versioned package in training and serving.

A provisional six-hour inference window requires at least 95% of expected observations in each required low-rate channel, at least one valid vibration feature packet in the latest 15 minutes, and compliance with the 30-minute staleness rule. These are **DESIGN ASSUMPTIONS**, not universal sensor-quality standards. Boundary and fault-injection tests verify that incomplete, corrupt, out-of-range, duplicate, out-of-order, and stale inputs are quarantined or return `Insufficient Data` with a visible reason.

Dataset manifests retain source partitions, validation output, transformation and label-policy versions, and extraction time. Training, validation, and test partitions are separated by time and motor or motor group where feasible. Overlapping windows from one source recording cannot cross partitions.

### 2.4.3 Privacy, confidentiality, and security

Motor telemetry, predictions, model artifacts, and maintenance records are operationally sensitive. User identity is retained only for acknowledgement, assignment, approval, and audit. Role- and purpose-based access, encryption, export controls, retention enforcement, deletion, secret management, and access logging apply to hot, cold, backup, and derived data. Unnecessary personal notes are not collected. This design does not assert a plant-specific legal regime; a site review must establish any additional privacy, safety, regulatory, or retention obligations.

### 2.4.4 Expected data volume

Using the explicit planning envelope, 100 motors produce approximately:

- scalar data: `100 × 86,400 × 256 bytes = 2.21 GB/day`;
- vibration feature data: `100 × 1,440 × 4 KiB = 0.59 GB/day`; and
- combined logical volume: approximately `2.80 GB/day`, `252 GB/90 hot days`, and `2.05 TB/two cold years`.

At 500 motors, the corresponding logical envelope is approximately `14.0 GB/day`, `1.26 TB hot`, and `10.2 TB cold`. These **DESIGN ASSUMPTIONS** exclude replication, indexes, compression, object metadata, selectively uploaded raw vibration, and filesystem overhead. Capacity review must add measured factors and compare actual growth with the envelope before storage exhaustion.

# 3. Goals and Metrics

## 3.1 Goal Hierarchy and Measurement Policy

The hierarchy is `Business Goal → System Goal → Model Goal → Metric → Baseline → Target`. Business outcomes are measured against the same motors’ most recent 12 usable pre-pilot months, normalized by motor operating hours. The 12-month baseline and a minimum six-month pilot are **DESIGN ASSUMPTIONS**; if insufficient events exist, observation continues rather than manufacturing a result. A staggered rollout or matched non-pilot comparison should be used where practical to reduce confounding by workload, motor mix, season, staffing, and maintenance-policy changes.

No deployed-system baseline exists. Load, integration, and fault-injection tests establish the first system measurements. Model candidates are compared on identical time- and asset-separated data against `Always Normal`, `Always High Risk`, a maintenance-approved deterministic rule, and logistic regression. Public benchmark scores are not substituted for project baselines.

| Hierarchy | Metric and baseline | Target / acceptable threshold | Status |
|---|---|---|---|
| BG-01 reduce unplanned downtime → SG timely/available/valid predictions → MG detect failures | BM-01: unplanned motor downtime hours per 1,000 operating hours; historical same-motor baseline. | ≥15% relative reduction. | **PROPOSED DESIGN TARGET**, not an achieved result. |
| BG-02 reduce emergency burden without shifting cost → SG alert/data reliability → MG recall with actionable alerts | BM-02 emergency labor-hours and BM-03 total motor-maintenance cost per 1,000 operating hours. | Emergency labor-hours ≥10% lower; total cost no more than 5% above baseline while BM-01 improves. | **PROPOSED DESIGN TARGET**. |
| BG-03 avoid unnecessary work and fatigue → SG deduplicated alert workflow → MG control false alerts | BM-04: ML-initiated completed interventions with no actionable condition; historical comparable or shadow-mode baseline. | ≤20%; model alert precision/FPR gates below must also pass. | **PROPOSED DESIGN TARGET**. |

Metric governance follows the decision hierarchy. The Plant Operations Manager owns BM-01 and the BM-03 cost guardrail; the Maintenance Engineer owns BM-02 and BM-04 disposition quality; the ML/Platform Engineer owns SM-01–SM-05 and produces MM-01–MM-06 release evidence for joint maintenance/operations approval. Business metrics use the minimum six-month pilot and a rolling 12-month view when enough outcomes exist. System metrics are reviewed continuously and summarized monthly, except load/reconciliation acceptance tests, which run per release. Model metrics are calculated per candidate on the frozen untouched test set and then on matured operational outcomes; counts, ratios, uncertainty intervals, and relevant motor/operating-regime slices are reported together. Thus model quality is necessary for timely and actionable alerts, system quality is necessary for the model output to reach users, and neither alone establishes business benefit.

## 3.2 System Goals and Metrics

| ID | Metric and measurement point | Baseline | Proposed target |
|---|---|---|---|
| SM-01 | Valid new/updated `High Risk` episodes persisted to the dashboard within five minutes of scheduled window close; duplicate retries update the same episode. | End-to-end acceptance test. | ≥99% within five minutes and zero duplicate active episodes for the same motor/horizon. |
| SM-02 | Window close to dashboard visibility, reported at p50/p95/p99/maximum. | First representative load test. | p95 ≤2 minutes and normal-path maximum <15 minutes at 100 and 500 motors. |
| SM-03 | Eligible minutes in which scheduled scoring and dashboard retrieval both succeed. | First measured pre-production/pilot month. | ≥99.5% per month, excluding approved maintenance. |
| SM-04 | Reconciliation of gateway-acknowledged message IDs to validated/persisted records; processing delay and unexplained loss reported separately. | Test manifest and first pilot month. | 100% reconciliation in one-hour load tests; ≥99.5% available for processing within five minutes in normal operation; zero unexplained loss. |
| SM-05 | Valid predictions divided by eligible scheduled motor-windows; invalid attempts carry reason codes. | First shadow-mode month. | ≥95% valid coverage; 100% of non-valid attempts visibly classified, not returned as `Normal`. |

## 3.3 Model Goals, Evaluation Units, and Metrics

A scoring window is one valid prediction opportunity. Repeated high-risk windows are grouped into a high-risk alert episode. One episode may match at most one approved failure and one failure at most one episode; a match requires the episode to begin within the 24 hours before failure. A negative motor-day is an adequately observed motor-day with no approved failure in the following 24 hours and no censoring condition. Post-deployment episodes followed by preventive intervention are censored unless an approved outcome policy supports attribution; they are not automatically false positives.

| ID | Metric | Baseline | Proposed promotion target |
|---|---|---|---|
| MM-01 | Failure-event recall `TP/(TP+FN)`. | `Always Normal = 0`; rule and logistic baselines measured on the same test set. | ≥85%. |
| MM-02 | Alert-episode precision `TP/(TP+FP)`. | `Always High Risk` equals prevalence induced by the episode policy. | ≥50%. |
| MM-03 | F1 `2PR/(P+R)`. | `Always Normal = 0`; practical baselines measured. | ≥0.63 while separate recall and precision gates also pass. |
| MM-04 | Negative-motor-day false-positive rate `FP_day/(FP_day+TN_day)`. | `Always Normal = 0`; `Always High Risk = 1`. | ≤1%. |
| MM-05 | Window-level area under the precision–recall curve (PR-AUC). | Held-out positive-window prevalence plus rule/logistic comparators. | ≥2× held-out prevalence and above both practical baselines; report a 95% bootstrap interval. |
| MM-06 | Brier score and calibration by motor/operating-regime slice. | Constant training-prevalence probability on the same test set. | Lower Brier score than the constant baseline and no material unexplained slice miscalibration. |

PR-AUC is included because the operational question is whether positive predictions remain useful as recall increases in an imbalanced problem; accuracy is excluded because an `Always Normal` policy can appear strong while detecting no failures. Recall receives priority because a false negative leaves an impending failure without an ML warning. It is constrained to prevent an `Always High Risk` policy and alert fatigue: apply the frozen episode policy, find validation thresholds with recall ≥85%, reject any with precision <50% or false-positive rate >1%, then select the highest-recall survivor. F1, expected alert volume, lead time, and calibration are secondary comparisons. Freeze this policy before untouched testing. If no threshold passes all gates, do not promote the model.

## 3.4 Acceptance and Promotion

Promotion requires an approved failure definition; leakage-resistant time/asset partitions; all MM-01–MM-06 gates on untouched data; representative SM-01–SM-05 integration, load, and fault tests; denominators, uncertainty, and important slices; versioned model/data/feature/threshold artifacts; monitoring and rollback readiness; and human approval. Business metrics are pilot exit criteria, not prerequisites for shadow mode. A target revision must be approved before the untouched test set is examined. No threshold in this section represents achieved performance.

# 4. High-Level Architecture Design

## 4.1 Architectural Stance and Diagram

The proposed deployment is a small central on-premises VM/container cluster connected to the OT area through industrial gateways and an authenticated MQTT broker. Gateways perform acquisition, identity/time/unit normalization, versioned high-rate vibration feature extraction, and durable buffering. Central services validate, store, train, register, score, monitor, and deliver alerts. Training and serving share the same versioned feature package. One XGBoost binary classifier is the primary predictive model; deterministic rules and logistic regression remain non-production comparators.

![High-level training, inference, and feedback architecture](../diagrams/architecture.png)

**Figure 1. Proposed production ML architecture.** Solid flows carry normal-path data or artifacts; dotted flows represent invalid-data, monitoring, or feedback behavior. The architecture is prospective and does not represent an existing deployment.

## 4.2 Data Flow and ML Pipeline

### 4.2.1 Training path

`sensor/history data → ingestion and immutable storage → batch validation and curation → shared preprocessing/feature engineering → XGBoost training → independent evaluation → MLflow registry → human-approved deployment`

Gateways publish scalar envelopes and vibration feature packets through MQTT over TLS. Ingestion authenticates, deduplicates, and durably lands raw records before acknowledgement. Batch validation quarantines records that fail identity, schema, timestamp, unit, range, continuity, or lineage rules. The CMMS/EAM adapter joins approved outcomes using the governed `(t, t+24 hours]` policy. The shared feature package builds the six-hour window and fits any learned transformation on training data only. Airflow creates immutable dataset manifests and time/asset splits, trains the single candidate, and runs independent evaluation. A failed mandatory gate prevents registry promotion. Passing bundles contain the model, transformations, schema, threshold/episode policy, evaluation evidence, and lineage. Human approval, compatibility checks, shadow operation, and rollback readiness precede activation.

### 4.2.2 Inference path

`new sensor data → MQTT ingestion → online validation and recent storage → shared preprocessing/features → cached approved model → failure-risk probability → versioned threshold/hysteresis → Normal/High Risk → dashboard/alert`

Every 15 minutes, or on authorized demand, the feature assembler retrieves a valid six-hour motor window. Missing, stale, corrupt, or incompatible data return `Insufficient Data`. Otherwise the cached approved XGBoost bundle returns one probability. A deterministic policy converts it to `Normal` or `High Risk` and maintains one alert episode per motor/horizon. The system first persists the immutable prediction and durable alert event, then delivers the dashboard notification. The engineer can inspect, acknowledge, assign, investigate, close, and link a work order. No inference result actuates equipment.

### 4.2.3 Feedback and MLOps path

`service/data/model/outcome monitoring → validated quality, drift, or performance signal → human investigation → approved retraining trigger → validation/training/evaluation → registry → shadow/approved deployment or rejection`

Monitoring covers gateway health, missing/stale data, validation failures, queue lag, feature freshness, latency, availability, prediction distribution, episode rate, drift indicators, and delayed performance/calibration. Distribution change opens an investigation; it is not automatic proof of concept drift [10]. Monthly review and validated signals can start the controlled Airflow path, but no training process can directly update serving. Alert dispositions and CMMS outcomes are retained immutably, and missing outcomes never become negative labels.

## 4.3 Component Responsibilities, Technologies, and Failure Behavior

| ID / component | Responsibility; input → output | Candidate technology and rationale | Important failure behavior |
|---|---|---|---|
| C01 Sensors/asset registry | Observe vibration, temperature, current, RPM and bind channel metadata → timestamped readings. | Existing plant-grade sensors and asset registry; avoids a vendor assumption. | Invalid/unregistered channels become unavailable, not healthy defaults. |
| C02 Edge gateway/buffer | Normalize identity/time/unit, compute vibration features, buffer → scalar envelopes, feature packets, quality metadata. | Industrial Linux, containerized Python/NumPy/SciPy, durable SQLite/filesystem queue. | Buffer 24-hour envelope, alarm on utilization, preserve original timestamps. |
| C03 MQTT broker | Authenticate and decouple gateways/consumers → per-motor topics and delivery metrics. | MQTT 5/EMQX; lightweight publish/subscribe with explicit delivery semantics [7]. | Gateways buffer outage; consumers deduplicate at-least-once delivery. |
| C04 Ingestion/persistence | Validate transport, assign idempotency key, land immutable raw record → validation work item/audit. | Python, Pydantic, MQTT/MinIO/PostgreSQL clients. | Do not acknowledge before durable landing; quarantine poison messages. |
| C05 Validation/quarantine | Apply stream/batch quality rules → valid normalized or quarantined records. | Pydantic/Pandera and batch validation reports. | Invalid data isolated with reason; rules failure stops the partition. |
| C06 Historical store | Retain raw, quarantine, curated, manifests, artifacts → versioned objects. | On-premises MinIO/S3 lifecycle storage. | Failed/checksum-invalid writes block or quarantine; training pauses if unavailable. |
| C07 Operational store | Serve recent windows, predictions, episodes, audit → queryable state. | PostgreSQL with TimescaleDB; time-window and relational integrity at moderate scale. | Transactional retry; stale read-only view is labelled; no fresh score from stale data. |
| C08 CMMS/EAM adapter/labels | Join work orders/outcomes and apply label policy → versioned labels/censoring. | Authenticated REST/file adapter and Airflow Python/SQL job. | Missing/ambiguous outcomes remain delayed/censored, never negative by default. |
| C09 Shared feature package | Align modalities, check validity, transform → versioned vector and quality lineage. | Versioned Python package using NumPy/SciPy/pandas/scikit-learn transformers. | Version mismatch fails training or returns `Insufficient Data`; no online refit. |
| C10 Orchestrator/dataset builder | Schedule curation, snapshots, splits, jobs → immutable manifest/run state. | Apache Airflow; suited to auditable scheduled dependencies. | Failed dependency stops downstream work; serving is independent. |
| C11 Primary training | Fit one candidate → XGBoost artifact and metadata. | Containerized Python/XGBoost. | Invalid features/classes or compute failure produces no candidate or deployment authority. |
| C12 Evaluation gate | Calculate metrics, uncertainty, slices, baselines, contracts → signed pass/fail report. | Separate scikit-learn evaluation container and policy checks. | Missing lineage, leakage, failed metric, or schema incompatibility rejects promotion. |
| C13 Registry | Retain immutable bundle, evidence, aliases, approvals → approved/candidate versions. | Self-hosted MLflow backed by PostgreSQL/MinIO. | Outage blocks change, not cached serving; unsigned/corrupt artifact is rejected. |
| C14 Deployment controller | Human approval, smoke/shadow checks, atomic activation → running version/audit. | CI runner, OCI registry, Docker on on-premises VMs, Ansible-equivalent rollout. | Failed checks leave active version unchanged; regression rolls back. |
| C15 Feature/inference worker | Schedule scores, validate window, load cache → probability or `Insufficient Data`. | Python worker, FastAPI for on-demand requests, XGBoost runtime, durable queue. | Retry idempotently; only verified cached approved model may serve. |
| C16 Risk/episode policy | Threshold, hysteresis, deduplicate → class and durable episode event. | Versioned Python policy and PostgreSQL transaction/outbox. | Invalid probability cannot become `Normal`; configuration mismatch blocks classification. |
| C17 Dashboard/notification | Present risk/context and workflow → user action and CMMS linkage. | Internal web application, API, PostgreSQL workflow, configurable notifier. | Persist before notify; queue outage recovery; deny unauthorized state/control action. |
| C18 Observability/ML monitoring | Aggregate service, data, input, output, outcome signals → dashboards, alerts, reports. | Prometheus/Grafana, structured logs, scheduled Python/SQL monitors. | Independent heartbeat; monitoring loss blocks promotion after authorized grace period. |
| C19 Retraining controller | Convert validated signal and owner decision → approved/rejected Airflow trigger. | Ticket/approval workflow and Airflow trigger API. | Drift alone cannot deploy; missing labels/approval leaves serving unchanged. |
| C20 Security/audit controls | Enforce identity, segmentation, encryption, secrets, audit → permitted flows and evidence. | Existing IdP/OIDC, PKI/mTLS, Vault-compatible secrets, firewall/DMZ, encrypted stores. | Fail closed; audit failure blocks privileged promotion, never motor operation. |

These products are candidates, not claims about existing plant technology. Equivalent standard tools may replace them if input/output contracts, controls, traceability, and failure semantics remain unchanged. MQTT delivery semantics require application-level idempotency; broker acknowledgement alone is not proof that a prediction was stored or delivered [7]. OT placement and security must respect reliability, safety, and performance constraints [8].

## 4.4 Deployment, Scaling, Reliability, and Maintenance

The gateway tier uses one industrial gateway per suitable motor group. Stateless central workers scale horizontally by motor-ID partition and durable queues; TimescaleDB and MinIO use backup/replication and capacity controls. Training runs in bounded containers with inference capacity reserved. A full Kubernetes estate and public-cloud dependency are deliberately unnecessary at the assumed scale.

During gateway/network loss, data buffer locally and later replay with original event time. Broker/ingestion loss causes bounded retry. Object-store loss pauses new durable landing and training; registry loss blocks release but cached approved serving continues. Operational-store loss prevents claiming new persisted predictions; the dashboard may show a clearly stale last-known view. Dashboard loss preserves durable alerts for replay. CMMS loss delays outcomes and performance measurement. Model/schema incompatibility fails closed. Candidate failure leaves the active model unchanged. These behaviors provide graceful degradation without converting infrastructure failure into `Normal` motor status.

Maintainability relies on immutable dataset and model lineage, shared transformation code, independent evaluation, human-gated registry aliases, shadow rollout, rollback, layered monitoring, and controlled retraining. This treats production readiness as a property of the complete system rather than of one offline score [9].

# 5. Trade-offs Analysis

## 5.1 TO-01 — False Negatives versus False Positives

**Engineering question.** How should the operating threshold balance missed 24-hour failures against false alerts and unnecessary maintenance?

- **Option A—recall-first lower threshold:** improves the chance of warning before a genuine failure and supports downtime reduction, but increases investigation work, false episodes, cost, and alert fatigue.
- **Option B—precision-first higher threshold:** produces a smaller and more actionable queue, but misses more failures and can make the system quiet rather than useful.

**Scenario consequence and decision.** Operations bears much of the false-negative downtime cost, while maintenance bears false-positive workload. The **DESIGN DECISION** is constrained recall-first selection: maximize validation recall among thresholds satisfying recall ≥85%, alert-episode precision ≥50%, negative-motor-day FPR ≤1%, and F1 ≥0.63. C16 applies hysteresis and one-active-episode deduplication, and C17 retains human review. If no threshold passes, do not deploy. Residual risks are uncertain plant prevalence and costs, rare-event uncertainty, slice disparities, and intervention-censored outcomes; monitor them through uncertainty, slice, alert-disposition, and business-cost reporting.

## 5.2 TO-02 — Edge versus Centralized/Cloud Inference

**Engineering question.** Should complete risk inference run on every gateway or in one governed central service?

- **Option A—full edge inference:** continues local scoring during central-path loss and reduces telemetry transport, but multiplies model rollout, resource, monitoring, rollback, and version-drift risks.
- **Option B—central inference:** provides one model/feature/threshold governance point and consistent fleet monitoring, but depends on the central path and creates a wider central failure domain. Public cloud would additionally introduce an unapproved OT boundary.

**Scenario consequence and decision.** The 24-hour horizon and 15-minute cadence do not require millisecond control, while stakeholders need consistent fleet results and governed releases. The **DESIGN DECISION** is gateway-side collection, buffering, and vibration features with centralized on-premises C15 inference; neither a gateway risk model nor public-cloud inference is deployed initially. This supports maintainability, monitoring, promotion, and the 100/500-motor envelope. Residual risks are central outage/capacity and interruptions beyond the 24-hour buffer; expose lag/freshness, replay recoverable data, return `Insufficient Data`, and retain rollback. A future edge failover requires a separate validated design.

## 5.3 TO-03 — Prediction/Data Freshness versus Infrastructure Cost

**Engineering question.** How much freshness is justified for a 24-hour advisory decision?

- **Option A—central raw streaming and sub-minute scoring:** can show changes sooner and retain richer data, but increases bandwidth, storage, compute, monitoring, and alert churn without evidence of better performance for this target.
- **Option B—bounded aggregation and cadence:** lowers cost and supports stable windows and testable capacity, but can delay fast changes and discard raw-waveform information.

**Scenario consequence and decision.** Maintenance needs stable actionable episodes, operations needs bounded cost, and platform engineering needs a reproducible capacity envelope. The **DESIGN DECISION** is the assumed one-second scalar envelope, one-minute edge vibration packet, six-hour feature window, 15-minute/on-demand scoring, 90-day hot and two-year cold retention, and selective raw-vibration upload. The p95 two-minute and normal-path <15-minute targets remain. Residual risks include fast-onset failures, an unsuitable lookback, and inaccurate payload/retention estimates. Pilot measurements of lead time, coverage, queue lag, storage growth, and missed-event timing must drive any versioned revision.

## 5.4 TO-04 — Model Complexity/Performance versus Latency and Maintainability

**Engineering question.** Should the primary model learn from raw temporal sequences or from governed engineered features?

- **Option A—temporal/deep model:** may learn patterns omitted by manual summaries, but requires more representative labelled histories, raw-data infrastructure, compute, tuning, monitoring, and diagnostic effort. No project evidence shows that it improves this exact target.
- **Option B—engineered features plus XGBoost:** supports fast bounded inference, shared feature lineage, calibration, and simpler rollback, but may miss temporal structure and depends on feature quality.

**Scenario consequence and decision.** The design has no verified plant dataset or achieved comparison; maintainability and evidence gates therefore matter more than speculative complexity. The **DESIGN DECISION** is one XGBoost primary classifier with deterministic and logistic baselines only. It must pass MM-01–MM-06 and system gates before promotion. This follows the evidence that method choice must fit the data and operating problem [4] and avoids treating public benchmark transfer as proven [5]. Residual risk is that engineered features/XGBoost may fail performance or calibration gates. That outcome triggers data/label review and a new controlled comparison, possibly including a temporal candidate; it does not justify relaxing gates or silently adding a second production model.

# 6. References

[1] A. K. S. Jardine, D. Lin, and D. Banjevic, “A review on machinery diagnostics and prognostics implementing condition-based maintenance,” *Mechanical Systems and Signal Processing*, vol. 20, no. 7, pp. 1483–1510, 2006. https://doi.org/10.1016/j.ymssp.2005.09.012

[2] S. Nandi, H. A. Toliyat, and X. Li, “Condition monitoring and fault diagnosis of electrical motors—A review,” *IEEE Transactions on Energy Conversion*, vol. 20, no. 4, pp. 719–729, 2005. https://doi.org/10.1109/TEC.2005.847955

[3] R. B. Randall and J. Antoni, “Rolling element bearing diagnostics—A tutorial,” *Mechanical Systems and Signal Processing*, vol. 25, no. 2, pp. 485–520, 2011. https://doi.org/10.1016/j.ymssp.2010.07.017

[4] T. P. Carvalho, F. A. A. M. N. Soares, R. Vita, R. da P. Francisco, J. P. T. V. Basto, and S. G. S. Alcalá, “A systematic literature review of machine learning methods applied to predictive maintenance,” *Computers & Industrial Engineering*, vol. 137, art. 106024, 2019. https://doi.org/10.1016/j.cie.2019.106024

[5] C. Lessmeier, J. K. Kimotho, D. Zimmer, and W. Sextro, “Condition monitoring of bearing damage in electromechanical drive systems by using motor current signals of electric motors: A benchmark data set for data-driven classification,” *PHM Society European Conference*, vol. 3, no. 1, 2016. https://doi.org/10.36001/phme.2016.v3i1.1577; companion dataset: https://mb.uni-paderborn.de/en/kat/research/bearing-datacenter/data-sets-and-download

[6] Case Western Reserve University, “Bearing Data Center,” Case School of Engineering. https://engineering.case.edu/bearingdatacenter/welcome (accessed Sep. 30, 2026).

[7] A. Banks, E. Briggs, K. Borgendale, and R. Gupta, eds., *MQTT Version 5.0*. OASIS Open, 2019. https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html

[8] K. Stouffer et al., *Guide to Operational Technology (OT) Security*, NIST Special Publication 800-82 Revision 3, 2023. https://doi.org/10.6028/NIST.SP.800-82r3

[9] E. Breck, S. Cai, E. Nielsen, M. Salib, and D. Sculley, “The ML Test Score: A rubric for ML production readiness and technical debt reduction,” in *2017 IEEE International Conference on Big Data*, 2017. https://doi.org/10.1109/BigData.2017.8258038

[10] J. Gama, I. Žliobaitė, A. Bifet, M. Pechenizkiy, and A. Bouchachia, “A survey on concept drift adaptation,” *ACM Computing Surveys*, vol. 46, no. 4, art. 44, pp. 1–37, 2014. https://doi.org/10.1145/2523813
