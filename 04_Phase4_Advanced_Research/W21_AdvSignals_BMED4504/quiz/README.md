# Week 21 Self-Quiz — Advanced Biomedical Signals (BMED4504)

> **Format**: Self-assessment | **Time**: ~60 minutes  

---

## Part I: Multiple Choice Questions (10 × 3 pts = 30 pts)

**Q1.** The Discrete Wavelet Transform (DWT) decomposes a signal into:
- (A) Frequency bands only (like FFT)  
- (B) Time-frequency tiles (approximation + details at multiple scales)  
- (C) Time-domain samples  
- (D) Statistical moments  
- (E) Principal components

---

**Q2.** In the ICA (Independent Component Analysis) model x = As, the rows of matrix A represent:
- (A) Independent sources  
- (B) Mixing coefficients  
- (C) Unmixing coefficients  
- (D) Principal components  
- (E) Wavelet coefficients

---

**Q3.** In LMS adaptive filtering, the step size μ must satisfy:
- (A) μ > 2/λ_max (largest eigenvalue)  
- (B) 0 < μ < 2/λ_max  
- (C) μ = λ_max/2  
- (D) μ = 0 (no adaptation)  
- (E) μ can be any positive value

---

**Q4.** The Wigner-Ville Distribution (WVD) provides excellent time-frequency resolution but suffers from:
- (A) Low-frequency artifacts  
- (B) Cross-term interference  
- (C) Phase distortion  
- (D) Gibbs phenomenon  
- (E) Aliasing

---

**Q5.** Compressive sensing requires the signal to be:
- (A) Sampled at Nyquist rate  
- (B) Sparse in some basis  
- (C) Gaussian distributed  
- (D) Stationary  
- (E) Periodic

---

**Q6.** The Daubechies wavelet db4 has:
- (A) 2 vanishing moments, support length 3  
- (B) 4 vanishing moments, support length 7  
- (C) 8 vanishing moments, support length 15  
- (D) 1 vanishing moment, support length 1  
- (E) 16 vanishing moments, support length 31

---

**Q7.** In ICA, kurtosis measures:
- (A) Mean value  
- (B) Variance  
- (C) Non-Gaussianity (deviation from Gaussian distribution)  
- (D) Frequency content  
- (E) Spectral entropy

---

**Q8.** For a signal with frequency content from 0-50 Hz, the Nyquist sampling rate is:
- (A) 50 Hz  
- (B) 100 Hz  
- (C) 200 Hz  
- (D) 25 Hz  
- (E) 500 Hz

---

**Q9.** The Short-Time Fourier Transform (STFT) has a fundamental trade-off between:
- (A) Time and frequency resolution  
- (B) Amplitude and phase  
- (C) Signal and noise  
- (D) Linear and nonlinear processing  
- (E) Real and imaginary parts

---

**Q10.** RLS (Recursive Least Squares) typically converges in approximately:
- (A) 100-1000 × filter length iterations  
- (B) 1 × filter length iterations  
- (C) 10 × filter length iterations  
- (D) Filter length squared iterations  
- (E) Does not converge

---

## Part II: Short Answer Questions (5 × 6 pts = 30 pts)

**SQ1.** Explain the difference between DWT and CWT. When would you use each one?

---

**SQ2.** In the cocktail party problem (separating multiple voices), why does ICA work better than PCA?

---

**SQ3.** Derive the stability condition for the LMS algorithm: 0 < μ < 2/λ_max

---

**SQ4.** Explain what the Restricted Isometry Property (RIP) means in compressive sensing and why it matters for signal reconstruction.

---

**SQ5.** A 64-channel EEG is sampled at 500 Hz for 10 minutes. Calculate the data size in megabytes. If we use compressive sensing with 20× compression, how many measurements per channel are needed?

---

## Part III: Problem-Solving (2 × 20 pts = 40 pts)

**PQ1.** [20 pts] **LMS Adaptive Filter Design**

Design an LMS adaptive filter for removing 60 Hz power line noise from an ECG signal.

(a) [4 pts] If the autocorrelation matrix R = E[xx^T] has eigenvalues λ = [0.1, 0.5, 1.0, 2.0], what is the maximum allowable step size μ for stability?

