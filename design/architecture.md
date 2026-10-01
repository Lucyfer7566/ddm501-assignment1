# Phase 4 — High-Level Architecture Design

## 1. Scope and architectural stance

This architecture implements one production-oriented ML system for the fixed binary task: predict whether an industrial motor is at high risk of failure within the next 24 hours from recent vibration, temperature, electrical-current, and RPM data. A valid inference returns a failure-risk probability and `Normal` or `High Risk`; invalid or stale input returns `Insufficient Data`, never a silent `Normal` result.

The architecture uses **one primary gradient-boosted tree classifier** trained on versioned, engineered time-window features. Logistic regression and deterministic sensor rules remain development comparators only; they are not separately deployed predictive models. The selected primary model is a **DESIGN DECISION**, not a claim that this algorithm has achieved the Phase 3B thresholds. Model development must still demonstrate that the candidate passes the defined evaluation and promotion gates [S04; MG-01–MG-05].

The deployment stance is a small, on-premises central ML platform connected to the OT environment through a controlled gateway/broker boundary:

- gateways collect scalar telemetry, perform versioned high-rate vibration windowing/feature extraction, and buffer transmissions;
- central services ingest, validate, store, train, and score on the 15-minute cadence;
- the maintenance dashboard and CMMS/EAM integration remain advisory and have no actuator/control path;
- training and serving use the same versioned feature definitions and transformation package;
- model promotion is human-gated, and serving retains the last approved bundle for rollback; and
- logical components may be co-deployed in a small VM/container cluster. The design does not require one microservice, database, or machine per box.

This stance is realistic for the assumed 100-motor initial and 500-motor growth envelope while respecting OT availability and security concerns [S08]. MQTT is used because its publish/subscribe and delivery semantics suit decoupled IoT collection, with idempotent consumers handling possible duplicate delivery [S07].

## 2. Architecture diagram

The authoritative raw Mermaid source is `diagrams/architecture.mmd`. It contains no Markdown wrapper. The diagram distinguishes:

1. the shared OT and data foundation;
2. the training path;
3. the inference path; and
4. the monitoring, feedback, retraining, and deployment loop.

## 3. End-to-end paths

### 3.1 Training path

**Required flow:** sensor/history data → ingestion/storage → validation → preprocessing/feature engineering → training → evaluation → model registry → deployment.

1. **Sensor/history acquisition.** Motor sensors provide vibration, temperature, electrical current, and RPM. The gateway attaches stable motor/sensor identity, timestamps, units, schema/feature versions, and quality status. It converts high-rate vibration windows to versioned feature packets and keeps raw vibration in a bounded local ring buffer for selective upload and investigation.
2. **Ingestion and immutable storage.** The gateway publishes scalar envelopes and vibration feature packets through MQTT 5 over the controlled OT/IT boundary. The ingestion service authenticates the source, assigns an idempotency key, writes an immutable raw record to object storage, and routes a copy for validation. Plant history imported from approved sources follows the same landing contract.
3. **Validation and curation.** Batch validation checks schema, units, asset identity, timestamps, continuity, duplicates, range rules, and lineage. Invalid records enter quarantine with reason codes. Valid records are written to versioned curated partitions; no record is silently repaired or dropped.
4. **Outcome and label construction.** The CMMS/EAM adapter imports approved failure and maintenance events. The label builder applies the versioned DR-07 policy: features use data at or before prediction time `t`, and a positive label requires an approved failure in `(t, t + 24 hours]`. Planned shutdowns, ambiguous outcomes, already-failed states, and intervention-censored windows are excluded or marked explicitly.
5. **Preprocessing and feature engineering.** A shared feature package constructs the provisional six-hour motor window, checks DR-06 completeness/freshness, aligns sensor modalities, incorporates RPM/operating context, and generates the exact feature schema used by serving. Dataset manifests retain source partitions, code versions, parameters, and validation results.
6. **Training.** Airflow launches a containerized training job. It uses time- and asset-separated partitions, fits preprocessing on training data only, trains the single XGBoost binary classifier, and records parameters, code/data versions, and artifacts. Logistic regression and rule baselines are evaluated but cannot become additional deployed primary models.
7. **Evaluation.** A separate evaluation job calculates Phase 3B metrics, uncertainty, calibration, motor/operating-regime slices, capacity impact, and comparison with declared baselines. Promotion fails if any mandatory gate is missing or unmet; evaluation does not rewrite thresholds after seeing the untouched test set.
8. **Registry.** Passing candidate bundles—model, feature/preprocessing package, threshold/episode-policy configuration, input/output schema, metrics, and lineage—are stored in MLflow with immutable artifact versions. Registration does not equal deployment.
9. **Deployment.** A maintenance/operations/ML approval record authorizes a registry version. The deployment controller verifies signatures and compatibility, runs contract/smoke checks, deploys it in shadow mode, then atomically changes the approved-model alias. The serving worker caches the last approved bundle so registry unavailability cannot stop inference. A failed rollout returns the alias/runtime to the prior approved version.

