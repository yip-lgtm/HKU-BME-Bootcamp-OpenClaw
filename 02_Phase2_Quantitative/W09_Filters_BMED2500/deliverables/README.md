# Week 9 Deliverables — FIR/IIR Filters & Signal Processing (BMED2500)

## Deliverable 1: Filter Design Challenge

- **Type**: Problem Set + Code
- **Deadline**: End of Week 9 (Sunday, 23:59 HKT)
- **Submission**: PDF + .ipynb/.py

### Requirements

**Problem 1 — FIR Filter Design (15 marks)**
Design a bandpass FIR filter for EEG analysis with the following specifications:
- Passband: 8-13 Hz (alpha rhythm)
- Stopband attenuation: ≥ 40 dB
- Maximum passband ripple: 0.5 dB
- Sampling rate: 250 Hz

a) Determine the minimum filter length M using the approximate formula for the Hann window.
b) Design the filter using `scipy.signal.firwin` with the Hann window.
c) Plot the magnitude response (dB) and phase response.
d) Apply the filter to a simulated EEG signal containing alpha (10 Hz), beta (20 Hz), and delta (2 Hz) components. Show before/after spectra.
e) Explain why a Type I FIR filter (M odd, symmetric) is preferred over Type II.

**Problem 2 — IIR Filter Analysis (15 marks)**
A biomedical instrument uses the following transfer function:

```
H(z) = (1 + z^{-1}) / (1 - 0.5z^{-1})
```

a) Identify poles and zeros. Plot them on the z-plane.
b) Determine if the system is BIBO stable. Prove your answer.
c) Is the system causal? Explain.
d) Compute and plot the frequency response |H(e^(jω))| for ω ∈ [0, π].
e) What type of filter (LP/HP/BP/BS) is this? Justify.

**Problem 3 — ECG Denoising (10 marks)**
An ECG recording has the following noise components:
- Baseline wander: 0.1-0.5 Hz drift
- Powerline interference: 50 Hz + harmonics (100 Hz, 150 Hz)
- EMG noise: broadband 20-200 Hz

Design a complete denoising pipeline using appropriate filters:
a) Specify filter type, order, and cutoff frequencies for each stage.
b) Implement the pipeline in Python using `scipy.signal` functions.
c) Compare Butterworth vs Chebyshev Type I for the powerline notch. Which has better 50 Hz rejection?
d) Show SNR improvement before and after filtering.
e) Discuss the tradeoff between noise reduction and ECG signal distortion.

**Problem 4 — Real-time Constraints (10 marks)**
A patient monitor processes ECG at fs = 500 Hz with a processing delay budget of 5 ms.

a) What is the maximum filter length for an FIR filter to fit within the budget?
b) If you use an IIR filter of order 4, what is the expected group delay in samples and milliseconds?
c) The filter must have fc = 40 Hz for QRS detection. Which filter type (FIR or IIR) would you choose for this real-time application? Justify.
d) For a 10-second ECG recording, compare the computational cost (multiplications per second) of a 201-tap FIR filter vs a 4th-order IIR filter.

### Rubric

- [ ] **P1**: M calculated correctly (3pts), filter designed in Python (4pts), plotted (3pts), applied to EEG (3pts), explanation (2pts)
- [ ] **P2**: Poles/zeros identified (3pts), stability proven (3pts), causality explained (2pts), frequency response plotted (3pts), filter type (2pts)
- [ ] **P3**: Pipeline designed (3pts), implemented (3pts), Butterworth vs Chebyshev compared (2pts), SNR computed (2pts)
- [ ] **P4**: FIR length budget calculated (2pts), IIR delay computed (2pts), filter type decision justified (3pts), computational cost compared (3pts)

**Total: 50 marks**

---

## Deliverable 2: EEG/ECG Denoising Report

- **Type**: Code + Written Report (3-4 pages)
- **Deadline**: End of Week 9

### Task
Design and evaluate a complete signal denoising system for either EEG or ECG.

### Options
- **Option A**: EEG denoising for sleep stage classification
  - Remove: ocular artifacts (EOG, 0-5 Hz drift), muscular artifacts (EMG, >20 Hz), electrode motion (slow drift <1 Hz)
  - Preserve: Sleep spindle (12-15 Hz), K-complex, slow oscillation (0.5-1 Hz)

- **Option B**: ECG denoising for arrhythmia detection
  - Remove: baseline wander (<0.5 Hz), powerline (50/60 Hz), EMG noise (20-150 Hz)
  - Preserve: QRS complex (10-40 Hz), P wave (0.5-10 Hz), T wave (0.5-10 Hz)

### Required Analysis

1. **Filter Design & Comparison (15 marks)**:
   - Design 2 FIR filters (different window types) and 2 IIR filters (different types)
   - Compare: magnitude response, phase linearity, group delay, stopband attenuation
   - Recommend the best filter for your application

2. **Performance Evaluation (15 marks)**:
   - Compute SNR improvement
   - Measure QRS/alpha wave preservation (correlation with clean signal)
   - Compare filtered output with ground truth

3. **Clinical Discussion (10 marks)**:
   - What diagnostic information could be lost by filtering?
   - What are the risks of over-filtering?
   - How would you validate in a clinical setting?

### Rubric

- [ ] Filter designs correct and well-evaluated (15 pts)
- [ ] Performance metrics computed (SNR, correlation) (15 pts)
- [ ] Clinical discussion (10 pts)
- [ ] Code quality and reproducibility (5 pts)

**Total: 45 marks**

---

## Deliverable 3: Filter Design Competition (Bonus)

Design the most efficient filter to meet these specifications:
- Passband: 8-13 Hz, ripple < 0.1 dB
- Stopband: 0-5 Hz and 16-50 Hz, attenuation > 50 dB
- fs = 250 Hz
- Evaluation: minimum filter order/coefficients (computational efficiency)

**Bonus 5 points** for the most efficient design.

---

## Submission Instructions

- `[StudentID]_W09_[DeliverableName].[ext]`
- Include `requirements.txt`
- Code must be reproducible

**Maintainer**: BME Bootcamp Agent | **Week 9** | **BMED2500: FIR/IIR Filters**
