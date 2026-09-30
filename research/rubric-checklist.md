# Assignment 1 Rubric Checklist

## Scope and counting convention

- Authoritative assignment inspected: `assignment/INDIVIDUAL ASSIGNMENT 1.pdf` (3 pages).
- The PDF states that this assignment is worth **5% of the course**.
- This checklist records **37 explicit requirements**: **28 weighted report-rubric criteria** and **9 unweighted scenario/submission constraints**.
- A compound rubric line is kept as one requirement when the PDF presents it as one criterion (for example, “Performance: latency, throughput, response time expectations”). Its individual elements are retained in the evidence column so none are lost.
- The trade-off topics listed in the PDF are examples, not mandatory topics. They are therefore not counted as five separate requirements.
- Every status is initialized to **NOT STARTED** because this phase analyzes the assignment only; it does not draft or satisfy the final report.

## A. Scenario eligibility and complexity constraints

| ID | Grading section | Weight | Exact requirement in concise form | Evidence/content the final report must contain | Expected destination section/file | Status |
|---|---|---:|---|---|---|---|
| SC-01 | Guidelines — Required Characteristics | Not specified | Use a problem that requires an ML task such as prediction, classification, recommendation, or anomaly detection. | An explicit ML task, prediction target, output type, and explanation of the decision supported by the prediction. | Final report — Problem Definition | NOT STARTED |
| SC-02 | Guidelines — Required Characteristics | Not specified | Include at least two distinct stakeholders with different goals and concerns. | At least two named stakeholder roles; their distinct goals, decisions, and concerns. | Final report — Problem Definition / Stakeholders | NOT STARTED |
| SC-03 | Guidelines — Required Characteristics | Not specified | Design for real-world deployment and consider scalability, reliability, and maintenance. | Production context plus explicit scalability, reliability, operations, monitoring, maintenance, and lifecycle considerations. | Final report — Requirements Analysis and Architecture | NOT STARTED |
| SC-04 | Guidelines — Required Characteristics | Not specified | Describe realistic or hypothetical data sources and consider collection, quality, and privacy. | Data origins and collection path; whether sources are real or assumed; quality risks/controls; privacy concerns. | Final report — Requirements Analysis / Data Requirements | NOT STARTED |
| SC-05 | Guidelines — Required Characteristics | Not specified | Provide measurable business value. | A business outcome, metric, measurement method, baseline/comparator, and target or acceptance criterion. | Final report — Goals and Metrics | NOT STARTED |
| SC-06 | Guidelines — Complexity Level | Not specified | Avoid an overly simple application with an obvious solution and no production considerations. | Enough system-level scope to demonstrate ingestion, operations, reliability, monitoring, retraining, stakeholder concerns, and non-trivial decisions. | Report-wide; especially Architecture and Trade-offs | NOT STARTED |
| SC-07 | Guidelines — Complexity Level | Not specified | Avoid multiple interacting ML models or extremely specialized domain knowledge. | A bounded design whose ML scope and required domain knowledge are understandable and defensible within one report. | Final report — Problem Definition / Scope | NOT STARTED |

## B. Weighted report rubric

### Problem definition