### 3.2 Inference path

**Required flow:** new sensor data → ingestion → validation/preprocessing → deployed model → failure-risk probability → risk classification → maintenance dashboard/alert.

1. **New telemetry.** The gateway sends one-second scalar envelopes and one-minute vibration feature packets under the Phase 3A planning envelope. A local durable buffer tolerates the assumed 24-hour disconnection case; replay preserves original event timestamps.
2. **Ingestion.** The MQTT broker decouples gateway delivery from central consumers. The ingestion service deduplicates at-least-once retries, writes immutable raw events, and forwards them to stream validation.
3. **Online validation.** Records with unknown schema, invalid identity/unit/timestamp, corrupt payload, or failed quality rules are quarantined and counted. Valid normalized data are written to TimescaleDB for recent-window assembly. Missing data remain observable.
4. **Scheduled/on-demand feature assembly.** Every 15 minutes per eligible motor, or for an authorized on-demand request, the online feature assembler retrieves the provisional six-hour window. It invokes the same versioned feature package used in training. If DR-06 completeness or NFR-05 freshness fails, it emits `Insufficient Data` with reason codes and a data-health event instead of invoking the classifier.
5. **One deployed model.** The inference worker loads the approved XGBoost bundle from its local cache, checks schema compatibility, and produces a probability in `[0,1]`. The bundle and every result carry model, feature, preprocessing, and threshold-policy versions.
6. **Risk classification.** A deterministic policy compares the probability with the frozen Phase 3B operating threshold and applies versioned hysteresis/deduplication. This policy is not a second predictive model. It creates `Normal` or `High Risk` and maintains one active high-risk episode per motor/horizon.
7. **Prediction and alert persistence.** The system stores the immutable score, class, source window, quality status, versions, and timing. A transition/update to `High Risk` creates or updates one durable alert episode; delivery is retried idempotently.
8. **Maintenance interaction.** The dashboard shows priority, probability, recent trends, freshness, and alert state. The engineer can acknowledge, assign, comment, and link a CMMS/EAM work order. The prediction remains advisory; there is no command path to motor actuation.

### 3.3 Feedback and MLOps path

**Required flow:** monitoring → data/model drift or performance signal → retraining trigger → validation → registry/deployment.

