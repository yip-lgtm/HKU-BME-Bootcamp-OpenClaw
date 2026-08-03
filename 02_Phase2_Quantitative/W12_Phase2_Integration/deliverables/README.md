# Week 12 Deliverables — Phase 2 Integration Portfolio

## Deliverable 1: Phase 2 Portfolio Project

- **Type**: Capstone Project (Comprehensive)
- **Deadline**: End of Week 12 (Sunday, 23:59 HKT)
- **Submission**: PDF report + .ipynb code

### Overview
The Phase 2 Portfolio is the capstone project that integrates all three domains covered in Phase 2:
- **Signal Processing** (Weeks 7-9)
- **Statistical Analysis** (Week 10)
- **Physiological Modeling** (Week 11)

You will build a complete biomedical analysis pipeline for one of the following clinical applications:

| Option | Application | Key Integration |
|--------|-------------|-----------------|
| A | Sleep Stage Classification (EEG) | FFT/STFT + Statistics |
| B | Drug Effect Monitoring (PK/PD) | ODE modeling + Statistics |
| C | Arrhythmia Detection (ECG) | Filtering + HRV statistics + PK/PD |
| D | Neural Spike Sorting (MEA) | Detection + Clustering + HH model |

### Required Components

**Component 1: Signal Processing Pipeline (25 marks)**
- Generate or load a biomedical signal dataset (minimum 20 recordings)
- Apply appropriate preprocessing: filtering (FIR/IIR), artifact removal
- Extract features in both time domain and frequency domain
- Validate preprocessing quality with before/after comparisons
- Document filter specifications (type, order, cutoff frequencies)

**Component 2: Statistical Validation (25 marks)**
- State formal hypotheses for your clinical question
- Perform appropriate statistical tests (at least 2 different tests)
- Report: test statistics, p-values, confidence intervals, effect sizes
- Discuss clinical significance vs statistical significance
- Address multiple comparisons if applicable

**Component 3: Physiological Model (25 marks)**
- Build a mechanistic model relevant to your application
- Fit model to data OR use model to make predictions
- Perform sensitivity analysis (vary 1-2 key parameters ±50%)
- Interpret model parameters in clinical/biological terms
- Discuss model limitations

**Component 4: Comprehensive Report (15 marks)**
- 8-12 page written report (excluding code)
- Structure: Introduction, Methods, Results, Discussion, Conclusion
- Include at least 5 high-quality figures
- Proper scientific writing style
- References to relevant literature

**Component 5: Code Quality (10 marks)**
- Clean, documented, reproducible code
- Functions with clear docstrings
- Random seeds set for reproducibility
- Clear visualization with proper labels

### Rubric

- [ ] Component 1 (Signal Processing): Pipeline complete and well-documented (25 pts)
- [ ] Component 2 (Statistics): Hypotheses, tests, CIs, effect sizes (25 pts)
- [ ] Component 3 (Modeling): ODE model with parameter analysis (25 pts)
- [ ] Component 4 (Report): Scientific writing, figures, interpretation (15 pts)
- [ ] Component 5 (Code): Clean, reproducible, documented (10 pts)

**Total: 100 marks**

---

## Deliverable 2: Phase 2 Knowledge Synthesis

- **Type**: Written Synthesis (3-4 pages)
- **Deadline**: End of Week 12

### Task
Write a synthesis essay addressing the following prompt:

**"How do signal processing, statistical analysis, and physiological modeling complement each other in biomedical engineering? Provide specific examples from Phase 2 content."**

### Structure

1. **Introduction** (1 paragraph): Define the three domains and their roles in BME
2. **Domain Roles** (2-3 paragraphs): Explain what each domain contributes
3. **Integration Examples** (2-3 paragraphs): Specific Phase 2 examples of integration
4. **Challenges & Limitations** (1-2 paragraphs): When does integration fail?
5. **Conclusion** (1 paragraph): Future directions

### Rubric

- [ ] Clear explanation of all three domains (10 pts)
- [ ] At least 3 specific Phase 2 examples (15 pts)
- [ ] Discussion of integration benefits (10 pts)
- [ ] Critical analysis of limitations (10 pts)
- [ ] Quality of writing (5 pts)

**Total: 50 marks**

---

## Deliverable 3: Phase 2 Review Presentation

- **Type**: Presentation (10-12 slides)
- **Deadline**: End of Week 12

### Task
Create a presentation summarizing Phase 2 for a multidisciplinary audience (clinicians, engineers, and life scientists).

### Slide Requirements

1. **Title slide** — Phase 2 overview
2. **Biomedical Signals** — Why ECG/EEG matter in healthcare
3. **Signal Processing Pipeline** — Filtering for clinical data
4. **Statistics in Medicine** — How clinical trials work
5. **Physiological Models** — From cells to whole body
6. **Integration Demo** — One key example from your project
7. **Clinical Impact** — Real-world applications
8. **Challenges** — Limitations and open problems
9. **Summary** — Key takeaways

### Rubric

- [ ] Content accuracy (10 pts)
- [ ] Clarity for multidisciplinary audience (10 pts)
- [ ] Visual quality (10 pts)
- [ ] Integration of all three domains (10 pts)
- [ ] Clinical relevance emphasized (10 pts)

**Total: 50 marks**

---

## Grading Summary

| Deliverable | Weight | Deadline |
|-------------|--------|----------|
| Portfolio Project | 60% | Week 12 |
| Knowledge Synthesis | 25% | Week 12 |
| Presentation | 15% | Week 12 |

**Total Phase 2 Grade: 100 marks**

---

## Phase 2 Learning Outcomes

Upon completing Phase 2, you should be able to:

| # | Learning Outcome | Evidence |
|---|-----------------|----------|
| 1 | Generate and analyze biomedical signals | ECG/EEG generation code |
| 2 | Design FIR/IIR filters for clinical applications | Filter design and ECG denoising |
| 3 | Perform statistical hypothesis tests on clinical data | t-tests, ANOVA, regression |
| 4 | Build ODE models of physiological systems | PK/PD model, HH model |
| 5 | Integrate all three domains in one pipeline | Capstone project |
| 6 | Present technical results to multidisciplinary audiences | Phase 2 presentation |

**Maintainer**: BME Bootcamp Agent | **Week 12** | **Phase 2 Integration**
