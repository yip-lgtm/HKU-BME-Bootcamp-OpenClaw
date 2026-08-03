# Week 4 Deliverables — Cell Biology & Tissue Structure

## Overview
本周交付物旨在整合細胞生物學的理論知識與實際應用，包括概念圖建構、問題集解答和模擬報告。

## Deliverable 1: Cell Biology Concept Map (細胞生物學概念圖)

### Description
創建一個全面的細胞生物學概念圖，展示細胞膜結構與胞內胞器之間的結構-功能關係。

### Requirements
1. **Main Concept**: Cell as a Functional Unit (細胞作為功能單位)
2. **Required Components**:
   - Fluid Mosaic Model (流動鑲嵌模型)
   - Membrane organelles and their functions
   - Membrane transport mechanisms
   - Cytoskeleton network
   - Cell signaling pathways
   - Energy production systems

### Technical Specifications
- Format: Digital (Miro, Lucidchart, or hand-drawn scanned)
- Minimum size: A3 or equivalent digital canvas
- Color coding:
  - 🔴 Red: Membrane structure components
  - 🔵 Blue: Transport mechanisms
  - 🟢 Green: Cytoskeleton elements
  - 🟡 Yellow: Signaling pathways
  - 🟣 Purple: Energy systems

### Evaluation Criteria (Rubric)

| Criterion | Excellent (A) | Good (B) | Satisfactory (C) | Needs Work (D) |
|-----------|---------------|----------|------------------|-----------------|
| Completeness | All 6 components included with detailed connections | 5 components with good connections | 4 components with basic connections | < 4 components |
| Accuracy | All scientific concepts are accurate | Minor errors (< 2) | Some errors (3-5) | Multiple significant errors |
| Visual Design | Clear hierarchy, professional appearance | Clear hierarchy, good appearance | Adequate organization | Poor organization |
| Integration | Shows clear structure-function relationships | Shows some relationships | Limited integration | No clear integration |
| BME Application | Strong clinical/BME relevance | Some relevance | Minimal relevance | No relevance |

### Submission Format
- PDF file: `W04_concept_map.pdf`
- Source file (optional): `.mm` (FreeMind), `.graphml` (yEd), or image format

---

## Deliverable 2: Problem Set — Membrane Transport Calculations

### Problem Set Overview
本問題集包含 5 道計算題和 2 道概念題，涵蓋細胞膜運輸的關鍵計算。

### Problem Set File
📄 **File**: `W04_problem_set.md`

### Problems

#### Problem 1: Fick's Law Calculation (20 points)
**Question**: Oxygen diffuses from the alveolus (PO₂ = 100 mmHg) into the capillary blood (PO₂ = 40 mmHg). The alveolar-capillary membrane is 0.5 μm thick, and the diffusion coefficient for O₂ in tissue is 2.0 × 10⁻⁹ m²/s.

**Tasks**:
a) Calculate the flux of O₂ (in mol/m²/s) (10 points)
b) If the capillary surface area is 70 m² and the transit time is 0.75 s, calculate the total O₂ transferred (10 points)

**Answer Key**:
```
a) J = -D × (dC/dx)
    Henry's Law: C = α × P
    α(O₂) = 1.3 × 10⁻⁶ mol/(L·mmHg)
    ΔC = (100 - 40) × 1.3 × 10⁻⁶ = 7.8 × 10⁻⁵ mol/L = 0.078 mol/m³
    dC/dx = 0.078 / (0.5 × 10⁻⁶) = 156,000 mol/m⁴
    J = 2.0 × 10⁻⁹ × 156,000 = 3.12 × 10⁻⁴ mol/(m²·s)

b) Total O₂ = J × A × t = 3.12 × 10⁻⁴ × 70 × 0.75 = 1.64 × 10⁻² mol
    = 16.4 mmol per pass through the capillary
```

#### Problem 2: Goldman Equation (25 points)
**Question**: Calculate the resting membrane potential using the Goldman equation with the following ion concentrations:

| Ion | [Inside] (mM) | [Outside] (mM) | Permeability (m/s) |
|-----|---------------|----------------|-------------------|
| K⁺ | 150 | 5 | 5 × 10⁻⁶ |
| Na⁺ | 15 | 150 | 5 × 10⁻⁸ |
| Cl⁻ | 10 | 120 | 1 × 10⁻⁶ |

T = 37°C (310 K)

**Tasks**:
a) Calculate the Goldman voltage (15 points)
b) Calculate the equilibrium potential for each ion using Nernst equation (10 points)

#### Problem 3: Na⁺/K⁺-ATPase Energy Analysis (20 points)
**Question**: Calculate the total energy required for one complete cycle of the Na⁺/K⁺-ATPase.

