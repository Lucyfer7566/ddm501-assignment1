# Scenario

## Title

Design of an ML-Based Predictive Maintenance System for Industrial Motors Using IoT Sensor Data

## Problem

Design a production-oriented Machine Learning system that predicts whether an industrial motor is at high risk of failure within the next 24 hours based on recent IoT sensor measurements.

## ML Task

Binary classification.

The system predicts:

- Normal
- High Risk

and also returns a failure-risk probability.

## Input Data

Recent time-series sensor measurements, primarily:

- vibration
- temperature
- electrical current
- rotational speed (RPM)

Additional operational variables may be considered only when justified by research or explicitly identified as design assumptions.

## Primary Stakeholders

1. Maintenance Engineer
2. Plant Operations Manager
3. ML / Platform Engineer

## Business Objective

Reduce unplanned motor downtime while avoiding unnecessary preventive maintenance and excessive false alarms.

## High-Level Deployment Concept

Industrial motors and sensors
→ IoT / edge gateway
→ data ingestion
→ centralized data storage
→ preprocessing and feature engineering
→ ML training and evaluation
→ model registry
→ inference service
→ maintenance dashboard / alerts
→ monitoring and retraining

## Scope

Use one primary predictive ML model.

Do not change the task into:

- remaining useful life prediction
- general anomaly detection
- multi-class fault diagnosis
- a multi-model system

unless explicitly approved by the user.

## Important Assumption Policy

This is a system design assignment, not evidence of an actually deployed system.

Values such as:

- number of motors
- sensor sampling frequency
- latency target
- availability target
- prediction frequency
- retraining interval
- model performance threshold

must be classified as either:

1. externally supported facts with citations, or
2. explicit engineering/design assumptions with justification.

Never present hypothetical design assumptions as measured real-world facts.
