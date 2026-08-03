# Week 18 Quiz — Medical Imaging II (MRI, Nuclear & Optical Imaging)

> **Course**: BMED3501/BMED3504 — Medical Imaging / Molecular Imaging  
> **Duration**: 45 minutes | **Total points**: 50 (MCQ: 30 pts, Short Answer: 20 pts)  
> **Instructions**: Scientific calculators permitted. Constants provided on last page.

---

## Part I: Multiple Choice Questions (30 points — 3 pts each)

**Q1.** The Larmor frequency for ¹H at 3.0 T is approximately:
- (A) 42.58 MHz
- (B) 63.9 MHz
- (C) 127.7 MHz
- (D) 298.1 MHz

**Q2.** In MRI, the T1 relaxation time represents:
- (A) Decay of transverse magnetization due to spin-spin interactions
- (B) Recovery of longitudinal magnetization to thermal equilibrium
- (C) Dephasing due to static magnetic field inhomogeneities
- (D) The decay of the free induction decay (FID) signal

**Q3.** The T2* (T2-star) relaxation time is:
- (A) Always equal to T2
- (B) Always shorter than T2 due to additional dephasing from field inhomogeneity
- (C) Longer than T2 because it includes spin echo contributions
- (D) Independent of the magnetic field strength B₀

**Q4.** The spatial resolution of PET is primarily limited by:
- (A) The speed of light in tissue
- (B) The positron range and non-colinearity of annihilation photons
- (C) The wavelength of 511 keV γ-photons
- (D) The gyromagnetic ratio of ¹⁸F

**Q5.** The SUV (standardized uptake value) for a tissue with activity concentration of 8 kBq/g, injected dose of 370 MBq, and patient weight of 80 kg is approximately:
- (A) 0.9
- (B) 1.7
- (C) 8.5
- (D) 17.2

**Q6.** After a 90° RF pulse in MRI, the transverse magnetization M_xy at time t = TE (where TE >> T2) is approximately:
- (A) M₀ (unchanged)
- (B) M₀ × e^(-TE/T1)
- (C) M₀ × e^(-TE/T2)
- (D) Approximately zero

**Q7.** In a spin echo MRI sequence, the 180° pulse:
- (A) Eliminates T1 relaxation effects
- (B) Refocuses spins dephased by static field inhomogeneities (but not T2 effects)
- (C) Increases the signal by a factor of 2
- (D) Reverses the direction of precession

**Q8.** The half-life of ¹⁸F used in FDG-PET is approximately:
- (A) 6 hours
- (B) 20 minutes
- (C) 68 minutes
- (D) 110 minutes

**Q9.** In T1-weighted MRI, fat appears bright because:
- (A) Fat has a very long T1, so it is still saturated at typical TR values
- (B) Fat has a very short T1, so it recovers most fully between TR intervals
- (C) Fat has a long T2, so it appears bright on T2W images
- (D) Fat contains gadolinium from the contrast agent

**Q10.** The primary advantage of OCT over other imaging modalities is:
- (A) Deep penetration (5-10 cm)
- (B) Micron-level axial resolution (3-10 μm)
- (C) No need for coupling medium
- (D) Ability to image through bone

---

## Part II: Short Answer Questions (20 points — 4 pts each)

**Q11.** State the Larmor equation and calculate the resonance frequency f₀ for ¹H at 1.5 T. If the B₀ field varies by ±5 ppm across a 1 mm voxel, what is the resulting frequency variation? (4 pts)

**Q12.** Compare the information content of the center vs. periphery of k-space in MRI. If you acquire only the outer 50% of k-space (zeroing the center), what happens to the image? (4 pts)

**Q13.** Write the SUV formula and calculate SUV for: tissue activity = 12 kBq/g, injected dose = 370 MBq, patient weight = 65 kg. What does an SUV > 4 suggest about the tissue? (4 pts)

**Q14.** Explain why PET requires coincidence detection rather than single γ-photon detection. What are the three types of coincidences in PET, and which one is the primary noise source? (4 pts)

**Q15.** Explain the principle of optical coherence tomography (OCT). Why is interferometry required rather than direct time-of-flight measurement (as in ultrasound)? (4 pts)

---

## Answer Key

### Part I: MCQ Answers

