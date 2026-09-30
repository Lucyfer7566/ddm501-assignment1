# Phase 2 Research Notes

## Research boundary

- These notes support later design work; they are not report prose.
- The task remains binary classification: high risk of industrial-motor failure within the next 24 hours.
- No source reviewed demonstrates that the proposed hypothetical system has been trained, tested, deployed, or achieved any performance level.
- Source IDs refer to `research/sources.md` and BibTeX keys to `research/references.bib`.

## A. Predictive maintenance for rotating machinery and motors

### Evidence

1. Jardine, Lin, and Banjevic organize condition-based maintenance into data acquisition, data processing, and maintenance decision-making. They distinguish diagnostics after/at fault occurrence from prognostics before failure and discuss multi-sensor information [S01; `jardine2006cbm`].
2. Nandi, Toliyat, and Li review motor-fault signatures across current, speed, torque, vibration, noise, and thermal measurements [S02; `nandi2005motors`].
3. Randall and Antoni explain why localized rolling-element bearing defects can generate impulsive vibration responses and why speed variation affects frequency-domain diagnosis [S03; `randall2011bearing`].

### Design relevance

- The system boundary must include acquisition, processing, prediction delivery, and a maintenance decision workflow. A classifier alone would not satisfy the condition-based-maintenance lifecycle.
- The fixed 24-hour prediction is prognostic rather than merely diagnostic. Fault-state benchmark accuracy cannot be treated as evidence of 24-hour failure prediction.
- Multiple signal types are defensible because they provide different views of motor and drive condition. Their value must still be validated for the eventual plant and motor population.
- The final problem justification should compare the ML design with existing threshold/rule-based monitoring and planned maintenance. Research does not justify removing deterministic protection or expert review.

## B. Condition-monitoring signals

| Signal | Verified support | Material implication | Limitation to retain |
|---|---|---|---|
| Vibration | Bearing faults can create impulsive vibration signatures; envelope analysis and speed-aware processing are established techniques [S03]. Paderborn and CWRU publish motor/drive vibration records [S05, S06]. | Preserve raw or sufficiently rich windows for feature extraction; consider time, frequency, envelope, and speed-normalized features. | Sensor position, mounting, sampling, machine structure, load, and speed affect observed signals. Public test rigs do not prove transfer to a plant motor. |
| Electrical current | Motor-current signature analysis is an established motor-monitoring family [S02]. Paderborn includes synchronous current and vibration [S05]. | Current can complement mechanical sensing and may be available through drive electronics. | Paderborn reports that current-based classification is sensitive to training-data damage type and is weaker than vibration in its experiments; do not generalize that comparison to all motors. |
| Temperature | Thermal measurement is among the motor condition-monitoring techniques reviewed by Nandi et al.; Paderborn includes supportive temperature measurements [S02, S05]. | Temperature can provide slower condition/operating-context information and support plausibility checks. | Temperature response may lag fast faults and is affected by ambient/load conditions. No reviewed source establishes a universal alarm or prediction threshold. |
| Rotational speed (RPM) | Motor speed is a monitored signal [S02]; bearing characteristic frequencies depend on shaft speed, and speed variation can require order tracking [S03]. Both Paderborn and CWRU expose speed information [S05, S06]. | Treat RPM as operating context and a feature/preprocessing input, not only as another independent sensor reading. | Speed effects must be separated from degradation effects to reduce false drift/fault indications. |

### Cross-sensor implications

- Sensor timestamps and asset identity must be aligned before feature generation.
- Operating regime variables such as RPM and load are needed to interpret vibration/current changes.
- Missing or stale channels must be represented explicitly; silent imputation could conceal a sensor or ingestion failure.
- The research supports investigating fusion, but it does not prove that every sensor must be available for every prediction.

## C. ML approaches relevant to the fixed task

### Evidence

- The predictive-maintenance literature uses multiple ML families and emphasizes matching the method to available historical data, preprocessing, validation, and maintenance needs [S04].
- Paderborn demonstrates data-driven classification using engineered signal information, while also showing sensitivity to the damage population used for training [S05].
- None of the reviewed sources identifies a universally superior model for this assignment’s exact target.

### Candidate approach classes for later evaluation

| Approach | Why it is relevant | Principal risk or prerequisite |
|---|---|---|
| Interpretable linear/logistic baseline | Directly matches binary probability output and provides a minimum-complexity comparator. | May not capture nonlinear interactions or time-frequency structure. |
| Tree-based ensemble on engineered window features | Handles nonlinear interactions and mixed sensor/operating features while remaining practical for tabular pipelines. | Feature quality, calibration, and transfer across motors/loads must be tested. |
| Kernel/SVM classifier | Commonly applicable to engineered condition-monitoring features and smaller datasets. | Probability calibration and scaling with dataset size require attention. |
| One-dimensional temporal/deep model | Can learn representations from raw or lightly processed sequences. | Requires enough representative labelled data, stronger compute/latency controls, and more demanding interpretability/maintenance work. |

### Evaluation cautions for later phases

- Keep the fixed binary target and probability output; do not substitute RUL regression, anomaly detection, or multi-class fault diagnosis.
- Split evaluation by time and asset/motor where possible. Randomly splitting overlapping windows from the same recording can create an optimistic estimate; the exact split policy remains a design question until the data schema is defined.
- Model selection must consider rare failures, asymmetric false-negative/false-positive consequences, probability calibration, latency, maintainability, and delayed labels. Thresholds and metrics belong to Phase 3.
- Public benchmark results are not project results and must never be presented as achieved performance.

