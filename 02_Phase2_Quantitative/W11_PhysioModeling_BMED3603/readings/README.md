# Week 11 Readings — Physiological Modeling (ODEs) (BMED3603)

## Required Readings

### Primary Textbooks
- **Keenan, A. et al.** (2017) *Mathematical Physiology* (Keenan & Klavins), 2nd ed. Springer — Chapters 1-4 (Mathematical foundations, cellular dynamics)
- **Cobelli, C. & Carson, E.** (2008) *Introduction to Modeling in Physiology and Medicine* — Academic Press — Chapters 1-6
- **Murray, J.D.** (2002) *Mathematical Biology I: An Introduction*, 3rd ed. Springer — Chapters 1-3

### HKU Course Materials
- **BMED3603** — Biostatistics and Modeling for Biomedical Engineers (HKU SBME)
- Topics: First-order ODEs, second-order systems, compartmental models, Hodgkin-Huxley, cardiovascular models

### Key Papers / Historical References
1. **Hodgkin, A.L. & Huxley, A.F.** (1952) "A Quantitative Description of Membrane Current and Its Application to Conduction and Excitation in Nerve" — *Journal of Physiology*, 117(4), 500-544 — Nobel Prize 1963
2. **Windkessel, O.** (1899) *Die Lehre von der Herztätigkeit* — German treatise on cardiovascular modeling
3. **Kety, S.S.** (1948) "Measurement of Local Blood Flow by the Exchange of an Inert, Diffusible Substance" — *Methods in Medical Research*
4. **Sheppard, C.W.** (1948) "Properties ofTracer Currents" — Indicator dilution theory

### Online Resources
- MATLAB Documentation: **ODE Solvers** — ode45, ode15s, ODE Suite
- Python SciPy: **solve_ivp, odeint** — ODE integration
- Wolfram Alpha: **Differential Equations** — Symbolic and numeric solutions
- NIH Physiome Project: **CellML Models** — Repository of physiological models
- BioModels Database: **Curated Models** — Standard physiological models

## Pre-Week Objectives (BMED3603 Week 11)
By the end of Week 11, students should be able to:
1. Formulate first-order ODEs for physiological systems (RC circuit analogy)
2. Solve linear first-order ODEs using integrating factor method
3. Analyze second-order systems: overdamped, critically damped, underdamped
4. Build one-compartment and two-compartment pharmacokinetic models
5. Understand the Hodgkin-Huxley model: 4 gating variables, 4 currents
6. Analyze cardiovascular Windkessel models: 2-element, 3-element
7. Apply numerical methods: Euler's method, Runge-Kutta (RK4)
8. Perform sensitivity analysis on physiological parameters

## Mathematical Foundations

### Core Equations (Week 11)

#### First-Order Linear ODEs
1. **Standard form**: τ(dy/dt) + y = Kx(t)
2. **Solution**: y(t) = y(0)e^(-t/τ) + (K/τ)∫x(τ')e^(-(t-τ')/τ)dτ'
3. **Time constant**: τ (tau) — characterizes system response speed
4. **Steady-state gain**: K = lim(y(t)/x(t)) as t→∞

#### Second-Order Systems
5. **Standard form**: d²y/dt² + 2ζω_n(dy/dt) + ω_n²y = ω_n²x(t)
6. **Natural frequency**: ω_n = √(k/m) [rad/s]
7. **Damping ratio**: ζ = c/(2√(km))
8. **Character roots**: s = -ζω_n ± ω_n√(ζ²-1)
9. **Underdamped (ζ < 1)**: y(t) = Ae^(-ζω_nt)cos(ω_dt + φ), ω_d = ω_n√(1-ζ²)
10. **Overdamped (ζ > 1)**: Two real negative roots
11. **Critically damped (ζ = 1)**: Double root at s = -ω_n

#### Hodgkin-Huxley Model
12. **Membrane equation**: C(dV/dt) = -I_mem = -(I_Na + I_K + I_L)
13. **Sodium current**: I_Na = g_Na·m³·h·(V - V_Na)
14. **Potassium current**: I_K = g_K·n⁴·(V - V_K)
15. **Leak current**: I_L = g_L·(V - V_L)
16. **Gating dynamics**: dm/dt = α_m(1-m) - β_mm
17. **Equilibrium potentials**: V_Na = +60mV, V_K = -90mV, V_L = -70mV

#### Pharmacokinetic Compartmental Models
18. **One-compartment (oral dose)**: dC/dt = -k_e·C + k_a·D·e^(-k_a·t)/V_d
19. **Two-compartment**: Central compartment (V_c) + Peripheral compartment (V_p)
20. **Clearance**: CL = k_e·V_d = rate of elimination / C
21. **Half-life**: t_1/2 = ln(2)/k_e ≈ 0.693·τ

#### Windkessel Cardiovascular Model
22. **2-element**: R·C(dP/dt) + P = R·Q(t)
23. **3-element**: Z_c·C(dP/dt) + (1 + R_1/R_2)·P = Z_c·Q(t) + R_1·C(dQ/dt)
24. **Characteristic impedance**: Z_c = ρ·c / A (pulse wave velocity)

### Physiological Parameters Reference Values

| Parameter | Symbol | Value | Units |
|-----------|--------|-------|-------|
| Membrane capacitance | C | 1.0 | μF/cm² |
| Na conductance | g_Na | 120 | mS/cm² |
| K conductance | g_K | 36 | mS/cm² |
| Leak conductance | g_L | 0.3 | mS/cm² |
| Resting potential | V_rest | -70 | mV |
| Drug elimination rate | k_e | 0.1-0.5 | hr⁻¹ |
| Volume of distribution | V_d | 10-50 | L |
| Systemic vascular resistance | R | 1.0-1.5 | mmHg·s/mL |
| Arterial compliance | C | 1.5-2.0 | mL/mmHg |

## Assessment Alignment
- BMED3603 Problem Set 2 (Week 12): ODE modeling project
- BMED3603 Quiz 2 (Week 12): Model simulation and analysis
- HKU SBME Lab 2: Hodgkin-Huxley simulation in Python/MATLAB

## Key Concepts for Deep Dive
1. **Why Linearize?** — Taylor expansion around operating point
2. **Phase Portraits** — Visualizing 2nd-order system dynamics
3. **Bifurcation** — When small parameter changes cause qualitative changes
4. **Parameter Sensitivity** — Which physiological parameters matter most?
5. **Model Validation** — Comparing model predictions to experimental data

## Supplementary Reading
- **Artoli, A.M.** (2013) *Physiological Modeling with MATLAB* — Imperial College Press
- **Michaels, J.A.** (1985) *Physiological Systems: Numerical Methods* — Saunders
- **Guyton, A.C. & Hall, J.E.** (2020) *Textbook of Medical Physiology*, 14th ed. Elsevier

## Reflection Questions
1. Why does the Hodgkin-Huxley model use m³h and n⁴ for gating variables?
2. What determines whether a second-order system is overdamped or underdamped?
3. How does compartmental modeling relate to actual drug distribution in the body?
4. Why is the Windkessel model still used despite its simplifications?
5. What is the relationship between time constant τ and system bandwidth?
