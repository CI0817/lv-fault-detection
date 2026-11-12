# LV Fault Detection & Localisation — Project Roadmap

**Duration:** 12 weeks  
**Commitment:** 4 hours/week (48 hours total)  
**Developer:** Individual project  
**Status:** Planning

## 1. Project Overview

### Objective

Develop an interactive, literature-informed fault detection and localisation platform for low-voltage (LV) distribution networks using synthetic smart-meter measurements and known network topology.

The project builds upon prior experience in LV network topology estimation, phase identification, and synthetic smart-meter data generation.

### Primary Goals

1. **Technical understanding:** Develop a strong understanding of fault detection and localisation techniques, including their mathematical foundations, assumptions, and limitations.
2. **Algorithm implementation:** Study existing literature, select a suitable method, and implement and validate it using synthetic data.
3. **Interactive platform:** Build a web-based platform to visualise network conditions, simulate faults, and display detection and localisation results.
4. **Documentation:** Maintain records of research, engineering decisions, experiments, and findings.

### Project Priorities

1. Technical depth and understanding.
2. Engineering implementation.
3. Portfolio presentation.

Implementation and portfolio presentation are equally important, but neither should compromise technical understanding.

## 2. Scope

### In Scope

- A small, radial LV distribution network.
- Synthetic network and smart-meter data using Python and OpenDSS.
- Known network topology and phase connections.
- Simulated fault scenarios with known ground truth.
- Fault detection and localisation using a literature-based method.
- Quantitative validation and performance analysis.
- An interactive Streamlit dashboard.
- Technical documentation and reproducible experiments.

### Out of Scope (Initial Version)

- Real-time deployment on utility networks.
- Integration with operational smart meters.
- High-speed protection and fault clearing.
- Large-scale machine learning models.
- Multiple simultaneous faults.
- Production-grade infrastructure.

**Measurement limitation:** Conventional smart-meter readings sampled every five minutes cannot reliably capture short-duration faults that clear between samples. The prototype will focus on simulated disturbances observable under its measurement assumptions. Faster sampling assumptions, if introduced, must be explicitly identified.

## 3. Proposed Technical Architecture

1. **Network modelling:** Reuse and simplify the honours thesis synthetic LV network generation pipeline.
2. **Fault simulation:** Generate fault scenarios using OpenDSS under varying network conditions.
3. **Measurement generation:** Export synthetic customer voltage measurements with corresponding fault labels.
4. **Fault detection:** Identify anomalous network conditions from meter measurements.
5. **Fault localisation:** Estimate the most likely faulted location using a literature-informed algorithm.
6. **Visualisation:** Display network conditions, measurements, fault predictions, and evaluation metrics using Streamlit.

### Preliminary Technology Stack

| Component | Technology |
|---|---|
| Programming | Python |
| Network modelling and power flow | OpenDSS |
| Numerical computation | NumPy, Pandas |
| Network representation | NetworkX |
| Visualisation | Plotly |
| Web platform | Streamlit |
| Version control | Git and GitHub |
| Documentation | Markdown |

Technology choices may change as implementation requirements become clearer.

## 4. Learning Methodology

The project follows a **research-first, implementation-driven learning approach**.

The learning process combines four activities:

### 1. Study

Establish the theoretical foundation before tackling substantial implementation challenges.

For foundational topics, use coherent, well-referenced research chapters that explain:
- Engineering motivation and context.
- Fundamental physical principles.
- Mathematical formulations.
- Worked examples and practical applications.
- Assumptions and limitations.
- Relevant academic references and further resources.

The core reading should generally fit within 30–45 minutes, excluding optional supplementary material.

### 2. Experiment

Apply concepts through small practical experiments.

Experiments should:
- Investigate a clearly defined engineering question.
- Connect theoretical predictions with observed behaviour.
- Be reproducible.
- Produce interpretable results.

### 3. Reflect

Record what was learned, what the experiments demonstrated, and what remains uncertain.

### 4. Discuss

Use the study material and experimental findings to guide deeper technical conversations.

Discussions should emerge naturally from questions, unexpected observations, or engineering decisions.

### Learning Emphasis by Phase

