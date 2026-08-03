# Week 19 Deliverables — Tissue Engineering (BMED4604)

> **Due**: End of Week 19 weekend (Sunday midnight)  
> **Submission**: Push to `04_Phase4_Advanced_Research/W19_TissueEng_BMED4604/deliverables/`

---

## Deliverable 1: Tissue Engineering Research Proposal — Bone Defect Repair

### Type
**Research Proposal** (3000-4000 words, PDF + Python code)

### Scenario

You are a postdoc in the Langer Lab (MIT) proposing a tissue engineering strategy for critical-sized femoral bone defects in elderly osteoporotic patients. Your committee has asked you to develop a comprehensive proposal addressing all aspects of the tissue engineering triad.

### Required Sections

**1. Abstract** (200 words)

**2. Specific Aims** (300 words)
- Aim 1: Design and fabricate 3D-printed PLGA/β-TCP composite scaffold with optimized porosity (70%) and pore size (300-400 μm)
- Aim 2: Optimize MSC seeding density and BMP-2 delivery for osteogenic differentiation
- Aim 3: Validate in ovariectomized rat critical-sized defect model at 12 weeks

**3. Background and Significance** (800 words)
- Critical-sized bone defects: definition (cannot heal spontaneously > 2 cm in humans)
- Osteoporosis: 200 million patients worldwide, 8.9 million fractures/year
- Current treatments: autograft (gold standard but limited supply, morbidity), allograft (immune risk), BMP-2 (high dose, side effects)
- Tissue engineering as alternative: address all three limitations

**4. Research Design and Methods** (1500 words)

*4a. Scaffold Design*:
- Material: PLGA 50:50 / β-TCP 70:30 composite
- Fabrication: 3D printing (Extrusion-based, 300 μm nozzle, 70% porosity)
- Characterize: SEM, micro-CT, compressive mechanical testing
- **Calculate**: Scaffold stiffness if ρ_scaffold = 0.65 g/cm³, ρ_material = 1.4 g/cm³ (PLGA), E_material = 1 GPa

*4b. Cell Source and Expansion*:
- Autologous bone marrow MSC (patient-matched)
- Isolation: Ficoll density gradient, 1.077 g/mL
- Expansion: α-MEM + 10% FBS + FGF-2 (5 ng/mL)
- Target: 10 × 10⁶ cells per scaffold (seeding density: 1 × 10⁶ cells/mL)
- **Calculate**: Starting from 5 mL bone marrow aspirate (~50,000 CFU-F), expand to passage 3 for 10×10⁶ cells. How many doublings needed?

*4c. Growth Factor Delivery*:
- BMP-2 dose: 5 μg/mL (lower than clinical 1.5 mg/mL)
- Delivery system: Heparin-functionalized PLGA microspheres (200-300 μm)
- Release profile: 30% burst (day 1), sustained release ~0.5 μg/day (days 2-21)
- **Model**: Calculate total BMP-2 released over 21 days using biphasic kinetics (M_fast = 0.3, M_slow = 0.65, k_fast = 0.5 day⁻¹, k_slow = 0.05 day⁻¹)

*4d. Bioreactor Culture*:
- Perfusion bioreactor: Q = 1-2 μL/min
- Target wall shear stress: 0.05-0.1 Pa
- Duration: 14 days dynamic culture
- Mechanical stimulation: 5% cyclic compression, 1 Hz, 1 hr/day
- **Calculate**: Required flow rate for τ_w = 0.08 Pa in 300 μm pore radius

*4e. In Vivo Validation*:
- Ovx rat model (8-week-old, 200 g)
- Critical-sized defect: 5 mm femoral condyle
- Groups: (1) Empty defect (negative control), (2) Scaffold only, (3) Scaffold + MSC, (4) Scaffold + MSC + BMP-2, (5) Autograft (positive control)
- Endpoint: 12 weeks
- Outcomes: Radiography, μCT (BV/TV), biomechanical testing (3-point bend)

**5. Expected Outcomes** (300 words)

**6. Limitations and Alternative Approaches** (200 words)

**7. Budget Justification** (200 words)

### Calculations Required (show all work)

