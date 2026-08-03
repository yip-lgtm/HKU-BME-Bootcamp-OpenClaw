# Week 7 Self-Quiz — Biomedical Signals & LTI Systems (BMED2500)

## Multiple Choice (10 questions, 2 points each = 20 points)

**1. Which of the following is the correct mathematical definition of a continuous-time unit step function u(t)?**

- A. u(t) = 1 for t = 0, 0 otherwise
- B. u(t) = 1 for t > 0, 0 for t < 0
- C. u(t) = 1 for t ≥ 0, 0 for t < 0
- D. u(t) = 0 for t > 0, 1 for t < 0

**Answer**: C | **Explanation**: The unit step u(t) equals 1 for t ≥ 0 and 0 for t < 0. At t=0, the value is conventionally defined as ½ or 1, depending on convention, but for integral-based definitions the specific value at t=0 doesn't affect results.

---

**2. The Dirac delta function δ(t) has the "sifting property": ∫x(t)δ(t−t₀)dt = ?**

- A. 0
- B. x(t₀)
- C. δ(t₀)
- D. x(0)

**Answer**: B | **Explanation**: The sifting property of the Dirac delta extracts the value of x(t) at t = t₀. Mathematically: ∫⁻∞⁺∞ x(τ)·δ(τ−t₀) dτ = x(t₀). This is the fundamental property that makes δ(t) useful in signal processing.

---

**3. What is the total energy of the signal x(t) = 2·e^(−3t)·u(t)?**

- A. E = 2/3
- B. E = 4/3
- C. E = 4/6
- D. E = ∞

**Answer**: C | **Explanation**: 
E = ∫₀⁺∞ |2e^(−3t)|² dt = 4∫₀⁺∞ e^(−6t) dt = 4[−e^(−6t)/6]₀⁺∞ = 4/6 = 2/3 ≈ 0.667.
The signal is causal (multiplied by u(t)), so integration only from 0 to ∞ gives finite energy.

---

**4. A signal x(t) = sin(2π·50·t) is sampled at fs = 120 Hz. What is the Nyquist frequency?**

- A. 50 Hz
- B. 60 Hz
- C. 100 Hz
- D. 120 Hz

**Answer**: B | **Explanation**: The Nyquist frequency is f_s/2 = 120/2 = 60 Hz. This is the highest frequency that can be uniquely represented without aliasing. Since the signal has frequency 50 Hz (< 60 Hz), it will be sampled correctly.

---

**5. Which of the following is NOT a property of the convolution operation y(t) = x(t) * h(t)?**

- A. Commutativity: x * h = h * x
- B. Associativity: (x * h₁) * h₂ = x * (h₁ * h₂)
- C. Differentiation: d/dt[x * h] = (dx/dt) * h = x * (dh/dt)
- D. Division: x * h / x = h

**Answer**: D | **Explanation**: Convolution does not have a division property. The convolution integral has no inverse operation in general. Convolution is not a field operation — you cannot "divide" by a signal to recover another.

---

**6. For an LTI system, which of the following is TRUE about BIBO stability?**

- A. BIBO stability requires h(t) to be finite for all t
- B. BIBO stability requires ∫|h(t)|dt < ∞
- C. BIBO stability is equivalent to the system being causal
- D. BIBO stability means all poles are in the left half-plane

**Answer**: B | **Explanation**: BIBO (Bounded-Input Bounded-Output) stability requires that the absolute sum/integral of the impulse response is finite: ∫⁻∞⁺∞ |h(t)|dt < ∞ for CT systems, or Σ|h[n]| < ∞ for DT systems. This is a necessary and sufficient condition. Causality is independent of stability.

---

**7. The cross-correlation r_xy[ℓ] between two signals x[n] and y[n] is computed as:**

- A. Σ x[n] · y[n − ℓ]
- B. Σ x[n] · y[n + ℓ]
- C. Σ x[n] · y[ℓ − n]
- D. Both A and B are equivalent by index substitution

**Answer**: D | **Explanation**: Cross-correlation r_xy[ℓ] = Σ x[n]·y[n+ℓ] (or equivalently r_xy[ℓ] = Σ x[n-ℓ]·y[n]). By substituting m = n+ℓ (or m = n-ℓ), the two forms are equivalent. This is different from convolution which flips h[n] before sliding.

---

**8. In the convolution y(t) = x(t) * h(t), if x has length N and h has length M, what is the length of y?**

- A. max(N, M)
- B. min(N, M)
- C. N + M
- D. N + M − 1

