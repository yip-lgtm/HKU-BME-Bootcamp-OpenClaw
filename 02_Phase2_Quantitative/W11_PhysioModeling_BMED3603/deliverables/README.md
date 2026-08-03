# Week 11 Deliverables — Physiological Modeling (BMED3603)

## Deliverable 1: ODE Modeling Problem Set

- **Type**: Problem Set + Code
- **Deadline**: End of Week 11 (Sunday, 23:59 HKT)
- **Submission**: PDF + .ipynb

### Requirements

**Problem 1 — One-Compartment Pharmacokinetics (15 marks)**
A drug has the following PK parameters: V_d = 40 L, k_e = 0.15 hr⁻¹.

a) Calculate the half-life and time constant.
b) A patient receives 500 mg IV bolus. What is the initial concentration?
c) Plot C(t) for 48 hours. Mark the half-life and therapeutic window (2-8 mg/L).
d) The patient needs to maintain C > 3 mg/L. How often should doses be given?
e) After reaching steady state with 200 mg doses, what is the C_max and C_min?

**Problem 2 — Second-Order Cardiovascular Model (15 marks)**
A cardiovascular monitoring system is modeled as a second-order system with ω_n = 3 rad/s.

a) For ζ = 0.3, calculate: overshoot (%), peak time, settling time (2% criterion), rise time.
b) Compare responses for ζ = 0.1, 0.3, 0.5, 1.0, 2.0. Which has the fastest rise time? Which has no overshoot?
c) A hypertensive patient has stiffer vessels (higher ω_n = 5 rad/s). How does this change the response?
d) Plot the step response for all cases. Label key features (overshoot, settling time).

**Problem 3 — Hodgkin-Huxley Analysis (10 marks)**
a) Explain the biological role of each gate variable (m, h, n) in the HH model.
b) At V = 0 mV, calculate α_m, β_m, α_h, β_h, α_n, β_n using the standard formulas.
c) Simulate the model with I_stim = 10 μA/cm² for 1 ms. Report peak V and spike width.
d) What is the minimum current needed to generate an action potential (rheobase)?
e) Explain why the refractory period prevents the neuron from firing again immediately.

**Problem 4 — Parameter Fitting (10 marks)**
Given concentration-time data for a drug: t = [0.5, 1, 2, 4, 8, 12, 24] hours, C = [9.1, 8.5, 7.4, 5.8, 3.4, 2.0, 0.5] mg/L:

a) Fit a one-compartment model using curve_fit.
b) Report V_d and k_e with 95% confidence intervals.
c) Calculate R² for the fit.
d) Predict the concentration at t = 48 hours.
e) Calculate the time to reach 50% of steady-state concentration.

### Rubric

- [ ] **P1**: Half-life calculated (3pts), initial concentration (2pts), plot (3pts), dosing interval (3pts), steady state (4pts)
- [ ] **P2**: Characteristics calculated for all ζ (4pts), comparison (3pts), plot (4pts), clinical interpretation (4pts)
- [ ] **P3**: Gating variable explanation (3pts), rate constants calculated (3pts), simulation (2pts), threshold (2pts)
- [ ] **P4**: Parameter fit (3pts), CIs (2pts), R² (2pts), prediction (3pts)

**Total: 50 marks**

---

## Deliverable 2: Physiological Modeling Project

- **Type**: Research Report + Code
- **Deadline**: End of Week 11

### Task
Build a physiological model for one of the following systems and write a brief research report.

**Options**:
- **Option A**: Two-compartment pharmacokinetic model (oral dosing)
- **Option B**: Cardiovascular Windkessel model
- **Option C**: Glucose-insulin regulatory system (minimal model)
- **Option D**: Respiratory gas exchange model

### Required Components

1. **Model Derivation (10 marks)**:
   - Write the ODE system mathematically
   - Explain the physiological meaning of each parameter
   - Include a block diagram

2. **Numerical Simulation (15 marks)**:
   - Implement the model in Python using scipy.integrate
   - Solve for realistic parameter values
   - Plot key state variables over time
   - Validate with literature values

3. **Sensitivity Analysis (10 marks)**:
   - Vary one key parameter ±50% and show the effect
   - Discuss physiological implications

4. **Clinical Application (10 marks)**:
   - Connect to a real clinical scenario
   - Discuss how the model could guide treatment decisions

### Rubric

- [ ] Model derivation with physiological explanation (10 pts)
- [ ] Correct implementation and simulation (15 pts)
- [ ] Sensitivity analysis (10 pts)
- [ ] Clinical application (10 pts)
- [ ] Code quality (5 pts)

**Total: 50 marks**

---

## Deliverable 3: Model Validation Challenge (Bonus)

Fit a model to noisy data using least squares. Evaluate your model using residual analysis.

**Bonus 5 points** for best model fit.

---

## Submission Instructions

- `[StudentID]_W11_[DeliverableName].[ext]`
- Include `requirements.txt`

**Maintainer**: BME Bootcamp Agent | **Week 11** | **BMED3603: Physiological Modeling**