(b) [4 pts] If μ = 0.05 and filter length M = 32, estimate the steady-state MSE.

(c) [4 pts] How does increasing μ affect: (i) convergence speed, (ii) steady-state error?

(d) [4 pts] Design an alternative approach using wavelet denoising instead of adaptive filtering. What wavelet would you use and why?

(e) [4 pts] Simulate and compare the SNR improvement of LMS vs. fixed FIR filter for this application.

---

**PQ2.** [20 pts] **Compressive Sensing for EEG**

A 128-channel EEG system with 1000 Hz sampling rate needs to reduce power consumption for wireless transmission. You want to use compressive sensing with 10× compression.

(a) [4 pts] Calculate the data rate at 1000 Hz × 128 channels × 16-bit samples.

(b) [4 pts] With 10× compression, what is the target compressed data rate?

(c) [4 pts] If EEG signals are sparse in the wavelet domain with K = 50 non-zero coefficients per channel, how many measurements m are needed per channel for reliable reconstruction?

(d) [4 pts] Design a measurement matrix Φ. What properties must it have? What is the RIP condition?

(e) [4 pts] If you use OMP for reconstruction with K = 50, how many iterations are needed? What is the computational complexity?

---

## Answer Key

### MCQ Answers

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | DWT produces approximation (low-freq) and detail (high-freq) coefficients at multiple scales |
| 2 | **B** | A contains mixing coefficients defining how sources combine to form observations |
| 3 | **B** | LMS stability requires 0 < μ < 2/λ_max |
| 4 | **B** | Cross-terms appear between multiple signal components in WVD |
| 5 | **B** | Compressed sensing requires sparsity in some representation basis |
| 6 | **B** | dbN has N vanishing moments and support length 2N-1 |
| 7 | **C** | Kurtosis measures deviation from Gaussian (κ = 0 for Gaussian) |
| 8 | **B** | f_s ≥ 2 × f_max = 2 × 50 = 100 Hz |
| 9 | **A** | Heisenberg uncertainty: can't simultaneously have perfect time and frequency resolution |
| 10 | **B** | RLS converges in order M iterations (vs. LMS which needs 100-1000M) |

### Short Answer Solutions

**SQ1**: DWT uses dyadic scales and downsampling (O(N) computation), best for denoising/compression. CWT uses continuous scales without downsampling (redundant, O(N²)), best for time-frequency visualization.

**SQ2**: PCA finds uncorrelated (not independent) directions. ICA maximizes statistical independence (non-Gaussianity), which is required to separate physically distinct sources.

**SQ3**: From error correlation: E[w(n+1) - w_opt]² = (1 - 2μλ + μ²λ²) × previous error. For convergence: |1 - 2μλ| < 1 → 0 < μ < 2/λ_max.

**SQ4**: RIP: For all K-sparse vectors x, (1-δ)||x||² ≤ ||Φx||² ≤ (1+δ)||x||². RIP ensures unique reconstruction of sparse signals from compressed measurements.

**SQ5**: Data = 500 × 60 × 128 × 16 bits = 73,728,000 bits = 9.2 MB per minute. With 20× compression: m = N/20 = 500/20 = 25 measurements per channel.

### Problem Solutions

**PQ1**: (a) λ_max = 2.0, μ_max = 2/2.0 = 1.0. (b) MSE_ss ≈ μσ_x²M/4 ≈ 0.05 × 1 × 32/4 = 0.4. (c) Higher μ = faster convergence but higher steady-state error. (d) Use db4 or sym4 wavelet; captures multi-scale features of ECG morphology.

**PQ2**: (a) 1000 × 128 × 16 = 2.048 Mbps. (b) 204.8 kbps. (c) m ≥ c × K × log(N/K) ≈ 10 × 50 × log(500/50) ≈ 1150 measurements (but practically m = N/10 = 50 with Gaussian matrix). (d) Random Gaussian or Bernoulli matrix; RIP satisfied with m ≥ cK log(N/K). (e) 50 iterations; O(mKN) complexity.

---

## Self-Scoring Guide

| Score | Grade | Interpretation |
|-------|-------|-----------------|
| 85-100 | A | Excellent |
| 70-84 | B | Good |
| 55-69 | C | Satisfactory |
| < 55 | D | Needs review |