| Q | Answer | Topic |
|---|--------|-------|
| Q1 | **C** | f₀ = 42.58 × 3.0 = 127.7 MHz |
| Q2 | **B** | T1: recovery of M_z to M₀ (energy transfer to lattice) |
| Q3 | **B** | T2* < T2; T2* = T2 + dephasing from static B₀ inhomogeneities |
| Q4 | **B** | PET resolution limited by: positron range (~0.5 mm), non-colinearity (~0.5 mm), detector resolution (~4 mm) |
| Q5 | **B** | SUV = 8/(370/80000) = 8/0.004625 = 1.73 ≈ 1.7 |
| Q6 | **D** | M_xy decays with T2* during TE; at TE >> T2, M_xy ≈ 0 without spin echo |
| Q7 | **B** | 180° pulse reverses phase accrued from static inhomogeneities; true T2 decay is irreversible |
| Q8 | **D** | T½(¹⁸F) = 109.8 min ≈ 110 min |
| Q9 | **B** | Fat T1 ~ 250 ms (short) → full recovery at TR=500 ms → bright on T1W |
| Q10 | **B** | OCT provides 3-10 μm axial resolution (best of all clinical modalities); limited depth (1-3 mm) |

### Part II: Short Answer Model Answers

**Q11**: *Larmor equation: ω₀ = γB₀ or f₀ = (γ/2π)B₀ = 42.58 MHz/T × 1.5 T = 63.87 MHz. Field variation ±5 ppm = ±5×10⁻⁶ × 1.5 T = ±7.5 μT. Frequency variation = γ × ±7.5 μT = 2π × 42.58×10⁶ × 7.5×10⁻⁶ = ±2003 Hz ≈ ±2 kHz.*

**Q12**: *K-space center (low spatial frequencies) contains image contrast and SNR. K-space periphery (high spatial frequencies) contains edges, fine detail, resolution. If you zero the k-space center: you remove contrast information → the image has no brightness variation (all gray) but edges remain. If you zero the periphery: the image becomes blurry but retains contrast.*

**Q13**: *SUV = r_tissue / (D_inj/W) = 12 kBq/g / (370 MBq/65000 g) = 12 / (370/65000) = 12/0.00569 = 2109... wait: 370 MBq = 370000 kBq. SUV = 12 / (370000/65000) = 12/5.69 = 2.11. SUV > 4 suggests high suspicion for malignancy.*

**Q14**: *PET requires coincidence detection because: (1) a single 511 keV γ is non-directional and can't be localized; (2) two γs at 180° define a line of response (LOR) through the body. Three coincidence types: (1) True coincidences (both γs from same annihilation, no scatter) — the signal; (2) Scatter coincidences (one γ scattered, wrong LOR) — main noise source; (3) Random coincidences (unrelated γs within coincidence window) — background noise.*

**Q15**: *OCT uses low-coherence interferometry: light is split into reference and sample arms; backscattered light interferes with reference light; the interferogram is detected and FFT gives depth-resolved reflectivity. Direct time-of-flight is impossible because light travels 3×10⁸ m/s; for 1 mm depth: t = 2×0.001/3×10⁸ = 6.7 fs, requiring <10 fs resolution (impossible with electronic detectors). Interferometry measures optical path difference directly via the interference pattern, achieving μm resolution.*

---

## Constants and Equations (provided for quiz)

| Constant | Value |
|----------|-------|
| γ/2π (¹H) | 42.58 MHz/T |
| Speed of light c | 3×10⁸ m/s |
| Speed of sound in tissue | 1540 m/s |
| ¹⁸F half-life | 110 min |
| 511 keV γ attenuation (soft tissue) | μ ≈ 0.095 cm⁻¹ |
| OCT: δz ≈ 0.44·λ₀²/Δλ (n=1) | — |
| Larmor: ω₀ = γB₀ | f₀ = (γ/2π)·B₀ |
| SUV = r_tissue / (D_inj/W) | — |

---

## Bloom's Taxonomy Distribution

| Level | Questions | Points |
|-------|----------|--------|
| Knowledge (recall) | Q1, Q8, Q9 | 9 pts |
| Comprehension | Q2, Q3, Q7, Q10 | 12 pts |
| Application | Q4, Q5, Q6, Q11-15 | 29 pts |

---

## Common Student Errors

1. **Q2**: Confusing T1 (longitudinal recovery) with T2 (transverse decay). T1: energy transfer to lattice → M_z recovers. T2: spin-spin dephasing → M_xy decays.
2. **Q4**: Thinking PET resolution is limited by 511 keV γ wavelength (λ = 3.5 pm). Actually, resolution is limited by positron range (~0.5 mm) and non-colinearity (~0.5 mm).
3. **Q5**: SUV formula error — mixing MBq and kBq. Must convert: 370 MBq = 370,000 kBq.
4. **Q11**: Confusing ppm field variation with frequency variation. ppm × B₀ = ΔB, then Δf = γΔB.
5. **Q15**: Thinking OCT uses "time of flight" like ultrasound. It uses interferometry because light travel time is too fast for direct electronic measurement.
