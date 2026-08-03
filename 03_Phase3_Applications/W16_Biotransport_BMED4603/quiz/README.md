# Week 16 Quiz — Biotransport (Diffusion and Fluid Mechanics)

> **Course**: BMED4603 — Transport Processes in Biological Systems  
> **Duration**: 45 minutes | **Total points**: 50 (MCQ: 30 pts, Short Answer: 20 pts)  
> **Instructions**: Write answers on lined paper. Scientific calculators permitted.

---

## Part I: Multiple Choice Questions (30 points — 3 pts each)

**Q1.** The diffusion coefficient D for oxygen in water at 37°C is approximately:
- (A) 2.4×10⁻² cm²/s
- (B) 2.4×10⁻⁵ cm²/s
- (C) 2.4×10⁻⁸ cm²/s
- (D) 2.4×10⁻¹¹ cm²/s

**Q2.** The characteristic diffusion time τ for a solute to diffuse across a distance L is proportional to:
- (A) L/D
- (B) L²/D
- (C) D/L²
- (D) L·D

**Q3.** Which of the following statements about Starling forces is FALSE?
- (A) Net filtration occurs at the arteriolar end of a systemic capillary
- (B) The reflection coefficient σ is 1.0 for glucose
- (C) The glycocalyx layer affects capillary permeability
- (D) Edema results when filtration exceeds lymphatic return

**Q4.** In Poiseuille flow through a cylindrical tube, halving the radius reduces the flow rate Q by a factor of:
- (A) 2
- (B) 4
- (C) 8
- (D) 16

**Q5.** The wall shear stress τ_w in Poiseuille flow is given by:
- (A) ΔP·R/L
- (B) ΔP·R/(2L)
- (C) ΔP·R²/(4L)
- (D) 8μQ/(πR³)

**Q6.** The Fåhræus-Lindqvist effect describes the observation that blood apparent viscosity:
- (A) Increases as vessel diameter decreases below 300 μm
- (B) Decreases as vessel diameter decreases below 300 μm
- (C) Is constant regardless of vessel diameter
- (D) Increases with hematocrit

**Q7.** In the Krogh cylinder model for tissue oxygenation, the tissue P_O₂:
- (A) Is constant at all radial distances from the capillary
- (B) Decreases parabolically with the square of radial distance
- (C) Increases linearly with radial distance
- (D) Is independent of oxygen consumption rate

**Q8.** The reflection coefficient σ in Kedem-Katchalsky equations ranges from:
- (A) 0 to 10
- (B) 0 to 1
- (C) -1 to 1
- (D) 1 to 100

**Q9.** The Peclet number Pe = J_v(1-σ)/P characterizes:
- (A) The relative importance of convection vs. diffusion for solute transport
- (B) The Reynolds number for blood flow
- (C) The diffusion time scale
- (D) The osmotic pressure difference

**Q10.** The endothelial glycocalyx layer (EGL) in capillaries:
- (A) Increases hydraulic conductivity L_p
- (B) Acts as the primary permeability barrier and determines σ
- (C) Is absent in pulmonary capillaries
- (D) Prevents all protein transport

---

## Part II: Short Answer Questions (20 points — 4 pts each)

**Q11.** State Fick's first law of diffusion in words and write its mathematical form. Define all variables including units. (4 pts)

**Q12.** Write the Starling equation for net transcapillary fluid flux J_v. Define L_p, σ, ΔP, and Δπ. Why is the glycocalyx important in the revised Starling hypothesis? (4 pts)

**Q13.** Using the Stokes-Einstein relation D = k_B T/(6πμr), calculate the ratio of diffusion coefficients for O₂ (r = 0.18 nm) vs. glucose (r = 0.37 nm) in water at 37°C. Which diffuses faster and by how much? (4 pts)

**Q14.** Sketch the velocity profile u(r) for Poiseuille flow in a cylindrical tube. Label the maximum velocity and show that u_avg = u_max/2 for this parabolic profile. (4 pts)

