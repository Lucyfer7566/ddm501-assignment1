# Independent rubric audit of `drafts/report-v1.md`

Reviewed against the three-page `assignment/INDIVIDUAL ASSIGNMENT 1.pdf`, `scenario.md`, `AGENTS.md`, and `research/rubric-checklist.md`. The PDF supplies 28 weighted report criteria plus seven scenario criteria and two submission constraints, as enumerated in the checklist. Locations below refer to line numbers in the reviewed draft. `Full` means the required content is present in the draft, not that proposed targets have been achieved or that a final PDF has been inspected. `Partial` identifies a substantive ambiguity or incomplete specification. `Pending` applies only to final-deliverable checks.

## CRITICAL Findings

- **C1 — Submission artifact pending (SUB-01, SUB-02; submission only).** No report PDF was found outside `assignment/`, so final rendering, page-scale diagram legibility, and the required `DDM501_Assignment1_[StudentID]_[Name].pdf` filename cannot yet be verified. The draft itself is a Markdown file, and the actual student ID and name are not supplied in the reviewed inputs. This blocks *final submission*, not the draft's report-content coverage. Create and inspect the PDF after revision and populate the filename from the student's real details; do not guess them.
- **No critical report-content omission found.** The fixed binary motor-failure target and next-24-hour horizon are explicit at `drafts/report-v1.md:17-19`; the training, inference, monitoring, and retraining paths are all described at lines 178-194.

## MAJOR Findings

- **M1 — Business false-alarm measure has no usable comparator (SC-05, GM-02, GM-05).** Section 3.1, `drafts/report-v1.md:133`, sets BM-04 to “ML-initiated completed interventions with no actionable condition” with a ≤20% target and a “historical comparable or shadow-mode baseline.” The text does not explicitly define the percentage denominator. Historical periods have no ML-initiated interventions, and shadow mode cannot by itself observe interventions caused by an ML alert. This weakens the measurable-business-impact and acceptable-baseline requirements. Define the numerator, denominator, observation/attribution rules, and a comparator that can actually be observed during the proposed pilot; retain the target as a proposed design target until validated.
- **M2 — False-positive units are not reproducible (GM-04, GM-05, TO-01).** Section 3.3 defines prediction windows, alert episodes, failure events, and negative motor-days at `drafts/report-v1.md:149`, then gives `FP_day/(FP_day+TN_day)` at line 156 without saying when an episode makes a negative motor-day positive, how a multi-day episode is counted, or how overlapping 24-hour horizons are assigned. The “Always High Risk = 1” FPR baseline and the ≤1% promotion gate therefore depend on an unstated conversion rule. Section 5.1 repeats this gate at line 240. Specify a deterministic day-level alert indicator and matching/counting policy, then recompute both baseline definitions and the selection gate on that common unit.
- **M3 — Threshold selection can choose a candidate that the promotion rule rejects (GM-04, GM-05, TO-01).** Section 3.3, `drafts/report-v1.md:155-160`, requires F1 ≥0.63 but the stated validation search filters only recall, precision, and FPR before selecting the highest-recall survivor. Section 3.4 requires *all* MM-01–MM-06 gates (`drafts/report-v1.md:164`), while TO-01 again includes F1 in the threshold decision (`drafts/report-v1.md:240`). Include every mandatory numerical gate in the search, or designate F1 as diagnostic consistently. Also replace MM-06's “no material unexplained slice miscalibration” (`drafts/report-v1.md:158`) with a testable acceptance rule or an explicit, documented human-review decision procedure before calling it a promotion gate.
- **M4 — Several numerical acceptance levels are labelled but only weakly justified (GM-05).** The draft correctly labels targets as proposed, yet the business targets (15% downtime, 10% labor, 5% cost, 20% unnecessary interventions at `drafts/report-v1.md:131-133`) and model gates (85% recall, 50% precision, 1% FPR, 0.63 F1, 2× prevalence at lines 153-158) lack a stated plant-cost, capacity, or event-rate rationale for why those values are *acceptable* for this scenario. The PDF requires acceptable thresholds and baselines; `AGENTS.md` also requires justified thresholds. Give each threshold a decision rationale tied to the defined pilot, workload, or operational tolerance, and state how the provisional values will be reviewed before an untouched test set is used. No external performance number or achieved result is needed.