| ID | Grading section | Weight | Exact requirement in concise form | Evidence/content the final report must contain | Expected destination section/file | Status |
|---|---|---:|---|---|---|---|
| PD-01 | Problem definition — Context and Background | 20% section | Describe the domain, organization context, and why the problem matters. | Sourced domain background; explicit organizational/operating context; consequences of the problem; assumptions clearly labelled where the scenario is silent. | Final report — Problem Definition / Context and Background | NOT STARTED |
| PD-02 | Problem definition — Problem Statement | 20% section | State the problem clearly and in measurable terms. | Population/asset, available inputs, binary outcome, fixed 24-hour horizon, prediction timing, and measurable success framing. | Final report — Problem Definition / Problem Statement | NOT STARTED |
| PD-03 | Problem definition — Current Situation | 20% section | Explain how the problem is currently handled. | An as-is maintenance/monitoring workflow, its actors and decision points, and documented limitations; unsupported details must be labelled as design assumptions. | Final report — Problem Definition / Current Situation | NOT STARTED |
| PD-04 | Problem definition — Justification | 20% section | Explain why ML is appropriate. | Scenario-specific comparison with plausible non-ML approaches; why patterns in available data warrant ML; limits and conditions under which ML adds value. | Final report — Problem Definition / Justification for ML | NOT STARTED |
| PD-05 | Problem definition — Stakeholder Identification | 20% section | List all stakeholders and their concerns. | A stakeholder inventory covering direct users, business/operations owners, and technical operators; each role’s goals, decisions, risks, and concerns. | Final report — Problem Definition / Stakeholders | NOT STARTED |

### Requirements analysis

| ID | Grading section | Weight | Exact requirement in concise form | Evidence/content the final report must contain | Expected destination section/file | Status |
|---|---|---:|---|---|---|---|
| RA-F01 | Requirements analysis — Functional | 20% section | Define the core ML functionality: predictions or decisions made. | Binary high-risk/normal prediction for failure within 24 hours; risk probability; prediction cadence/trigger and the operational decision supported. | Final report — Requirements Analysis / Functional Requirements | NOT STARTED |
| RA-F02 | Requirements analysis — Functional | 20% section | Specify inputs and outputs. | Input schema/window and sensor fields; validation expectations; output schema including class, probability, timestamp/asset identity, and any user-facing context justified by the design. | Final report — Requirements Analysis / Functional Requirements | NOT STARTED |
| RA-F03 | Requirements analysis — Functional | 20% section | Define integration requirements with existing systems. | Interfaces and data/control flows among sensors/gateways, ingestion/storage, inference, maintenance applications, dashboards/alerts, and any assumed plant system. | Final report — Requirements Analysis / Functional Requirements | NOT STARTED |
| RA-F04 | Requirements analysis — Functional | 20% section | Define user interaction requirements. | Role-specific workflow for viewing and acting on predictions/alerts, including feedback or acknowledgement where justified. | Final report — Requirements Analysis / Functional Requirements | NOT STARTED |
| RA-N01 | Requirements analysis — Non-Functional / Performance | 20% section | Define latency, throughput, and response-time expectations. | Quantified service expectations with units, load conditions, measurement points, and justification; values must be sourced facts or labelled design assumptions. | Final report — Requirements Analysis / Non-Functional Requirements | NOT STARTED |
| RA-N02 | Requirements analysis — Non-Functional / Scalability | 20% section | Define expected load and scaling strategy. | Expected assets/events/predictions and growth assumptions; bottlenecks; horizontal/vertical or edge/cloud scaling approach and capacity rationale. | Final report — Requirements Analysis / Non-Functional Requirements | NOT STARTED |
| RA-N03 | Requirements analysis — Non-Functional / Reliability | 20% section | Define uptime, failure handling, and graceful degradation. | Availability objective; failure modes; retries/buffering/fallbacks; behavior during sensor, network, service, or model unavailability; recovery expectations. | Final report — Requirements Analysis / Non-Functional Requirements | NOT STARTED |
| RA-N04 | Requirements analysis — Non-Functional / Maintainability | 20% section | Define update frequency, retraining needs, and monitoring requirements. | Ownership and change process; software/model update expectations; retraining triggers/cadence; service, data, drift, and model monitoring. | Final report — Requirements Analysis / Non-Functional Requirements | NOT STARTED |
| RA-D01 | Requirements analysis — Data | 20% section | Define data sources. | Sensor and operational/maintenance-label sources, provenance, collection path, historical versus online use, and fact/assumption labels. | Final report — Requirements Analysis / Data Requirements | NOT STARTED |
| RA-D02 | Requirements analysis — Data | 20% section | Define data quality requirements. | Completeness, validity, timing/alignment, missing/outlier handling, label quality, validation checks, and ownership. | Final report — Requirements Analysis / Data Requirements | NOT STARTED |
| RA-D03 | Requirements analysis — Data | 20% section | Define privacy requirements. | Data classification and relevant privacy/confidentiality concerns; access, retention, minimization, and protection requirements appropriate to the scenario. | Final report — Requirements Analysis / Data Requirements | NOT STARTED |
| RA-D04 | Requirements analysis — Data | 20% section | Define data volume. | Estimated or sourced volume/rate/retention with calculation basis and units; implications for storage, processing, and scaling. | Final report — Requirements Analysis / Data Requirements | NOT STARTED |

