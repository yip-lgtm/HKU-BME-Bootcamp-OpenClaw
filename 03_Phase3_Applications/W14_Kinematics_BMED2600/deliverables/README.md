# Week 14 Deliverables — Kinematics, Gait, and Bone Fracture

> **Due**: End of Week 14 weekend  
> **Submission**: Push to `03_Phase3_Applications/W14_Kinematics_BMED2600/deliverables/`

---

## Deliverable 1: Motion Capture Data Analysis Report

### Type
**Technical Report** (1500-2000 words + Python notebook)

### Scenario

You are a BME clinical researcher analyzing gait data from a patient with unilateral knee osteoarthritis (OA) compared to an age-matched healthy control.

### Data (synthetic, based on literature)

**Healthy control**:
- Walking speed: 1.40 m/s
- Cadence: 110 steps/min
- Knee flexion during stance (max): 5°
- Knee flexion during swing (max): 65°
- Peak knee extension moment: 45 N·m
- Double support time: 0.12 s

**Knee OA patient**:
- Walking speed: 1.05 m/s (reduced)
- Cadence: 95 steps/min
- Knee flexion during stance (max): 8° (stiffer, less flexion)
- Knee flexion during swing (max): 52° (reduced swing flexion)
- Peak knee extension moment: 38 N·m (reduced — joint degeneration)
- Double support time: 0.18 s (increased — instability)

### Tasks

1. **[40 pts] Kinematic comparison**: Plot and compare knee angle-time curves for both subjects across the full gait cycle (0-100%). Comment on the differences.

2. **[30 pts] Kinetic analysis**: Compute and compare peak joint moments for hip, knee, and ankle. Explain why the OA patient has reduced knee extension moment (hint: consider quadriceps weakness and pain avoidance).

3. **[20 pts] Clinical interpretation**: Using the six determinants of gait (Saunders), explain which determinants are likely affected in knee OA and how they contribute to the observed gait deviations.

4. **[10 pts] Treatment recommendations**: Propose two orthotic or surgical interventions with biomechanical justification.

### Rubric

| Criterion | Excellent | Good | Satisfactory |
|-----------|-----------|------|-------------|
| Kinematic comparison | Clear plots, proper axis labels, quantitative comparison | Good plots, minor analysis gaps | Basic plots, limited interpretation |
| Kinetic analysis | All moments computed correctly, proper units | Correct with minor errors | Partial analysis |
| Clinical interpretation | All 6 determinants discussed, linked to OA | 4-5 determinants discussed | Limited connection |
| Recommendations | Two interventions with biomechanical rationale | One well-justified intervention | Generic recommendations |

---

## Deliverable 2: Bone Fracture Risk Assessment Report

### Type
**Engineering Analysis Report** (1000-1500 words)

### Scenario

A 65-year-old woman (weight 60 kg) sustains a fall from standing height. She has osteoporosis (BMD T-score = -3.2 at the femoral neck). You must assess fracture risk at three sites: vertebral body (L1), femoral neck, and tibial shaft.

### Data
- L1 vertebral body: cross-sectional area = 11 cm², height = 2.4 cm
- Femoral neck: cross-sectional area = 3.0 cm², neck-shaft angle = 125°
- Tibial shaft: diameter = 2.4 cm, wall thickness = 0.3 cm

### Tasks

1. **[30 pts] L1 compression fracture risk**: Estimate compressive stress during standing. Compare with vertebral body ultimate strength (~100 MPa compressive for osteoporotic bone).

2. **[30 pts] Femoral neck fracture risk**: Calculate tensile stress at the superior (lateral) femoral neck during single-leg stance. Use the femoral neck cross-section. Compare with tensile strength (σ_ult_tension ≈ 100 MPa for osteoporotic bone).

3. **[20 pts] Tibial shaft bending**: If the fall creates a 3-point bending moment M = 100 N·m at the tibial shaft, calculate the bending stress and compare with tibial bending strength.

4. **[20 pts] Clinical recommendations**: Based on your analysis, rank the three sites by fracture risk and suggest prophylactic interventions (pharmacological, surgical, lifestyle).

### Rubric

| Criterion | Pts | Notes |
|-----------|-----|-------|
| Accurate stress calculations | 30 | All three sites, proper units |
| Correct fracture criterion application | 20 | Compare to appropriate strength values |
| Risk ranking with justification | 20 | Clear ranking + clinical reasoning |
| Treatment recommendations | 20 | Match analysis findings |
| Report quality | 10 | Organization, clarity, figures |

---

## Deliverable 3: Weekly Reflection Log

### Type
**Reflection Journal** (300-500 words)

### Prompts

1. What surprised you most about the complexity of normal walking gait? How does this inform your understanding of gait pathology?
2. How does inverse dynamics (computing forces from motion) differ from structural engineering analysis (computing motion from forces)? Which direction is more commonly used in BME and why?
3. Rate your confidence (1-10) on: (a) computing joint angles from marker data, (b) performing inverse dynamics, (c) analyzing GRF patterns. For any <7, what needs more practice?
