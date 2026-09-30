# Verified Source Ledger

## Scope

- Research date: 2026-09-30.
- Selected evidence set: 10 strong sources.
- Selection rule: retain a source only when its identity can be verified and it materially supports the fixed industrial-motor, binary, 24-hour failure-risk system design.
- Stable DOI links are preferred for publications. Official organization or dataset pages are used where no DOI exists.
- Dataset suitability is assessed separately from source quality. A high-quality dataset can still be unsuitable as the production training set for this target.

## Accepted sources

### S01 — Condition-based maintenance foundation

- **Title:** A review on machinery diagnostics and prognostics implementing condition-based maintenance
- **Authors:** Andrew K. S. Jardine; Daming Lin; Dragan Banjevic
- **Year:** 2006
- **Type:** Peer-reviewed review article, *Mechanical Systems and Signal Processing*, 20(7), 1483–1510
- **DOI:** `10.1016/j.ymssp.2005.09.012`
- **Stable URL:** https://doi.org/10.1016/j.ymssp.2005.09.012
- **BibTeX key:** `jardine2006cbm`
- **Verified against:** Elsevier article record and DOI metadata.
- **Can support:**
  - condition-based maintenance as a flow from data acquisition to data processing to maintenance decision-making;
  - the distinction between diagnostics and prognostics;
  - the relevance of multiple-sensor data and sensor fusion;
  - the need to connect predictions to maintenance decisions rather than treating the model as a standalone artifact.
- **Cannot support:** A 24-hour threshold, this project’s fleet size, sensor cadence, business savings, or achieved model performance.

### S02 — Electrical-motor condition monitoring modalities

- **Title:** Condition Monitoring and Fault Diagnosis of Electrical Motors—A Review
- **Authors:** Subhasis Nandi; Hamid A. Toliyat; Xiaodong Li
- **Year:** 2005
- **Type:** Peer-reviewed review article, *IEEE Transactions on Energy Conversion*, 20(4), 719–729
- **DOI:** `10.1109/TEC.2005.847955`
- **Stable URL:** https://doi.org/10.1109/TEC.2005.847955
- **BibTeX key:** `nandi2005motors`
- **Verified against:** IEEE Xplore/DOI record.
- **Can support:**
  - motor-current signature analysis as a recognized motor condition-monitoring approach;
  - the use of vibration, speed, torque, noise, and thermal measurements as fault-related signals;
  - the existence of different motor fault types and corresponding signal signatures;
  - the rationale for a multimodal sensor design rather than reliance on one unqualified signal.
- **Cannot support:** That all listed signals are equally informative for every motor, or that this hypothetical system has validated any sensor combination.

### S03 — Vibration analysis for rotating bearings

- **Title:** Rolling element bearing diagnostics—A tutorial
- **Authors:** Robert B. Randall; Jérôme Antoni
- **Year:** 2011
- **Type:** Peer-reviewed tutorial/review, *Mechanical Systems and Signal Processing*, 25(2), 485–520
- **DOI:** `10.1016/j.ymssp.2010.07.017`
- **Stable URL:** https://doi.org/10.1016/j.ymssp.2010.07.017
- **BibTeX key:** `randall2011bearing`
- **Verified against:** Elsevier article record and DOI metadata.
- **Can support:**
  - vibration acceleration as an informative signal for rolling-element bearing faults;
  - envelope analysis and spectral-kurtosis-based band selection as established diagnostic techniques;
  - the effect of shaft speed and speed variation on characteristic frequencies;
  - use of order tracking or speed-aware preprocessing when operating speed varies.
- **Cannot support:** A claim that bearing faults are the only motor failure mode, or that a laboratory vibration classifier directly predicts failure within 24 hours.

### S04 — ML methods in predictive maintenance

- **Title:** A systematic literature review of machine learning methods applied to predictive maintenance
- **Authors:** Thyago Peres Carvalho; Fabrízzio Alphonsus A. M. N. Soares; Roberto Vita; Roberto da Piedade Francisco; João Pedro Tavares Vieira Basto; Symone G. S. Alcalá
- **Year:** 2019
- **Type:** Peer-reviewed systematic review, *Computers & Industrial Engineering*, 137, 106024
- **DOI:** `10.1016/j.cie.2019.106024`
- **Stable URL:** https://doi.org/10.1016/j.cie.2019.106024
- **BibTeX key:** `carvalho2019pdm`
- **Verified against:** Elsevier DOI record and DBLP metadata.
- **Can support:**
  - use of ML for predictive-maintenance applications with multivariate industrial data;
  - the need to select methods according to the data and operational problem rather than assuming one universally best algorithm;
  - a lifecycle that includes historical data selection, preprocessing, model selection, training, validation, and maintenance;
  - labelled-data availability and model selection as practical concerns.
