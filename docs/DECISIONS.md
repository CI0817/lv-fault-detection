# Engineering Decision Log

This document records significant design decisions, alternatives considered, trade-offs, and technical justification.

Decisions should be revisited only when new evidence or project constraints warrant reconsideration.

## Decision Summary

| ID | Decision | Status | Related week |
|---|---|---|---|
| D001 | Reuse synthetic LV network generation concepts | Accepted | Planning |
| D002 | Use Python/OpenDSS | Accepted | Planning |
| D003 | Use Streamlit for the initial platform | Provisional | Planning |
| D004 | Select fault detection/localisation algorithm | Pending | Week 2 |

**Status options:** Proposed / Provisional / Accepted / Superseded / Rejected

---

## D001 — Reuse Synthetic LV Network Generation Concepts

**Status:** Accepted

**Problem:** Establish a suitable LV network simulation environment within the project timeline.

**Alternatives:**
- Reuse and adapt the honours thesis simulation pipeline.
- Develop a new simulation pipeline.
- Use an existing benchmark network.

**Decision:** Reuse the existing pipeline as a starting point.

**Rationale:** Existing familiarity reduces setup effort and allows greater focus on fault detection and localisation.

**Trade-offs:** Existing code may require adaptation for fault simulation and controlled experiments.

---

## D002 — Python and OpenDSS

**Status:** Accepted

**Problem:** Select the primary programming and power-system simulation environment.

**Alternatives:**
- Python with OpenDSS.
- Python with pandapower.
- MATLAB-based simulation.

**Decision:** Use Python with OpenDSS.

**Rationale:** Existing experience, compatibility with the previous simulation pipeline, and relevant fault simulation capabilities.

**Trade-offs:** Requires validation of fault modelling assumptions and OpenDSS simulation behaviour.

---

## D003 — Streamlit Platform

**Status:** Provisional

**Problem:** Select a framework for the interactive demonstration.

**Alternatives:**
- Streamlit.
- Dash.
- React with a Python backend.

**Provisional decision:** Use Streamlit unless later requirements justify an alternative.

**Rationale:** Rapid development and integration with Python-based analysis.

**Trade-offs:** Less frontend flexibility than a dedicated web application framework.

---

## Decision Template

### DXXX — [Decision Title]

**Date:** [Date]  
**Status:** Proposed / Provisional / Accepted / Superseded / Rejected

**Problem:** [What needs to be decided?]

**Constraints:** [Technical, time, or resource limitations.]

**Alternatives:**

| Option | Advantages | Disadvantages |
|---|---|---|
| A | | |
| B | | |
| C | | |

**Decision:** [Selected approach.]

**Rationale:** [Why was it selected?]

**Trade-offs:** [What compromises were accepted?]

**Evidence:** [References, experiments, or technical reasoning.]

**Revisit when:** [Conditions that justify reassessment.]

**Related records:** [Research or experiment links.]