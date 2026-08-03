# Week 5 Deliverables — Homeostasis & Excitable Tissues

## Overview
本周交付物旨在整合穩態生理學的理論知識與實際應用，包括動作電位模擬報告和血糖調節案例研究。

## Deliverable 1: Action Potential Simulation Report

### Description
撰寫一份完整的 Hodgkin-Huxley 動作電位模擬實驗報告。

### Report Structure

```
1. Abstract (摘要)
   - Key findings and significance

2. Introduction (引言)
   - Historical background (Hodgkin & Huxley, 1939-1952)
   - Objectives of the simulation

3. Methods (方法)
   - HH model equations
   - Simulation parameters
   - Numerical methods

4. Results (結果)
   - Action potential waveform
   - Gating variable dynamics
   - Ionic currents
   - Refractory periods

5. Discussion (討論)
   - Comparison with experimental data
   - Model limitations
   - Clinical applications

6. Conclusion (結論)

7. References (參考文獻)
```

### Technical Requirements
- Length: 2000-3000 words
- Figures: At least 4 (from simulation output)
- References: Minimum 5 peer-reviewed sources

### Submission Format
- PDF: `W05_AP_simulation_report.pdf`

---

## Deliverable 2: Homeostasis Case Study — Blood Glucose Regulation

### Case Study: Type 2 Diabetes Mellitus

**Patient Presentation**:
Mr. Chen, 58-year-old male
- Chief complaint: Fatigue, increased thirst, frequent urination for 2 months
- PMH: Hypertension (controlled), obesity (BMI 32)
- Family history: Father had Type 2 DM
- Fasting blood glucose: 180 mg/dL (normal < 100 mg/dL)
- HbA1c: 8.5% (normal < 5.7%)

### Questions

#### Question 1: Normal Glucose Homeostasis (15 points)
Explain how normal blood glucose homeostasis is maintained, including:
- The role of insulin and glucagon
- Target tissues (liver, muscle, adipose)
- Key enzymes regulated
- Feedback mechanisms

#### Question 2: Pathophysiology of Type 2 DM (20 points)
Using the concept of homeostasis, explain what goes wrong in Type 2 DM:
- Insulin resistance vs. β-cell dysfunction
- Which part of the negative feedback loop is impaired?
- Why does hyperglycemia occur despite high insulin levels?

#### Question 3: Treatment Rationale (25 points)
The physician prescribes metformin 500 mg twice daily. Explain:
- Mechanism of action of metformin
- How this addresses the pathophysiology
- Why lifestyle modification (diet + exercise) is also essential
- Expected HbA1c reduction with treatment

#### Question 4: Complication Risk Assessment (20 points)
Mr. Chen is at risk for both microvascular and macrovascular complications.
- List and explain the major complications
- Which homeostatic systems are affected?
- What monitoring is recommended?

#### Question 5: Integration Essay (20 points)
Connect Week 4 (cell biology) and Week 5 (homeostasis) concepts:
- How does hyperglycemia affect cellular function?
- What happens to membrane transport in diabetes?
- How does this relate to the clinical presentation?

### Answer Key

**Q1: Normal Glucose Homeostasis**
```
Key Points:
1. Insulin (β cells) - anabolic hormone
   - ↑ GLUT4 translocation → ↑ glucose uptake
   - ↑ Glycogen synthesis (liver, muscle)
   - ↓ Gluconeogenesis (liver)
   - ↑ Lipogenesis (adipose)

2. Glucagon (α cells) - catabolic hormone
   - ↑ Glycogenolysis (liver)
   - ↑ Gluconeogenesis (liver)
   - ↓ Glycogen synthesis

3. Set point: ~5 mM (90 mg/dL) fasting
4. Normal range: 4.4-6.1 mM (80-110 mg/dL)
```

**Q2: Pathophysiology of Type 2 DM**
```
Normal feedback loop:
Stimulus → ↑ Glucose → β cells → ↑ Insulin → ↓ Glucose → Negative feedback

Type 2 DM:
1. Insulin resistance (primary defect)
   - ↓ Insulin receptor signaling
   - ↓ GLUT4 translocation
   - ↓ Glucose uptake despite normal insulin

2. Compensatory hyperinsulinemia
   - β cells try to overcome resistance
   - ↑ Insulin secretion

3. β-cell exhaustion (secondary defect)
   - ↓ Insulin production over time
   - β-cell apoptosis

4. Result: Hyperglycemia despite high insulin
```

**Q3: Treatment Rationale**
```
Metformin mechanism:
1. Activates AMP-kinase (AMPK)
2. ↓ Hepatic gluconeogenesis (major effect)
3. ↑ Insulin sensitivity (peripheral)
4. ↓ Intestinal glucose absorption (minor)
5. Weight neutral or slight loss

Expected HbA1c reduction: 1-1.5%
(From 8.5% → ~7-7.5%)

Lifestyle modification:
- 5-7% weight loss → 0.5-1% HbA1c reduction
- ↑ Physical activity → ↑ GLUT4 expression
- ↑ Insulin sensitivity
```

---

## Deliverable 3: Negative Feedback Loop Diagram

### Description
Create a visual diagram showing the negative feedback loop for ONE of the following systems:
1. Blood glucose regulation
2. Body temperature regulation
3. Blood pressure regulation
4. Thyroid hormone regulation

### Requirements
- Format: Digital (Miro, PowerPoint, or hand-drawn scanned)
- Must include all components:
  - Stimulus
  - Receptor/Sensor
  - Integrating center
  - Effector
  - Response
  - Negative feedback arrow
- Include specific values (set points, normal ranges)
- Color code different components
- Add BME clinical application

### Evaluation Criteria

| Component | Points | Description |
|-----------|--------|-------------|
| Completeness | 20 | All 6 components present |
| Accuracy | 30 | Scientifically correct |
| Clarity | 20 | Clear visual hierarchy |
| Clinical relevance | 15 | Includes BME application |
| Creativity | 15 | Visual appeal and organization |

---

## Submission Checklist

- [ ] Action Potential Simulation Report (PDF)
- [ ] Case Study Answers (all 5 questions)
- [ ] Negative Feedback Loop Diagram (PDF or image)
- [ ] References in APA format
- [ ] All files named correctly

---

## Grading Rubric (Total: 100 points)

| Deliverable | Points |
|-------------|--------|
| AP Simulation Report | 35 |
| Case Study | 40 |
| Feedback Loop Diagram | 25 |

---

## BME Real-World Connections

本周作業與以下 BME 應用相關：

### Medical Devices
- **胰島素泵**: 連續皮下胰島素輸注 (CSII)
- **連續血糖監測 (CGM)**: 組織間液葡萄糖感測器
- **心律調節器**: 電子心臟節律管理

### Computational Biology
- **生理系統建模**: Ordinary differential equation models
- **藥物動力學模擬**: PK/PD modeling
- **系統生物學**: Network analysis of physiological systems

### Clinical Engineering
- **遠程監測**: 家庭血壓、血糖監測
- **人工胰臟**: 閉環控制系統
- **康復工程**: 生物反饋訓練

---

*BMED2302 Week 5 Deliverables | HKU BME Bootcamp | 2026-07*