### Goals and metrics

| ID | Grading section | Weight | Exact requirement in concise form | Evidence/content the final report must contain | Expected destination section/file | Status |
|---|---|---:|---|---|---|---|
| GM-01 | Goals and metrics — Hierarchy | 20% section | Define a clear hierarchy of goals and corresponding metrics. | Traceability from business goals to system goals to model goals; metric definition, direction, owner/consumer, measurement window, and dependency relationships. | Final report — Goals and Metrics / Goal Hierarchy | NOT STARTED |
| GM-02 | Goals and metrics — Business | 20% section | Define business goals. | Measurable downtime/maintenance/false-alarm outcomes, their business metric definitions, and how value will be evaluated without claiming achieved results. | Final report — Goals and Metrics / Business Goals | NOT STARTED |
| GM-03 | Goals and metrics — System | 20% section | Define system goals. | Operational goals and metrics such as end-to-end timeliness, availability, throughput, data freshness, or alert delivery, selected for this scenario. | Final report — Goals and Metrics / System Goals | NOT STARTED |
| GM-04 | Goals and metrics — Model | 20% section | Define model goals. | Evaluation goals and metrics suited to imbalanced failure risk and asymmetric error costs; evaluation unit/window and threshold-selection method. | Final report — Goals and Metrics / Model Goals | NOT STARTED |
| GM-05 | Goals and metrics — Acceptance | 20% section | Define acceptable thresholds and baselines. | Justified acceptance thresholds and comparator baselines for business, system, and model metrics; each number identified as sourced, assumed, or to be established during validation. | Final report — Goals and Metrics / Thresholds and Baselines | NOT STARTED |

### High-level architecture design

| ID | Grading section | Weight | Exact requirement in concise form | Evidence/content the final report must contain | Expected destination section/file | Status |
|---|---|---:|---|---|---|---|
| AR-01 | High-level architecture design — Diagram | 25% section | Present clear architecture diagrams. | At least one legible, correctly rendered high-level diagram with labelled components, boundaries, flows, training and inference paths, monitoring, and retraining feedback. | Final report — High-Level Architecture / Figure(s) | NOT STARTED |
| AR-02 | High-level architecture design — Data Flow | 25% section | Show data flow through the system. | Directional flow from sensor generation and ingestion through storage/processing to training and serving, prediction consumers, monitoring, and feedback. | Final report — High-Level Architecture / Diagram and narrative | NOT STARTED |
| AR-03 | High-level architecture design — ML Pipeline | 25% section | Include data ingestion, preprocessing, training, and inference stages. | Explicit training path and inference path with consistent preprocessing and model promotion/serving boundaries; model evaluation/registry and feedback loops required by the project instructions. | Final report — High-Level Architecture / ML Pipeline | NOT STARTED |
| AR-04 | High-level architecture design — Components | 25% section | Describe each component’s purpose and responsibility. | A component inventory mapping every major diagram element to its inputs, outputs, owner/function, and responsibility. | Final report — High-Level Architecture / Component Descriptions | NOT STARTED |
| AR-05 | High-level architecture design — Technology | 25% section | State technologies or tools considered. | Candidate technology/tool categories or products for major components, selection criteria, and scenario-specific rationale; avoid unexplained product lists. | Final report — High-Level Architecture / Technology Choices | NOT STARTED |

