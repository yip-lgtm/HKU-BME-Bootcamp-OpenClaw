# Week 8 Deliverables — Fourier Transform & Spectrum Analysis (BMED2500)

## Deliverable 1: FFT Analysis of Biomedical Signals

- **Type**: Problem Set (Written + Code)
- **Deadline**: End of Week 8 (Sunday, 23:59 HKT)
- **Submission**: PDF (written) + .ipynb/.py (code)

### Requirements

**Problem 1 — FFT Computation (15 marks)**
Given x[n] = [1, 2, 3, 4, 5, 6, 7, 8]:

a) Compute the 8-point DFT X[k] by hand using the formula X[k] = Σₙ x[n]·e^(-j2πkn/8).
b) Write Python code to verify your answer using np.fft.fft().
c) Plot the magnitude |X[k]| and phase ∠X[k] for k = 0, 1, ..., 7.
d) What is the physical frequency (in Hz) corresponding to X[2] if fs = 1000 Hz?
e) Compute the inverse DFT to verify you recover x[n].

**Problem 2 — Sampling Theorem (15 marks)**
A medical imaging device samples a physiological signal with the following components:
- Respiration rate: 0.25 Hz
- Heart rate: 1.2 Hz  
- Muscle artifact: 60 Hz

a) What is the minimum sampling rate to avoid aliasing? Show your calculation.
b) The device uses fs = 100 Hz. Which frequency components will alias? What aliased frequencies will appear?
c) Design an anti-aliasing filter specification: what cutoff frequency would you choose, and what type (LP/HP/BP)?
d) The Nyquist criterion states f_s ≥ 2B. In practice, we use f_s > 2B × (2-5). Why is this necessary?

**Problem 3 — STFT Analysis (10 marks)**
A patient has an EEG recording with a seizure event at t = 30-35 seconds. The sampling rate is 250 Hz.

a) Design an STFT with nperseg = 256, noverlap = 192. Calculate:
   - Time resolution Δt
   - Frequency resolution Δf
   - Number of time frames for a 60-second recording
b) Will you be able to localize the seizure event in time with ±0.5s accuracy? Explain.
c) Will you be able to resolve frequency components separated by 2 Hz? Explain.
d) How would you modify the window length to improve frequency resolution at the expense of time resolution?

**Problem 4 — Laplace Transform (10 marks)**
A drug absorption system has the transfer function:

```
H(s) = K / (τ₁s + 1)(τ₂s + 1)

where K = 1, τ₁ = 2 hours, τ₂ = 4 hours
```

a) Identify the poles and zeros of H(s).
b) Is this system stable? Justify using the pole locations.
c) Compute the impulse response h(t).
d) Plot the step response y(t) for t ∈ [0, 20] hours.
e) What is the time constant of the dominant pole? What percentage of steady-state is reached at t = 3τ?

### Rubric

- [ ] **P1**: DFT computed correctly (6 pts), verified in Python (4 pts), plotted (3 pts), frequency interpretation (2 pts)
- [ ] **P2**: Minimum fs found (4 pts), aliasing analysis (5 pts), filter design (4 pts), practical consideration (2 pts)
- [ ] **P3**: STFT parameters calculated (4 pts), time resolution analysis (3 pts), frequency resolution analysis (3 pts)
- [ ] **P4**: Poles/zeros identified (2 pts), stability proven (3 pts), impulse response (3 pts), step response plotted (2 pts)

**Total: 50 marks**

---

## Deliverable 2: EEG Spectral Analysis Report

- **Type**: Code + Written Report (3-4 pages)
- **Deadline**: End of Week 8
- **Submission**: .ipynb (with output) + PDF report

### Task
Analyze a simulated EEG recording to detect event-related changes in alpha band power.

### Data Generation
Generate a 20-second EEG signal at fs = 250 Hz with:
- Background activity: pink noise (1/f spectrum)
- Alpha rhythm: 10 Hz, amplitude 0.5
- Event: Alpha burst at t = 8-12 seconds, amplitude 2.5
- Muscle artifact: brief burst at t = 5 seconds, 30 Hz, duration 0.5s

### Analysis Required

1. **Preprocessing** (5 marks): Design and apply appropriate filters to remove the muscle artifact while preserving alpha rhythm.

2. **Spectral Analysis** (10 marks):
   - Compute FFT of the full signal
   - Compute power spectral density (PSD) using Welch's method
   - Identify dominant frequency peaks
   - Compare PSD during event vs rest periods

3. **Time-Frequency Analysis** (10 marks):
   - Compute STFT spectrogram with appropriate window length
   - Track alpha band (8-13 Hz) power over time
   - Quantify event-related synchronization (ERS): % increase in alpha power during event

4. **Medical Interpretation** (10 marks):
   - Relate alpha burst to clinical context (e.g., eyes-closed resting state)
   - Compare with literature values for alpha power
   - Discuss limitations of the analysis

5. **Code Quality** (5 marks)

### Rubric

- [ ] Preprocessing filters correctly applied (5 pts)
- [ ] FFT and PSD analysis with correct interpretation (10 pts)
- [ ] STFT with appropriate window selection and ERS quantification (10 pts)
- [ ] Clinical interpretation with limitations discussed (10 pts)
- [ ] Clear, reproducible code (5 pts)

**Total: 40 marks**

---

## Deliverable 3: Aliasing Art Challenge (Bonus)

- **Type**: Creative Coding Challenge
- **Deadline**: End of Week 8
- **Submission**: .ipynb

### Challenge
Create a visual demonstration of aliasing by generating audio/visual artifacts. Your submission should:

1. Show a high-frequency pattern (e.g., rotating spokes, striped pattern) that appears to "freeze" or reverse when sampled at a low rate
2. Include a slider or animation showing the effect of varying sampling rate
3. Explain the aliasing phenomenon in a clinical context (e.g., stroboscopic effect in surgery, monitor flicker)

**Bonus 5 points** for creative visualization + clinical connection.

---

## Submission Instructions

- All files: `[StudentID]_W08_[DeliverableName].[ext]`
- Include all dependencies in a `requirements.txt`
- Code must be reproducible with random seeds set

**Maintainer**: BME Bootcamp Agent | **Week 8** | **BMED2500: Fourier & Sampling**
