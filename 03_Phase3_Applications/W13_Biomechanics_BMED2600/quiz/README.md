# Week 13 Self-Quiz — Biomechanics (BMED2600)

> **Format**: Self-assessment | **Time**: ~45 minutes  
> **Instructions**: Attempt all questions. Check answers against solution key at bottom.

---

## Part I: Multiple Choice Questions (10 × 3 pts = 30 pts)

**Q1.** A bone specimen with diameter d = 6 mm is loaded in tension with F = 8,482 N. What is the tensile stress σ?
- (A) 50 MPa  
- (B) 150 MPa  
- (C) 200 MPa  
- (D) 300 MPa  
- (E) 450 MPa

---

**Q2.** Which of the following best describes the relationship between cortical bone Young's modulus and trabecular bone apparent modulus?
- (A) E_cortical ≈ E_trabecular (they are similar)  
- (B) E_cortical ≈ 10× E_trabecular  
- (C) E_cortical ≈ 20× E_trabecular  
- (D) E_cortical ≈ 100× E_trabecular  
- (E) E_trabecular > E_cortical

---

**Q3.** In a Kelvin-Voigt viscoelastic model under constant stress σ₀, the strain:
- (A) Increases linearly without bound (Maxwell behavior)  
- (B) Approaches an asymptotic value σ₀/E  
- (C) Instantly jumps to σ₀/E  
- (D) Decreases exponentially  
- (E) Remains constant

---

**Q4.** Which model predicts stress relaxation under constant strain?
- (A) Kelvin-Voigt only  
- (B) Maxwell only  
- (C) Both Maxwell and Kelvin-Voigt  
- (D) Neither  
- (E) Standard Linear Solid only

---

**Q5.** In Fung's quasi-linear viscoelastic (QLV) theory, the stress at time t is expressed as:
- (A) σ = E · ε (elastic only)  
- (B) σ(t) = G(t) · σ^e(ε) (separable elastic × relaxation)  
- (C) σ(t) = σ₀ · e^(-t/τ) (Maxwell form)  
- (D) σ(t) = σ₀ · t/η (linear viscous)  
- (E) σ(t) = ∫ K(t-τ) · dε/dτ dτ (convolution integral)

---

**Q6.** The von Mises stress criterion is appropriate for predicting yielding in:
- (A) Brittle materials (e.g., ceramics)  
- (B) Ductile materials (e.g., metals, cortical bone at high strain rates)  
- (C) Viscoelastic materials only  
- (D) Composite materials only  
- (E) All materials equally

---

**Q7.** The Poisson's ratio of cortical bone is approximately:
- (A) 0.05  
- (B) 0.15  
- (C) 0.30  
- (D) 0.60  
- (E) 0.90

---

**Q8.** In the Gibson-Ashby model for trabecular bone, E ∝ ρ*². If apparent density doubles from 0.2 to 0.4 g/cm³, the apparent modulus:
- (A) Doubles (×2)  
- (B) Triples (×3)  
- (C) Quadruples (×4)  
- (D) Increases by √2  
- (E) Increases by 8×

---

**Q9.** In a femoral hip prosthesis, "stress shielding" refers to:
- (A) The bone absorbing too much stress from the implant  
- (B) The implant carrying load that would normally go to bone, causing bone resorption  
- (C) Stress concentration at the fracture site  
- (D) Elastic mismatch between implant and bone causing fatigue failure  
- (E) Shear stress at the bone-cement interface

---

**Q10.** The relaxation time τ = η/E in a Maxwell model has units of:
- (A) Pa·s  
- (B) Pa/s  
- (C) s (seconds)  
- (D) 1/s  
- (E) Dimensionless

---

## Part II: Short Answer Questions (5 × 6 pts = 30 pts)

**SQ1.** A cortical bone specimen (L₀ = 40 mm, diameter = 10 mm) is loaded to F = 18,850 N.
(a) Calculate the engineering stress σ (2 pts)
(b) Calculate the engineering strain ε if E = 18 GPa (2 pts)
(c) State whether the bone has yielded, given σ_yield = 120 MPa (2 pts)

---

**SQ2.** Compare and contrast the Maxwell and Kelvin-Voigt models of viscoelasticity in terms of:
(a) Their differential equations (1 pt each)
(b) Their predictions for instantaneous strain under step stress (1 pt each)
(c) Which material behavior each best models in the body (1 pt each)

---

**SQ3.** Define the "toe region" in tendon/ligament stress-strain behavior. What microstructural mechanism causes it? Name the approximate strain range and modulus in this region.

---

**SQ4.** A patient has a femoral neck crack a = 3 mm. Using fracture mechanics (K = Yσ√πa, with Y = 1.0, K_IC = 4 MPa√m), calculate the stress at which fracture occurs. If body weight creates σ = 50 MPa at the femoral neck, what is the safety factor?

---

**SQ5.** Explain Frost's mechanostat theory of bone remodeling. Include the four strain threshold zones and their associated remodeling responses (bone formation, resorption, quiescence).

---

## Part III: Problem-Solving Questions (2 × 20 pts = 40 pts)

**PQ1.** [20 pts] **Viscoelastic Model Fitting**

A Kelvin-Voigt element (E = 2 GPa, η = 10 GPa·s) is subjected to a step stress σ₀ = 20 MPa at t = 0.