- **Cannot support:** Choosing a final algorithm before the project’s label quality, sample size, class balance, and deployment constraints are known.

### S05 — Paderborn motor-bearing benchmark and data portal

- **Title:** Condition Monitoring of Bearing Damage in Electromechanical Drive Systems by Using Motor Current Signals of Electric Motors: A Benchmark Data Set for Data-Driven Classification
- **Authors:** Christian Lessmeier; James Kuria Kimotho; Detmar Zimmer; Walter Sextro
- **Year:** 2016
- **Type:** Peer-reviewed conference paper with companion official university dataset
- **DOI:** `10.36001/phme.2016.v3i1.1577`
- **Stable URLs:**
  - Publication: https://doi.org/10.36001/phme.2016.v3i1.1577
  - Official dataset: https://mb.uni-paderborn.de/en/kat/research/bearing-datacenter/data-sets-and-download
- **BibTeX key:** `lessmeier2016paderborn`
- **Verified against:** PHM Society publication, full paper, and Paderborn University Bearing DataCenter.
- **Can support:**
  - existence of a public electromechanical-drive dataset with synchronous vibration and motor-current signals;
  - supportive speed, torque, radial-load, and temperature measurements;
  - multiple operating conditions and both artificial and accelerated-life bearing damage;
  - the caution that results depend on damage type and training-data representativeness;
  - using the dataset for signal-processing and classifier prototyping.
- **Important limitation:** The records are short labelled condition measurements, not continuous production histories with native “failure within the next 24 hours” labels. They cannot demonstrate target validity or production performance for this assignment.

### S06 — Case Western Reserve University bearing dataset

- **Title:** Welcome to the Case Western Reserve University Bearing Data Center Website / Download a Data File
- **Author/organization:** Case Western Reserve University, Case School of Engineering
- **Year:** Not stated on the official pages
- **Type:** Official university dataset repository
- **DOI:** None identified
- **Stable URLs:**
  - Overview: https://engineering.case.edu/bearingdatacenter/welcome
  - Data files: https://engineering.case.edu/bearingdatacenter/download-data-file
- **BibTeX key:** `cwruBearingData`
- **Verified against:** Official university pages.
- **Can support:**
  - availability of public normal and faulty motor-bearing vibration data;
  - documented motor loads, rotational speeds, accelerometer locations, sampling rates, and seeded bearing-fault status;
  - use as an auxiliary vibration-processing and cross-operating-condition benchmark.
- **Important limitation:** Faults were seeded and the dataset is fault-diagnosis data, not a natural run-to-failure or 24-hour risk dataset. It does not include the scenario’s full sensor set.

### S07 — IoT messaging standard

- **Title:** MQTT Version 5.0
- **Editors:** Andrew Banks; Ed Briggs; Ken Borgendale; Rahul Gupta
- **Organization:** OASIS Open
- **Year:** 2019
- **Type:** OASIS Standard
- **DOI:** None
- **Stable URL:** https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html
- **BibTeX key:** `oasis2019mqtt`
- **Verified against:** OASIS standard and citation block.
- **Can support:**
  - MQTT as a lightweight client-server publish/subscribe transport for M2M and IoT contexts;
  - application decoupling through publish/subscribe messaging;
  - explicit delivery semantics: at most once, at least once, and exactly once;
  - abnormal-disconnection notification and the availability of authentication, authorization, and secure-communication mechanisms.
- **Cannot support:** Selecting a QoS level without considering loss, duplication, bandwidth, latency, and idempotency requirements for this system.

### S08 — Operational-technology security and reliability

- **Title:** Guide to Operational Technology (OT) Security
- **Authors:** Keith Stouffer; Michael Pease; CheeYee Tang; Timothy Zimmerman; Victoria Pillitteri; Suzanne Lightman; Adam Hahn; Stephanie Saravia; Aslam Sherule; Michael Thompson
- **Organization:** National Institute of Standards and Technology
- **Year:** 2023
- **Type:** NIST Special Publication 800-82 Revision 3
- **DOI:** `10.6028/NIST.SP.800-82r3`
- **Stable URL:** https://doi.org/10.6028/NIST.SP.800-82r3
- **BibTeX key:** `nist2023otsecurity`
- **Verified against:** NIST CSRC final publication record.
- **Can support:**
  - treating plant telemetry and gateways as operational technology with distinct performance, reliability, and safety requirements;
  - considering OT topology, threats, vulnerabilities, and security countermeasures;
  - avoiding an ingestion design that ignores availability or physical-process consequences.