### Trade-offs analysis

| ID | Grading section | Weight | Exact requirement in concise form | Evidence/content the final report must contain | Expected destination section/file | Status |
|---|---|---:|---|---|---|---|
| TO-01 | Trade-offs analysis | 15% | Analyze at least 3–4 significant design trade-offs. | At least four meaningful trade-offs under the repository instructions. For each: alternatives, benefits, disadvantages/risks, scenario-specific reasoning, and final design decision. The PDF’s example topics are illustrative, not mandatory. | Final report — Trade-offs Analysis | NOT STARTED |

## C. Submission constraints

| ID | Grading section | Weight | Exact requirement in concise form | Evidence/content the final report must contain | Expected destination section/file | Status |
|---|---|---:|---|---|---|---|
| SUB-01 | Submission Guidelines | Not specified | Submit a written report as a PDF. | A final rendered PDF whose content and diagrams have been visually checked. | Final deliverable PDF | NOT STARTED |
| SUB-02 | Submission Guidelines — File Naming | Not specified | Name the file `DDM501_Assignment1_[StudentID]_[Name].pdf`. | Filename populated with the student’s actual ID and name, with the required prefix and PDF extension. | Final deliverable filename | NOT STARTED |

## Scenario-to-rubric comparison

### Requirements already supported by `scenario.md`

| Assignment area | Scenario coverage | Assessment |
|---|---|---|
| ML problem | Fixed binary classification of motor failure risk within the next 24 hours; outputs normal/high risk plus probability. | Satisfies SC-01 and gives a strong basis for PD-02 and RA-F01. |
| Stakeholder minimum | Names Maintenance Engineer, Plant Operations Manager, and ML / Platform Engineer. | Satisfies the minimum count in SC-02, but not yet the “all stakeholders and concerns” detail in PD-05. |
| Real-world deployment | Provides an end-to-end concept from motors/sensors and gateway through ingestion, storage, training, registry, inference, dashboard/alerts, monitoring, and retraining. | Supports SC-03 and the architecture rubric; detailed operating requirements remain open. |
| Data sources | Identifies vibration, temperature, electrical current, and RPM time series; restricts added variables to researched or explicit assumptions. | Supports SC-04 and RA-D01 at a high level. |
| Business value | States the objective of reducing unplanned downtime while avoiding unnecessary maintenance and excessive false alarms. | Supports SC-05 qualitatively; metrics, baselines, and targets are not yet defined. |
| Complexity | Restricts the design to one primary predictive model while requiring a production lifecycle. | No direct conflict. The report must make the system-level complexity visible so it does not appear to be a basic standalone classifier. |

### Scenario gaps or potential rubric risks

These are not changes to `scenario.md`; they identify content that later phases must research or label as design assumptions.