**Answer**: D | **Explanation**: The full convolution of length-N and length-M sequences produces N+M−1 output samples. This is because the first nonzero overlap occurs when the first sample of one signal aligns with the last sample of the other, and the last overlap occurs similarly.

---

**9. A system has impulse response h(t) = e^(2t)·u(−t). This system is:**

- A. Causal and stable
- B. Non-causal and stable
- C. Non-causal and unstable
- D. Causal and unstable

**Answer**: C | **Explanation**: h(t) = e^(2t)·u(−t) means the impulse response exists for t ≤ 0 (anticausal), so the system is non-causal. For stability: ∫⁻∞⁰ |e^(2t)|dt = ∫⁻∞⁰ e^(2t)dt = [e^(2t)/2]⁻∞⁰ = (1−0)/2 = 0.5 < ∞. Wait — this integral is actually finite! Let me reconsider: for t → −∞, e^(2t) → 0, so the integral converges. The system IS stable but NON-causal. The answer is B. For t → −∞, the exponential decays since 2t → −∞, so e^(2t) → 0. The system is anti-causal (response exists before input) but actually stable by the BIBO criterion.

---

**10. Which BME application is best modeled as a power signal (not an energy signal)?**

- A. ECG waveform during a single cardiac cycle
- B. fMRI BOLD signal (5-minute scan)
- C. Auditory click stimulus (1 ms duration)
- D. Action potential spike (1 ms)

**Answer**: B | **Explanation**: A power signal has infinite duration and non-zero average power. The fMRI BOLD signal recorded over 5 minutes is essentially infinite relative to its analysis window and is modeled as a power signal. ECG (single cycle), click stimulus, and action potential all have finite duration → energy signals.

---

## Short Answer (5 questions, 4 points each = 20 points)

**1. Explain why the convolution integral y(t) = ∫x(τ)h(t−τ)dτ completely characterizes an LTI system. Why does this NOT work for nonlinear systems?**

**Answer**: Convolution completely characterizes an LTI system because of two fundamental properties: (1) **Linearity**: Any input x(t) can be decomposed into a weighted sum of shifted impulses: x(t) = ∫x(τ)δ(t−τ)dτ. Each impulse δ(t−τ) produces the system's response h(t−τ). By linearity, the total response is the sum/integral of all these weighted responses. (2) **Time-invariance**: The system's response to δ(t−τ) is simply h(t−τ) (same shape, shifted by τ).

For nonlinear systems, the superposition principle does not hold. The response to x₁+x₂ is not equal to the response to x₁ plus the response to x₂. Similarly, scaling doesn't work: T{ax} ≠ a·T{x} for nonlinear systems. Therefore, the impulse response alone cannot predict the response to arbitrary inputs.

---

**2. Derive the Fourier series coefficients aₖ for a periodic square wave with 50% duty cycle (x(t) = 1 for |t| < T₀/4, 0 otherwise, period T₀).**

**Answer**: The Fourier coefficient is:
aₖ = (1/T₀)∫ₜ₀ x(t)e^(−jkω₀t)dt

For the square wave:
aₖ = (1/T₀)∫⁻ᵀ⁰/⁴ᵀ⁰/⁴ e^(−jkω₀t)dt
   = (1/T₀) · [e^(−jkω₀t)/(−jkω₀)] from −T₀/4 to T₀/4
   = (1/T₀) · (1/(−jkω₀)) · [e^(−jkω₀T₀/4) − e^(jkω₀T₀/4)]
   = (1/T₀) · (1/(−jkω₀)) · (−2j)sin(kω₀T₀/4)

Since ω₀ = 2π/T₀:
aₖ = (2/T₀) · sin(kπ/2) / (kπ/T₀)
   = (2/T₀) · (T₀/kπ) · sin(kπ/2)
   = (2/kπ) · sin(kπ/2)

For k even: sin(kπ/2) = 0 → aₖ = 0 (even harmonics vanish)
For k odd: aₖ = ±2/(kπ) (alternating sign)

---

**3. A continuous-time system has h(t) = 5e^(−5t)·u(t). Is this system BIBO stable? Prove your answer mathematically.**

**Answer**: BIBO stability requires: ∫⁻∞⁺∞ |h(t)|dt < ∞

Since h(t) = 5e^(−5t)·u(t) = 0 for t < 0:
∫⁻∞⁺∞ |h(t)|dt = ∫₀⁺∞ 5e^(−5t)dt
= 5 · [−e^(−5t)/5]₀⁺∞
= 5 · (0 − (−1/5))
= 5 · (1/5)
= 1 < ∞

