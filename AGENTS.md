# DDM501 Individual Assignment 1

This repository contains an academic ML System Design assignment.

## Authoritative Inputs

1. `assignment/INDIVIDUAL_ASSIGNMENT_1.pdf` is the authoritative assignment specification.
2. `scenario.md` is the authoritative scenario definition.

Do not silently change either.

## Objective

Produce a technically defensible, production-oriented ML System Design Document that satisfies every explicit item in the assignment rubric.

The work must focus on the complete ML system lifecycle, not merely the ML algorithm.

## Required Workflow

Do not immediately write the final report.

Work in this order:

1. inspect the assignment
2. extract the rubric
3. create a requirements checklist
4. research external evidence
5. record assumptions
6. design requirements
7. define goals and metrics
8. design architecture
9. analyze trade-offs
10. draft the report
11. perform independent reviews
12. revise
13. produce the final document

## Evidence Rules

Never fabricate:

- citations
- authors
- papers
- URLs
- DOI identifiers
- datasets
- statistics
- experimental results
- model accuracy
- deployment results

Every externally verifiable factual claim in the final report must be traceable to a real source.

Prefer sources in this order:

1. peer-reviewed research
2. official datasets
3. standards or authoritative organizations
4. official technical documentation
5. reputable secondary sources only when necessary

## Facts, Assumptions, and Decisions

Clearly distinguish:

- FACT: externally supported by a source
- DESIGN ASSUMPTION: chosen for this hypothetical system
- DESIGN DECISION: an engineering choice made after considering alternatives

Never convert an assumption into a fact.

## ML Scope

The fixed ML task is binary classification:

Predict whether an industrial motor is at high risk of failure within the next 24 hours.

Do not silently change the prediction target or prediction horizon.

## Assignment Coverage

The final report must address:

### Problem Definition

- context and background
- measurable problem statement
- current situation
- justification for ML
- stakeholders and stakeholder concerns

### Requirements Analysis

Functional requirements:
- core ML functionality
- inputs and outputs
- integration requirements
- user interaction

Non-functional requirements:
- latency / response time
- throughput where relevant
- scalability
- reliability
- graceful degradation
- maintainability
- monitoring
- retraining

Data requirements:
- sources
- quality
- privacy
- volume

### Goals and Metrics

Separate:

- business goals
- system goals
- model goals

Define metrics and justified thresholds/baselines.

Do not invent achieved model results.

### Architecture

Architecture must show both:

TRAINING PATH:
data → preprocessing → training → evaluation → model registry

INFERENCE PATH:
new sensor data → preprocessing → deployed model → risk prediction → user/application

Also address monitoring and retraining feedback loops.

### Trade-offs

Analyze at least four meaningful engineering trade-offs.

Each trade-off must contain:

- alternatives
- benefits
- disadvantages/risks
- scenario-specific reasoning
- final design decision

## Writing

The final report should use clear academic English.

Prefer precise engineering reasoning over generic textbook explanations.

Avoid unnecessary introductory explanations of basic ML concepts.

## Quality Gate

The project is not complete until:

- every rubric item is covered
- no fabricated citations remain
- no unsupported factual numbers remain
- problem, data, metrics, architecture and trade-offs are internally consistent
- training and inference paths are present
- monitoring and retraining are addressed
- architecture diagram renders correctly
- final report filename follows the assignment requirement