1. **Scaffold porosity**: ε = 1 - (ρ_scaffold/ρ_material) = ?
2. **Scaffold modulus**: E_scaffold = E_material × (1 - ε) = ?
3. **MSC expansion**: Days needed for 50,000 → 10,000,000 cells at T_d = 36 hr
4. **BMP-2 cumulative release**: M(21) = M_fast(1-e^(-0.5×21)) + M_slow(1-e^(-0.05×21)) = ?
5. **Perfusion flow rate**: Q for τ_w = 0.08 Pa, r = 300 μm, μ = 1 mPa·s

### Rubric

| Criterion | Points | Description |
|-----------|--------|-------------|
| Scientific rigor | 30 | All calculations correct, assumptions stated |
| Scaffold design rationale | 20 | Evidence-based choices, referenced literature |
| Trilineage integration | 15 | MSC biology addressed comprehensively |
| Growth factor kinetics | 15 | Quantitative model with real parameters |
| Bioreactor engineering | 10 | Mechanical stimulation justified |
| Clinical relevance | 10 | Clear pathway to osteoporotic fracture |
| **Total** | **100** | |

---

## Deliverable 2: Scaffold Design Optimization Report

### Type
**Technical Report with Python Analysis** (1500 words + code)

### Task

Design an optimal scaffold for osteochondral interface engineering (bone-cartilage interface). The interface is 2-3 mm thick and connects subchondral bone (E = 1-5 GPa) to articular cartilage (E = 0.01-0.1 MPa) — a 10,000× stiffness gradient.

### Requirements

1. **Design criteria**:
   - Cartilage side: E = 0.1-1 MPa, porosity > 70%, pore size 20-100 μm
   - Bone side: E = 1-5 GPa, porosity 60-70%, pore size 200-500 μm
   - Interface zone: continuous gradient over 2 mm

2. **Material system**:
   - Base: PEGDA (photopolymerizable, tunable mechanical properties)
   - Ceramic reinforcement: HA nanoparticles (20-50 nm)
   - Bioactive factors: TGF-β3 (cartilage side), BMP-2 (bone side)

3. **Analysis tasks** (Python required):
   - Calculate mechanical properties at different PEGDA/HA compositions
   - Model diffusion of TGF-β3 and BMP-2 in the graded scaffold
   - Simulate release profiles from the biphasic system
   - Optimize interface gradient (linear vs. exponential vs. step)

4. **Submit**: Python code (fully commented) + analysis plots + written interpretation

### Rubric

| Criterion | Points |
|-----------|--------|
| Mechanical gradient design | 25 |
| Diffusion modeling (both factors) | 25 |
| Release kinetics optimization | 25 |
| Code quality and reproducibility | 15 |
| Written interpretation | 10 |
| **Total** | **100** |

---

## Deliverable 3: Weekly Research Reflection

### Type
**Research Journal** (400-600 words)

### Prompts

1. Which tissue engineering challenge (vascularization, scale-up, immune response, regulatory) do you think is the most fundamental — and why? Be specific.

2. The Langer & Vacanti (1993) paper is 30+ years old. What predictions from that paper have come true? What has not? How has the field changed?

3. If you were writing a grant proposal to NIH, what would be your "innovation" statement for a bone tissue engineering project? (2-3 sentences)

4. Identify one specific paper from this week's reading that you would cite in your literature review. Write a 3-sentence annotation explaining why.

5. Rate your confidence (1-10) for each of the 5 core mental models. For any < 7, what do you need to study further?

### Sample Entry

> "The most fundamental challenge in tissue engineering is vascularization. Without blood vessels, any tissue thicker than ~200 μm will necrose in the center due to oxygen diffusion limits. I've read about pre-vascularization strategies (VEGF delivery, microfluidic networks, organ-on-a-chip), but the leap from lab to clinical is still enormous. Atala's 2009 review acknowledges that the first fully vascularized organ (a bladder, 2001) used a simple approach: the body's own tissue as a 'bioreactor' for the scaffold. This suggests that in situ tissue engineering — using the body as the bioreactor — may be more clinically viable than ex vivo approaches for complex organs."