1. **Monitoring collection.** Prometheus/Grafana and scheduled Python/SQL monitors collect sensor availability, missing/stale/corrupt data, queue lag, validation failures, feature freshness, inference latency/error, alert delivery, prediction distribution, episode rate, and service availability. Delayed CMMS outcomes enable recall, precision, F1, false-positive-rate, PR-AUC, and calibration monitoring when labels mature [S09].
2. **Drift/performance analysis.** Scheduled jobs compare feature distributions and operating-regime coverage with approved reference data. Input distribution change produces an investigation signal, not proof of concept drift or an automatic retrain command [S10].
3. **Retraining trigger and review.** A monthly review or validated data-quality, drift, accumulated-label, or performance trigger opens a retraining review record. The ML/platform owner verifies label maturity, data quality, affected slices, and whether retraining is justified.
4. **Controlled retraining.** An approved trigger starts the same Airflow training path from batch validation and dataset construction. It never bypasses data/label validation.
5. **Re-evaluation and promotion.** The candidate must pass the same untouched-test, baseline, uncertainty, system, security, and rollback gates. Only a human-approved registry version reaches shadow and active deployment.
6. **Feedback retention.** Alert acknowledgement, work-order disposition, confirmed failure time, actionable-condition findings, and censored intervention state are immutable feedback inputs. A user cannot overwrite the original prediction or directly convert an unreviewed comment into a training label.

## 4. Major component catalogue

### 4.1 OT collection and ingestion

| ID / component | Purpose and responsibility | Input | Output | Candidate technology/tool and rationale | Important failure behavior |
|---|---|---|---|---|---|
| C01 — Motor sensors and asset registry | Observe motor condition and bind each channel to a stable motor/sensor identity, unit, location, and operating context. | Physical vibration, temperature, electrical current, RPM; approved asset metadata. | Timestamped raw readings and channel metadata. | Existing industrial accelerometers, temperature sensors, current/drive telemetry, RPM encoder/drive signal, plus the plant asset registry. Reusing plant-grade sensing avoids inventing a vendor-specific stack. | A disconnected, implausible, or unregistered channel is marked unavailable/invalid; it never generates a healthy default. Sensor health is exposed separately from motor risk. |
| C02 — Industrial edge gateway and local buffer | Collect protocols close to equipment, normalize identity/time/units, compute versioned high-rate vibration features, buffer outages, and publish without performing risk inference. | C01 readings, asset metadata, signed feature package/configuration. | One-second scalar envelopes, one-minute vibration feature packets, quality metadata, selectively uploaded raw windows. | Industrial Linux PC running containerized collectors; Python/NumPy/SciPy for the validated signal pipeline, with a compiled implementation only if profiling requires it; durable local filesystem/SQLite queue. This bounds central bandwidth while retaining a reproducible feature version. | During central/network outage, buffer up to the NFR-06 envelope and alarm on utilization. On overflow risk, preserve metadata/quality evidence, apply the documented retention priority, and never relabel missing periods as normal. A feature-package mismatch blocks that channel’s publication. |
| C03 — MQTT 5 broker | Authenticate gateways, decouple producers/consumers, apply topic ACLs, and provide observable delivery/backpressure semantics. | TLS-authenticated gateway publications. | Ordered-per-motor topics for ingestion consumers; broker delivery/connection metrics. | On-premises EMQX MQTT 5 deployment. MQTT is lightweight and publish/subscribe based; its QoS semantics are explicit [S07]. EMQX supplies clustered operation and access controls without adding a general-purpose streaming platform at this scale. | If unavailable, gateways buffer locally. At-least-once delivery may duplicate messages, so downstream writes are idempotent. Broker acknowledgement is not treated as proof of storage or prediction. |
| C04 — Ingestion and idempotent persistence service | Validate transport envelope/authenticity, attach ingestion time, deduplicate by event key, persist immutable raw events, and route online validation. | C03 telemetry and approved historical imports. | Raw object-store record, validation work item, ingestion audit/metrics. | Horizontally scalable Python service using Pydantic contracts, MQTT client, MinIO client, and PostgreSQL/TimescaleDB drivers. This is sufficient for 100–500 motors and keeps contracts testable. | Retry on transient storage failures with bounded backoff; do not acknowledge completion before durable raw persistence. Poison messages go to quarantine. Queue lag and rejected events alert operators. |
| C05 — Validation rules and quarantine | Apply one governed set of schema, identity, timestamp, unit, range, continuity, duplicate, ordering, and cross-channel alignment checks in stream and batch modes. | Raw events/partitions, schema registry, asset metadata, quality rules. | Valid normalized records or immutable quarantined records with reason codes and metrics. | Pydantic/Pandera for record/dataframe checks and Great Expectations-style batch reports, packaged from one versioned rules repository. Lightweight Python tooling matches the selected data scale. | Invalid data are isolated rather than repaired silently. If rules/configuration cannot load, processing stops for that partition while prior valid serving data remain available; bypass is forbidden without an audited waiver. |

