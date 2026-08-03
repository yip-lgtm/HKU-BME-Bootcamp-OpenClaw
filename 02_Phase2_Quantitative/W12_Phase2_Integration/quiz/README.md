# Week 12 Self-Quiz — Phase 2 Integration (BMED3603)

## Multiple Choice (10 questions, 2 points each = 20 points)

**1. In a complete ECG analysis pipeline, which sequence is most appropriate?**

- A. Filter → Detect peaks → Extract features → Statistical analysis
- B. Extract features → Filter → Detect peaks → Statistical analysis
- C. Statistical analysis → Filter → Extract features → Detect peaks
- D. All steps can be done simultaneously

**Answer**: A | **Explanation**: Proper signal processing requires filtering BEFORE feature extraction. Raw ECG contains noise (baseline wander, powerline interference, EMG), which would corrupt any features extracted from the signal. The correct pipeline: Acquire → Preprocess (filter) → Feature extraction → Statistical analysis. Features must be extracted from clean signals.

---

**2. Which Phase 2 concept is MOST directly used in the design of a clinical trial's sample size?**

- A. Fourier Transform
- B. IIR Filter design
- C. Statistical power analysis
- D. Hodgkin-Huxley model

**Answer**: C | **Explanation**: Statistical power analysis determines the sample size needed to detect a clinically meaningful effect with specified power (usually 80%) at a given significance level (α = 0.05). Power analysis uses effect size, variance, and α to calculate the required n. Fourier transforms and filter design are signal processing concepts; HH is a physiological model.

---

**3. A researcher finds that the RMSE of their PK model is 2.1 mg/L. They then add a second compartment to the model and the RMSE improves to 1.5 mg/L. Which statistical test should they use to determine if the improvement is significant?**

- A. Paired t-test
- B. F-test comparing residual variances
- C. Chi-square test
- D. ANOVA

**Answer**: B | **Explanation**: When comparing the goodness-of-fit of nested models (one-compartment vs two-compartment), the F-test comparing residual sum of squares (or RMSE²) is appropriate. The F-statistic = [(RSS_reduced - RSS_full)/(df_reduced - df_full)] / [RSS_full/df_full]. This tests whether adding parameters significantly improves the fit.

---

**4. Which combination of techniques is MOST appropriate for detecting atrial fibrillation (AF) in ECG?**

- A. Lowpass filter + peak detection + HRV (RMSSD) + arrhythmia classification
- B. Highpass filter + FFT + regression
- C. Bandpass filter + ODE solver + PK model
- D. Moving average + t-test + correlation

**Answer**: A | **Explanation**: AF detection requires: (1) preprocessing (bandpass 0.5-40 Hz to remove noise), (2) R-peak detection, (3) RR interval variability analysis (AF shows highly irregular RR intervals → very low HRV), (4) classification based on HRV metrics. FFT and regression alone don't capture the irregular timing pattern. ODE/PK models don't apply to AF.

---

**5. In physiological modeling, what is the PRIMARY advantage of a two-compartment PK model over a one-compartment model?**

- A. Simpler mathematics
- B. Fewer parameters to estimate
- C. Can model the distribution phase and terminal elimination phase separately
- D. Always has a higher R²

**Answer**: C | **Explanation**: A two-compartment model distinguishes between the central compartment (blood/plasma) where drug concentration equilibrates quickly, and the peripheral compartment (tissues) where equilibration is slower. This creates a biphasic elimination curve with an initial distribution phase and a terminal elimination phase. This is physiologically more realistic for many drugs.

---

**6. A clinical study reports "HR decreased by 15 ± 3 bpm (mean ± SEM) after treatment, p = 0.001." Which interpretation is MOST accurate?**

- A. The treatment always reduces HR by 15 bpm
- B. There is a 0.1% chance the treatment has no effect
- C. If there were no real effect, there's a 0.1% chance of observing data this extreme
- D. The effect size is 15 bpm