| Phase | Primary learning approach |
|---|---|
| Phase 1 | Structured research chapters, independent study, exploratory experiments |
| Phase 2 | Targeted technical study and simulation experiments |
| Phase 3 | Mathematical reasoning, algorithm implementation, validation |
| Phase 4 | Implementation-driven learning and system integration |
| Phase 5 | Experimental evaluation, critical analysis, technical documentation |

### Weekly Time Allocation

| Activity | Typical hours |
|---|---:|
| Study and research | 1 |
| Implementation and experimentation | 2 |
| Reflection and documentation | 1 |
| **Total** | **4** |

This is a flexible guideline. Phase 1 may require more study time and less implementation time.

### Weekly Completion Standard

A week is complete when:
1. Its learning objective has been addressed.
2. Required deliverables exist.
3. Results or concepts can be explained.
4. Findings and unresolved questions are documented.

Completing reading material alone does not demonstrate understanding.

---

## 5. Development Roadmap

### Phase 1 — Fundamentals and Algorithm Selection

**Duration:** Weeks 1–2 (8 hours)

**Goal:** Establish a coherent understanding of LV network faults, their observable effects, and available detection/localisation methods before selecting an algorithm.

#### Week 1 — Fundamentals of LV Network Faults

**Learning objectives**
- Understand common electrical fault types in LV distribution networks.
- Understand fault causes and physical mechanisms.
- Explore how faults affect current, voltage, and network operation.
- Understand conventional protection and fault detection principles.
- Explore how smart-meter measurements can support fault detection and localisation.
- Understand sampling, observability, and measurement limitations.

**Study materials**

Prepare a coherent, well-referenced technical chapter:

**Chapter 1 — Fundamentals of LV Distribution Network Faults**

Proposed structure:
1. LV distribution network characteristics and operating principles.
2. Electrical fault types, causes, and mechanisms.
3. Fault behaviour: current, voltage, impedance, and network response.
4. Conventional protection and fault detection mechanisms.
5. Smart-meter-based fault detection principles.
6. Fault localisation and the role of network topology.
7. Measurement limitations, observability, and uncertainty.
8. Worked examples and engineering applications.
9. References and recommended further reading.

The chapter should explain how concepts connect rather than presenting isolated definitions.

Provide credible academic references, technical resources, and optional educational videos where available.

**Activities**
- Study the core chapter independently.
- Explore additional references where needed.
- Identify questions or assumptions worth investigating.
- Conduct a small experiment showing normal and abnormal voltage behaviour, preferably using an existing OpenDSS model if readily available.
- Discuss findings and uncertainties arising from the study.

**Deliverables**
- [ ] Referenced Chapter 1 study material.
- [ ] Completed core reading and learning notes.
- [ ] Simple voltage disturbance experiment or visualisation.
- [ ] Documented observations and outstanding questions.

**Checkpoint:** Explain the relationship between electrical faults, measurable network changes, and the feasibility and limitations of smart-meter-based detection.

#### Week 2 — Detection Methods and Algorithm Selection

**Learning objectives**
- Understand the mathematical principles of candidate fault detection and localisation methods.
- Distinguish between model-based, measurement-based, and data-driven approaches.
- Identify the network information and measurements required by each approach.
- Evaluate implementation complexity, interpretability, and suitability.

**Study materials**

Prepare a focused literature review:

**Chapter 2 — Smart-Meter-Based Fault Detection and Localisation Methods**

Suggested structure:
1. Problem formulation and mathematical objectives.
2. Overview of candidate approaches.
3. Mathematical principles and required measurements.
4. Comparison of assumptions and limitations.
5. Applicability to the project's synthetic network.
6. Implementation considerations and research gaps.

Study at least three relevant approaches before selecting one.

**Activities**
- Study the literature review and primary research papers.
- Compare reasonable algorithm alternatives.
- Evaluate mathematical and computational requirements.
- Select one method for implementation.
- Derive its essential equations and develop pseudocode.

**Deliverables**
- [ ] Literature review comparing at least three approaches.
- [ ] Selected algorithm with verifiable reference.
- [ ] One-page explanation of the selected method.
- [ ] Mathematical formulation and pseudocode.
- [ ] Engineering decision record justifying the selection.

**Checkpoint:** Justify the selected approach in terms of measurement compatibility, physical validity, technical depth, and implementation feasibility.

**Milestone M1:** Research foundation and algorithm selection complete.

---

### Phase 2 — Synthetic Network and Fault Simulation

**Duration:** Weeks 3–5 (12 hours)

