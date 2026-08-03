# Week 7 Deliverables — Biomedical Signals & LTI Systems (BMED2500)

## Deliverable 1: Signal Processing Problem Set

- **Type**: Problem Set (Written + Code)
- **Deadline**: End of Week 7 (Sunday, 23:59 HKT)
- **Submission**: Upload to Canvas as a single PDF (written) + .ipynb/.py file (code)

### Requirements

1. **Problem 1 — Signal Classification (10 marks)**
   For each of the following biomedical signals, classify them according to:
   - Time domain: Continuous (CT) vs Discrete (DT)
   - Periodicity: Periodic vs Aperiodic
   - Energy/Power: Energy signal vs Power signal
   - Determinism: Deterministic vs Stochastic
   
   Signals:
   a) ECG waveform recorded for 10 seconds at 500 Hz
   b) EEG background activity (continuous monitoring)
   c) Action potential spike train (single neuron, 3 seconds)
   d) Blood pressure waveform (single cardiac cycle)
   e) Neural white noise (background cortical activity)
   
   For each signal, compute E (total energy) or P (average power) and show your work.

2. **Problem 2 — Convolution Calculation (15 marks)**
   Compute y[n] = x[n] * h[n] for the following signals by hand (show all steps):
   
   ```
   x[n] = {1, 2, 1} for n = 0,1,2 (0 otherwise)
   h[n] = {0.5, 1, 0.5} for n = 0,1,2 (0 otherwise)
   ```
   
   a) Write out the full convolution result y[n] for n = 0 to 4.
   b) Verify your answer using Python's `np.convolve()`.
   c) Plot x[n], h[n], and y[n] using matplotlib.
   d) Explain why len(y) = len(x) + len(h) - 1.

3. **Problem 3 — LTI System Analysis (15 marks)**
   A biomedical instrument has impulse response h(t) = e^(-2t)·u(t).
   
   a) Is this system BIBO stable? Prove it mathematically using ∫|h(t)|dt.
   b) Is this system causal? Explain.
   c) Compute the frequency response H(jω) using the Fourier transform.
   d) What type of filter (LP/HP/BP) does this represent?
   e) If the input is x(t) = sin(2t), compute the output y(t) using the convolution theorem.

4. **Problem 4 — BME Application (10 marks)**
   A pharmacokinetic model describes drug concentration as:
   
   ```
   C(t) = C₀ · e^(-k_e · t) · u(t)
   ```
   
   where k_e = 0.1 hr⁻¹ (elimination rate constant).
   
   a) What is the half-life of this drug?
   b) After how many hours does the concentration fall below 1% of C₀?
   c) Plot C(t)/C₀ for t ∈ [0, 50] hours.
   d) A patient takes 500mg dose. If C₀ = 10 mg/L, after how many hours is the patient safe to take another dose of 250mg (target < 5 mg/L)?

### Rubric

- [ ] **P1 (10 pts)**: All 5 signals correctly classified with justification (2 pts each)
- [ ] **P2 (15 pts)**: Convolution computed correctly by hand (8 pts), verified in Python (3 pts), plotted (2 pts), explained (2 pts)
- [ ] **P3 (15 pts)**: BIBO stability proven with integral (4 pts), causality explained (2 pts), H(jω) computed (5 pts), filter type identified (2 pts), output computed (2 pts)
- [ ] **P4 (10 pts)**: Half-life calculated (2 pts), time to <1% found (2 pts), plot generated (3 pts), clinical question answered (3 pts)
- [ ] **Code quality**: Clear comments, readable plots, reproducible results (bonus 2 pts)

**Total: 50 marks**

---

## Deliverable 2: LTI System Analysis Report

- **Type**: Code + Written Report
- **Deadline**: End of Week 7 (Sunday, 23:59 HKT)
- **Submission**: .ipynb file (with output cells) + 2-3 page PDF report

### Requirements