**Answer**: C | **Explanation**: p = 0.001 means: if the null hypothesis (no effect) were true, there is a 0.1% probability of observing data at least as extreme as what was observed. It does NOT mean there's a 0.1% chance H₀ is true (that's a Bayesian interpretation). The ± SEM is the standard error, not the standard deviation. Effect size should be reported separately (e.g., Cohen's d = 0.8).

---

**7. Which of the following is an example of the CONVERGENCE of signal processing and physiological modeling?**

- A. Using FFT to analyze the frequency content of an ECG signal
- B. Using an ODE model to predict blood pressure response to a drug
- C. Using a filter to remove baseline wander before applying an ODE model
- D. Using a t-test to compare two groups

**Answer**: C | **Explanation**: Signal processing (filtering) is used to clean data BEFORE applying a physiological model. This demonstrates convergence — preprocessing (signal processing) enables better modeling (ODE). FFT analysis (A) and ODE prediction (B) are domain-specific. t-test (D) is statistics. Only option C shows integration across domains.

---

**8. In HRV analysis, the LF/HF ratio is computed from:**

- A. Time-domain features (RR intervals)
- B. Frequency-domain features (power spectral density)
- C. Non-linear features (sample entropy)
- D. Morphological features (QRS shape)

**Answer**: B | **Explanation**: The LF/HF ratio is a frequency-domain HRV metric computed from the power spectral density:
- LF band: 0.04-0.15 Hz (associated with sympathetic activity)
- HF band: 0.15-0.4 Hz (associated with parasympathetic/vagal activity)
- LF/HF ratio: index of sympathovagal balance

This requires computing the PSD (via Welch's method or FFT) and integrating power in each frequency band.

---

**9. A diagnostic test has sensitivity = 95% and specificity = 90% for a disease with 5% prevalence in the population. What is the probability that a person with a POSITIVE test actually has the disease (PPV)?**

- A. 95%
- B. 90%
- C. ~32%
- D. ~95%

**Answer**: C | **Explanation**: Using Bayes' theorem:
PPV = (Se × Prev) / (Se × Prev + (1-Sp) × (1-Prev))
PPV = (0.95 × 0.05) / (0.95 × 0.05 + 0.10 × 0.95)
PPV = 0.0475 / (0.0475 + 0.095) = 0.0475 / 0.1425 ≈ 0.333 ≈ 33%

This is a fundamental principle: in low-prevalence diseases, even good tests have poor PPV. Most positive results are false positives when the disease is rare.

---

**10. Which step is MOST LIKELY to introduce systematic bias in a clinical signal processing pipeline?**

- A. Using FFT to compute the power spectrum
- B. Applying the same filter cutoff to all recordings without calibration
- C. Detecting R-peaks using a fixed threshold
- D. Computing RMSSD from RR intervals

**Answer**: B | **Explanation**: Applying the same filter parameters (cutoff frequencies, filter order) to all recordings without considering inter-subject variability can introduce systematic bias. For example, if some patients have higher baseline wander, the same 0.5 Hz highpass may be insufficient for some and too aggressive for others. Fixed threshold R-peak detection (C) can also be problematic but is less systematically biased across the entire population. FFT, RMSSD are computational steps without design choices that introduce systematic bias.

---

## Short Answer (5 questions, 4 points each = 20 points)

**1. A clinical study measures blood pressure before and after a new antihypertensive drug. Describe the complete analysis pipeline using ALL THREE Phase 2 domains: signal processing, statistics, and physiological modeling.**

**Answer**:
**Signal Processing**:
- Acquire BP waveform (continuous or cuff-based)
- Preprocessing: remove artifacts (bandpass filter 0.5-30 Hz), detect systolic peaks
- Feature extraction: systolic BP, diastolic BP, mean arterial pressure, HR
- Frequency domain: compute BPV (blood pressure variability) spectrum

**Statistical Analysis**:
- Descriptive: mean ± SD for pre and post measurements
- Paired t-test (or Wilcoxon if non-normal): test if BP significantly changed
- Effect size (Cohen's d): is the change clinically meaningful?
- Confidence interval for mean change: how precise is our estimate?
- Power analysis: was the study adequately powered?

**Physiological Modeling**:
- PK/PD model: one-compartment oral absorption (k_a, k_e, V_d) + direct effect on heart rate/vascular resistance
- Fit model to individual patient data
- Predict: optimal dosing interval, time to steady state, minimum effective dose
- Sensitivity analysis: how does BP change with ±20% variation in elimination rate?

This integrated pipeline provides: clinical evidence (statistics) + mechanism (modeling) + measurement quality (signal processing).

---

**2. Explain why simply reporting p-values without effect sizes and confidence intervals is insufficient for clinical research. Provide an example from Phase 2 content.**

**Answer**: p-values alone are insufficient because:

1. **They don't measure magnitude**: A highly significant p (p < 10^-10) could correspond to a clinically trivial effect (e.g., BP changes by 0.1 mmHg). Statistical significance ≠ clinical significance.

2. **They don't convey precision**: p = 0.04 with n=1000 vs p = 0.04 with n=10 convey very different levels of evidence. CI tells us the range of plausible effect sizes.

3. **They don't indicate practical importance**: Effect size (Cohen's d, odds ratio) tells us how large the effect is in practical terms.

**Phase 2 Example**:
Drug A study: BP change = 15 ± 3 mmHg (SEM), p = 0.001
Drug B study: BP change = 5 ± 1 mmHg (SEM), p = 0.001

Both have p = 0.001, but:
- Drug A: Cohen's d ≈ 0.5 (medium effect)
- Drug B: Cohen's d ≈ 0.1 (negligible effect)

With CI:
- Drug A CI: [9, 21] mmHg (entirely clinically meaningful)
- Drug B CI: [3, 7] mmHg (borderline clinically meaningful)

Reporting only "p = 0.001" hides this critical difference.

---

**3. In the Hodgkin-Huxley model, the action potential is an emergent property of the interaction between ion channels. How does this motivate the use of mechanistic models over purely statistical models in biomedical engineering?**

**Answer**: 
**Limitation of purely statistical models**:
Statistical models (regression, ML) can accurately predict outcomes from data, but they:
- Don't explain WHY the system behaves as it does
- Don't generalize to unseen conditions (no mechanistic insight)
- Can't predict responses to novel interventions (e.g., new drug target)
- Provide no insight into pathophysiology

**Advantage of mechanistic models (like HH)**:
1. **Interpretable parameters**: g_Na, g_K, E_Na have clear biophysical meanings
2. **Predictive under novel conditions**: Can predict response to new drug, new mutation, new species
3. **Identifiable structure**: The model structure (equivalent circuit) reflects known physiology
4. **Enables intervention design**: Can predict which ion channel to target to achieve desired effect

**Example from Phase 2**:
- Statistical model: "Patients with AF have RMSSD < 15 ms" — useful diagnostic
- HH model: "Na+ channel blocker prolongs APD by reducing I_Na" — enables drug design

Both are valuable, but mechanistic models provide deeper understanding and can guide therapy development.

---

**4. A researcher collects EEG data from 50 patients during a cognitive task. They perform FFT on each recording and compare spectral power between groups using t-tests at 100 frequency bins. They report p < 0.05 at several frequency bins. What is the major statistical problem here, and how would you address it?**

**Answer**: **Multiple comparisons problem** — performing 100 independent t-tests inflates the family-wise error rate.

With α = 0.05 per test and 100 tests:
Expected false positives = 100 × 0.05 = 5
If we observe 6-7 significant bins, they could all be false positives!

**Solutions**:
1. **Bonferroni correction**: α_adj = 0.05/100 = 0.0005 (very conservative)
2. **False Discovery Rate (FDR)**: Benjamini-Hochberg procedure (less conservative, controls proportion of false discoveries)
3. **Cluster-based correction**: In EEG, significant clusters in time-frequency are tested rather than individual points
4. **Pre-specified regions of interest**: Only test frequency bands of interest (alpha, beta, gamma) rather than all bins
5. **Permutation testing**: Non-parametric approach that properly controls for multiple comparisons

**Best practice**: Pre-register your analysis plan (which bins/frequencies you'll test) to avoid data snooping.

---

**5. Describe how signal processing and physiological modeling can be combined in a closed-loop drug delivery system for general anesthesia. What are the key integration challenges?**

**Answer**:
**Closed-Loop Anesthesia System**:

1. **Signal Processing**:
   - Acquire EEG (processed EEG index: BIS, SEF, SAiR)
   - Real-time filtering to remove EMG artifact
   - Feature extraction: spectral edge frequency, burst suppression ratio
   - Classification: awake vs sedated vs deep anesthesia states

2. **Statistical Modeling** (for initial calibration):
   - Population PK/PD model for propofol and remifentanil
   - Bayesian updating based on individual response
   - Individualize dosing parameters

3. **Physiological Modeling**:
   - PK model: one/two-compartment with effect site equilibration
   - PD model: Emax model linking concentration to EEG effect
   - ODE solver for real-time concentration prediction

4. **Closed-Loop Control**:
   - Controller: PID or model predictive control
   - Input: current EEG depth
   - Output: drug infusion rate
   - Goal: maintain target BIS (e.g., 40-60)

**Integration Challenges**:
1. **Latency**: Signal processing delay (100-500 ms) + infusion delay (effect site equilibration ~1-2 min)
2. **Inter-patient variability**: Model parameters vary 3-5 fold between patients
3. **Safety**: Robustness to signal dropout, actuator failure, overdosing
4. **Regulatory**: FDA approval requires extensive validation
5. **Model mismatch**: Individual patient may deviate significantly from population model

---

## True or False (5 questions, 1 point each = 5 points)

**1. The convolution theorem allows us to replace convolution in the time domain with multiplication in the frequency domain, which is computationally faster.**
**Answer**: True. Convolution in time ↔ multiplication in frequency. For large signals, FFT-based convolution (FFT → multiply → IFFT) is faster than direct convolution: O(N log N) vs O(N²). This is the foundation of fast filtering and spectral analysis.

---

**2. A statistically significant result (p < 0.05) is always clinically important.**
**Answer**: False. Statistical significance does not equal clinical significance. With large sample sizes, even tiny effects (0.001 mmHg BP reduction) can be statistically significant. Clinical importance requires effect sizes above a clinically meaningful threshold, not just statistical significance.

---

**3. The Hodgkin-Huxley model and a linear regression model are both examples of "models" but serve fundamentally different purposes in biomedical engineering.**
**Answer**: True. HH is a mechanistic/white-box model — its parameters have physical meaning (conductances, capacitances, Nernst potentials). Regression is a statistical/empirical/black-box model — it describes relationships without mechanistic insight. Both are useful but for different purposes: HH for understanding mechanism and predicting under novel conditions; regression for prediction and association testing.

---

**4. In the PK/PD闭环 system, using a faster ADC (higher sampling rate) always improves the quality of the physiological model parameter estimates.**
**Answer**: False. Beyond the Nyquist frequency, increasing sampling rate doesn't improve parameter estimates — it just adds redundant data. The key determinant of PK/PD parameter accuracy is the timing of blood samples relative to the system's time constants, not the ADC rate. Oversampling beyond what's needed increases computational burden without improving estimates.

---

**5. An integrative approach combining signal processing, statistics, and physiological modeling provides more robust clinical decision support than any single method alone.**
**Answer**: True. This is the fundamental premise of Phase 2. Signal processing ensures data quality. Statistical analysis quantifies uncertainty and validates findings. Physiological modeling provides mechanistic understanding and enables prediction. Together, they form a robust pipeline that is both data-driven (signal processing, statistics) and knowledge-driven (physiological modeling), providing both reliability and interpretability.

---

## Score Guide

| Score | Grade | Interpretation |
|-------|-------|---------------|
| 40–45 | A | Excellent! Phase 2 Master — ready for Phase 3 |
| 32–39 | B | Strong understanding — review missed questions |
| 24–31 | C | Good foundation — review Phase 2 integration |
| < 24 | D | Review all Phase 2 content before proceeding |

---

## Phase 2 Final Reflection Questions

Answer these for your own learning (not graded):

1. What was the most challenging concept in Phase 2, and how do you now understand it better?
2. Which Phase 2 skill will be most useful in your future research/clinical work?
3. How do the three Phase 2 domains (signals, statistics, modeling) connect to your Phase 1 knowledge?
4. What questions do you still have about Phase 2 content?
5. How will you apply Phase 2 skills in Phase 3 (Clinical/Biomedical Engineering)?

**Maintainer**: BME Bootcamp Agent | **Week 12** | **Phase 2 Integration**