## D. Dataset assessment

### Selected public datasets

| Dataset | Signals/conditions verified | Useful for | Not sufficient for |
|---|---|---|---|
| Paderborn University Bearing DataCenter [S05] | Synchronous vibration and motor current; supportive speed, torque, radial load, and temperature; multiple operating conditions; healthy, artificial-damage, and accelerated-life-damage bearings. | Multimodal ingestion/preprocessing prototypes; signal-feature experiments; operating-condition robustness studies; demonstration of dataset provenance and licensing. | Native 24-hour failure-risk labels, production prevalence, natural plant workflows, or claimed deployment performance. |
| Case Western Reserve University Bearing Data Center [S06] | Motor-bearing vibration; documented accelerometer locations, load/speed conditions, normal data, and seeded inner-race/ball/outer-race faults. | Vibration pipeline prototyping and cross-load/speed checks. | Multimodal training, natural degradation, production failure prediction, or the assignment’s 24-hour target. |

### Dataset conclusion

- No selected public dataset natively represents the complete target: an industrial motor’s probability of failure within the next 24 hours using recent vibration, temperature, current, and RPM.
- Public datasets may support preprocessing and method prototyping only. A defensible production design requires plant telemetry joined to timestamped failure/maintenance events and a precisely defined label-generation rule.
- Combining unrelated public datasets does not automatically create a valid multimodal training set; signals must be time-aligned on the same assets and operating periods.
- Dataset licenses and provenance must be retained. Paderborn permits non-commercial academic use with attribution; the repository terms must be checked again before any actual reuse.

### Rejected core candidates

- AI4I 2020: synthetic and missing vibration/current; immediate machine-failure target rather than a natural 24-hour horizon.
- NASA C-MAPSS: simulated turbofan RUL, which changes the asset and target.
- IMS bearings: useful run-to-failure vibration data, but not the full motor sensor/target combination and less aligned than the selected motor-drive datasets.

## E. IoT and industrial data ingestion

### Evidence

- MQTT 5.0 is a lightweight publish/subscribe transport with three delivery quality-of-service levels and decoupled producers/consumers [S07].
- NIST SP 800-82 Rev. 3 treats industrial control/measurement environments as OT and emphasizes their performance, reliability, safety, topology, and security constraints [S08].

### Requirements to carry into system design

- Every observation needs asset/sensor identity, event timestamp, ingestion timestamp, units, schema/version information, and quality status.
- Gateway and broker interruption must not silently convert missing data into healthy evidence. The system needs observable buffering, retry, backpressure, and stale-data behavior.
- At-least-once delivery can produce duplicates; consumers must support deduplication or idempotent writes if that delivery level is selected.
- QoS is a trade-off, not a synonym for end-to-end reliability. Broker acknowledgement does not prove that data were stored, processed, or used in a prediction.
- Authentication, authorization, encryption, credential lifecycle, network segmentation, and least-privilege access must be considered at the OT/IT boundary.
- High-rate raw vibration may require gateway-side batching or feature extraction, but the location and amount of edge processing remain open design questions.

## F. Production ML monitoring, drift, retraining, and reliability

### Evidence

- Breck et al. identify production-readiness tests and monitoring across data, features, models, infrastructure, and live behavior; offline accuracy is not sufficient [S09].
- Gama et al. define concept drift around change in the relationship between inputs and target over time and survey detection/adaptation approaches [S10].
- NIST requires OT security measures to respect performance, reliability, and safety needs [S08].

### Monitoring layers to carry forward

1. **Sensor/data health:** channel availability, missingness, stale timestamps, range/unit violations, clock skew, duplicate/out-of-order events, and operating-regime coverage.
2. **Pipeline/service health:** gateway backlog, broker/consumer lag, feature freshness, inference errors, response time, prediction/alert delivery, and fallback activation.
3. **Model input behavior:** feature-distribution changes by motor and operating regime, unseen categories/ranges, and training-serving transformation consistency.
4. **Model output behavior:** probability/class distribution, alert rate, confidence/calibration checks when labels become available, and behavior by important asset/operating slices.
5. **Outcome performance:** false negatives, false positives, recall/precision or other selected metrics after a reliable failure label matures; delayed labels must be handled explicitly.

### Retraining implications

- A distribution alert is evidence for investigation, not automatic proof that the input-target relationship changed.
- Automatic retraining solely on a calendar schedule is not justified by the reviewed evidence.
- A later design can combine scheduled review with triggers based on validated data quality, drift, accumulated labelled events, and observed performance degradation.
- Candidate models must be evaluated and approved before registry promotion; rollback and previous-model retention are reliability requirements, not performance claims.

## Evidence gaps remaining after Phase 2

1. No public source or dataset provides the complete scenario-specific 24-hour failure label.
2. No evidence defines the plant’s operational meaning of “motor failure” or the event source that confirms it.
3. No organization-specific baseline exists for downtime, maintenance cost, false alarms, failure prevalence, or current maintenance performance.
4. No scenario fact specifies fleet size, sensor sampling rates, data volume, retention, prediction cadence, latency, throughput, or availability.
5. No evidence establishes acceptable model/system thresholds for this hypothetical organization.
6. No plant-specific privacy, safety, cybersecurity, network, or regulatory constraints have been supplied.
7. No evidence establishes how well a model trained on public test rigs would generalize across the target plant’s motor types, loads, mounting conditions, and environments.
8. No current-state maintenance workflow or existing integration endpoint has been supplied.

