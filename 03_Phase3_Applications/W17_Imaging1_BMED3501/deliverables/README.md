# Week 17 Deliverables — Medical Imaging I (X-ray, CT, Ultrasound)

> **Course**: BMED3501 — Medical Imaging  
> **Due**: End of Week 17 | **Format**: PDF or Jupyter Notebook  
> **Total points**: 100 (problem set: 60 pts, imaging analysis: 40 pts)

---

## Assignment 1: Imaging Physics Problem Set (60 points)

### Section A: X-ray Physics (20 points)

**Q1** (5 pts) — Lambert-Beer Law  
X-rays of initial intensity I₀ = 100 units pass through 8 cm of muscle (μ = 0.22 cm⁻¹).

(a) Calculate the transmitted intensity I using the Lambert-Beer law.  
(b) What fraction of X-rays are absorbed in the muscle?  
(c) Calculate the half-value layer (HVL) for muscle.  
(d) A CT scanner uses 120 kVp X-rays with effective energy ~50 keV. If the HVL for soft tissue is ~3.2 cm, what is the effective μ?

**Q2** (5 pts) — Hounsfield Units  
Calculate the Hounsfield units for the following tissues:
- Compact bone: μ_bone = 0.65 cm⁻¹
- Lung: μ_lung = 0.18 cm⁻¹
- Fat: μ_fat = 0.20 cm⁻¹

Given: μ_water = 0.20 cm⁻¹. Classify each tissue as having HU below or above water.

**Q3** (5 pts) — X-ray interactions  
Compare the photoelectric effect and Compton scattering:
(a) Which effect dominates at 30 keV? At 100 keV?  
(b) Why does photoelectric effect create image contrast in radiographs?  
(c) How does Compton scattering reduce image quality?  
(d) A dental X-ray uses 60 kVp (~40 keV effective). Is this optimal for soft tissue contrast or bone contrast? Explain.

**Q4** (5 pts) — X-ray spectrum  
Sketch a typical diagnostic X-ray spectrum (intensity vs. energy). Label:
- Bremsstrahlung continuum
- Characteristic peaks (Kα, Kβ)
- Effective energy
- How does increasing kVp affect the spectrum?

---

### Section B: CT Physics (20 points)

**Q5** (5 pts) — Filtered Back Projection  
(a) State the Fourier Slice Theorem in your own words.  
(b) Why does simple back projection (without filtering) produce a blurry image?  
(c) What is the purpose of the ramp filter |ω| in FBP?  
(d) Write the FBP reconstruction formula (you may use symbols for the transforms).

**Q6** (5 pts) — CT artifacts  
Name and explain the mechanism of two of the following CT artifacts:
- Beam hardening artifact
- Partial volume artifact
- Ring artifact
For each, describe one correction method.

**Q7** (5 pts) — CT dose  
A chest CT scan delivers CTDI_vol = 20 mGy. The effective dose D_eff is approximately D_eff = 0.017 × CTDI_vol (mSv) for chest CT.

(a) Calculate D_eff for this scan.  
(b) Compare this to natural background radiation (~2.4 mSv/year in Hong Kong).  
(c) What techniques can reduce CT dose while maintaining diagnostic image quality?

**Q8** (5 pts) — CT numbers  
A CT scan shows a liver lesion with HU = +65 and surrounding liver tissue with HU = +55.

(a) What is the contrast between the lesion and liver?  
(b) If contrast resolution requires CNR > 3 for detection, what is the minimum noise level σ needed?  
(c) Would this lesion be more visible on CT or MRI? Explain your reasoning.

---

### Section C: Ultrasound Physics (20 points)

**Q9** (5 pts) — Acoustic impedance  
Calculate the acoustic impedance mismatch between:
(a) Fat (ρ = 925 kg/m³, c = 1450 m/s) and muscle (ρ = 1080 kg/m³, c = 1580 m/s)  
(b) Muscle and cortical bone (ρ = 1900 kg/m³, c = 3600 m/s)  
(c) For case (b), what fraction of ultrasound energy is reflected at the muscle-bone interface?

**Q10** (5 pts) — Ultrasound resolution  
A 7.5 MHz linear array transducer is used for thyroid imaging (depth ~3 cm).

(a) Calculate the wavelength λ in soft tissue.  
(b) What is the theoretical axial resolution?  
(c) If the pulse contains 4 cycles, what is the axial resolution?  
(d) Why might the practical resolution be worse than the theoretical value?

**Q11** (5 pts) — Doppler equation  
Blood flow in the carotid artery is measured with:
- Transducer frequency f_t = 5 MHz
- Doppler angle θ = 60°
- Measured Doppler shift Δf = 3.5 kHz

(a) Calculate the blood flow velocity v.  
(b) If θ is reduced to 30°, what is the new Δf?  
(c) Why is it important to measure θ accurately for velocity calculations?

**Q12** (5 pts) — Aliasing in pulsed Doppler  
A pulsed Doppler system has:
- PRF (pulse repetition frequency) = 8 kHz
- f_t = 3 MHz
- θ = 0°

(a) Calculate the maximum measurable velocity (Nyquist limit).  
(b) If actual blood velocity is 1.5 m/s, will aliasing occur?  
(c) How can you redesign the system to measure velocities up to 2 m/s without aliasing?

---

## Assignment 2: Ultrasound Imaging Analysis (40 points)

### Overview
Using the provided ultrasound image dataset (or a simulated dataset), perform a quantitative analysis and write a 2-3 page technical report.

### Dataset Options (choose one)

**Option A: Clinical Ultrasound Image Analysis**  
Analyze a provided B-mode ultrasound image of the liver or thyroid:
1. Measure echogenicity of the target structure vs. surrounding tissue
2. Calculate contrast-to-noise ratio (CNR)
3. Assess image quality (near-field vs. far-field TGC effectiveness)
4. Identify artifacts (shadowing, reverberation, etc.)
5. Discuss clinical implications

**Option B: Doppler Ultrasound Velocity Analysis**  
Analyze a spectral Doppler waveform:
1. Identify peak systolic velocity (PSV) and end-diastolic velocity (EDV)
2. Calculate resistive index (RI = (PSV-EDV)/PSV) and pulsatility index (PI)
3. Classify the waveform (low-resistance vs. high-resistance)
4. Discuss clinical applications (e.g., carotid stenosis grading)

**Option C: CT Angiography Case Study**  
Analyze a CT angiography case:
1. Identify and label key anatomical structures in axial slices
2. Measure Hounsfield units of plaque vs. lumen vs. contrast
3. Assess image quality (spatial resolution, contrast, noise)
4. Identify one artifact and explain its mechanism
5. Discuss radiation dose considerations

### Report Format
- Title, author, affiliation
- Abstract (100 words)
- Introduction
- Methods
- Results (with figures and quantitative data)
- Discussion
- References

### Grading Rubric

| Criterion | Excellent (A) | Good (B) | Satisfactory (C) | Incomplete (D/F) |
|-----------|--------------|---------|-----------------|-----------------|
| Quantitative analysis (20 pts) | All calculations correct | Minor errors | Partial | Missing |
| Image interpretation (10 pts) | Detailed, insightful | Accurate | Basic | Absent |
| Technical writing (10 pts) | Clear, well-organized | Good | Adequate | Poor |

---

## Submission Instructions

1. **Problem Set**: PDF or handwritten scan with all working shown.
2. **Imaging Analysis Report**: PDF with embedded figures.
3. **Code**: Python/Jupyter notebook (.ipynb) for computational sections.

**Late policy**: 10% per day. Max 3 days.
