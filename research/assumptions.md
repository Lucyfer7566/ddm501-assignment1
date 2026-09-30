# Facts, Assumptions, Decisions, and Open Questions

## Usage rule

- **FACT** entries require an external source or an explicit authoritative project constraint.
- **DESIGN ASSUMPTION** entries are hypothetical conditions used to make later design work concrete. They are not measured facts.
- **DESIGN DECISION** entries are choices already fixed by `scenario.md` or by the project’s evidence policy. Architecture/model choices not yet made remain open.
- **OPEN QUESTION** entries must be resolved by evidence, stakeholder input, or an explicit later design assumption.

## FACT

### FACT-01 — Condition-based maintenance lifecycle

- Condition-based maintenance links data acquisition, data processing, and maintenance decision-making.
- Evidence: S01 / `jardine2006cbm`.
- Boundary: This does not establish how the hypothetical plant currently performs maintenance.

### FACT-02 — Motor condition can be observed through multiple signal families

- Published motor-condition-monitoring literature discusses motor current, vibration, speed, torque, and thermal measurements.
- Evidence: S02 / `nandi2005motors`.
- Boundary: It does not establish that every signal is available or equally useful in the target plant.

### FACT-03 — Speed affects vibration interpretation

- Rolling-bearing characteristic frequencies depend on shaft speed, and varying speed can require speed-aware processing such as order tracking.
- Evidence: S03 / `randall2011bearing`.

### FACT-04 — Paderborn is a relevant multimodal benchmark, not a 24-hour target dataset

- The Paderborn dataset contains synchronous vibration and motor-current data plus supporting speed, torque, radial-load, and temperature measurements under multiple operating conditions.
- Its published records are labelled condition experiments rather than native 24-hour failure-risk examples.
- Evidence: S05 / `lessmeier2016paderborn` and the official dataset page.

### FACT-05 — CWRU is a vibration fault-diagnosis benchmark

- CWRU provides normal and seeded-fault motor-bearing vibration data with documented load, speed, and sampling conditions.
- Evidence: S06 / `cwruBearingData`.
- Boundary: It is not evidence of natural production failure prediction.

### FACT-06 — MQTT provides distinct delivery semantics

- MQTT 5.0 specifies at-most-once, at-least-once, and exactly-once delivery qualities, each with different loss/duplication/overhead behavior.
- Evidence: S07 / `oasis2019mqtt`.

### FACT-07 — Production ML and OT require monitoring beyond model accuracy

- Production ML literature identifies data, feature, infrastructure, and live-model monitoring needs; NIST OT guidance additionally emphasizes performance, reliability, safety, and security constraints.
- Evidence: S08 and S09 / `nist2023otsecurity`, `breck2017mltestscore`.

### FACT-08 — Concept drift is not identical to input distribution change

- Concept drift concerns change in the relationship between inputs and the target over time; input drift alone does not prove performance degradation.
- Evidence: S10 / `gama2014drift`.

## DESIGN ASSUMPTION

All entries below are **PROPOSED, NOT VERIFIED**. Later phases may revise them.

### DESIGN ASSUMPTION-01 — Hypothetical operating organization

- The system is designed for a hypothetical industrial plant with multiple instrumented motors; no real organization or deployed fleet is claimed.
- Reason: the assignment requires organizational context, but `scenario.md` supplies none.

### DESIGN ASSUMPTION-02 — Failure event source exists

- A maintenance/work-order or failure-event system can provide motor identity and a trustworthy event timestamp for label construction.
- Reason: the 24-hour target cannot be trained or evaluated without timestamped outcomes.

### DESIGN ASSUMPTION-03 — Sensor time can be aligned

- Gateways provide sufficiently synchronized event timestamps and stable motor/sensor identifiers to construct recent multimodal windows.
- Reason: vibration, temperature, current, and RPM must refer to the same asset and period.

### DESIGN ASSUMPTION-04 — Operating regimes vary

- Motor load and speed vary enough that RPM and available operating variables must be treated as context during preprocessing, evaluation, and monitoring.
- Reason: S03, S05, and S06 show operating-condition relevance.

### DESIGN ASSUMPTION-05 — Outcome labels are delayed

- Confirmed failure/non-failure outcomes are not available immediately after prediction and arrive through maintenance operations.
- Reason: this is typical of the defined 24-hour horizon but has not been verified for a real plant.

### DESIGN ASSUMPTION-06 — Network or gateway interruptions can occur