### 4.2 Data foundation and shared transformations

| ID / component | Purpose and responsibility | Input | Output | Candidate technology/tool and rationale | Important failure behavior |
|---|---|---|---|---|---|
| C06 — Historical object store | Retain immutable raw, quarantine, curated, dataset-manifest, selectively uploaded raw-vibration, and model-artifact data under lifecycle policies. | C04 raw events, C05 batch results, training datasets/artifacts. | Versioned S3 objects/partitions with checksums, lineage, retention class, and access policy. | On-premises MinIO with S3 API, encryption, replication/erasure protection, and lifecycle rules. It separates large historical data from the operational query store and supports reproducible training snapshots. | Failed writes block downstream acknowledgement; checksum failure quarantines the object. Capacity alarms precede retention exhaustion. Loss of this store blocks training/replay but serving continues with cached features/model until operational retention limits are reached. |
| C07 — Operational time-series and prediction store | Serve recent validated telemetry/features, prediction records, alert state, idempotency keys, and dashboard queries. | C05 valid records; C15 predictions; C16 episode updates. | Queryable six-hour windows, immutable predictions, alert state, audit records. | PostgreSQL with TimescaleDB extension. It supports time-window queries, relational integrity, deduplication, and moderate projected volume without a separate feature-store platform. | Use transactions, backups, and a replica appropriate to the 99.5% target. If writes fail, durable ingestion/alert work is retried; inference does not claim successful delivery. Read-only degradation may show the last timestamp clearly but may not create fresh scores from stale data. |
| C08 — CMMS/EAM adapter and label builder | Exchange work-order/maintenance feedback and produce governed 24-hour outcome labels without modifying source records. | Approved CMMS/EAM events, C07 prediction/alert IDs, failure-definition policy. | Versioned outcome/label tables, censored states, linked dispositions, integration status. | Authenticated REST/file adapter plus scheduled Python/SQL label job under Airflow. This accommodates an unknown plant product while preserving a stable internal contract. | If the CMMS is unavailable, queue outbound links and delay label maturity. Do not infer negative outcomes from missing work orders. Invalid/ambiguous events remain censored and cannot enter training automatically. |
| C09 — Shared feature and preprocessing package | Provide identical feature definitions for edge/batch/online paths: alignment, missingness indicators, RPM context, aggregation, transformations, and schema. | Valid telemetry windows, feature configuration, fitted training-only transformation state. | Versioned feature vector, completeness/freshness result, lineage and quality indicators. | Versioned Python wheel/container using NumPy, SciPy, pandas, and scikit-learn-compatible transformers. One package reduces training-serving skew and is appropriate for engineered tabular features. | Schema/version mismatch returns `Insufficient Data` online and fails training jobs. Fitted transformations are immutable within a model bundle. No online code may refit imputation/scaling state. |

### 4.3 Training, evaluation, registry, and deployment