| Rubric requirement(s) | Scenario gap or risk | Required later treatment |
|---|---|---|
| PD-01 | No specific organization, plant context, motor fleet, operating regime, or business consequences are supplied. | Use credible domain evidence and explicitly labelled design assumptions; do not imply a real organization. |
| PD-03 | The current maintenance and monitoring process is not described. | Define a defensible as-is workflow as an explicit scenario/design assumption unless a factual generic practice is supported by research. |
| PD-05 / SC-02 | Only primary stakeholders are listed, and their distinct concerns are not stated. The meaning of “all stakeholders” is open-ended. | Add justified secondary roles if needed and map every included stakeholder to goals and concerns. |
| RA-F02 | The sensor lookback window, sampling/aggregation, validation rules, and complete output contract are unspecified. | Define them later as researched facts or design assumptions. |
| RA-F03 | The scenario names architectural stages but no actual existing plant systems, protocols, interfaces, or maintenance-management integration. | Specify interface-level requirements without claiming an existing deployment. |
| RA-F04 | Dashboard/alerts are named, but role-specific workflows, acknowledgement, escalation, and feedback are unspecified. | Design user interactions and human decision boundaries. |
| RA-N01 to RA-N04 | No latency, throughput, response time, load, scaling, uptime, failure-handling, update, retraining, or monitoring targets are supplied. | Establish justified design targets/assumptions and define how each will be measured. |
| RA-D01 to RA-D04 | Failure-label source, collection details, data-quality criteria, privacy/confidentiality controls, data rate, volume, and retention are unspecified. | Research feasible sources and calculations; label hypothetical values as assumptions. |
| GM-01 to GM-05 | No metric definitions, hierarchy, baselines, acceptance thresholds, or achieved results are supplied. | Define prospective goals and acceptance criteria only; do not invent achieved performance. |
| AR-01 to AR-05 | The scenario provides a linear concept, not a rendered diagram, detailed component boundaries, responsibilities, or candidate tools. | Produce and verify these in the architecture phase. |
| TO-01 | No alternatives or trade-off decisions are analyzed in the scenario. | Analyze at least four significant trade-offs later, using the repository’s required structure. |
| SUB-02 | Student ID and student name are not provided. | Human input is required before final filename validation. |

### Conflict assessment

- **No direct scenario/rubric conflict was found.**
- There is a potential presentation tension between the scenario’s “one primary predictive ML model” constraint and the PDF’s warning against an overly simple single-model application. The PDF specifically warns against a simple model **with no production considerations**; the scenario’s ingestion, deployment, monitoring, retraining, reliability, and stakeholder scope can satisfy the intended complexity without introducing multiple models.
- The project instructions require at least four trade-offs, which is compatible with the PDF’s ambiguous “at least 3–4” wording and adopts the safer interpretation.

## Assignment ambiguities and human-attention items

| ID | Ambiguity | What is known; do not invent | Human attention |
|---|---|---|---|
| AM-01 | “Analyze at least 3–4” is internally awkward: it may mean a minimum of three, a target range of three to four, or at least four. | The repository instruction resolves this conservatively to at least four. | No immediate decision required unless the instructor says otherwise. |
| AM-02 | “Clear architecture diagrams” is plural, but the required number and acceptable notation are not stated. | One integrated diagram may cover all flows, but the PDF does not confirm whether that is sufficient. | Optional instructor clarification; otherwise use the smallest number of legible diagrams that fully covers the rubric. |
| AM-03 | “All stakeholders” has no boundary for indirect stakeholders such as safety, cybersecurity, IT, vendors, or regulators. | The scenario lists three primary roles only. | A later design decision is needed; ask the instructor only if they expect a prescribed stakeholder set. |
| AM-04 | “Acceptable thresholds and baselines” does not say whether values must come from literature, organizational data, assumptions, or experiments. | The repository policy permits sourced facts or explicit design assumptions and prohibits invented achieved results. | No immediate decision required; keep provenance explicit. |
| AM-05 | No report length, page limit, required template, citation style, minimum source count, submission platform, or deadline appears in the supplied PDF. | None of these may be inferred from the provided files. | Human/instructor attention is required if such rules exist elsewhere. |
| AM-06 | The PDF does not define the expected depth of named technologies/tools or whether product names are required. | The rubric only says “Technologies/tools considered.” | No immediate clarification required; later justify candidates at a level consistent with a high-level design. |
| AM-07 | The final filename requires `[StudentID]` and `[Name]`, but neither value is supplied. | Placeholders must not be guessed. | Human input required before finalization. |
| AM-08 | `AGENTS.md` names `assignment/INDIVIDUAL_ASSIGNMENT_1.pdf`, while the actual repository file is `assignment/INDIVIDUAL ASSIGNMENT 1.pdf`. | The existing three-page PDF with spaces was inspected; it was not renamed or modified. | Repository-owner attention recommended to avoid automation/path failures. |