**Task**: Analyze a real-world LTI system related to biomedical engineering.

**Choose ONE of the following system types**:

| Option | System | Input | Output |
|--------|--------|-------|--------|
| A | Cardiovascular (Windkessel model) | Heartbeat impulse sequence | Blood pressure waveform |
| B | Pharmacokinetics (1-compartment) | Drug dose input | Plasma concentration |
| C | Neural membrane (RC circuit) | Current injection | Membrane voltage |
| D | EMG recording system | Muscle electrical activity | Measured EMG signal |

**Your analysis must include**:

1. **System Definition (5 marks)**
   - Define the impulse response h[n] or h(t) mathematically
   - Explain the biological/physical meaning of h(t)
   - Identify system properties: linear, time-invariant, causal, BIBO stable

2. **Convolution Analysis (10 marks)**
   - Compute the output y[n] = x[n] * h[n] for a given input
   - Use both direct convolution and FFT-based methods
   - Verify the convolution theorem: y(t) = x(t) * h(t) ↔ Y(ω) = X(ω)·H(ω)
   - Plot input, impulse response, and output on the same figure

3. **Frequency Response (10 marks)**
   - Compute H(jω) or H(e^(jω)) analytically
   - Plot magnitude |H(ω)| and phase ∠H(ω)
   - Identify: Is it lowpass, highpass, or bandpass?
   - Explain the -3dB bandwidth in biological terms

4. **Clinical/Biological Interpretation (10 marks)**
   - Explain what the system model tells us about the physiology
   - Discuss limitations of the LTI assumption
   - Propose one improvement to the model (e.g., adding nonlinearity, time-variance)

5. **Code Quality (5 marks)**
   - Clean, commented code
   - Reproducible (random seeds set)
   - Clear visualizations with proper labels

### Rubric

- [ ] System definition with correct impulse response and properties (5 pts)
- [ ] Convolution analysis with both time and frequency domain methods (10 pts)
- [ ] Frequency response correctly computed and interpreted (10 pts)
- [ ] Clinical interpretation of model results and limitations (10 pts)
- [ ] Well-documented, reproducible code (5 pts)

**Total: 40 marks**

---

## Deliverable 3: ECG Signal Generator Challenge

- **Type**: Coding Challenge (Competitive/Bonus)
- **Deadline**: End of Week 7
- **Submission**: .py file or .ipynb

### Challenge Goal
Write a Python function `generate_ecg(heart_rate, duration, fs, noise_level)` that generates a realistic ECG waveform and can be used to:
1. Detect R-peaks automatically
2. Compute instantaneous heart rate
3. Compute heart rate variability (HRV) metrics

### Scoring Criteria

| Criterion | Points | Description |
|-----------|--------|-------------|
| Waveform realism | 15 | ECG has recognizable P, QRS, T waves with correct morphology |
| R-peak detection | 10 | Correctly identifies R-peaks within ±50ms of ground truth |
| Heart rate accuracy | 10 | Computed HR within 5 bpm of actual HR |
| HRV computation | 10 | Correctly computes RMSSD and SDNN |
| Noise robustness | 10 | Handles noise levels 0.0–0.5 gracefully |
| Code quality | 5 | Clean, documented, efficient code |

### Bonus (5 points)
- Implement atrial fibrillation detection (irregular RR intervals)
- Add respiratory sinus arrhythmia (HR variation with breathing)

**Total: 60 marks (bonus 5)**

---

## Submission Instructions

1. All files: `[StudentID]_W07_[DeliverableName].[ext]`
   Example: `123456789_W07_ProblemSet.pdf`
2. Code files must be runnable without modification
3. Include a `requirements.txt` if you use non-standard libraries
4. Report must follow the rubric structure exactly

**Late Policy**: -10% per day (max 5 days)

**Maintainer**: BME Bootcamp Agent | **Week 7** | **BMED2500: Signals and Systems**