- The ingestion path must tolerate temporary connectivity loss without interpreting missing telemetry as normal operation.
- Reason: required to design graceful degradation; duration and recovery target are not yet selected.

### DESIGN ASSUMPTION-07 — Predictions are advisory

- A high-risk prediction informs maintenance prioritization and does not autonomously shut down a motor.
- Reason: consistent with the maintenance-dashboard/alert scenario and safer until authority and safety integration are specified.

### DESIGN ASSUMPTION-08 — Public data are auxiliary

- Public bearing datasets may be used to prototype preprocessing or evaluation code, while target-model training/evaluation requires representative plant histories and labels.
- Reason: no selected public dataset natively matches the complete target.

### DESIGN ASSUMPTION-09 — Numeric service targets are not facts

- Latency, throughput, availability, retention, sampling, prediction cadence, and retraining values will be explicitly labelled as design assumptions unless later supported by an authoritative source or stakeholder requirement.
- Reason: `scenario.md` and the assignment provide no values.

## DESIGN DECISION

### DESIGN DECISION-01 — Fixed ML target

- Use binary classification to predict whether an industrial motor is at high risk of failure within the next 24 hours.
- Basis: mandatory `scenario.md` constraint.
- Status: FIXED.

### DESIGN DECISION-02 — Fixed outputs

- Return `Normal` or `High Risk` and a failure-risk probability.
- Basis: mandatory `scenario.md` constraint.
- Status: FIXED.

### DESIGN DECISION-03 — One primary predictive model

- Design around one primary predictive model; do not convert the system into RUL prediction, generic anomaly detection, multi-class diagnosis, or a multi-model solution.
- Basis: mandatory `scenario.md` scope.
- Status: FIXED.

### DESIGN DECISION-04 — No benchmark result becomes a project result

- Treat published/dataset benchmark findings as external evidence only. Do not report them as achieved accuracy, deployment evidence, or proof that the hypothetical system meets a threshold.
- Basis: evidence rules and the mismatch between public dataset targets and the project target.
- Status: FIXED.

## OPEN QUESTION

### OPEN QUESTION-01 — What counts as failure?

- Is the event a forced stop, protective trip, component replacement, confirmed fault, inability to meet load, or another operational definition?

### OPEN QUESTION-02 — How will labels be generated?

- What timestamp and system of record establish a positive event, and how are planned shutdowns, duplicate work orders, censored observations, and post-maintenance periods handled?

### OPEN QUESTION-03 — What is the recent-data lookback window?

- The scenario says “recent” measurements but gives no duration or aggregation policy.

### OPEN QUESTION-04 — What are the sensor sampling and edge-processing policies?

- Raw vibration may require a much higher sampling rate than temperature or RPM; the rates, batching, compression, and edge features are unspecified.

### OPEN QUESTION-05 — What is the fleet and operating envelope?

- Motor count, types, power ratings, loads, duty cycles, environments, and criticality classes are unknown.

### OPEN QUESTION-06 — How often is risk scored?

- Event-triggered, fixed-interval, and on-demand prediction remain alternatives.

### OPEN QUESTION-07 — What current systems must be integrated?

- The plant historian, SCADA/DCS, CMMS/EAM, identity platform, notification service, and dashboard technology are unknown.

### OPEN QUESTION-08 — What are the acceptable business, system, and model thresholds?

- Downtime, false-alarm burden, recall, precision, calibration, latency, freshness, throughput, availability, and cost thresholds require stakeholder input or explicit later assumptions.

### OPEN QUESTION-09 — What baseline is available?

- Current planned/reactive maintenance performance and a non-ML/rule-based comparison have not been supplied.

### OPEN QUESTION-10 — What class balance and label delay occur in practice?

- Failure prevalence and outcome latency materially affect evaluation and monitoring, but no organization-specific data exist.

### OPEN QUESTION-11 — Where will inference run?

- Edge, on-premises central service, and cloud-hosted inference require later trade-off analysis under OT connectivity and security constraints.

### OPEN QUESTION-12 — Which MQTT or other transport semantics are appropriate?

- QoS, buffering, deduplication, ordering, and backpressure requirements depend on the selected ingestion architecture and acceptable data loss/latency.

### OPEN QUESTION-13 — What triggers retraining and promotion?

- The balance among calendar review, drift alerts, accumulated labels, performance degradation, manual approval, and rollback is not yet designed.

### OPEN QUESTION-14 — Which plant-specific security, privacy, safety, and retention rules apply?

- NIST provides general OT guidance, but no organization-specific policy or regulatory regime has been supplied.

