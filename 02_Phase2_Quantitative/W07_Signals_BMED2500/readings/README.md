# Week 7 Readings — Biomedical Signals & Linear Systems (BMED2500)

## Required Readings

### Primary Textbooks
- **Oppenheim, A.V. & Willsky, A.S.** (1997) *Signals and Systems*, 2nd ed. Prentice Hall — Chapters 1-3 (Signals, Systems, LTI Systems)
- **Proakis, J.G. & Manolakis, D.G.** (2006) *Digital Signal Processing: Principles, Algorithms, and Applications*, 4th ed. Prentice Hall — Chapters 1-2
- **Rangayyan, R.M.** (2015) *Biomedical Signal Analysis: A Case-Study Approach*, 2nd ed. Wiley-IEEE Press — Chapters 1-3

### HKU Course Materials
- **BMED2500** — Signals and Systems for Biomedical Engineers (Course Outline, HKU SBME)
- Course topics: CT/DT signals, Fourier series, Fourier transforms, Laplace transforms, z-transforms, LTI systems, convolution

### Key Papers / Historical References
1. **Nyquist, H.** (1928) "Certain Topics in Telegraph Transmission Theory" — *Transactions of the American Institute of Electrical Engineers*, 47(2), 617-644. DOI: 10.1109/T-AIEE.1928.505502 — Nyquist Sampling Theorem foundation
2. **Shannon, C.E.** (1949) "Communication in the Presence of Noise" — *Proceedings of the IRE*, 37(1), 10-21. DOI: 10.1109/JRPROC.1949.232969 — Shannon-Nyquist Sampling Theorem
3. **Bracewell, R.N.** (1956) "Radio Astronomy" — Strip integration and convolution in radio astronomy applications
4. **Hodgkin, A.L. & Huxley, A.F.** (1952) "A Quantitative Description of Membrane Current" — *Journal of Physiology*, 117(4), 500-544. DOI: 10.1113/jphysiol.1952.sp004764 — Nobel Prize, action potentials as signals

### Online Resources
- MIT OpenCourseWare: **6.003 Signals and Systems** — https://ocw.mit.edu/courses/6-003-signals-and-systems-fall-2011/
- MIT OpenCourseWare: **6.341 Discrete-Time Signal Processing** — https://ocw.mit.edu/courses/6-341-discrete-time-signal-processing-fall-2005/
- Khan Academy: **Signals and Systems** — https://www.khanacademy.org/science/electrical-engineering/ee-signals
- Academic Earth: **Signals and Systems** — Video lectures by Oppenheim (MIT)
- Wolfram Demonstrations: **Convolution** — Interactive convolution visualizations
- PhysioNet: **PhysioBank** — Real biomedical signal databases (ECG, EEG, blood pressure)

## Pre-Week Objectives (BMED2500 Week 7)
By the end of Week 7, students should be able to:
1. Classify signals: continuous-time (CT) vs discrete-time (DT), periodic vs aperiodic, energy vs power
2. Perform signal operations: time shifting, scaling, reversal, convolution
3. Explain LTI system properties: linearity, time-invariance, causality, stability
4. Compute convolution integrals (CT) and convolution sums (DT)
5. Understand basic signals: unit step u(t), unit impulse δ(t), exponentials
6. Apply Fourier Series for periodic signals (CTFS)
7. Recognize BME applications: ECG waveforms, neural action potentials, blood pressure signals

## Mathematical Foundations

### Core Equations (Week 7)
1. **Unit Impulse**: δ(t) = 0 for t ≠ 0, ∫δ(t)dt = 1
2. **Unit Step**: u(t) = 1 for t ≥ 0, 0 otherwise
3. **Convolution Integral (CT)**: y(t) = x(t) * h(t) = ∫x(τ)h(t-τ)dτ
4. **Convolution Sum (DT)**: y[n] = x[n] * h[n] = Σx[k]h[n-k]
5. **Fourier Series (CTFS)**: x(t) = Σaₖe^(jkω₀t), where ω₀ = 2π/T
6. **Complex Exponential**: e^(jθ) = cos(θ) + j sin(θ)
7. **Euler's Formula**: cos(θ) = (e^(jθ) + e^(-jθ))/2

## BME-Specific Signal Examples
- **ECG waveform**: P wave (~0.1s), QRS complex (~0.08s), T wave (~0.2s)
- **Neural spike**: ~1ms duration, ~100mV amplitude
- **Blood pressure**: ~120/80 mmHg systolic/diastolic, ~1Hz heart rate
- **EEG rhythms**: Delta (0.5-4 Hz), Theta (4-8 Hz), Alpha (8-13 Hz), Beta (13-30 Hz)

## Assessment Alignment
- BMED2500 Problem Set 1 (Week 8): Convolution, LTI systems
- BMED2500 Quiz 1 (Week 9): Signal classification, Fourier Series
- HKU SBME Lab 1: MATLAB signal generation and convolution

## Key Concepts for Deep Dive
1. **LTI Systems and Convolution** — Why convolution fully characterizes LTI systems
2. **Dirac Delta and Sampling** — The delta function as a mathematical idealization
3. **Fourier Series Convergence** — Dirichlet conditions for convergence
4. **Gibbs Phenomenon** — Overshoot near discontinuities (9% for Fourier Series)
5. **Causality and Stability** — BIBO stability criterion

## Supplementary Reading
- **Hsu, H.P.** (2010) *Schaum's Outline of Signals and Systems*, 3rd ed. McGraw-Hill
- **Lathi, B.P. & Green, R.** (2018) *Essentials of Digital Signal Processing* — Cambridge University Press
- **Webster, J.G.** (2010) *Medical Instrumentation: Application and Design*, 4th ed. Wiley

## Reflection Questions
1. Why is the impulse response h(t) sufficient to characterize any LTI system?
2. What happens to Gibbs phenomenon as we add more Fourier terms?
3. How does signal classification (CT vs DT, periodic vs aperiodic) affect our analysis approach?
4. Why is causality important for real-time biomedical signal processing?
5. How would you model an ECG signal as a combination of basic signals?