| ID / component | Purpose and responsibility | Input | Output | Candidate technology/tool and rationale | Important failure behavior |
|---|---|---|---|---|---|
| C10 — Workflow orchestrator and dataset builder | Schedule validation, label joins, dataset snapshots, training/evaluation jobs, monthly reviews, and approved retraining runs with auditable dependencies. | C06 curated partitions, C08 labels, C09 package/configuration, approved trigger. | Immutable dataset manifest, time/asset splits, job records, downstream tasks. | Apache Airflow on the central VM cluster. The workload is predominantly scheduled/batch, and Airflow provides retries, dependency state, and auditable DAG runs without a real-time feature platform. | A failed task stops dependent work; retries are bounded and idempotent. Training never proceeds from an incomplete manifest. Orchestrator outage does not stop the already deployed inference service. |
| C11 — Primary model training service | Fit one production candidate and record reproducible parameters/artifacts. | C10 training/validation partitions and C09 fitted preprocessing state. | XGBoost binary classifier candidate, training metadata, baseline results. | Containerized Python with XGBoost and scikit-learn evaluation adapters. Gradient-boosted trees handle nonlinear interactions in engineered sensor/context features while remaining fast to serve and less infrastructure-intensive than a raw-sequence deep model. | Compute/data failure produces no candidate. NaN/invalid-feature or class/label checks fail early. A completed training job has no deployment authority. |
| C12 — Independent evaluation and promotion gate | Evaluate model/system acceptance criteria, calibration, slices, uncertainty, baselines, schema compatibility, and policy threshold on untouched data. | C11 candidate, C10 test manifest, Phase 3B metric policy, serving contract. | Signed evaluation report with pass/fail, frozen threshold/episode policy, and any rejection reasons. | Separate Python evaluation container using scikit-learn metrics, bootstrap jobs, and machine-readable policy checks. Separation prevents the trainer from declaring itself deployable. | Missing metric, inadequate lineage, leakage, baseline regression, schema mismatch, or failed mandatory threshold rejects promotion. Thresholds are not changed after test inspection. |
| C13 — Model registry and artifact lineage | Store immutable model bundles, evaluation evidence, aliases, approvals, and rollback history. | C11 artifacts and C12 signed evaluation report. | Versioned candidate/approved model bundle and registry metadata. | Self-hosted MLflow Tracking/Model Registry backed by PostgreSQL and MinIO. It is sufficient for experiment lineage, artifacts, approval aliases, and rollback at this scale. | Registry outage blocks new registration/deployment but does not stop C15, which uses a verified local cache. Corrupt or unsigned artifacts are never loaded. Previous approved versions remain retained. |
| C14 — Deployment controller and container runtime | Enforce human approval, compatibility/smoke tests, shadow rollout, atomic activation, and rollback on the central serving cluster. | Approved C13 bundle, signed container image, deployment approval. | Running approved version, deployment audit, health/rollback status. | Self-hosted CI runner, Harbor-compatible OCI registry, and Docker containers on a small on-premises VM cluster; Ansible or equivalent performs repeatable rollout. This avoids requiring Kubernetes for the projected load. | A failed signature, contract, smoke, or shadow check leaves the active version unchanged. Runtime health regression rolls back to the last approved bundle. No training process can directly update active serving. |

### 4.4 Inference and maintenance interaction

