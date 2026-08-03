# Week 9 Readings — FIR/IIR Filters & Signal Processing Tools (BMED2500)

## Required Readings

### Primary Textbooks
- **Oppenheim, A.V. & Schafer, R.W.** (2010) *Discrete-Time Signal Processing*, 3rd ed. Prentice Hall — Chapters 6-10 (Filter Design)
- **Proakis, J.G. & Manolakis, D.G.** (2006) *Digital Signal Processing*, 4th ed. Prentice Hall — Chapters 7-10
- **Rangayyan, R.M.** (2015) *Biomedical Signal Analysis*, 2nd ed. Wiley-IEEE Press — Chapters 4-6 (Biomedical filtering)

### HKU Course Materials
- **BMED2500** — Signals and Systems for Biomedical Engineers (HKU SBME)
- Topics: FIR filter design (windowing, Parks-McClellan), IIR filter design (Butterworth, Chebyshev, elliptic), filter structures, implementation

### Key Papers / Historical References
1. **Butterworth, S.** (1930) "On the Theory of Filter Amplifiers" — *Experimental Wireless and the Wireless Engineer*, 7, 536-541 — Butterworth filter design
2. **Parks, T.W. & McClellan, J.H.** (1972) "A Program for the Design of Digital FIR Filters" — *IEEE Transactions on Audio and Electroacoustics*, AU-20(3), 195-199 — Parks-McClellan algorithm
3. **Chebyshev, P.L.** (1899) *Theory of Mechanisms Adapted to the Calculation of Machines* — Chebyshev polynomials for filter design
4. **Widnall, W.S. & Liu, B.** (1979) "Minimizing the Group Delay Ripple of Multi-Band Filters" — *IEEE Transactions on Acoustics, Speech, and Signal Processing*

### Online Resources
- Iowa Hills Software: **FIR Filter Design** — Interactive Parks-McClellan design tool
- MATLAB/Octave: **fir1, fir2, firls,remez** — FIR filter design functions
- MATLAB/Octave: **butter, cheby1, cheby2, ellip** — IIR filter design functions
- SciPy Documentation: **signal filter design** — scipy.signal module
- dsprelated.com: **Filter Design** — Comprehensive filter design tutorials

## Pre-Week Objectives (BMED2500 Week 9)
By the end of Week 9, students should be able to:
1. Compare FIR vs IIR filters: linear phase, stability, computational complexity
2. Design FIR filters using windowing methods (Rectangular, Hamming, Hann, Blackman)
3. Design FIR filters using Parks-McClellan optimal equiripple method
4. Design IIR filters: Butterworth (maximally flat), Chebyshev Type I/II, Elliptic
5. Determine filter specifications: passband cutoff ω_p, stopband ω_s, ripple, attenuation
6. Implement filter structures: Direct Form I/II, Transposed, Cascade, Parallel
7. Apply filters to biomedical signals: ECG denoising, EEG spike detection, baseline wander removal

## Mathematical Foundations

### Core Equations (Week 9)

#### Filter Transfer Functions
1. **FIR Filter**: H(z) = Σb_k z^(-k), k = 0 to M (all-pole at z=0)
2. **IIR Filter**: H(z) = Σb_k z^(-k) / (1 + Σa_k z^(-k)), M≥N

#### Butterworth Magnitude Response
3. **|H(jω)|² = 1 / [1 + (ω/ω_c)^(2n)** — maximally flat in passband
4. **3dB cutoff**: |H(jω_c)|² = 1/2, where ω_c is the cutoff frequency

#### Chebyshev Type I
5. **|H(jω)|² = 1 / [1 + ε²T_n²(ω/ω_p)** — equiripple in passband
6. **ε**: passband ripple factor; T_n: Chebyshev polynomial of degree n

#### Linear Phase FIR Conditions
7. **Type I**: h[n] = h[M-n], M even, zero at ω=π
8. **Type II**: h[n] = h[M-n], M odd, zero at ω=π
9. **Type III**: h[n] = -h[M-n], M even, zeros at ω=0 and ω=π
10. **Type IV**: h[n] = -h[M-n], M odd, zero at ω=0

### Window Functions Comparison

| Window | Main Lobe Width | Peak Sidelobe (dB) | Attenuation (dB) |
|--------|-----------------|-------------------|------------------|
| Rectangular | 4π/(M+1) | -13.3 | -21 |
| Hamming | 8π/(M+1) | -41.9 | -53 |
| Hann | 8π/(M+1) | -31.5 | -44 |
| Blackman | 12π/(M+1) | -58.1 | -74 |
| Kaiser (β=8) | Variable | -56 | -70 |

### BME Filter Applications
- **ECG baseline wander removal**: High-pass filter, cutoff 0.5-5 Hz
- **ECG powerline interference**: Notch filter at 50/60 Hz, Q=30-50
- **ECG QRS detection**: Bandpass 5-15 Hz, enhances QRS complex
- **EMG noise removal**: Bandpass 20-500 Hz
- **EEG artifact removal**: ICA-based, bandpass 0.5-50 Hz

## Filter Design Parameters

### Typical Specifications
- **Passband ripple**: δ_p = 0.01-0.1 (0.1-1 dB)
- **Stopband attenuation**: δ_s = 0.001-0.01 (60-100 dB)
- **Transition width**: Δω = 0.1π-0.2π
- **Filter order**: N ≈ (A - 8) / (2.285Δω) for Chebyshev

### BME-Specific Filter Requirements
- **Medical device safety**: FDA Class IIa/b filters require validation
- **Latency**: Real-time systems need minimal group delay
- **Phase distortion**: ECG/EEG require linear phase for waveform morphology

## Assessment Alignment
- BMED2500 Problem Set 3 (Week 10): Filter design project
- BMED2500 Midterm (Week 10): FIR/IIR comparison, design methods
- HKU SBME Lab 3: Real-time ECG filtering implementation

## Key Concepts for Deep Dive
1. **Linear Phase** — Why it matters for biomedical signal fidelity
2. **Gibbs Phenomenon in Windowed FIR** — Truncation artifacts
3. **IIR Stability** — Bilinear transform mapping from s-plane to z-plane
4. **Filter Order Trade-offs** — Computational cost vs frequency selectivity
5. **Biomedical Filter Validation** — Clinical testing requirements

## Supplementary Reading
- **Antoniou, A.** (2017) *Digital Filters: Analysis, Design, and Signal Processing*, 2nd ed. McGraw-Hill
- **Dutta Roy, S.C.** (2015) *Introduction to Digital Signal Processing*, 2nd ed. McGraw-Hill
- **Ifeachor, E.C. & Jervis, B.W.** (2002) *Digital Signal Processing: A Practical Approach*, 2nd ed. Prentice Hall

## Reflection Questions
1. When would you choose FIR over IIR for biomedical applications?
2. How does the bilinear transform handle frequency warping?
3. Why do biomedical filters often use cascaded stages?
4. What is the clinical consequence of phase distortion in ECG analysis?
5. How does the Kaiser window's β parameter control the filter response?