**Goal:** Develop a reproducible environment for generating normal and faulted LV network measurements.

#### Week 3 — LV Network Modelling

**Learning objectives**
- Review the relationship between topology, loading, and voltage distribution.
- Understand the network parameters required for fault simulation.

**Activities**
- Review concise technical material on OpenDSS network modelling.
- Adapt the existing honours thesis pipeline into a small test network.
- Simulate normal operating conditions.
- Extract customer voltage measurements.
- Visualise network topology and voltage profiles.

**Deliverables**
- [ ] Working radial LV network model.
- [ ] Normal operating condition simulation.
- [ ] Network topology and voltage visualisation.

**Checkpoint:** Reproduce baseline network measurements without manually modifying the model for each run.

#### Week 4 — Fault Simulation

**Learning objectives**
- Understand fault impedance, fault location, and fault-induced voltage changes.
- Learn how OpenDSS represents fault conditions.

**Activities**
- Review the required fault modelling concepts.
- Implement fault injection using OpenDSS.
- Simulate faults at different network locations.
- Compare faulted and normal voltage measurements.
- Investigate spatial voltage patterns.

**Deliverables**
- [ ] Working fault injection script.
- [ ] At least two distinct simulated fault locations.
- [ ] Normal versus faulted voltage comparison.

**Checkpoint:** Demonstrate measurable differences between normal and faulted operation, including at least one location-dependent effect.

#### Week 5 — Fault Dataset Generation

**Learning objectives**
- Understand dataset design for fault detection and localisation.
- Explore the effects of loading conditions and fault parameters.

**Activities**
- Automate variations in fault location and resistance.
- Include different loading conditions and no-fault cases.
- Export measurements and ground-truth labels.
- Prepare separate reference and evaluation scenarios.

**Deliverables**
- [ ] Reproducible dataset generation script.
- [ ] At least 50 labelled normal/fault scenarios, including at least 20 held out for evaluation.
- [ ] Data format and scenario parameter documentation.

**Checkpoint:** Generate the dataset consistently from a single script without manual intervention.

**Milestone M2:** Fault simulation and data pipeline complete.

---

### Phase 3 — Detection and Localisation Algorithm

**Duration:** Weeks 6–8 (12 hours)

**Goal:** Implement and quantitatively evaluate the selected algorithm.

#### Week 6 — Fault Detection Baseline

**Learning objectives**
- Understand anomaly detection using voltage deviations.
- Explore detection thresholds, false alarms, and sensitivity.

**Activities**
- Review relevant detection equations and assumptions.
- Develop a simple voltage-based baseline detector.
- Compare observations against expected normal behaviour.
- Test the effect of load variations and thresholds.

**Deliverables**
- [ ] Working fault detection baseline.
- [ ] Initial detection results.
- [ ] Threshold sensitivity experiment.

**Checkpoint:** Distinguish the tested faulted scenarios from normal operation and identify the detector's weaknesses.

#### Week 7 — Literature-Based Fault Localisation

**Learning objectives**
- Understand how spatial voltage information can estimate fault location.
- Connect the selected mathematical method to practical implementation.

**Activities**
- Review the algorithm's essential mathematical steps.
- Implement the selected localisation algorithm.
- Generate fault predictions from simulated measurements.
- Compare predictions with ground truth.
- Investigate ambiguous or incorrect predictions.

**Deliverables**
- [ ] Working localisation function.
- [ ] Ranked candidate fault locations.
- [ ] Example prediction with mathematical explanation.

**Checkpoint:** Execute the detection and localisation pipeline for a known test scenario.

#### Week 8 — Algorithm Validation

**Learning objectives**
- Understand detection and localisation performance metrics.
- Investigate performance differences between reference and unseen scenarios.

**Activities**
- Evaluate on held-out scenarios.
- Calculate precision, recall, and false-positive rate.
- Evaluate top-one and top-three localisation accuracy.
- Analyse representative failure cases.

**Deliverables**
- [ ] Automated evaluation script.
- [ ] Performance metrics table.
- [ ] Localisation results visualisation.
- [ ] Discussion of errors and algorithm limitations.

**Checkpoint:** Report reproducible performance on unseen scenarios without tuning parameters against the final test set.

**Milestone M3:** Fault detection and localisation algorithm implemented and evaluated.

---