**Q15.** Write the Krogh equation for tissue P_O₂ at radius r from the capillary center. What parameters would you expect to differ significantly between resting muscle and a solid tumor, and in what direction? (4 pts)

---

## Answer Key (for instructor use)

### Part I: MCQ Answers

| Q | Answer | Topic |
|---|--------|-------|
| Q1 | **B** | Fick's laws; D_O₂ in water |
| Q2 | **B** | Characteristic diffusion time τ = L²/D |
| Q3 | **B** | Starling forces; σ for glucose ≈ 0 (freely permeable) |
| Q4 | **D** | Poiseuille: Q ∝ r⁴; (1/2)⁴ = 1/16 |
| Q5 | **B** | Wall shear: τ_w = ΔP·R/(2L) |
| Q6 | **B** | Fåhræus-Lindqvist: μ_app ↓ as d ↓ below 300 μm |
| Q7 | **B** | Krogh model: P(r) = P_cap - Ṁr²/(4Dα) |
| Q8 | **B** | Reflection coefficient: 0 ≤ σ ≤ 1 |
| Q9 | **A** | Peclet number: convection/diffusion ratio |
| Q10 | **B** | Glycocalyx: primary permeability barrier |

### Part II: Short Answer Model Answers

**Q11**: *Fick's first law states that the molar flux of a solute is proportional to the negative concentration gradient. Mathematically: J = -D·(dC/dx) where J = molar flux (mol/m²·s), D = diffusion coefficient (m²/s), C = concentration (mol/m³), x = distance (m). The negative sign indicates flux is opposite to the direction of increasing concentration (diffusion from high to low concentration).*

**Q12**: *J_v = L_pS[(P_c - P_i) - σ(π_c - π_i)] where L_p = hydraulic conductivity (mL/min/mmHg/cm²), σ = reflection coefficient (0-1), ΔP = capillary minus interstitial hydrostatic pressure, Δπ = capillary minus interstitial oncotic pressure. The glycocalyx (EGL, ~0.5 μm) is the key structure determining L_p and σ; the revised Starling hypothesis includes the pressure drop across the EGL as a separate term.*

**Q13**: *D ∝ 1/r, so D_O₂/D_glucose = r_glucose/r_O₂ = 0.37/0.18 ≈ 2.06. Oxygen diffuses approximately 2× faster than glucose due to its smaller molecular radius.*

**Q14**: *Parabolic profile: u(r) = u_max(1 - (r/R)²) where u_max = ΔP·R²/(4μL). Average velocity u_avg = Q/(πR²) = u_max/2 (integration of parabolic profile).*

**Q15**: *P_O₂(r) = P_cap - Ṁr²/(4Dα). Tumor parameters vs. resting muscle: ↑intercapillary distance, ↓P_cap (poor perfusion), ↑Ṁ (hypoxic cells), ↓D (compressed extracellular matrix). Result: lower tissue P_O₂, larger hypoxic regions.*

---

## Bloom's Taxonomy Distribution

| Level | Questions | Points |
|-------|----------|--------|
| Knowledge (recall) | Q1, Q2 | 6 pts |
| Comprehension | Q3, Q6, Q8, Q10 | 12 pts |
| Application | Q4, Q5, Q7, Q9, Q11-15 | 32 pts |

---

## Common Student Errors

1. **Q3**: Confusing σ for glucose (≈ 0) with σ for albumin (≈ 0.95). Glucose is freely permeable; albumin is nearly completely retained.
2. **Q4**: Forgetting the r⁴ dependence in Poiseuille's law. Halving radius → factor of 2⁴ = 16 reduction.
3. **Q14**: Confusing wall shear stress formula (ΔP·R/2L) with resistance formula (8μL/πr⁴).
4. **Q15**: Using linear instead of quadratic radial dependence in the Krogh equation.