| ID / component | Purpose and responsibility | Input | Output | Candidate technology/tool and rationale | Important failure behavior |
|---|---|---|---|---|---|
| C15 — Online feature assembler and inference worker | Schedule 15-minute/on-demand scores, enforce window validity, load the approved cached bundle, and produce one probability per valid motor-window. | C07 recent data, C09 package, C13/C14 approved cached model bundle, authorized request. | Failure-risk probability or `Insufficient Data`, versions, timestamps, latency/quality metrics. | Python worker plus FastAPI for authenticated on-demand requests, XGBoost runtime, and a durable work queue. The small request rate does not justify a separate high-scale serving platform. | Missing/stale data return `Insufficient Data`; schema/model incompatibility blocks scoring. A process failure retries idempotently. If registry is down, use only the verified cached approved version; if no valid cache exists, fail closed and alert. |
| C16 — Risk policy and alert-episode manager | Convert probability to `Normal`/`High Risk`, apply frozen threshold/hysteresis/deduplication, and maintain one active episode per motor/horizon. | C15 probability/result, approved threshold-policy version, current episode state. | Immutable classification, episode create/update/close event, audit record. | Small Python policy module/service with PostgreSQL transactions and versioned configuration. Keeping the policy deterministic makes alert burden testable without adding a second predictive model. | Missing/invalid probability cannot become `Normal`. Transaction or notification failure retains a retryable outbox record. Configuration/version mismatch blocks classification and raises an operational alert. |
| C17 — Maintenance dashboard, notification, and CMMS link | Present prioritized risk and evidence; support acknowledgement, assignment, comment, closure, escalation, and work-order linkage. | C07 predictions/history, C16 alert events, C08 CMMS status, user identity/roles. | User-visible dashboard/notification, audited workflow actions, linked maintenance feedback. | Thin internal web application (React or equivalent with FastAPI), PostgreSQL-backed workflow, configurable email/SMS/enterprise-notification connector, and CMMS REST adapter. A purpose-built thin layer covers interactions that a monitoring dashboard alone cannot. | Prediction/alert persistence precedes notification. Dashboard outage queues delivery and preserves predictions; recovery replays idempotently. Stale views show freshness. Unauthorized users cannot alter state, approve labels/models, or actuate equipment. |

### 4.5 Monitoring, drift, and retraining control

| ID / component | Purpose and responsibility | Input | Output | Candidate technology/tool and rationale | Important failure behavior |
|---|---|---|---|---|---|
| C18 — Observability and ML monitoring | Collect service/data/model/output/outcome measures; route alerts; preserve queryable history and runbooks. | Metrics/logs from C02–C17, reference distributions, matured C08 outcomes. | Operational dashboards, alert events, drift/performance reports, SLO calculations. | Prometheus and Grafana for service metrics; centralized structured logs; scheduled Python/SQL jobs for data quality, drift, calibration, and delayed performance. This keeps ML monitoring transparent and avoids a second large platform. | Monitoring has its own heartbeat. Collector/dashboard failure alerts through an independent route; serving may continue temporarily, but deployment/promotion is blocked when required monitoring evidence is unavailable. |
| C19 — Retraining review and trigger controller | Turn monthly review or validated signals into an auditable investigation and, only after approval, start C10. | C18 data/drift/performance signal, accumulated labels, owner decision. | Review/ticket record, approved or rejected retraining trigger, link to C10 run. | Airflow trigger API plus the organization’s ticket/approval workflow. This supports human control and traceability rather than automatic drift-to-deployment. | Drift alone cannot start deployment. Missing label maturity, data-quality failure, or absent approval closes/pauses the trigger. Controller outage leaves current serving unchanged. |

### 4.6 Cross-cutting platform security

| ID / component | Purpose and responsibility | Input | Output | Candidate technology/tool and rationale | Important failure behavior |
|---|---|---|---|---|---|
| C20 — Identity, secrets, network, and audit controls | Enforce OT/IT segmentation, service/user identity, least privilege, encryption, secret lifecycle, artifact integrity, and privileged-action audit. | User/service identities, certificates/secrets, access policies, network flows. | Authenticated/authorized connections, encrypted data, audit events, denied-access evidence. | Existing plant identity provider using OIDC/OAuth2, internal PKI/mTLS, Vault-compatible secrets manager, host firewall/DMZ controls, encrypted MinIO/PostgreSQL volumes. Reuse of enterprise controls avoids a parallel identity stack and aligns with OT security guidance [S08]. | Authentication or secret-validation failure denies the action rather than bypassing control. Expiring certificates/secrets alert before expiry. Audit-store failure blocks privileged deployment/approval actions; it does not create a control path to motors. |

## 5. Deployment topology and scaling

The logical architecture can begin on a small on-premises cluster rather than a full Kubernetes estate:

