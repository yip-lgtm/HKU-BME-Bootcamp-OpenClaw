# Week 17 Quiz — Medical Imaging I (X-ray, CT, Ultrasound)

> **Course**: BMED3501 — Medical Imaging  
> **Duration**: 45 minutes | **Total points**: 50 (MCQ: 30 pts, Short Answer: 20 pts)  
> **Instructions**: Scientific calculators permitted. No smartphones during exam.

---

## Part I: Multiple Choice Questions (30 points — 3 pts each)

**Q1.** The linear attenuation coefficient μ for X-rays in compact bone at 60 keV is approximately:
- (A) 0.02 cm⁻¹
- (B) 0.20 cm⁻¹
- (C) 0.65 cm⁻¹
- (D) 2.50 cm⁻¹

**Q2.** The Hounsfield unit for air is:
- (A) -100
- (B) -500
- (C) -1000
- (D) 0

**Q3.** The photoelectric effect in X-ray imaging:
- (A) Dominates at high photon energies
- (B) Is proportional to Z³/E³
- (C) Is independent of atomic number
- (D) Is the main cause of scatter radiation

**Q4.** In CT, the ramp filter |ω| in filtered back projection:
- (A) Smooths the image to reduce noise
- (B) Amplifies high-frequency components to compensate for blurring
- (C) Removes low-frequency artifacts
- (D) Is only needed for iterative reconstruction

**Q5.** The acoustic impedance Z of soft tissue (ρ = 1060 kg/m³, c = 1540 m/s) is approximately:
- (A) 0.0004 MRayl
- (B) 1.63 MRayl
- (C) 6.8 MRayl
- (D) 1600 MRayl

**Q6.** Ultrasound is totally reflected at the tissue-air interface because:
- (A) The frequency is too high
- (B) The acoustic impedance mismatch is extremely large (Z ≈ 0.0004 MRayl vs. ~1.63 MRayl)
- (C) The wavelength is too small
- (D) The speed of sound in air is higher than in tissue

**Q7.** The Doppler frequency shift Δf for blood flow at velocity v = 0.5 m/s, f_t = 5 MHz, θ = 0° is approximately:
- (A) 650 Hz
- (B) 1.3 kHz
- (C) 3.2 kHz
- (D) 6.5 kHz

**Q8.** Aliasing in pulsed Doppler ultrasound occurs when:
- (A) The Doppler angle is greater than 60°
- (B) The Doppler shift exceeds half the PRF (Nyquist limit)
- (C) The transducer frequency is too low
- (D) The depth of the sample volume is greater than 10 cm

**Q9.** A CT scan shows a structure with HU = +800. This is most likely:
- (A) Fat
- (B) Muscle
- (C) Blood
- (D) Compact bone

**Q10.** The axial resolution of ultrasound is determined by:
- (A) Beam width
- (B) The wavelength λ (and thus frequency f_t)
- (C) The depth of imaging
- (D) The PRF

---

## Part II: Short Answer Questions (20 points — 4 pts each)

**Q11.** State the Lambert-Beer law for X-ray attenuation. If a 5 cm thick layer of muscle (μ = 0.22 cm⁻¹) reduces X-ray intensity from 100 to 33 units, confirm that this is consistent with the law. (4 pts)

**Q12.** Explain the Fourier Slice Theorem and why it is fundamental to CT image reconstruction. How does filtered back projection use this theorem? (4 pts)

**Q13.** A 4 MHz ultrasound transducer is used to image blood flow at θ = 30°. If the measured Δf = 2.5 kHz, calculate the blood flow velocity. Show your working. (4 pts)

**Q14.** Define acoustic impedance Z and write the reflection coefficient formula at an interface between two media with impedances Z₁ and Z₂. Calculate R for muscle (Z = 1.71 MRayl) and bone (Z = 6.8 MRayl). (4 pts)

**Q15.** Explain two advantages of ultrasound over CT/X-ray imaging and two limitations. (4 pts)

---

## Answer Key

### Part I: MCQ Answers

| Q | Answer | Topic |
|---|--------|-------|
| Q1 | **C** | Bone μ ≈ 0.65 cm⁻¹ (high Z, high density) |
| Q2 | **C** | Air HU = -1000 (μ ≈ 0) |
| Q3 | **B** | Photoelectric: μ ∝ Z³/E³ |
| Q4 | **B** | Ramp filter: |ω| compensates for blur in back projection |
| Q5 | **B** | Z = 1060×1540 = 1.63×10⁶ kg/m²·s = 1.63 MRayl |
| Q6 | **B** | Z_air ≈ 0.0004 vs. Z_tissue ≈ 1.63 → near-total reflection |
| Q7 | **C** | Δf = 2×5×10⁶×0.5×1/1540 = 3250 Hz ≈ 3.2 kHz |
| Q8 | **B** | Aliasing: Δf > PRF/2 |
| Q9 | **D** | HU +800 → compact bone |
| Q10 | **B** | Axial resolution = λ/2 = c/(2f) |

### Part II: Short Answer Model Answers

**Q11**: *Lambert-Beer: I = I₀e^(-μx). For x=5 cm, μ=0.22 cm⁻¹: I = 100×e^(-0.22×5) = 100×e^(-1.1) = 100×0.332 = 33.2 units. ✓ Consistent with the law.*

**Q12**: *The Fourier Slice Theorem states that the 1D Fourier transform of a projection at angle θ equals the 2D Fourier transform of the object evaluated along a line through the origin at angle θ. FBP uses this: take projections at many angles, FFT each → fill 2D frequency domain, multiply by ramp filter (|ω|), inverse FFT. The ramp filter compensates for the smearing that occurs in unfiltered back projection.*

**Q13**: *v = Δf·c/(2f_t·cosθ) = 2500×1540/(2×4×10⁶×0.866) = 0.56 m/s*

**Q14**: *Z = ρ·c (MRayl). R = ((Z₂-Z₁)/(Z₂+Z₁))². For muscle-bone: R = ((6.8-1.71)/(6.8+1.71))² = (5.09/8.51)² = 0.358 = 35.8%. So 36% of ultrasound energy is reflected at the muscle-bone interface (near-total reflection for bone).*

**Q15**: *Advantages: (1) No ionizing radiation — safe for fetus, repeated use; (2) Real-time imaging — cardiac, fetal; (3) Portable, inexpensive; (4) Doppler capability. Limitations: (1) Cannot penetrate bone/air — lungs, brain behind bone impossible; (2) Operator-dependent; (3) Limited resolution at depth; (4) Limited field of view.*

---

## Bloom's Taxonomy Distribution

| Level | Questions | Points |
|-------|----------|--------|
| Knowledge (recall) | Q1, Q2, Q5 | 9 pts |
| Comprehension | Q3, Q6, Q9 | 9 pts |
| Application | Q4, Q7, Q8, Q10, Q11-15 | 32 pts |

---

## Common Student Errors

1. **Q7**: Forgetting the factor of 2 in the Doppler equation (2f_t, not f_t). Confusing θ with sinθ.
2. **Q11**: Confusing μ (linear attenuation coefficient) with μ/ρ (mass attenuation coefficient).
3. **Q14**: Not squaring the numerator in R = ((Z₂-Z₁)/(Z₂+Z₁))².
4. **Q15**: Stating "ultrasound has no radiation" as both an advantage AND limitation — they're opposites, not the same.
