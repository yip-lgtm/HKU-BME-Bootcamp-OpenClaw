# Week 8 Self-Quiz — Fourier Transform & Spectrum Analysis (BMED2500)

## Multiple Choice (10 questions, 2 points each = 20 points)

**1. The DFT of a length-N sequence x[n] produces how many frequency samples?**

- A. N/2
- B. N
- C. N−1
- D. 2N

**Answer**: B | **Explanation**: The N-point DFT produces N complex frequency samples X[k], for k = 0, 1, ..., N−1. The DFT is periodic with period N, so k = 0 to N−1 represents one complete period. Only N/2 of these are unique for real-valued signals (due to conjugate symmetry), but the full DFT still has N points.

---

**2. A signal with maximum frequency component of 200 Hz must be sampled at at least what rate to avoid aliasing?**

- A. 100 Hz
- B. 200 Hz
- C. 400 Hz
- D. 500 Hz

**Answer**: C | **Explanation**: The Nyquist-Shannon sampling theorem requires f_s ≥ 2·f_max. With f_max = 200 Hz, the minimum sampling rate is 400 Hz. In practice, we typically oversample (e.g., 500 Hz) to account for non-ideal anti-aliasing filters.

---

**3. In the radix-2 Cooley-Tukey FFT algorithm, the computational complexity is:**

- A. O(N) — linear
- B. O(N log N) — quasi-linear
- C. O(N²) — quadratic
- D. O(log N) — logarithmic

**Answer**: B | **Explanation**: The FFT reduces the computational cost from O(N²) (naive DFT) to O(N log₂ N). For N = 1024, this is approximately 10240 operations (FFT) vs 1,048,576 (naive DFT) — a 100x speedup.

---

**4. Zero-padding a signal before computing its FFT:**

- A. Increases the actual frequency resolution
- B. Decreases the actual frequency resolution
- C. Only improves the visual (interpolated) frequency resolution
- D. Has no effect on frequency analysis

**Answer**: C | **Explanation**: Zero-padding adds zeros to the end of a signal, increasing N in the FFT. This improves the visual density of frequency points (apparent resolution), but does NOT increase the actual spectral resolution, which is determined by the duration of the original signal: Δf = f_s/N_original. More zeros means more interpolated points between the same true frequency bins.

---

**5. A discrete-time signal x[n] = cos(πn/2) has a DTFT that is:**

- A. Continuous and periodic with period 2π
- B. Discrete and periodic with period π
- C. Discrete and periodic with period 2π
- D. Continuous and non-periodic

**Answer**: A | **Explanation**: DTFT is always continuous in frequency ω and periodic with period 2π: X(e^(j(ω+2π))) = X(e^(jω)). For x[n] = cos(πn/2) = (e^(jπn/2) + e^(-jπn/2))/2, the DTFT has impulses at ω = ±π/2, but the DTFT function itself is continuous and 2π-periodic.

---

**6. Which window function has the widest main lobe (worst frequency resolution) but lowest peak sidelobe level?**

- A. Rectangular
- B. Hann
- C. Hamming
- D. Blackman

**Answer**: D | **Explanation**: The Blackman window has the narrowest main lobe among common windows... actually, no. Let me reconsider: Rectangular has the narrowest main lobe but highest sidelobes (-13 dB). Blackman has the widest main lobe but lowest sidelobes (-58 dB). There is a fundamental tradeoff: you cannot have both narrow main lobe AND low sidelobes simultaneously. The answer is D (Blackman).

---

**7. In the STFT, increasing the window length (nperseg) while keeping fs constant will:**

- A. Improve time resolution, worsen frequency resolution
- B. Improve frequency resolution, worsen time resolution
- C. Improve both resolutions
- D. Worsen both resolutions

**Answer**: B | **Explanation**: This is the fundamental time-frequency uncertainty principle. A longer window means each FFT is computed over more samples, giving finer frequency resolution (Δf = fs/nperseg decreases), but the time localization worsens because each time frame covers a longer time span (Δt = nperseg/fs increases).

---

**8. A system has transfer function H(s) = 1/(s+2). The pole location in the s-plane is:**

- A. s = +2 (RHP — unstable)
- B. s = −2 (LHP — stable)
- C. s = 0 (on imaginary axis — marginally stable)
- D. s = 2j (upper half-plane — marginally stable)