- **Gateway tier:** one industrial gateway per suitable motor group, each with local durable buffering and signed feature package.
- **Messaging/application tier:** at least two broker/application VMs where required to meet the 99.5% availability target. Ingestion, validation, inference, policy, and dashboard containers scale horizontally by motor-ID partition or durable work queue.
- **Data tier:** TimescaleDB with backup/replica strategy and MinIO with replicated/erasure-protected storage. Retention and capacity follow DR-10’s 100/500-motor envelope.
- **ML batch tier:** Airflow launches bounded training/evaluation containers on the same central compute pool or a dedicated worker. Training failure cannot consume serving resources below reserved inference capacity.

Scale-out is driven by broker/queue lag, validation throughput, feature freshness, inference latency, and storage utilization. Partitioning by motor ID preserves per-motor ordering while allowing independent workers. A platform already standardized on Kubernetes may host the same containers, but Kubernetes is not required by this design.

## 6. Reliability and graceful degradation

| Failure condition | Required behavior |
|---|---|
| Sensor/channel missing, corrupt, or stale | Quarantine/mark quality failure; use only a model explicitly validated for that available-channel pattern, otherwise return `Insufficient Data`; raise data-health alert; never default to `Normal`. |
| Gateway/network interruption | Buffer locally for the NFR-06 planning envelope, show connectivity/buffer health, replay original timestamps, deduplicate centrally, and prevent expired replay from becoming a current alert. |
| Broker or ingestion unavailable | Gateway retains data; consumers resume from durable delivery state; bounded retries/backpressure; no acknowledgement is interpreted as end-to-end success. |
| Raw object store unavailable | Stop acknowledgement of new durable landings and retry; training/replay pauses. Cached online data/model may serve only while freshness rules remain satisfied. |
| TimescaleDB unavailable | Durable ingestion/outbox work waits; no new score/alert is claimed persisted. Dashboard may show a clearly stale last-known view. |
| Feature package/schema mismatch | Stop affected training/inference; quarantine or `Insufficient Data`; do not dynamically coerce an unknown schema. |
| Inference worker/model cache unavailable | Retry on another worker; use only verified last-approved cache. With no valid model, fail closed and alert rather than applying an unapproved artifact or reporting normal. |
| Registry unavailable | Continue serving the verified cached approved model; block registration, promotion, and deployment. |
| Candidate evaluation or deployment fails | Reject/retain candidate evidence, leave active model unchanged, or roll back to the previous approved version. |
| Dashboard/notification unavailable | Preserve prediction and durable alert outbox; retry delivery idempotently; expose recovery backlog. The model never actuates equipment as a fallback. |
| CMMS/outcome feed unavailable | Queue linkage, delay labels/performance metrics, and prevent missing outcomes from becoming negatives or retraining data. |
| Monitoring unavailable | Alert via independent heartbeat; continue current approved serving only for the authorized grace period; block promotion until observability is restored. |

## 7. Technology decision summary

| Concern | Selected candidate | Why reasonable here | Deliberately not selected |
|---|---|---|---|
| OT messaging | MQTT 5 / EMQX | Native lightweight publish/subscribe, explicit delivery semantics, gateway decoupling [S07]. | A general-purpose Kafka platform at the OT edge; unnecessary for the assumed load. |
| Historical storage | MinIO S3 object storage | Immutable low-cost partitions, lineage, lifecycle, training snapshots. | Storing all raw/history only in a relational database. |
| Operational queries | PostgreSQL + TimescaleDB | Time windows plus relational prediction/alert integrity at moderate scale. | A separate online feature-store product before demonstrated need. |
| Validation/features | Versioned Python packages using Pydantic/Pandera, NumPy/SciPy/pandas, scikit-learn transformers | Shared, testable logic across batch/online paths; suited to engineered sensor features. | Independent notebook transformations or duplicated training/serving code. |
| Batch orchestration | Apache Airflow | Auditable scheduled dependencies and controlled retries for validation/training/retraining. | Automatic retraining directly from a drift alert. |
| Primary model | XGBoost binary classifier | Nonlinear tabular interactions, fast central inference, and lower operational complexity than raw-waveform deep learning. | Multiple interacting production models or an unvalidated deep temporal stack. |
| Registry | MLflow + MinIO/PostgreSQL | Experiment/artifact lineage, aliases, approvals, and rollback without a large managed platform. | Treating a model file in object storage as deployment approval. |
| Runtime | Docker containers on on-premises VMs | Repeatable deployment and horizontal workers without requiring Kubernetes at the projected scale. | A mandatory cloud or Kubernetes dependency before trade-off analysis/organizational need. |
| Monitoring | Prometheus/Grafana plus scheduled Python/SQL model monitors | Covers infrastructure and explicit ML checks with transparent logic [S09]. | Offline model accuracy as the only production monitor. |