(a) [3 pts] Calculate the relaxation time τ.
(b) [4 pts] Write the equation for strain ε(t) and calculate ε at t = 1s, 5s, and 10s.
(c) [4 pts] Calculate the strain rate dε/dt at t = 0 (instantaneous) and at t = 10s.
(d) [3 pts] If the specimen gauge length is L₀ = 25 mm, calculate the change in length ΔL at t = 10s.
(e) [3 pts] At what time does the strain reach 63.2% of its equilibrium value?
(f) [3 pts] Is this material more likely to model bone, tendon, or cartilage? Justify your answer using the magnitude of E.

---

**PQ2.** [20 pts] **Trabecular Bone Scaffold Design**

You are designing a 3D-printed trabecular bone scaffold for a critical-sized tibial defect (dimensions: 10 × 10 × 15 mm).

(a) [4 pts] Using the Gibson-Ashby model E = 3,390 × (ρ/1.9)² MPa, what apparent density should you target to match natural trabecular bone stiffness (E_target = 500 MPa)?

(b) [4 pts] If the scaffold porosity is P = 1 - (ρ/ρ_solid), what porosity does your target density correspond to? (ρ_solid = 1.9 g/cm³)

(c) [4 pts] Calculate the scaffold stiffness k = EA/L for your design (L = 15 mm, A = 100 mm²).

(d) [4 pts] The scaffold will be loaded with F = 500 N during early mobilization. Calculate the resulting stress σ and strain ε in your scaffold. Is it within the elastic limit?

(e) [4 pts] Name two additional mechanical properties (besides stiffness) that are critical for bone scaffold performance and explain why each matters clinically.

---

## Answer Key

### MCQ Answers
| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **D** (300 MPa) | A = π(3mm)² = 28.27 mm² = 2.827×10⁻⁵ m²; σ = 8482/2.827×10⁻⁵ = 300 MPa |
| 2 | **C** (≈20×) | E_cortical ≈ 18 GPa; E_trabecular ≈ 0.1-2 GPa; ratio 9-180×, most commonly cited ~20× |
| 3 | **B** | Kelvin-Voigt approaches σ₀/E asymptotically under constant stress |
| 4 | **B** | Maxwell shows exponential stress decay under constant strain; Kelvin-Voigt does not show true relaxation |
| 5 | **B** | QLV separates elastic and time-dependent components: σ(t) = G(t) × σ^e(ε) |
| 6 | **B** | Von Mises is a distortional energy theory for ductile materials |
| 7 | **C** | ν ≈ 0.28-0.35 for cortical bone |
| 8 | **C** (×4) | E ∝ ρ²; doubling ρ increases E by 2² = 4× |
| 9 | **B** | Stiffer implant bears load → bone understressed → resorption per Wolff's law |
| 10 | **C** | τ = η/E: (Pa·s)/(Pa) = s |

### Short Answer Solutions

**SQ1**: (a) A = π(5mm)² = 78.54 mm²; σ = 18,850/78.54 = 240 MPa. (b) ε = σ/E = 240/18,000 = 0.0133 = 1.33%. (c) Yes, σ=240 MPa > σ_yield=120 MPa → bone has yielded and likely fractured.

**SQ2**: See notes section — Maxwell: series spring+dashpot, instantaneous strain, good for stress relaxation. Kelvin-Voigt: parallel, no instantaneous strain, good for creep in polymers and soft tissues.

**SQ3**: Toe region: ε < 1-2%, collagen fiber uncrimping, E ~ 5-10 MPa (much lower than linear region).

**SQ4**: σ_f = K_IC/(Y√πa) = 4/(1×√(3.14×0.003)) = 4/0.097 = 41.2 MPa. Safety factor = 41.2/50 = 0.82 (<1.0 → HIGH RISK of fracture!).

**SQ5**: Frost mechanostat: ε > 1500 με → bone formation; 800-1500 με → quiescent; < 800 με → bone resorption.

### Problem Solutions

**PQ1**: (a) τ = η/E = 10 GPa·s / 2 GPa = 5 s. (b) ε(t) = (σ₀/E)(1-e^(-t/τ)). At 1s: 0.01×(1-e⁻·²)=0.0018; 5s: 0.01×(1-e⁻¹)=0.0063; 10s: 0.01×(1-e⁻²)=0.0086. (c) dε/dt = (σ₀/η)·e^(-t/τ); at t=0: 20/10000=0.002/s; at t=10s: 20/10000·e⁻²=0.00027/s. (d) ΔL = ε(10s)×25mm = 0.0086×25 = 0.215 mm. (e) t = τ = 5 s (63.2% of equilibrium = 1-e⁻¹). (f) E=2 GPa is high → closer to bone (E~18 GPa) than tendon (~0.5-1 GPa) → likely bone or bone-analogue material.

**PQ2**: (a) 500 = 3390×(ρ/1.9)² → ρ = 1.9×√(500/3390) = 1.9×0.384 = 0.73 g/cm³. (b) P = 1 - 0.73/1.9 = 1 - 0.384 = 0.616 = 61.6% porosity. (c) E=500 MPa; k = EA/L = 500 MPa×100mm²/15mm = 3333 N/mm. (d) σ = 500N/100mm² = 5 MPa; ε = 5/500 = 0.01 = 1%; within trabecular bone elastic range. (e) (1) Compressive strength — must not fail under body weight loading; (2) Fatigue resistance — cyclic loading during walking; (3) Permeability — cell migration and nutrient transport; (4) Degradation rate — matched to new bone formation rate.

---

## Self-Scoring Guide

| Score | Grade | Interpretation |
|-------|-------|----------------|
| 85-100 | A | Excellent — ready for Week 14 |
| 70-84 | B | Good — review topics with score < 6/10 |
| 55-69 | C | Satisfactory — revisit notes, redo problems |
| < 55 | D | Needs significant review — re-read all Week 13 materials |
