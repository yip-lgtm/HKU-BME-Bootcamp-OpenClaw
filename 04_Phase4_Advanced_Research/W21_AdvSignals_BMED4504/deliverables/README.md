# Week 21 Deliverables — Advanced Biomedical Signals (BMED4504)

> **Due**: End of Week 21 weekend (Sunday midnight)  
> **Submission**: Push to `04_Phase4_Advanced_Research/W21_AdvSignals_BMED4504/deliverables/`

---

## Deliverable 1: EEG Signal Processing Pipeline Report

### Type
**Technical Report** (2000-2500 words + Python code)

### Scenario

You are a biomedical signal processing engineer developing an EEG-based brain-computer interface (BCI) for a startup. The EEG data contains significant artifacts (eye blinks, muscle activity, line noise) that must be removed before feature extraction for motor imagery classification.

### Required Sections

**1. Introduction** (300 words)
- EEG fundamentals: 10-20 system, frequency bands (δ, θ, α, β, γ)
- Motor imagery BCI: event-related desynchronization (ERD)
- Challenge: real-time artifact removal

**2. Data Description** (200 words)
- 64-channel EEG, 1000 Hz sampling rate
- 5-minute recording from healthy subject
- Artifacts: eye blinks, eye movements, muscle activity, line noise

**3. Processing Pipeline** (1200 words)

Design a pipeline with the following stages:

(a) **Preprocessing**: Bandpass filter (0.5-100 Hz), notch filter (60 Hz)
(b) **ICA**: FastICA for artifact removal
(c) **Wavelet denoising**: DWT with db4 wavelet, 5 decomposition levels
(d) **Epoching**: Extract 2-second epochs around motor imagery events
(e) **Feature extraction**: ERD/ERS in alpha band (8-12 Hz)

**4. Implementation** (Python required)

Implement the complete pipeline with:
- Loading raw EEG data (or generate synthetic)
- All filtering steps
- ICA decomposition and component classification
- Wavelet denoising
- Performance metrics (SNR before/after)

**5. Results and Discussion** (500 words)

### Rubric

| Criterion | Points |
|-----------|--------|
| Pipeline completeness | 25 |
| ICA component analysis | 20 |
| Wavelet denoising | 20 |
| Performance evaluation | 20 |
| Code quality | 15 |
| **Total** | **100** |

---

## Deliverable 2: Adaptive Noise Cancellation Design

### Type
**Design Document** (1000 words + Python simulation)

### Task

Design an adaptive noise cancellation system for extracting fetal ECG from maternal abdominal recordings.

Requirements:
- Maternal ECG on chest (reference)
- Fetal ECG on abdominal (primary) - mixed signal
- Adaptive filter: LMS or RLS (your choice, justify)
- Evaluate performance with SNR improvement

### Calculations Required
- Convergence analysis
- Steady-state error calculation
- Filter length optimization

---

## Deliverable 3: Weekly Reflection

### Type
**Research Journal** (400-600 words)

### Prompts

1. Which advanced signal processing technique (wavelet, ICA, adaptive filtering, compressive sensing) has the most potential for your capstone project?

2. What are the computational constraints for real-time biomedical signal processing? How do these affect algorithm choice?

3. Rate your confidence (1-10) on each of the 5 mental models. Identify areas for deeper study.