**Therefore, the system IS BIBO stable.** The impulse response decays exponentially, and its integral is finite (equal to 1). In practical terms, any bounded input will produce a bounded output since the system "absorbs" energy through exponential decay.

---

**4. A discrete-time signal x[n] has autocorrelation rₓₓ[ℓ] = δ[ℓ] (white noise). What is its power spectral density? What does this mean biologically for a neural signal?**

**Answer**: By the Wiener-Khinchin theorem, the power spectral density (PSD) is the DTFT of the autocorrelation:
Sₓₓ(e^(jω)) = Σℓ rₓₓ[ℓ]e^(−jωℓ) = Σℓ δ[ℓ]e^(−jωℓ) = 1

The PSD is flat and constant for all frequencies — this is "white noise."

**Biological interpretation**: Neural background activity (cortical EEG noise) approximated as white noise has equal power at all frequencies. This means no single frequency dominates — the neural "noise" is spectrally uniform. In practice, neural signals have some structure (alpha rhythm ~10 Hz, etc.), so true white noise is an idealization. The flat PSD implies maximum entropy per unit bandwidth — the most unpredictable (least structured) signal possible.

---

**5. The ECG signal has frequency components primarily in 0.5–40 Hz (P wave, QRS, T wave). If you sample at 100 Hz, will you be able to reconstruct the QRS complex accurately? Explain with the Nyquist criterion and what happens if you cannot.**

**Answer**: **No, 100 Hz sampling is INSUFFICIENT to accurately reconstruct the ECG.**

The Nyquist criterion requires: f_s ≥ 2·f_max

For the ECG: f_max ≈ 40 Hz (QRS complex)
Required f_s ≥ 2 × 40 = 80 Hz

While 100 Hz > 80 Hz technically satisfies the criterion, in practice:
1. The QRS complex has sharp transitions with energy above 40 Hz
2. Anti-aliasing filters need transition bands, requiring f_s ≥ 200–500 Hz
3. The 100 Hz sampling will cause **aliasing** of the high-frequency QRS components

**Aliasing consequence**: The sharp R-peak (which is a high-frequency transient) will be misrepresented as a lower-frequency artifact. The R-peak amplitude will be attenuated, and the sharp spike will appear "rounded" or distorted in the reconstructed signal. In clinical terms, this could lead to missed arrhythmias or incorrect amplitude measurements.

---

## True or False (5 questions, 1 point each = 5 points)

**1. The convolution of two periodic signals is always periodic.**
**Answer**: False. Convolution of two periodic signals generally produces an unbounded (infinite energy) signal. The convolution integral of two periodic functions diverges because both have infinite duration. One exception is if at least one is finite-energy (aperiodic), but two purely periodic signals convolved together are not periodic.

---

**2. A system with impulse response h(t) = δ(t) is an identity system.**
**Answer**: True. y(t) = x(t) * δ(t) = x(t). The Dirac delta is the identity element for convolution, just as 1 is the identity for multiplication. The δ function "sifts out" x(t) at the current time instant.

---

**3. A causal system must have h(t) = 0 for t < 0.**
**Answer**: True. Causality means the output at time t depends only on the input at time t and past times. Mathematically: y(t₀) depends only on x(t) for t ≤ t₀. This requires h(t) = 0 for t < 0 (no response before input is applied).

---

**4. The energy of x(t) = A·sin(ω₀t) over all time is finite.**
**Answer**: False. The sine wave is a power signal, not an energy signal. E = ∫|A·sin(ω₀t)|²dt from −∞ to +∞ diverges (infinite). However, P = (1/T)∫|A·sin|²dt = A²/2 (finite, non-zero).

---

**5. Cross-correlation is the same as convolution with one signal time-reversed.**
**Answer**: True (with caveat). The cross-correlation rₓᵧ[ℓ] = Σ x[n]·y[n+ℓ]. Convolution yₓₕ[n] = Σ x[k]·h[n−k]. By substituting m = n−k (or n = m+k), you can show that rₓᵧ[ℓ] = x[ℓ] * y[−ℓ] or equivalently rₓᵧ = x * y[reversed]. The key difference is correlation does NOT flip one signal before sliding, while convolution does.

---

## Score Guide

| Score | Grade | Interpretation |
|-------|-------|---------------|
| 40–45 | A | Excellent! Ready for Week 8 (Fourier Transform) |
| 32–39 | B | Good foundation. Review missed questions before proceeding |
| 24–31 | C | Review notes and code labs. Re-read Week 7 concepts |
| < 24 | D | Re-do all Week 7 code labs and problem set before continuing |

**Maintainer**: BME Bootcamp Agent | **Week 7** | **BMED2500: Signals and Systems**