**Answer**: B | **Explanation**: The pole of H(s) = 1/(s+2) is at s = −2, which is in the left half-plane (LHP). Since the real part σ = −2 < 0, the impulse response h(t) = e^(−2t)·u(t) decays exponentially. The system is stable (BIBO).

---

**9. The Laplace transform of x(t) = e^(−3t)·u(t) is X(s) = 1/(s+3). What is the region of convergence (ROC)?**

- A. Re{s} > 3
- B. Re{s} < −3
- C. Re{s} > −3
- D. −3 < Re{s} < 3

**Answer**: C | **Explanation**: For a right-sided exponential signal x(t) = e^(−3t)·u(t), the ROC is Re{s} > −3 (to the right of the pole at s = −3). This ensures |e^(−3t)e^(−st)| = e^(−(σ+3)t) decays for σ > −3, making the Laplace integral converge.

---

**10. A clinical EEG recording at 250 Hz shows a spike at 125 Hz in the spectrum. This is most likely:**

- A. A real neural oscillation at 125 Hz
- B. Aliasing of a high-frequency artifact
- C. DC offset in the recording
- D. Quantization noise

**Answer**: B | **Explanation**: At f_s = 250 Hz, the Nyquist frequency is f_N = 125 Hz. Any frequency component above 125 Hz will alias back into the band 0-125 Hz. A spectral peak at exactly 125 Hz is highly suspicious — it's likely an aliased component (perhaps 125 Hz itself could be an alias of some other frequency). The most common source is power line interference at 50 Hz (or 60 Hz in some countries) or its harmonics.

---

## Short Answer (5 questions, 4 points each = 20 points)

**1. Derive the frequency mapping between continuous-time (CT) and discrete-time (DT) systems. If a CT signal with frequency f = 50 Hz is sampled at f_s = 200 Hz, what is the corresponding DTFT frequency ω in radians/sample?**

**Answer**: The mapping between CT frequency f (Hz) and DTFT frequency ω (rad/sample) is:
ω = 2π · f / f_s

For f = 50 Hz and f_s = 200 Hz:
ω = 2π · 50/200 = 2π · 0.25 = π/2 rad/sample

This means one complete cycle of the analog signal corresponds to 4 samples in the discrete-time representation. The DTFT is periodic with period 2π, so frequencies f = 50 + 200·k Hz all map to ω = π/2 + 2πk rad/sample.

---

**2. The DFT X[k] of a real-valued signal x[n] has conjugate symmetry: X[k] = X*[N−k]. Explain the physical meaning of this symmetry and why it always holds for real-valued signals.**

**Answer**: The conjugate symmetry X[k] = X*[N−k] means:
- |X[k]| = |X[N−k]| — magnitude is symmetric
- ∠X[k] = −∠X[N−k] — phase is antisymmetric

**Physical meaning**: For real-valued x[n], the DFT represents the signal as a sum of complex exponentials. Since x[n] is real, each positive-frequency component at ω must be paired with its complex conjugate at −ω to cancel the imaginary parts and produce a real-valued sum. This is why the spectrum of real signals is symmetric about f_s/2.

**Why it holds**: x[n] = (1/N)Σ X[k]e^(j2πkn/N). If x[n] is real, then x[n] = x*[n]. Substituting: (1/N)Σ X[k]e^(j2πkn/N) = (1/N)Σ X*[k]e^(-j2πkn/N). By changing the summation index k → N−k, we get X[k] = X*[N−k].

---

**3. A neuron fires action potentials at approximately 50 spikes/second. We want to analyze the interspike interval (ISI) distribution with frequency resolution of at least 5 Hz. What is the minimum recording duration required?**

**Answer**: Frequency resolution: Δf = 1/T, where T is the recording duration.

To resolve frequencies at the scale of neural firing patterns (say, up to 50 Hz or 20 ms ISI variations), we need:
Δf ≤ 5 Hz → T ≥ 1/5 = 0.2 seconds = 200 ms

However, to characterize the ISI distribution reliably and see spectral features related to spike timing, we typically need T >> the correlation time. A common rule: T ≥ 10/Δf_min for stable spectral estimates.

For Δf = 1 Hz: T ≥ 10 seconds
For Δf = 5 Hz: T ≥ 2 seconds

**Practical answer**: Minimum T = 1/Δf = 1/5 = 0.2 seconds (for basic resolution), but for statistically reliable spectral estimates, T ≥ 2-10 seconds is recommended.

---