### Phase 4 — Interactive Web Platform

**Duration:** Weeks 9–10 (8 hours)

**Goal:** Transform the simulation and algorithm into an interactive engineering demonstration.

#### Week 9 — Network Visualisation Dashboard

**Activities**
- Develop the initial Streamlit interface.
- Display network topology and measurements.
- Visualise normal and faulted voltage profiles.
- Display detection status.

**Deliverables**
- [ ] Working Streamlit application.
- [ ] Interactive LV network plot.
- [ ] Voltage measurement visualisation.

**Checkpoint:** Explore a precomputed network scenario through the dashboard.

#### Week 10 — Interactive Fault Simulation and Analysis

**Activities**
- Add controls for fault location and resistance.
- Integrate simulation and prediction functions.
- Display predicted and actual fault locations.
- Provide useful detection and localisation insights.

**Deliverables**
- [ ] Fault simulation controls.
- [ ] Integrated detection and localisation workflow.
- [ ] End-to-end demonstration.

**Checkpoint:** Select a fault scenario, execute the analysis, and visualise the result without modifying source code.

**Milestone M4:** Interactive fault analysis platform operational.

---

### Phase 5 — Robustness, Documentation and Showcase

**Duration:** Weeks 11–12 (8 hours)

**Goal:** Evaluate engineering limitations and prepare a reproducible, portfolio-quality project.

#### Week 11 — Robustness Analysis

**Learning objectives**
- Understand measurement and modelling uncertainty.
- Identify limitations of the chosen fault detection method.

**Activities**
- Select at least two robustness factors, such as measurement noise, loading variation, missing measurements, or fault resistance.
- Rerun evaluations with controlled changes.
- Explain performance differences.

**Deliverables**
- [ ] Robustness experiments.
- [ ] Performance comparison plots.
- [ ] Engineering discussion of limitations.

**Checkpoint:** Explain when and why the algorithm performs poorly.

#### Week 12 — Finalisation and Technical Documentation

**Activities**
- Refactor and test important functions.
- Improve reproducibility and project organisation.
- Finalise the technical report and README.
- Record a short demonstration.
- Prepare a concise resume project description.

**Deliverables**
- [ ] Organised GitHub repository.
- [ ] Reproducible setup instructions.
- [ ] Completed technical documentation.
- [ ] Demonstration video.
- [ ] Resume-ready project summary.

**Checkpoint:** Another developer can understand the method, run the prototype, and reproduce the main evaluation results.

**Milestone M5:** Project complete and ready for portfolio presentation.

---

## 6. Project Acceptance Criteria

- [ ] A relevant literature-based algorithm has been studied, justified, and implemented.
- [ ] Normal and faulted LV conditions can be simulated reproducibly.
- [ ] The fault detection and localisation pipeline runs end-to-end.
- [ ] Detection performance and localisation accuracy are quantified.
- [ ] The method's assumptions and limitations are documented.
- [ ] The Streamlit platform demonstrates fault injection and analysis.
- [ ] The repository includes research materials, setup instructions, experiments, and technical documentation.

No specific accuracy target is assumed before benchmarking.

## 7. Risk Management and Scope Control

| Risk | Mitigation |
|---|---|
| Research becomes too extensive | Keep core chapters scoped to weekly objectives; make deeper reading optional |
| Algorithm too complex | Reassess at Week 2 and prioritise a feasible method |
| OpenDSS modelling difficulties | Start with a small feeder and limited scenarios |
| Insufficient measurement observability | Define observable conditions and avoid unrealistic claims |
| Poor localisation accuracy | Investigate ambiguities and document limitations |
| Development falls behind schedule | Prioritise algorithm validation over dashboard enhancements |

### Minimum Viable Project

If time becomes constrained, prioritise:

1. Reproducible synthetic fault scenarios.
2. One evaluated literature-based detection/localisation method.
3. A minimal Streamlit demonstration.
4. Documentation explaining the method and results.

## 8. Planned Final Outputs

- Referenced study chapters and research notes.
- Python/OpenDSS simulation pipeline.
- Fault detection and localisation implementation.
- Evaluation scripts and results.
- Interactive Streamlit application.
- Weekly learning and progress log.
- Technical report and GitHub README.
- Project demonstration.

**Guiding principle:** Study the fundamentals, investigate them through experiments, understand the results, and document the engineering reasoning.