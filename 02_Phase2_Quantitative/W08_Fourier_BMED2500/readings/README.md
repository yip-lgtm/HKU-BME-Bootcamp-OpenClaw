# Week 8 Readings — Fourier, Convolution, Sampling, z-transform (BMED2500)

## Required Readings

### Primary Textbooks
- **Oppenheim, A.V. & Willsky, A.S.** (1997) *Signals and Systems*, 2nd ed. Prentice Hall — Chapters 4-6 (Fourier Transform, Sampling, Laplace Transform, z-Transform)
- **Proakis, J.G. & Manolakis, D.G.** (2006) *Digital Signal Processing*, 4th ed. Prentice Hall — Chapters 3-7
- **Haykin, S. & Van Veen, B.** (2002) *Signals and Systems*, 2nd ed. Wiley — Chapters 4-5

### HKU Course Materials
- **BMED2500** — Signals and Systems for Biomedical Engineers (HKU SBME)
- Topics: CTFT, DTFT, DFT, FFT, Sampling Theorem, Laplace transform, z-transform, filter relationships

### Key Papers / Historical References
1. **Cooley, J.W. & Tukey, J.W.** (1965) "An Algorithm for the Machine Calculation of Complex Fourier Series" — *Mathematics of Computation*, 19(90), 297-301. DOI: 10.2307/2003354 — The FFT paper, O(N log N) complexity
2. **Shannon, C.E.** (1949) "Communication in the Presence of Noise" — *Proceedings of the IRE*, 37(1), 10-21 — Sampling theorem formalization
3. **Gabor, D.** (1946) "Theory of Communication" — *Journal of the IEE*, 93(26), 429-457 — Time-frequency analysis, Gabor transforms
4. **Stockham, T.G.** (1966) "High-Speed Convolution and Correlation" — *AFIPS Spring Joint Computer Conference* — FFT in signal processing

### Online Resources
- MIT OCW: **6.003 Signals and Systems** — Fourier Transform lectures
- MIT OCW: **6.341 Discrete-Time Signal Processing** — DFT/FFT algorithms
- DSPRelated.com: **Fourier Transform Tutorials** — Practical FFT applications
- Wolfran Alpha: **Fourier Transform Calculator** — Interactive computation
- Justin Success: **Fourier Transform Explained** — Intuitive explanations with animations

## Pre-Week Objectives (BMED2500 Week 8)
By the end of Week 8, students should be able to:
1. Compute Continuous-Time Fourier Transform (CTFT) and its inverse
2. Understand the duality between time and frequency domains
3. Apply the Sampling Theorem: f_s ≥ 2B (Nyquist rate)
4. Explain aliasing and anti-aliasing filters
5. Compute Discrete-Time Fourier Transform (DTFT) and Discrete Fourier Transform (DFT)
6. Understand the Fast Fourier Transform (FFT) algorithm and its O(N log N) complexity
7. Apply Laplace Transform: X(s) = ∫x(t)e^(-st)dt, s = σ + jω
8. Apply z-Transform: X(z) = Σx[n]z^(-n), z = re^(jω)
9. Determine Region of Convergence (ROC) for Laplace and z-transforms

## Mathematical Foundations

### Core Equations (Week 8)

#### Fourier Transforms
1. **CTFT**: X(jω) = ∫x(t)e^(-jωt)dt, x(t) = (1/2π)∫X(jω)e^(jωt)dω
2. **DTFT**: X(e^(jω)) = Σx[n]e^(-jωn), x[n] = (1/2π)∫X(e^(jω))e^(jωn)dω
3. **DFT**: X[k] = Σx[n]e^(-j2πkn/N), x[n] = (1/N)ΣX[k]e^(j2πkn/N)

#### Sampling and Reconstruction
4. **Sampling Theorem**: x_s(t) = x(t)Σδ(t-nT_s), where T_s = 1/f_s
5. **Nyquist Rate**: f_s ≥ 2B, where B is the bandwidth
6. **Aliasing**: ω_alias = |ω mod 2ω_s - ω_s|

#### Laplace Transform
7. **Bilateral Laplace Transform**: X(s) = ∫x(t)e^(-st)dt, s = σ + jω
8. **ROC Properties**: ROC is bounded by poles; causal → right-sided

#### z-Transform
9. **z-Transform**: X(z) = Σx[n]z^(-n), z = re^(jω)
10. **Relationship to DTFT**: X(e^(jω)) = X(z)|z=e^(jω)

### FFT Complexity Comparison
- **Naive DFT**: N² complex multiplications (e.g., N=1024 → 1,048,576 operations)
- **FFT (Cooley-Tukey)**: (N/2)log₂(N) operations (e.g., N=1024 → 5,120 operations)
- **Speedup Factor**: ~200x for N=1024, ~1000x for N=10,000

### BME Applications
- **ECG spectral analysis**: P wave ~0-10 Hz, QRS ~10-40 Hz, T wave ~0-10 Hz
- **EEG frequency bands**: Delta (0.5-4 Hz), Theta (4-8 Hz), Alpha (8-13 Hz), Beta (13-30 Hz), Gamma (30-100 Hz)
- **Sampling rates**: ECG typically 500 Hz, EEG typically 250-1000 Hz, audio 44.1 kHz

## Key Properties Summary Table

| Property | Time Domain | Frequency Domain |
|----------|-------------|------------------|
| Convolution | x(t) * h(t) | X(jω) · H(jω) |
| Multiplication | x(t) · h(t) | (1/2π)X(jω) * H(jω) |
| Time shift | x(t-t₀) | X(jω)e^(-jωt₀) |
| Time scaling | x(at) | (1/|a|)X(jω/a) |
| Differentiation | dx/dt | jωX(jω) |
| Integration | ∫x(τ)dτ | X(jω)/jω + πX(0)δ(ω) |

## Assessment Alignment
- BMED2500 Problem Set 2 (Week 9): Fourier Transform, Sampling
- BMED2500 Midterm (Week 10): Full coverage of Weeks 7-9
- HKU SBME Lab 2: FFT-based spectral analysis of ECG signals

## Key Concepts for Deep Dive
1. **Why FFT?** — From O(N²) to O(N log N): the divide-and-conquer revolution
2. **Poles and Zeros** — Visualizing system behavior in the s-plane and z-plane
3. **ROC and Causality** — Why ROC determines if system is causal/stable
4. **Aliasing in Biomedical Signals** — Clinical consequences of undersampling
5. **Parseval's Theorem** — Energy conservation between time and frequency domains

## Supplementary Reading
- **Bracewell, R.N.** (2000) *The Fourier Transform and Its Applications*, 3rd ed. McGraw-Hill
- **Smith, S.W.** (1997) *The Scientist and Engineer's Guide to Digital Signal Processing* — Free online
- **Lyons, R.G.** (2010) *Understanding Digital Signal Processing*, 3rd ed. Prentice Hall

## Reflection Questions
1. Why is the FFT one of the most important algorithms in all of engineering?
2. What determines the ROC for a given signal's Laplace transform?
3. How does windowing affect spectral analysis? (spectrogram vs FFT)
4. Why must real-world anti-aliasing filters be analog (before sampling)?
5. What is the relationship between z-transform and discrete-time systems?
