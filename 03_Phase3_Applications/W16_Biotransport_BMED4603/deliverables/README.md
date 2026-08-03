# Week 16 Deliverables — Biotransport (Diffusion and Fluid Mechanics)

> **Course**: BMED4603 — Transport Processes in Biological Systems  
> **Due**: End of Week 16 | **Format**: PDF or Jupyter Notebook  
> **Total points**: 100 (problem set: 60 pts, modeling report: 40 pts)

---

## Assignment 1: Biotransport Problem Set (60 points)

### Section A: Fick's Laws of Diffusion (20 points)

**Q1** (5 pts) — Diffusion distance calculation  
Oxygen (D = 2×10⁻⁵ cm²/s in tissue) diffuses from a capillary into surrounding muscle tissue.

(a) Calculate the characteristic diffusion time τ = L²/D for L = 50 μm, 100 μm, 200 μm.  
(b) If the tissue surface concentration is maintained at C_s = 10⁻⁷ mol/cm³ and the deep tissue concentration is 0, calculate the steady-state flux J (mol/m²·s) for a 100 μm tissue slab.  
(c) How long does it take for the concentration at x = 50 μm to reach 50% of its steady-state value? Use the error function solution for a semi-infinite medium.

**Q2** (5 pts) — Molecular size and diffusivity  
Use the Stokes-Einstein relation D = k_B T / (6πμr) to answer:

(a) Calculate the diffusivity of glucose (r ≈ 0.37 nm) in water at 37°C (μ = 0.7 cP).  
(b) Compare to the diffusivity of hemoglobin (r ≈ 3.1 nm, 64 kDa). How much slower is hemoglobin diffusion?  
(c) Estimate the time for glucose to diffuse across a cell membrane thickness of 8 nm (D in lipid = 10⁻⁸ cm²/s).  

**Q3** (5 pts) — Transient diffusion  
A drug patch releases drug into the skin (D = 5×10⁻⁸ cm²/s). The drug concentration at the patch-tissue interface is maintained at C₀ = 10⁻⁴ mol/cm³. Initially, the tissue concentration is 0. After 1 hour, what is the concentration at depths of x = 0.1 mm and x = 0.5 mm?

**Q4** (5 pts) — Concentration profile derivation  
Derive the steady-state concentration profile C(x) for a tissue slab (0 ≤ x ≤ L) with surface concentrations C(0) = C₁ and C(L) = C₂, using Fick's first law. Show all steps.

---

### Section B: Starling Forces and Microcirculation (20 points)

**Q5** (5 pts) — Net filtration calculation  
For a systemic capillary with:
- P_c (arteriolar) = 35 mmHg, P_c (venular) = 15 mmHg  
- P_i = -2 mmHg (interstitial)  
- π_c = 25 mmHg (plasma oncotic pressure)  
- π_i = 2 mmHg (interstitial)  
- L_p = 0.01 mL/min/mmHg/cm², σ = 0.95 (for albumin)  

(a) Calculate net effective pressure ΔP_eff at both ends.  
(b) If the capillary surface area is 500 cm² per 100g of tissue, calculate the filtration rate at the arteriolar end.  
(c) State whether net filtration or reabsorption occurs at the venular end.

**Q6** (5 pts) — Edema pathophysiology  
Left-sided heart failure increases mean capillary pressure from 25 mmHg to 45 mmHg.

(a) Calculate the new net filtration rate (per 100g tissue).  
(b) If lymphatic return can compensate for only 2 mL/min/100g of filtration, how much fluid accumulates per hour?  
(c) Name two other conditions that can cause edema through different mechanisms.

**Q7** (5 pts) — Reflection coefficient significance  
The reflection coefficient σ for glucose is approximately 0, while for albumin σ ≈ 0.95. Explain this difference in physical terms. What does σ = 0.95 mean about albumin transport across the capillary wall?

**Q8** (5 pts) — Glycocalyx and revised Starling hypothesis  
Explain the role of the endothelial glycocalyx layer (EGL, ~0.5 μm thick) in the revised Starling hypothesis. Why does the EGL make the simple Starling equation incomplete for quantitative predictions?

---

### Section C: Fluid Mechanics and Oxygen Transport (20 points)

**Q9** (5 pts) — Poiseuille's law  
Blood flows through an arteriole of radius R = 100 μm, length L = 5 mm, at a pressure gradient ΔP = 20 mmHg.

