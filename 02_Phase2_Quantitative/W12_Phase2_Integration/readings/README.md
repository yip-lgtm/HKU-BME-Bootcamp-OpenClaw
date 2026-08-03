# Week 12 Readings — Phase 2 Integration + Diagnostic Tool Development

## Required Readings

### Integration Reading
- **Review all BMED2500 and BMED3603 core concepts** from Weeks 7-11
- **Integration Paper**: Compile signal processing, statistics, and modeling into one diagnostic framework

### Primary Textbooks
- **All Phase 2 textbooks** — Oppenheim, Rangayyan, Bland, Keenan, Cobelli
- **Rangayyan, R.M.** (2015) *Biomedical Signal Analysis*, 2nd ed. Wiley-IEEE Press — Chapters 7-10 (Advanced applications)

### HKU Course Materials
- **BMED2500** — Signals and Systems (Integration review)
- **BMED3603** — Biostatistics and Modeling (Integration review)
- **Capstone Preparation** — Engineering design process for medical devices

### Key Papers / Historical References
1. **Friesen, G.M. et al.** (1994) "A Comparison of the Noise Sensitivity of Nine QRS Detection Algorithms" — *IEEE Transactions on Biomedical Engineering*, 37(1), 85-98 — ECG algorithm comparison
2. **Thakor, N.V. et al.** (1984) "Reliable R-wave Detection from Ambulatory Subjects" — *Biomedical Sciences Instrumentation*, 20, 73-78 — Real-time ECG processing
3. **Webster, J.G.** (2010) *Medical Instrumentation: Application and Design*, 4th ed. Wiley — Chapter 15 (Diagnostic equipment)

### Online Resources
- PhysioNet: **Computing in Cardiology Challenge** — Annual ECG/EEG challenges
- PhysioNet: **MIMIC-III Database** — Critical care waveforms and clinical data
- Kaggle: **Medical Data** — ECG, EEG, and medical imaging datasets
- IEEE TBME: **Recent Diagnostic Algorithm Papers** — State-of-the-art methods

## Pre-Week Objectives (Week 12)
By the end of Week 12, students should be able to:
1. Integrate signal processing (filtering, FFT) with statistical analysis (hypothesis testing)
2. Combine mathematical modeling (ODEs) with experimental data fitting
3. Design a complete diagnostic tool pipeline: signal acquisition → preprocessing → feature extraction → classification → clinical output
4. Validate diagnostic algorithms against gold-standard clinical measures
5. Write a technical report following IEEE/ACM standards
6. Present results with appropriate statistical rigor and clinical relevance
7. Demonstrate Phase 2 competency through a comprehensive capstone project

## Integration Framework

### Complete Diagnostic Tool Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    DIAGNOSTIC TOOL PIPELINE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────┐  │
│  │ Signal       │───▶│ Preprocessing│───▶│ Feature          │  │
│  │ Acquisition  │    │ (Filtering)  │    │ Extraction       │  │
│  └──────────────┘    └──────────────┘    └──────────────────┘  │
│                                                  │              │
│  ┌──────────────┐    ┌──────────────┐            │              │
│  │ Clinical    │◀───│ Classification│◀──────────┘              │
│  │ Decision    │    │ & Prediction │                           │
│  └──────────────┘    └──────────────┘                           │
│                                                                  │
│  All stages validated with statistical analysis (Week 10)        │
│  All dynamics modeled with ODE analysis (Week 11)               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Example: ECG-Based Cardiac Abnormality Detection

| Pipeline Stage | Method | Reference Week |
|----------------|--------|----------------|
| Acquisition | 500 Hz sampling, 12-lead ECG | Week 8 (Sampling) |
| Preprocessing | Bandpass 0.5-50 Hz, notch 50 Hz | Week 9 (Filters) |
| Feature Extraction | R-peak detection, HRV analysis, ST-segment | Week 7-8 (Signals, FFT) |
| Classification | SVM/RF with statistical validation | Week 10 (Biostatistics) |
| Model | Pharmacokinetic-dynamic model for drug effects | Week 11 (ODEs) |

### Phase 2 Competency Matrix

| Competency | Week(s) | Assessment |
|------------|---------|------------|
| Signal classification | 7 | Quiz 1 |
| LTI systems & convolution | 7 | Problem Set 1 |
| Fourier transforms | 8 | Problem Set 2 |
| Filter design (FIR/IIR) | 9 | Lab 3 |
| Hypothesis testing (t-test, ANOVA) | 10 | Problem Set 1 |
| ODE modeling (Hodgkin-Huxley) | 11 | Lab 2 |
| Integration project | 12 | Final Deliverable |

## Assessment Alignment
- **Phase 2 Integration Project** (Week 12): Complete diagnostic tool with full documentation
- **Phase 2 Mock Exam** (Week 12): Comprehensive assessment covering Weeks 7-11
- **HKU SBME Capstone Prerequisites**: All Phase 2 concepts validated

## Key Integration Concepts
1. **Signal-to-Noise Ratio (SNR)** — Engineering measure correlated with diagnostic accuracy
2. **ROC Analysis** — Combining signal processing (sensitivity/specificity) with statistics
3. **System Identification** — Using frequency response (Week 8) to fit ODE models (Week 11)
4. **Clinical Trial Design** — Linking signal biomarkers to clinical endpoints
5. **FDA Regulatory Framework** — 510(k) clearance requirements for diagnostic software

## Supplementary Reading
- **Brown, M. et al.** (2019) "FDA's Guidance on Software as a Medical Device (SaMD)" — FDA
- **Good Clinical Practice** — ICH E6(R2) guidelines for clinical trials
- **ISO 13485** — Quality management systems for medical devices

## Reflection Questions
1. How do you validate that your signal processing improves clinical outcomes?
2. What is the minimum SNR required for reliable feature extraction?
3. How would you design a clinical study to validate your diagnostic algorithm?
4. What are the ethical considerations for AI-based medical diagnosis?
5. How does the integration of signals, statistics, and modeling create clinical value?