**4. The bilateral Laplace transform of x(t) = e^(2t)u(−t) is X(s) = 1/(s−2) with ROC: Re{s} < 2. Explain why the ROC is Re{s} < 2 and not Re{s} > 2.**

**Answer**: For x(t) = e^(2t)·u(−t) (left-sided signal, anti-causal):
X(s) = ∫₋∞⁰ e^(2t)·e^(−st)dt = ∫₋∞⁰ e^(−(s−2)t)dt

For convergence: Re{s−2} > 0 → Re{s} > 2? No, let me reconsider.

When t → −∞: e^(2t)·e^(−st) = e^(−(s−2)t) = e^(−(σ−2)·t)
Since t is negative: as t → −∞, we need −(σ−2)·t → −∞ (decaying)
This requires σ−2 < 0 → σ < 2

Therefore ROC: Re{s} < 2 (to the LEFT of the pole at s = 2).

This is a left-sided signal, so its ROC is to the left of the rightmost pole. If it were right-sided (u(t) instead of u(−t)), the ROC would be Re{s} > 2.

---

**5. A 4-point DFT is computed. The resulting frequency bins correspond to which analog frequencies if the sampling rate is 1000 Hz? List all four frequencies.**

**Answer**: DFT frequency bins: k = 0, 1, 2, 3

The k-th DFT bin corresponds to: f_k = k · f_s / N = k · 1000/4 = k · 250 Hz

- k = 0: f₀ = 0 Hz (DC component)
- k = 1: f₁ = 250 Hz (Nyquist frequency = f_s/2)
- k = 2: f₂ = 500 Hz (f_s, aliased to 0 in real signals)
- k = 3: f₃ = 750 Hz (aliased version of f_s − 750 = 250 Hz, but in the DFT representation it's treated as a negative frequency)

For real-valued signals, k = 0 and k = 2 are real-valued. k = 1 and k = 3 are complex conjugates (representing the same physical frequency at 250 Hz). The unique frequencies are f₀ = 0 Hz and f₁ = 250 Hz.

---

## True or False (5 questions, 1 point each = 5 points)

**1. The CTFT (Continuous-Time Fourier Transform) of a periodic signal is a continuous spectrum (not impulses).**
**Answer**: False. The CTFT of a periodic signal is a line spectrum — impulses at harmonic frequencies. The Fourier Series coefficients are scaled impulses: X(ω) = 2πΣ aₖ δ(ω − kω₀). The CTFT of an aperiodic signal is continuous.

---

**2. Zero-padding a signal from N to 2N samples halves the frequency resolution.**
**Answer**: False. Zero-padding does NOT affect actual frequency resolution. Actual resolution is determined by the duration of the nonzero signal: Δf = f_s/N. Zero-padding increases N in the FFT computation, providing more interpolated points between true frequency bins, but the actual spectral content (where the energy is concentrated) remains the same.

---

**3. DTFT is always periodic in frequency with period 2π, but DFT is always aperiodic.**
**Answer**: False (first part) and True (second part). DTFT is periodic in ω with period 2π: X(e^(j(ω+2π))) = X(e^(jω)). DFT of an N-point sequence is defined for k = 0, 1, ..., N−1 and the N DFT values are discrete and represent one period of the DTFT.

---

**4. The STFT of a signal is invertible: you can recover the original signal from its spectrogram.**
**Answer**: False (for magnitude spectrogram). The STFT magnitude |X(t, ω)| loses phase information, which is required for perfect reconstruction. However, the full complex STFT X(t, ω) is invertible (up to windowing effects). Techniques like Griffin-Lim algorithm can approximately invert from magnitude spectrogram alone.

---

**5. A pole at s = 0 in the Laplace domain corresponds to infinite gain at DC (ω = 0).**
**Answer**: True. H(s) = .../(s − 0) means at s = 0 (DC), the magnitude |H| → ∞. This corresponds to an integrator: y(t) = ∫x(τ)dτ. For an LTI system, a pole at the origin means the system accumulates input — it has infinite DC gain and integrates over time.

---

## Score Guide

| Score | Grade | Interpretation |
|-------|-------|---------------|
| 40–45 | A | Excellent! Ready for Week 9 (Filters) |
| 32–39 | B | Good foundation. Review missed questions |
| 24–31 | C | Review Week 8 notes and re-do code labs |
| < 24 | D | Focus on Week 8 before proceeding |

**Maintainer**: BME Bootcamp Agent | **Week 8** | **BMED2500: Fourier & Sampling**
