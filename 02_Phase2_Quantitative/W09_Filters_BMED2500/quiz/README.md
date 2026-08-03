# Week 9 Self-Quiz — FIR/IIR Filters (BMED2500)

## Multiple Choice (10 questions, 2 points each = 20 points)

**1. Which of the following is TRUE about FIR filters?**

- A. FIR filters always have linear phase if h[n] is symmetric
- B. FIR filters are always stable (by definition)
- C. FIR filters require fewer multiplications per output than IIR filters of the same order
- D. FIR filters cannot achieve sharp roll-off without very high order

**Answer**: A | **Explanation**: A FIR filter with symmetric (or antisymmetric) impulse response has exactly linear phase. Since FIR filters have all poles at z=0 (inside the unit circle), they are always BIBO stable. However, FIR filters generally require MORE multiplications than IIR filters for the same frequency selectivity. The correct answer is A — linear phase is achievable with symmetric/antisymmetric FIR filters.

---

**2. The Hann window has sidelobe level of approximately −31 dB. What is the approximate stopband attenuation of an FIR filter designed with this window?**

- A. −13 dB (same as rectangular)
- B. −31 dB (limited by window)
- C. −41 dB
- D. −58 dB

**Answer**: B | **Explanation**: The window's sidelobe level directly limits the stopband attenuation of the designed filter. Since the Hann window's first sidelobe is approximately −31 dB below the main lobe peak, the stopband cannot achieve better than approximately −31 dB attenuation regardless of filter length. This is the fundamental limitation of the simple windowing method. Increasing filter length narrows the main lobe (better resolution) but does not improve the sidelobe level.

---

**3. A discrete-time system has transfer function H(z) = (1 − 0.5z⁻¹)/(1 + 0.9z⁻¹). The pole is at z = −0.9. The system is:**

- A. Stable and causal
- B. Unstable and causal
- C. Stable and non-causal
- D. Unstable and non-causal

**Answer**: A | **Explanation**: The pole is at z = −0.9, which has |p| = 0.9 < 1 (inside the unit circle). Therefore the system is BIBO stable. The pole is not at the origin, but the filter is causal because we assume the ROC is |z| > 0.9 (outside the outermost pole). The numerator degree (1) equals the denominator degree (1), so the filter is proper and causal.

---

**4. In a Butterworth lowpass filter, as the filter order n increases:**

- A. Passband ripple increases, stopband attenuation decreases
- B. Passband becomes flatter, transition band narrows
- C. Phase response becomes more linear
- D. Group delay decreases

**Answer**: B | **Explanation**: The Butterworth filter is defined as having maximally flat magnitude in the passband (monotonically decreasing, no ripple). As order n increases: (1) the passband remains flat (maximally flat property is preserved), (2) the transition band narrows (sharper roll-off), and (3) stopband attenuation increases. The phase response of Butterworth filters is nonlinear (like all IIR filters except Bessel), and group delay actually becomes more variable with higher order.

---

**5. A notch filter has transfer function with zeros at z = e^(±jω₀). Where should the poles be placed for a narrow notch (high Q)?**

- A. At z = 0
- B. On the unit circle at e^(±jω₀)
- C. Just inside the unit circle at r·e^(±jω₀) where r is close to 1
- D. At z = 1 (DC)