- **Cannot support:** Any plant-specific threat model, availability target, or regulatory obligation without additional context.

### S09 — Production ML readiness and monitoring

- **Title:** The ML Test Score: A Rubric for ML Production Readiness and Technical Debt Reduction
- **Authors:** Eric Breck; Shanqing Cai; Eric Nielsen; Michael Salib; D. Sculley
- **Year:** 2017
- **Type:** Peer-reviewed IEEE Big Data conference paper
- **DOI:** `10.1109/BigData.2017.8258038`
- **Stable URLs:**
  - DOI: https://doi.org/10.1109/BigData.2017.8258038
  - Authoritative author-organization page: https://research.google/pubs/the-ml-test-score-a-rubric-for-ml-production-readiness-and-technical-debt-reduction/
- **BibTeX key:** `breck2017mltestscore`
- **Verified against:** Google Research publication record and IEEE DOI.
- **Can support:**
  - production ML requiring tests and monitoring beyond offline model evaluation;
  - checks spanning data, features, model development, infrastructure, and live monitoring;
  - monitoring training-serving skew, data invariants, model quality, and serving behavior as production-readiness concerns.
- **Cannot support:** A claim that completing a generic checklist guarantees safety, reliability, or model quality for this motor system.

### S10 — Concept drift

- **Title:** A Survey on Concept Drift Adaptation
- **Authors:** João Gama; Indrė Žliobaitė; Albert Bifet; Mykola Pechenizkiy; Abdelhamid Bouchachia
- **Year:** 2014
- **Type:** Peer-reviewed survey, *ACM Computing Surveys*, 46(4), Article 44, 1–37
- **DOI:** `10.1145/2523813`
- **Stable URL:** https://doi.org/10.1145/2523813
- **BibTeX key:** `gama2014drift`
- **Verified against:** ACM Digital Library DOI record.
- **Can support:**
  - concept drift as change over time in the relationship between inputs and target in an online supervised-learning setting;
  - distinguishing drift detection/adaptation from ordinary service monitoring;
  - evaluating adaptation strategies rather than assuming that every distribution alert requires automatic retraining.
- **Cannot support:** A universal drift threshold, detector, or retraining cadence for this scenario.

## Rejected candidate sources and datasets

Rejection means “not selected as one of the ten core evidence sources.” It does not mean that the source is nonexistent or poor quality.

| Candidate | Verified metadata and URL | Reason rejected from the core set |
|---|---|---|
| **AI4I 2020 Predictive Maintenance Dataset** | UCI Machine Learning Repository, 2020. DOI `10.24432/C5HS5C`. https://doi.org/10.24432/C5HS5C | Official and convenient, but explicitly synthetic. It uses air/process temperature, rotational speed, torque, and tool wear; it lacks vibration and motor current and does not provide a natural 24-hour pre-failure label. It could demonstrate tabular code only, not evidence production validity for this motor scenario. |
| **C-MAPSS / Turbofan Engine Degradation Simulation Data Set** | Abhinav Saxena and Kai Goebel, NASA Prognostics Data Repository, 2008. https://www.nasa.gov/intelligent-systems-division/discovery-and-systems-health/pcoe/pcoe-data-set-repository/ | Authoritative and useful for run-to-failure/RUL research, but it concerns simulated turbofan engines and an RUL target. Using it as primary evidence would change the asset domain and prediction task. |
| **IMS Bearing Data Set** | J. Lee, H. Qiu, G. Yu, J. Lin, and Rexnord Technical Services, 2007; hosted by the NASA Prognostics Data Repository. https://www.nasa.gov/intelligent-systems-division/discovery-and-systems-health/pcoe/pcoe-data-set-repository/ | Relevant run-to-failure bearing vibration data, but it lacks the scenario’s multimodal motor signals and does not by itself establish an industrial-motor 24-hour label. Paderborn and CWRU provide a clearer motor/drive connection for the limited source budget. |
| **Unverified mirrors, Kaggle copies, GitHub repackagings, and tertiary blog summaries of bearing datasets** | Various | Rejected because provenance, transformations, licensing, or metadata may differ from the official dataset. Official university/NASA pages were used instead. |

## Coverage map

| Requested research area | Primary source IDs |
|---|---|
| A. Predictive maintenance for rotating machinery/motors | S01, S02, S03 |
| B. Vibration, temperature, current, and rotational speed | S02, S03, S05, S06 |
| C. Relevant ML approaches | S04, S05 |
| D. Realistic/public datasets | S05, S06; rejected-candidate analysis above |
| E. IoT/industrial ingestion | S07, S08 |
| F. Monitoring, drift, retraining, and reliability | S08, S09, S10 |