(a) Calculate the volumetric flow rate Q (μL/s). Blood viscosity μ = 3 cP.  
(b) Calculate the average velocity u_avg and maximum velocity u_max.  
(c) What is the wall shear stress τ_w?  
(d) If the arteriole vasoconstricts to R = 70 μm, by what factor does Q change?

**Q10** (5 pts) — Krogh cylinder oxygenation  
A capillary (R_c = 3 μm) supplies a cylindrical tissue volume (R_t = 25 μm radius) in resting skeletal muscle.

(a) Using the Krogh model P(r) = P_cap - Ṁr²/(4Dα), calculate P_O₂ at r = 15 μm. Given: P_cap = 40 mmHg, Ṁ = 3.5 mL O₂/100g/min, D = 1.5×10⁻⁵ cm²/s, α = 3×10⁻² mL O₂/100g/mmHg.  
(b) Is this tissue region hypoxic (P_O₂ < 10 mmHg)?  
(c) Calculate the critical capillary P_O₂ above which the entire tissue cylinder remains above the hypoxic threshold.

**Q11** (5 pts) — Convective vs. diffusive transport  
The Peclet number Pe = J_v(1-σ)/P characterizes the relative importance of convective vs. diffusive solute transport.

(a) For albumin (σ = 0.95, P = 10⁻⁷ cm/s) at a filtration rate J_v = 10⁻⁵ cm/s, calculate Pe.  
(b) For glucose (σ ≈ 0, P = 10⁻³ cm/s) at the same J_v, calculate Pe.  
(c) Interpret both results in terms of dominant transport mechanism for each solute.

**Q12** (5 pts) — Reynolds number and flow regime  
Calculate the Reynolds number for:
(a) Blood flow in the aorta (D = 2.5 cm, u_avg = 30 cm/s, ρ = 1.06 g/cm³, μ = 3 cP)  
(b) Blood flow in a venule (D = 100 μm, u_avg = 2 mm/s)  

State the flow regime for each and explain the physiological significance.

---

## Assignment 2: Microcirculation Modeling Report (40 points)

### Overview
Write a 2-3 page technical report (including figures) on a topic chosen from below.

### Option A: Pulmonary Edema Modeling
Model the development of pulmonary edema in acute respiratory distress syndrome (ARDS). Using the Starling equation:

1. Collect literature values for normal and ARDS pulmonary capillary parameters (L_p, P_c, σ, π_c)
2. Calculate the filtration rate under normal and ARDS conditions
3. Plot the change in filtration rate vs. time (assume lymphatic compensation of 20 mL/h)
4. Discuss the clinical implications

### Option B: Tumor Oxygenation and Hypoxia
Using the Krogh cylinder model:

1. Model oxygen tension in a solid tumor with intercapillary distance of 150 μm
2. Compare with normal muscle (intercapillary distance = 50 μm)
3. Plot P_O₂ profiles from capillary to tissue midpoint for both cases
4. Discuss implications for radiation therapy resistance

### Option C: Kedem-Katchalsky Analysis of Dialysis
Analyze the removal of urea vs. albumin in hemodialysis using the Kedem-Katchalsky framework:

1. Given: Urea σ ≈ 0, P_urea = 10⁻³ cm/s; Albumin σ ≈ 0.9, P_alb = 10⁻⁷ cm/s
2. For a filtration rate of J_v = 10⁻⁴ cm/s, calculate solute flux for both
3. Discuss why hemodialysis effectively removes urea but not albumin
4. Explain the role of convective transport in high-flux dialysis

### Report Format
- Title, author, affiliation
- Abstract (100 words)
- Introduction (1 paragraph)
- Methods (Starling/Krogh calculations)
- Results (figures + 1 paragraph)
- Discussion (1-2 paragraphs)
- References (3+ primary literature)

### Grading Rubric

| Criterion | Excellent (A) | Good (B) | Satisfactory (C) | Incomplete (D/F) |
|-----------|--------------|---------|-----------------|-----------------|
| Quantitative analysis (20 pts) | All calculations correct, complete | Minor errors | Partial attempt | Missing |
| Figures (10 pts) | Clear, labeled, informative | Readable | Basic | Missing |
| Discussion (10 pts) | Insightful, connected to BME | Descriptive | Brief | Missing |

---

## Submission Instructions

1. **Problem Set**: Submit as PDF or handwritten scan. Show all working for partial credit.
2. **Modeling Report**: Submit as PDF. Include Python code in appendix or supplementary file.
3. **Code outputs**: Attach generated figures as PNG/PDF files.

**Late policy**: 10% penalty per day late. Max 3 days late.
