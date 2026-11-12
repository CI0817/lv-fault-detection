# Low-Voltage Fault Detection & Localisation

**A Python-based fault detection and localisation research prototype for low-voltage distribution networks using smart-meter measurements and network topology.**

**Status:** Planning and initial research  
**Project duration:** 12 weeks

## Overview

Low-voltage distribution networks are becoming increasingly complex due to distributed energy resources, electrification, and changing customer demand.

Improving network visibility is important for identifying abnormal operating conditions and supporting distribution network reliability.

This project investigates how smart-meter measurements and network topology information can be used to detect and estimate the locations of faults in LV distribution networks.

The goal is to develop a literature-informed fault detection algorithm, validate it through synthetic network simulations, and integrate it into an interactive web platform.

The project builds on previous work involving LV network topology estimation, phase identification, and synthetic smart-meter data generation.

## Project Objectives

- Investigate fault detection and localisation methods from academic literature.
- Simulate normal and faulted LV network conditions using OpenDSS.
- Implement and validate a literature-based fault detection and localisation algorithm.
- Evaluate detection and localisation performance under different operating conditions.
- Develop an interactive platform for network visualisation and fault analysis.
- Document mathematical foundations, engineering decisions, experimental results, and limitations.

## Proposed System

The planned processing workflow is:

**LV Network Model → Fault Simulation → Smart-Meter Measurements → Fault Detection → Fault Localisation → Interactive Visualisation**

### 1. LV Network Simulation

Generate a synthetic radial LV network with known topology, customer connections, and electrical parameters.

### 2. Fault Scenario Generation

Simulate faulted network conditions using OpenDSS, varying selected fault and operating parameters.

### 3. Fault Detection and Localisation

Use a literature-based algorithm to identify abnormal measurements and estimate possible fault locations.

The final method will be selected after the initial literature review.

### 4. Interactive Platform

Develop a Streamlit application to:

- Visualise network topology and customer voltages.
- Select or simulate fault conditions.
- Run the detection and localisation pipeline.
- Display predicted fault locations.
- Compare predictions against simulated ground truth.
- Explore measurement patterns and performance results.

## Technology Stack

| Technology | Purpose |
|---|---|
| Python | Core implementation |
| OpenDSS | LV network and fault simulations |
| NumPy / Pandas | Numerical and data processing |
| NetworkX | Network topology representation |
| Plotly | Interactive visualisation |
| Streamlit | Web application |
| Git / GitHub | Version control and documentation |

## Project Roadmap

| Phase | Weeks | Focus |
|---|---|---|
| 1 | 1–2 | Fundamentals and algorithm selection |
| 2 | 3–5 | LV network and fault simulation |
| 3 | 6–8 | Detection, localisation, and validation |
| 4 | 9–10 | Interactive web platform |
| 5 | 11–12 | Robustness analysis and documentation |

See [ROADMAP.md](ROADMAP.md) for the detailed weekly objectives, milestones, and deliverables.

Project progress and learning outcomes are documented in [PROGRESS.md](PROGRESS.md).

## Installation and Usage

*To be added once the simulation and application environments are established.*

This section will eventually provide:

- Prerequisites and dependencies.
- Environment setup.
- Synthetic data generation instructions.
- Algorithm evaluation commands.
- Instructions for running the Streamlit application.

## Evaluation

The implementation will be assessed using quantitative metrics.

**Fault detection**
- Precision.
- Recall.
- False-positive rate.

**Fault localisation**
- Correct-location accuracy.
- Top-three candidate accuracy.
- Analysis of localisation errors.

**Robustness**
- Performance under selected measurement and operating variations.

Results will be reported using held-out simulation scenarios.

## Research and Technical Documentation

The final repository will include:

- Selected literature and references.
- Mathematical formulation of the chosen method.
- Algorithm implementation details.
- Experimental methodology.
- Performance analysis.
- Assumptions and limitations.
- Engineering decisions and lessons learned.

## Scope and Limitations

This is an educational research prototype, not a utility-grade fault protection system.

The initial implementation relies on synthetic data, known network topology, and selected simulated fault scenarios.

Conventional smart-meter data may not capture short-duration electrical faults. The project will explicitly define the fault conditions and measurement assumptions under which detection is feasible.

Results obtained from synthetic simulations will not be presented as validated performance on operational distribution networks.

## Future Work

Potential future extensions include:

- Validation using real-world smart-meter measurements.
- Incorporating uncertainty in network topology and phase identification.
- Investigating alternative detection and localisation algorithms.
- Improving robustness to sparse, missing, or noisy measurements.
- Scaling to larger LV distribution networks.

## Project Status

Development will proceed incrementally over 12 weeks.

The initial focus is understanding the mathematical foundations of fault detection and selecting an appropriate literature-based algorithm before implementing the simulation and analysis pipeline.

---

**Project focus:** Power systems · Low-voltage distribution networks · Smart-meter analytics · Fault detection · Fault localisation · Python · OpenDSS