These are candidate technologies for the proposed design, not claims about products already owned by the hypothetical plant. Equivalent organization-standard tools may replace them if they preserve the same contracts, controls, and failure behavior.

## 8. Requirements, goals, and rubric traceability

| Required coverage | Architecture evidence |
|---|---|
| AR-01 — clear diagram | `diagrams/architecture.mmd`, rendered and syntax-checked during Phase 4. |
| AR-02 — system data flow | Sections 3.1–3.3 and the directional diagram paths. |
| AR-03 — ingestion, preprocessing, training, inference | C02–C05, C09–C16 and explicit training/inference sequences. |
| AR-04 — component purpose/responsibility | C01–C20 catalogue documents purpose, responsibility, input, output, technology rationale, and failure behavior. |
| AR-05 — technologies/tools | Component catalogue and Section 7 provide scenario-specific candidates and rationale. |
| Training path required by `AGENTS.md` | Data → validation → feature package → XGBoost training → independent evaluation → MLflow registry → approved deployment. |
| Inference path required by `AGENTS.md` | New sensor data → MQTT ingestion → online validation/feature assembly → deployed model → probability → threshold/class → dashboard/alert. |
| Monitoring and retraining | C18/C19 implement data/service/model/outcome monitoring, investigation triggers, validation, registry, approval, and rollback. |
| FR-01 / one primary model | C11/C15 deploy one XGBoost binary classifier; deterministic thresholding is policy, not another predictive model. |
| FR-03 / NFR-05 / DR-06 | C05/C09/C15 preserve `Insufficient Data` and never map missing data to `Normal`. |
| NFR-01 / NFR-02 | Queue-based central serving is capacity-tested at 100/500 motors against p95 ≤2 minutes and normal-path <15 minutes. |
| NFR-04 / NFR-06 | Cached approved model, rollback, local gateway buffer, durable retries, and explicit degradation support reliability. |
| NFR-07 / NFR-08 / NFR-09 | Versioned packages/artifacts, C18 monitoring, C19 review trigger, independent C12 gate, and human C14 deployment. |
| SM-01–SM-05 | C03–C07/C15–C18 expose alert delivery, latency, availability, reconciliation, and valid-coverage measurements. |
| MM-01–MM-06 | C12 evaluates recall, precision, F1, negative-motor-day FPR, PR-AUC, and Brier/calibration before registration/promotion. |

## 9. Deferred items for Phase 5 and implementation planning

The architecture makes explicit decisions needed for a coherent design, but it does not replace the required trade-off analysis. Phase 5 must evaluate at least:

- false-negative reduction versus false-positive/alert burden;
- gateway/central on-premises inference versus full edge or cloud inference;
- 15-minute freshness and feature-packet cadence versus infrastructure/storage cost; and
- the XGBoost engineered-feature model versus simpler and more complex alternatives.

Plant-specific products, network zones, exact gateway count, raw-vibration local retention, CMMS/EAM API, notification route, compute sizing, backup/recovery objectives, and security policy remain dependent on a real site survey. Any later replacement must retain the documented input/output contracts, metric gates, and failure semantics.