## MINOR Findings

- **m1 — On-demand service load is outside the quantified performance envelope (RA-N01, RA-N02).** FR-04 permits authenticated on-demand scoring (`drafts/report-v1.md:68`), but NFR-01/02 and SM-02 size and test only the 400/2,000 scheduled predictions per hour (`drafts/report-v1.md:78-80,142`). Define a bounded on-demand burst and response-time criterion, or state how it shares the existing capacity budget. The scheduled cadence and fleet growth are otherwise well specified.
- **m2 — Availability measurement needs a precise sampling rule (RA-N03, GM-03).** SM-03 says “eligible minutes in which scheduled scoring and dashboard retrieval both succeed” (`drafts/report-v1.md:143`), although scoring is scheduled every 15 minutes (`drafts/report-v1.md:56,68`). State whether each minute tests access to the latest due score, how the grace interval works, and how approved maintenance is removed from the denominator. This makes the ≥99.5% monthly target auditable.
- **m3 — Supporting stakeholder concerns are implicit (PD-05).** Section 1.5 names security/identity administrators and label stewards at `drafts/report-v1.md:45` but does not give those included roles their own goals, decisions, or concerns as it does the three primary roles at lines 41-43. A short responsibility/concern sentence for each would satisfy the PDF's “all stakeholders and their concerns” language without expanding the ML scope.
- **m4 — Inference preprocessing is compressed in the figure (AR-01, AR-03).** Figure 1 at `drafts/report-v1.md:172-174` is a real image and was inspected at native size. Its inference lane combines “New validated sensor data / six-hour window and shared features” in one box before the deployed model. Section 4.2.2 explicitly names online validation and shared preprocessing (`drafts/report-v1.md:184-188`), so the content is present; a distinct preprocessing/feature step in the diagram would make the required inference path easier to trace. Verify native-size labels remain readable after PDF layout.
- **m5 — Source-path inconsistency in project instructions, not the report.** `AGENTS.md` names `assignment/INDIVIDUAL_ASSIGNMENT_1.pdf`; the inspected repository file is `assignment/INDIVIDUAL ASSIGNMENT 1.pdf`. The checklist records this at AM-08. Align the instruction path when convenient so final validation scripts inspect the authoritative file.

## PASS Items

- The scenario remains one primary binary classifier for `Normal`/`High Risk` plus probability of motor failure within the next 24 hours (`drafts/report-v1.md:17-21,65-68,170`). It does not drift into remaining-useful-life, generic anomaly detection, or multiple deployed models.
- The report clearly distinguishes sourced claims, design assumptions, decisions, and proposed targets; it does not claim achieved model or deployment results (`drafts/report-v1.md:3,21,51,125-127,164`).
- The assumed current workflow, its limitations, ML justification, and the three primary stakeholders' distinct concerns are explicit (`drafts/report-v1.md:23-45`).
- Functional, non-functional, and data requirements cover interfaces, user workflow, degradation, security, quality, privacy, and calculated logical volume (`drafts/report-v1.md:61-119`). The stated volume arithmetic is internally consistent to the shown rounding.
- The architecture narrative and image include sensor acquisition, ingestion, storage, preprocessing, training, evaluation, model registry, approved deployment, inference, dashboard/alert, monitoring, and controlled retraining (`drafts/report-v1.md:168-229`). The image exists at `diagrams/architecture.png` and opens at native resolution; final-PDF legibility remains pending.
- Four trade-offs each state alternatives, benefits, risks, scenario-specific reasoning, and a design decision (`drafts/report-v1.md:233-267`). Their topics are legitimate choices under the PDF's illustrative list.

