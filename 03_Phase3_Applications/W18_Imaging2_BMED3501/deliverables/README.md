# Week 18 Deliverables — Medical Imaging II (MRI, Nuclear & Optical Imaging)

> **Course**: BMED3501/BMED3504 — Medical Imaging / Molecular Imaging  
> **Due**: End of Week 18 | **Format**: PDF or Jupyter Notebook  
> **Total points**: 100 (problem set: 60 pts, imaging analysis: 40 pts)

---

## Assignment 1: MRI and PET Physics Problem Set (60 points)

### Section A: MRI Physics (20 points)

**Q1** (5 pts) — Larmor Frequency  
(a) Calculate the Larmor frequency ω₀ (in rad/s) and f₀ (in MHz) for ¹H at 3.0 T and 7.0 T. Use γ/2π = 42.58 MHz/T.  
(b) Why does the signal-to-noise ratio (SNR) in MRI increase approximately linearly with B₀?  
(c) What is the challenge of imaging at very high field strengths (7 T and above)?

**Q2** (5 pts) — T1 and T2 Relaxation  
(a) State the equations for T1 recovery and T2 decay of the magnetization vector.  
(b) For white matter (T1 = 600 ms, T2 = 70 ms at 1.5 T), calculate M_z/M₀ and M_xy/M₀ at t = TR = 600 ms (TE = 0) and t = TE = 80 ms (TR = 3000 ms).  
(c) Why is T1 of fat (~250 ms) much shorter than T1 of CSF (~3000 ms)?

**Q3** (5 pts) — K-space Encoding  
(a) State the relationship between k-space coordinates (k_x, k_y) and imaging gradients (G_x, G_y).  
(b) What information is encoded in the center of k-space vs. the periphery?  
(c) If a gradient G_x = 20 mT/m is applied for T_read = 8 ms during frequency encoding, what is the field of view (FOV) and pixel size Δx? Use γ/2π = 42.58 MHz/T.

**Q4** (5 pts) — Image Contrast  
Compare T1-weighted, T2-weighted, and FLAIR MRI sequences:
(a) What TR and TE values are used for each?  
(b) Which sequence would you use to detect: (i) acute multiple sclerosis plaques, (ii) brain tumors after gadolinium injection, (iii) acute stroke?  
(c) Explain why gadolinium contrast agents appear bright on T1W images.

---

### Section B: PET and Nuclear Imaging (20 points)

**Q5** (5 pts) — PET Physics  
(a) Write the decay equation for ¹⁸F. What is the half-life?  
(b) Explain the process of positron annihilation. Why are the two 511 keV γ-photons emitted at approximately 180° apart?  
(c) Name three factors that limit the spatial resolution in PET.

**Q6** (5 pts) — SUV Calculation  
FDG-PET scan parameters:
- Injected dose: 370 MBq
- Patient weight: 80 kg
- Tumor activity concentration: 8 kBq/g measured at 60 min post-injection

(a) Calculate the SUV.  
(b) Is this suspicious for malignancy (threshold SUV = 2.5)?  
(c) If the scan is performed at 90 min instead of 60 min, how does this affect the measured SUV (account for ¹⁸F decay with T½ = 110 min)?  
(d) Why is lean body mass (LBM) SUV sometimes preferred over weight-based SUV for obese patients?

**Q7** (5 pts) — PET Artifacts  
Name and explain the mechanism of two PET artifacts:
(a) Attenuation correction artifact  
(b) Motion artifact  
(c) Partial volume effect  
For each, describe the clinical impact and one correction strategy.

**Q8** (5 pts) — PET vs. SPECT  
Compare PET and SPECT for cardiac imaging (myocardial perfusion):
(a) What tracers are used in each?  
(b) Why does PET have superior sensitivity and resolution compared to SPECT?  
(c) What is the main advantage of SPECT that keeps it widely used clinically?

---

### Section C: Optical Imaging (20 points)

**Q9** (5 pts) — OCT Axial Resolution  
(a) The axial resolution of OCT is given by δz ≈ 0.44·λ₀²/Δλ. Calculate δz for a swept-source OCT system with λ₀ = 1310 nm and Δλ = 100 nm.  
(b) How does this compare to the axial resolution of a typical B-mode ultrasound system (λ = 0.3 mm at 5 MHz)?  
(c) Why can't OCT penetrate as deep as ultrasound?

**Q10** (5 pts) — Fluorescence Imaging  
(a) What is the Stokes shift in fluorescence? Why is a large Stokes shift important for in vivo imaging?  
(b) Compare near-infrared fluorescence (NIRF) window I (650-900 nm) vs. window II (1000-1700 nm) for deep tissue imaging.  
(c) Name one clinical application of fluorescence-guided surgery.

---

## Assignment 2: PET/MRI Imaging Analysis (40 points)

### Overview
Write a 2-3 page technical report analyzing a molecular imaging case.

### Option A: FDG-PET Cancer Staging  
Analyze a simulated FDG-PET/CT case:
1. Identify metabolically active lesions (SUV calculations)
2. Assess PET image quality (noise, artifacts)
3. Stage the cancer (TNM or Lugano classification)
4. Discuss treatment planning implications

### Option B: MRI Brain Tumor Characterization  
Analyze an MRI case with contrast-enhanced brain tumor:
1. Identify T1W, T2W, FLAIR, and post-gadolinium appearances
2. Calculate relative enhancement ratios
3. Characterize the lesion (high-grade vs. low-grade features)
4. Discuss differential diagnosis and biopsy planning

### Option C: Hybrid PET/MR Analysis  
Analyze a simulated PET/MR case (e.g., brain PET/MR for dementia):
1. Correlate FDG-PET metabolic findings with structural MRI
2. Quantify PET uptake (SUV, z-score vs. reference database)
3. Assess MRI sequences for atrophy patterns
4. Discuss integrated reporting for neurodegenerative disease

### Report Format
- Title, author, affiliation
- Abstract (100 words)
- Introduction
- Imaging findings (with figures)
- Discussion
- References

### Grading Rubric

| Criterion | Excellent (A) | Good (B) | Satisfactory (C) | Incomplete (D/F) |
|-----------|--------------|---------|-----------------|-----------------|
| Quantitative analysis (20 pts) | All calculations correct | Minor errors | Partial | Missing |
| Image interpretation (10 pts) | Detailed, insightful | Accurate | Basic | Absent |
| Clinical significance (10 pts) | Strong clinical context | Good | Adequate | Missing |

---

## Submission Instructions

1. **Problem Set**: PDF or handwritten scan showing all working.
2. **Imaging Analysis Report**: PDF with embedded figures and data tables.
3. **Code**: Python notebook (.ipynb) for computational sections.

**Late policy**: 10% per day. Max 3 days.
