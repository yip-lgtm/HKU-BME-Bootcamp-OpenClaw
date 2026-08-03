# Week 13 Deliverables — Biomechanics (BMED2600)

> **Due**: End of Week 13 weekend (Sunday midnight)  
> **Submission**: Push to `03_Phase3_Applications/W13_Biomechanics_BMED2600/deliverables/`

---

## Deliverable 1: Biomechanical Analysis Report — Hip Prosthesis Stem

### Type
**Technical Engineering Report** (2000–2500 words, PDF + Python code)

### Requirements

You are a BME engineer at a medical device company designing a cementless femoral stem for total hip arthroplasty. You must perform a biomechanical analysis comparing three candidate materials and justify your recommendation.

**Scenario**: The stem is 150 mm long, cross-sectional area = 400 mm², loaded with F = 2500 N (single-leg stance, 80 kg person). The proximal femur (zone 1-7, Gruen zones) applies distributed load.

**Tasks**:

1. **Stress Analysis** (40%)
   - Calculate normal stress σ = F/A for the stem
   - Apply stress concentration factor K_t = 2.5 at the stem-neck junction
   - Compute von Mises equivalent stress at critical section
   - Safety factor against yield for each material

2. **Material Comparison** (30%)
   Compare THREE candidate materials:
   | Property | Ti-6Al-4V | CoCrMo | SS316L |
   |----------|-----------|--------|--------|
   | E (GPa) | 110 | 210 | 190 |
   | σ_yield (MPa) | 880 | 450 | 215 |
   | σ_ult (MPa) | 950 | 900 | 515 |
   | ρ (g/cm³) | 4.43 | 8.3 | 8.0 |
   | Relative cost | High | High | Low |

3. **Stress Shielding Analysis** (20%)
   - Calculate stiffness ratio: E_implant / E_cortical_bone
   - Explain how stiffness mismatch causes bone resorption (use Frost's mechanostat theory, ε < 800 με triggers resorption)
   - Propose one design modification to reduce stress shielding

4. **Conclusion** (10%)
   - Recommend optimal material with engineering justification
   - State limitations and assumptions

### Rubric

| Criterion | Excellent (90-100%) | Good (75-89%) | Satisfactory (60-74%) | Needs Improvement (<60%) |
|-----------|---------------------|---------------|----------------------|--------------------------|
| Stress analysis accuracy | All calculations correct, units consistent, K_t applied properly | Minor errors in one calculation | Several calculation errors | Major errors or missing approach |
| Material comparison depth | All 3 materials compared with 5+ criteria, nuanced discussion | 3 materials with 3 criteria | 2 materials compared | Incomplete |
| Stress shielding analysis | Correct stiffness ratio calculation, clear Frost's theory link, practical design solution | Good analysis with minor gaps | Superficial treatment | Missing or incorrect |
| Writing quality | Clear, professional, proper units, diagrams included | Well-organized with minor clarity issues | Acceptable with some disorganization | Poorly organized |
| Python code | Clean, commented, produces all figures | Functional with minor issues | Partial functionality | Not submitted or non-functional |

---

## Deliverable 2: Viscoelastic Model Fitting to Experimental Data

### Type
**Computational Analysis Report** (1000–1500 words + Python notebook)

### Requirements

A uniaxial tensile test on a bovine tendon specimen was performed. The data below (simulated from literature) represents stress relaxation under 5% strain held for 300 seconds.

**Experimental Data (stress relaxation, σ MPa at given time t s)**:

| t (s) | σ (MPa) |
|-------|---------|
| 0 | 50.0 |
| 1 | 42.0 |
| 5 | 35.0 |
| 10 | 30.5 |
| 30 | 24.0 |
| 60 | 21.0 |
| 100 | 19.5 |
| 200 | 18.2 |
| 300 | 17.8 |

**Tasks**:

1. Fit THREE models to the data:
   - Maxwell model (σ(t) = σ₀·e^(-E₁t/η₁))
   - Kelvin-Voigt model (σ(t) = E·ε₀ - (E·ε₀ - σ₀)·(1 - e^(-t/τ)))
   - Standard Linear Solid (SLS): σ(t) = σ_∞ + (σ₀ - σ_∞)·e^(-t/τ)

2. For each model, compute:
   - Parameters via least-squares fitting
   - R² goodness of fit
   - RMSE (root mean square error)
   - Residual analysis

3. Use the best-fit SLS parameters to predict:
   - Relaxation time τ
   - Equilibrium stress σ_∞
   - Initial stress σ₀
   - Discuss whether tendon behaves more like a Maxwell or SLS material

4. **BONUS**: Fit Fung's QLV theory relaxation function $G(t) = \frac{1 + (t/t_1)}{1 + c(t/t_1)}$ and compare

### Rubric

| Criterion | Points |
|-----------|--------|
| Correct model implementations (3 models) | 30 pts |
| Least-squares fitting with proper optimization | 20 pts |
| R² and RMSE computed correctly | 15 pts |
| Residual analysis performed | 10 pts |
| Interpretation of results | 15 pts |
| Code quality (comments, structure, figures) | 10 pts |
| **Total** | **100 pts** |

---

## Deliverable 3: Weekly Reflection Log

### Type
**Reflection Journal** (300-500 words)

### Prompts

1. Which concept from Week 13 was most surprising or counterintuitive? Why?
2. How does your civil engineering background (statics, structural analysis) connect to what you learned this week? Give one specific example.
3. What is the most significant clinical/engineering challenge where Week 13 biomechanics knowledge would be essential? Describe it in one paragraph.
4. Rate your confidence on a scale of 1-10 for each of the 5 core mental models. For any rating < 7, identify what additional study you need.

### Sample Entry

> "The most counterintuitive concept was **stress shielding in hip prostheses**. I always thought 'stronger is better' for implants, but the mismatch between Ti alloy (E=110 GPa) and cortical bone (E=18 GPa) actually causes bone resorption — exactly the opposite of the intended outcome. This directly parallels structural engineering: a very stiff steel beam in a flexible concrete frame would similarly take all the load and leave the concrete understressed, leading to cracking and failure. The Frost mechanostat threshold (800 με) maps to the 'effective stress' concept in structural engineering where we design to keep stresses within serviceability limits."