### Item-by-item rubric coverage

| ID | Criterion from PDF/checklist | Draft evidence | Coverage |
|---|---|---|---|
| SC-01 | Appropriate ML problem | §1.2, lines 17-19 | Full |
| SC-02 | At least two distinct stakeholders | §1.5, lines 39-45 | Full |
| SC-03 | Real-world deployment, scale, reliability, maintenance | §§2.3, 4.2-4.4, lines 74-87, 176-229 | Full |
| SC-04 | Realistic/hypothetical sources, collection, quality, privacy | §2.4, lines 91-109 | Full |
| SC-05 | Measurable business value | §3.1, lines 125-135; M1 | Partial |
| SC-06 | Nontrivial production scope | §§2.2-2.4, 4-5 | Full |
| SC-07 | Bounded complexity | §§1.2, 4.1, lines 17-19, 170 | Full |
| PD-01 | Context, organization, importance | §1.1, lines 9-13 | Full |
| PD-02 | Clear measurable problem | §1.2, lines 17-21 | Full |
| PD-03 | Current handling | §1.3, lines 25-27 | Full |
| PD-04 | Justification for ML | §1.4, lines 31-35 | Full |
| PD-05 | All stakeholders and concerns | §1.5, lines 39-45; m3 | Partial |
| RA-F01 | Core predictions/decisions | FR-01/04, lines 65, 68 | Full |
| RA-F02 | Inputs and outputs | FR-02/03, lines 66-67 | Full |
| RA-F03 | Integration with existing systems | FR-05/08, lines 69, 72; §4.3 | Full |
| RA-F04 | User interaction | FR-06/07/08, lines 70-72 | Full |
| RA-N01 | Latency, throughput, response time | NFR-01/02, lines 78-79; m1 | Partial |
| RA-N02 | Expected load and scaling strategy | NFR-02/03, lines 79-80; m1 | Partial |
| RA-N03 | Uptime, failure handling, degradation | NFR-04/05/06, lines 81-83; §4.4 | Full |
| RA-N04 | Updates, retraining, monitoring | NFR-07/08/09, lines 84-86; §4.2.3 | Full |
| RA-D01 | Data sources | §2.4.1, lines 93-97 | Full |
| RA-D02 | Data quality | §2.4.2, lines 101-105 | Full |
| RA-D03 | Privacy | §2.4.3, line 109 | Full |
| RA-D04 | Data volume | §2.4.4, lines 113-119 | Full |
| GM-01 | Goal/metric hierarchy | §3.1, lines 123-135 | Full |
| GM-02 | Business goals | §3.1, lines 129-135; M1 | Partial |
| GM-03 | System goals | §3.2, lines 139-145; m2 | Partial |
| GM-04 | Model goals | §3.3, lines 149-160; M2-M3 | Partial |
| GM-05 | Acceptable thresholds and baselines | §§3.1-3.4, lines 125-164; M1-M4 | Partial |
| AR-01 | Clear architecture diagram(s) | Figure 1, lines 168-174; m4 | Partial |
| AR-02 | Data flow | Figure 1 and §4.2, lines 172-194 | Full |
| AR-03 | Ingestion, preprocessing, training, inference | §§4.2.1-4.2.3, lines 178-194 | Full |
| AR-04 | Component purpose and responsibility | §4.3, lines 198-221 | Full |
| AR-05 | Technologies/tools considered | §4.3, lines 198-221 | Full |
| TO-01 | At least four significant trade-offs | §§5.1-5.4, lines 233-267 | Full |
| SUB-01 | Final written PDF | No report PDF to inspect; C1 | Pending (submission) |
| SUB-02 | Required PDF filename | Student ID/name absent; C1 | Pending (submission) |

The PDF's example trade-off topics are illustrative; the four selected topics satisfy the required count and structure. The PDF says “clear architecture diagrams” without a fixed number. One integrated figure covers the paths, subject to its final page-scale check.