**Answer**: C | **Explanation**: Poles must be placed just inside the unit circle (r < 1, r ≈ 1) at the same angle as the zeros. This provides: (1) deep notch at ω₀ (zeros cause zero response), (2) pole placement inside unit circle for stability, (3) r close to 1 for narrow notch (pole nearly cancels zero's effect away from ω₀). If poles were at z = 0, the notch would be very wide. If poles were on the unit circle, the system would be marginally stable.

---

**6. Which IIR filter type has the SHARPEST transition band for a given order n?**

- A. Butterworth
- B. Chebyshev Type I
- C. Chebyshev Type II
- D. Elliptic

**Answer**: D | **Explanation**: For a given order n, the elliptic filter has the sharpest transition from passband to stopband because it allows equiripple in BOTH passband and stopband. This is the most efficient design in terms of the order required to meet given specifications. The order required for the same specs: Elliptic < Chebyshev I/II < Butterworth (Butterworth is least efficient, requiring highest order).

---

**7. In Direct Form II (canonical form) implementation of an IIR filter:**

- A. The number of delay elements equals len(b) + len(a) − 1
- B. The number of delay elements equals max(len(b), len(a)) − 1
- C. The number of delay elements equals len(a) − 1 only
- D. There are no delay elements (all computations are instantaneous)

**Answer**: B | **Explanation**: In Direct Form II, the number of state variables (delay elements) equals max(len(b), len(a)) − 1. This is because the input and output delays are combined into a single delay line. In contrast, Direct Form I requires len(b) + len(a) − 1 delays. Direct Form II is more efficient in terms of memory storage.

---

**8. The group delay of a filter at frequency ω₀ is defined as:**

- A. −d|H(ω)|/dω at ω₀
- B. −d∠H(ω)/dω at ω₀
- C. |H(ω)|/∠H(ω) at ω₀
- D. d∠H(ω)/dω at ω₀

**Answer**: B | **Explanation**: Group delay τ_g(ω) = −d∠H(ω)/dω. It represents the average delay experienced by a narrowband signal centered at ω. For linear phase systems where ∠H(ω) = −ωτ, the group delay is constant (τ_g = τ), meaning all frequency components are delayed by the same amount and the waveform shape is preserved. Negative group delay (from nonlinear phase) causes phase distortion.

---

**9. For a Type I FIR filter (M odd, symmetric impulse response), which of the following is always true?**

- A. The filter has no zeros at z = −1 (ω = π)
- B. The filter has linear phase
- C. The filter is always highpass
- D. The filter cannot have even symmetry

**Answer**: B | **Explanation**: Type I FIR filters (M odd, h[n] = h[M−1−n]) have exactly linear phase. Type II FIR filters (M even, symmetric) have zeros at z = −1 (ω = π), which means they cannot implement highpass or bandstop filters. The question asks which is ALWAYS true: only B is always true for Type I. Type I filters can be LP, HP, BP, or BS depending on their coefficients.

---

**10. A 50 Hz powerline interference is contaminating an ECG recording at fs = 250 Hz. After applying a 2nd-order Butterworth lowpass filter with fc = 40 Hz, the interference:**

- A. Is completely removed because 50 Hz > 40 Hz (in stopband)
- B. Is still present due to filter transition band and limited attenuation
- C. Is amplified by the filter
- D. Cannot be analyzed without knowing the filter order

**Answer**: B | **Explanation**: At fs = 250 Hz, the normalized frequency for 50 Hz is ω_N = 50/125 = 0.4π rad/sample. A Butterworth filter with fc = 40 Hz has normalized cutoff ω_c = 40/125 = 0.32π. At 50 Hz (ω = 0.4π), the filter provides SOME attenuation (typically 20-40 dB depending on order), but not complete removal. For complete 50 Hz rejection, you need a notch filter specifically designed at 50 Hz. The answer is B — the interference is attenuated but not completely removed.

---

## Short Answer (5 questions, 4 points each = 20 points)

**1. A FIR filter with h[n] = {1/3, 1/3, 1/3} for n=0,1,2 is applied to x[n] = {1, 2, 3, 4}. Compute y[n] by hand and explain what type of filter this is.**

**Answer**: This is a 3-point moving average filter.

y[n] = x[n] * h[n] = Σ x[k]·h[n−k]

Output:
- n=0: y[0] = x[0]·h[0] = 1·(1/3) = 1/3
- n=1: y[1] = x[0]·h[1] + x[1]·h[0] = 1·(1/3) + 2·(1/3) = 1
- n=2: y[2] = x[0]·h[2] + x[1]·h[1] + x[2]·h[0] = 1·(1/3) + 2·(1/3) + 3·(1/3) = 2
- n=3: y[3] = x[1]·h[2] + x[2]·h[1] + x[3]·h[0] = 2·(1/3) + 3·(1/3) + 4·(1/3) = 3
- n=4: y[4] = x[2]·h[2] + x[3]·h[1] = 3·(1/3) + 4·(1/3) = 7/3

This is a **lowpass filter** (3-point moving average). The frequency response is H(e^(jω)) = (1/3)(1 + 2cos(ω)) with maximum at ω=0 (DC gain=1) and zero at ω=π (if 2-term average). With 3 terms, it has moderate lowpass characteristics.

---

**2. Explain why elliptic filters can achieve the sharpest transition band but are generally not used in biomedical signal processing where waveform fidelity is critical.**

**Answer**: Elliptic filters achieve the sharpest transition band by allowing **equiripple** (alternating oscillations) in BOTH the passband and stopband. This means:
1. **Passband ripple**: The magnitude oscillates between 1 and 1−δ₁, causing distortion of signal amplitude
2. **Nonlinear phase**: The phase response is highly nonlinear (especially near the passband edge), causing different frequency components to be delayed by different amounts

For biomedical signals like ECG, EEG, and neural recordings, **waveform morphology** (the exact shape of QRS complexes, action potentials, evoked potentials) carries critical diagnostic information. Nonlinear phase causes **phase distortion** that changes the waveform shape, potentially masking or creating abnormal features.

**Butterworth filters** are preferred in BME because they have:
- Maximally flat passband (no ripple)
- Moderate transition band
- Predictable phase response

For applications where phase matters (ECG waveform analysis), **linear phase FIR filters** are preferred despite their higher computational cost.

---

**3. A patient monitor requires real-time ECG filtering with group delay < 20 ms. If fs = 500 Hz, what is the maximum group delay in samples? A 201-tap FIR filter with linear phase has group delay = (M−1)/2 = 100 samples. Is this acceptable?**

**Answer**: Maximum group delay in samples = 20 ms × 500 Hz = 10 samples.

The 201-tap FIR filter has group delay = (201−1)/2 = 100 samples = 100/500 = 0.2 seconds = 200 ms.

**This is NOT acceptable** — 200 ms >> 20 ms budget.

To meet the 20 ms (10 sample) budget, options include:
1. **Reduce FIR length**: M ≤ 2×10+1 = 21 taps (much less frequency selectivity)
2. **Use IIR filter**: A 4th-order Butterworth has group delay ≈ 4-8 samples depending on frequency
3. **Use offline processing**: If real-time is not required, longer filters are fine

For real-time ECG monitoring, **IIR filters** (Butterworth, Chebyshev) are preferred because they achieve the required frequency selectivity with far fewer samples of delay. For QRS detection where waveform shape matters less than peak timing, a linear-phase FIR is sometimes used with reduced order.

---

**4. Design a filter to remove 60 Hz powerline interference from an EEG recording at fs = 250 Hz, while preserving the alpha rhythm (8-13 Hz). Specify: filter type, order (if IIR), or length (if FIR), and cutoff frequencies.**

**Answer**: Recommended approach: **Bandstop (notch) filter** at 60 Hz.

**Option 1: IIR Notch (recommended)**
- Type: IIR second-order notch (two sections for 60 Hz and potentially 120 Hz harmonic)
- Center frequency: f₀ = 60 Hz
- Quality factor Q = f₀/BW. For narrow notch: Q = 30-60
- Bandwidth: BW ≈ 60/Q = 1-2 Hz
- This preserves alpha (8-13 Hz) completely while deeply attenuating 60 Hz
- Group delay: ~2-4 samples (≈8-16 ms at 250 Hz) — acceptable

**Option 2: FIR Bandstop**
- Length M ≈ 125-250 (depending on transition width)
- Cutoffs: 55 Hz (lower) and 65 Hz (upper)
- Transition width: 10 Hz
- Group delay: (M−1)/2 ≈ 62-125 samples (≈250-500 ms at 250 Hz)
- Too slow for real-time applications

**Option 3: Two-stage**
- Highpass filter: fc = 5 Hz (removes baseline and very low frequencies)
- Notch at 60 Hz (removes powerline specifically)
- Preserves full alpha band (8-13 Hz)

**Best choice**: IIR notch filter for real-time, or FIR if linear phase is critical and latency is acceptable.

---

**5. Explain the relationship between the filter order n and (a) transition band width, (b) stopband attenuation, (c) group delay. Which of these tradeoffs is most important for ECG QRS detection?**

**Answer**:

| Parameter | Effect of Increasing Order n |
|-----------|-------------------------------|
| Transition band | **Narrows** (sharper roll-off: ~6n dB/octave for Butterworth) |
| Stopband attenuation | **Increases** (approximately linearly: ~20n dB/decade for Butterworth) |
| Group delay | **Increases** (more samples of delay) |

**For ECG QRS detection** (identifying the sharp R-peak):

The **most critical** consideration is the **group delay/transition trade-off**:
- QRS complexes have energy up to 40 Hz with very sharp transitions
- A filter with narrow transition band (sharp cutoff) is needed to separate QRS from noise
- BUT: narrow transition band requires either high-order filter (large delay) OR IIR filter (nonlinear phase)

**Practical compromise**: 
- Use **IIR Butterworth** bandpass (4-8 Hz to 15-40 Hz) of order 4-6
- Accept some phase distortion (can be partially compensated with zero-phase filtering `filtfilt` if not strictly real-time)
- Group delay ~4-8 samples = 8-16 ms at 500 Hz — acceptable for most monitoring applications

The **phase response** matters less for QRS detection (which is based on amplitude threshold) than for applications like evoked potentials (EP) where waveform shape must be preserved.

---

## True or False (5 questions, 1 point each = 5 points)

**1. A Type II FIR filter (M even, symmetric) cannot be used to design a highpass filter.**
**Answer**: True. Type II FIR filters have zeros at z = −1 (ω = π) because of the symmetry constraint with even length. This means the frequency response at ω = π is always zero: H(e^(jπ)) = 0. Therefore Type II filters cannot implement highpass or bandstop filters — they are always lowpass or bandpass only.

---

**2. The bilinear transformation s = (2/T_s)(1 − z⁻¹)/(1 + z⁻¹) always preserves the frequency response shape of the analog filter.**
**Answer**: False. The bilinear transformation is nonlinear (it uses tan(ωT_s/2) instead of ωT_s/2), which causes **frequency warping**. High frequencies are compressed toward π. To compensate, we use **prewarping** — design the analog filter at prewarped frequencies ω_a = (2/T_s)tan(ω_dT_s/2). Without prewarping, the digital filter's cutoff frequencies will be distorted, especially near f_s/2.

---

**3. The impulse response of a causal IIR filter is theoretically infinite in duration but practically decays to near-zero after a few time constants.**
**Answer**: True. For stable IIR filters, h[n] → 0 as n → ∞ because all poles are inside the unit circle (|p| < 1). After approximately 5 time constants (where time constant τ = 1/|ln|p|| samples), the impulse response has decayed to < 1% of its peak. In practice, IIR filters are truncated when computing responses.

---

**4. Zero-phase filtering (applying a filter forward and backward) doubles the effective filter order and doubles the stopband attenuation.**
**Answer**: True (for magnitude, not phase). Applying `filtfilt` (forward + backward) doubles the filter order (from n to 2n), which doubles the roll-off rate (from 6n dB/octave to 12n dB/octave) and doubles the stopband attenuation (in dB). However, it completely eliminates phase shift (zero phase). The effective group delay is also zero. This is ideal for offline biomedical signal processing where real-time delay is not a constraint.

---

**5. In cascade (SOS) implementation, changing the order of second-order sections does not affect the overall frequency response.**
**Answer**: True. Cascade form: H(z) = H₁(z)·H₂(z)·...·H_K(z). Since multiplication is commutative, H₁·H₂ = H₂·H₁. However, changing the order CAN affect numerical performance (roundoff error accumulation) and the transient response (order of ringing). For maximum numerical stability, place sections with poles closest to the unit circle first (high Q sections first).

---

## Score Guide

| Score | Grade | Interpretation |
|-------|-------|---------------|
| 40–45 | A | Excellent! Ready for Week 10 (Biostatistics) |
| 32–39 | B | Good. Review missed questions. |
| 24–31 | C | Review Week 9 concepts before proceeding |
| < 24 | D | Re-do Week 9 code labs before moving on |

**Maintainer**: BME Bootcamp Agent | **Week 9** | **BMED2500: FIR/IIR Filters**