Given:
- Membrane potential: -70 mV
- [Na⁺]in = 15 mM, [Na⁺]out = 150 mM
- [K⁺]in = 150 mM, [K⁺]out = 5 mM
- Stoichiometry: 3 Na⁺ out, 2 K⁺ in per ATP

**Tasks**:
a) Calculate the electrical work (5 points)
b) Calculate the chemical work (concentration gradients) (10 points)
c) Compare with ATP hydrolysis energy (ΔG ≈ -50 kJ/mol) (5 points)

#### Problem 4: Osmosis and Cell Volume (20 points)
**Question**: A human red blood cell (RBC) with initial volume of 90 fL is placed in:
a) Distilled water (0 mOsm/L)
b) 0.9% NaCl solution (300 mOsm/L)
c) 3% NaCl solution (1000 mOsm/L)

Initial intracellular osmolarity: 300 mOsm/L

**Tasks**:
For each solution, predict:
a) Initial osmotic pressure difference (5 points)
b) Water movement direction (5 points)
c) Final cell state (normal, hemolysis, or crenation) (10 points)

#### Problem 5: Membrane Permeability Analysis (15 points)
**Question**: The table shows permeability coefficients for different molecules across a lipid bilayer:

| Molecule | log P | Permeability (cm/s) |
|----------|-------|-------------------|
| Water | 0.45 | 3 × 10⁻⁴ |
| O₂ | 0.73 | 4 × 10⁻⁴ |
| CO₂ | 0.83 | 5 × 10⁻⁴ |
| Glucose | -3.0 | 1 × 10⁻⁹ |

**Tasks**:
a) Explain the relationship between log P and permeability (5 points)
b) Why is glucose permeability so low despite being a small molecule? (5 points)
c) What is the optimal log P range for drug molecules? (5 points)

#### Problem 6: Receptor-Mediated Endocytosis (Bonus 10 points)
**Question**: Describe the pathway of LDL (low-density lipoprotein) from synthesis to degradation, including:
- Site of synthesis
- Receptor involved
- Clathrin-coated pit formation
- Endosome maturation
- Lysosomal degradation

---

## Deliverable 3: Membrane Diffusion Simulation Report

### Description
撰寫一份實驗室報告，描述並解釋你運行的膜擴散模擬結果。

### Report Structure

```
1. Abstract (摘要)
   - 200 words maximum
   - Key findings summary

2. Introduction (引言)
   - Background on Fick's Law
   - Objectives of the simulation

3. Methods (方法)
   - Simulation parameters
   - Numerical methods used

4. Results (結果)
   - Concentration profiles
   - Flux calculations
   - Permeability comparisons

5. Discussion (討論)
   - Interpretation of results
   - Limitations of the model
   - BME applications

6. Conclusion (結論)
   - Key takeaways

7. References (參考文獻)
   - Minimum 3 peer-reviewed references
```

### Technical Requirements
- Length: 1500-2500 words
- Figures: At least 2 (from simulation output)
- References: APA format
- Font: 12pt Times New Roman or Arial

### Submission Format
- Word document: `W04_simulation_report.docx` or
- PDF file: `W04_simulation_report.pdf`

---

## Submission Checklist

Before submitting, verify:

- [ ] Concept map with all 6 required components
- [ ] Concept map shows structure-function relationships
- [ ] Concept map includes BME applications
- [ ] Problem Set completed with all calculations shown
- [ ] Problem Set answers verified
- [ ] Simulation report with all sections
- [ ] Report includes at least 2 figures from simulation
- [ ] References formatted in APA style
- [ ] All files named correctly
- [ ] Content is original and properly attributed

---

## Grading Rubric (Total: 100 points)

| Deliverable | Points | Weight |
|-------------|--------|--------|
| Concept Map | 40 | 40% |
| Problem Set | 35 | 35% |
| Simulation Report | 25 | 25% |

---

## Academic Integrity Notice

本作業應為獨立完成。如有引用他人的內容或想法，請務必注明出處。

**引用格式 (APA 7th Edition)**:
- In-text: (Author, Year)
- Reference list: Author, A. A. (Year). Title. *Journal*, Volume(Issue), pages. DOI

---

## Extensions and Late Submissions

- **Extension requests**: Submit via email 48 hours before deadline
- **Late penalty**: 10% deduction per day (maximum 5 days)
- **Emergency**: Contact instructor immediately

---

## BME Real-World Connection

本周作業的內容直接應用於：
- **藥物傳遞系統設計**: 理解藥物如何穿過細胞膜
- **生物感測器開發**: 膜通透性測量原理
- **組織工程**: 細胞在不同滲透壓環境下的行為
- **神經工程**: Hodgkin-Huxley模型是神經假肢的基礎

---

*BMED2302 Week 4 Deliverables | HKU BME Bootcamp | 2026-07*
