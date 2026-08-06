# Deep Study Format Course Body
# Week 1: BME Introduction + Chemistry Foundations
## 生物醫學工程概論與化學基礎 — 第一週

---

# 🧠 5MM — Five Mental Models
## 五個核心心智模型

---

### Mental Model 1: The Henderson–Hasselbalch Buffer Framework
**心智模型一：Henderson–Hasselbalch 緩衝系統框架**

The Henderson–Hasselbalch equation (Hasselbalch 1916; Henderson 1908) is the single most powerful quantitative tool for understanding acid–base balance in biological systems. It transforms the equilibrium expression for any weak acid HA ⇌ H⁺ + A⁻ into a linear logarithmic form:

$$\text{pH} = \text{p}K_a + \log_{10}\frac{[\text{A}^-]}{[\text{HA}]}$$

Applied to blood (Hasselbalch 1916), with pK_a of carbonic acid = 6.10 (at 37 °C, 5% CO₂), and normal values [HCO₃⁻] = 24 mM, [CO₂] dissolved = 1.20 mM (PCO₂ = 40 mmHg × 0.0301 mM/mmHg):

$$\text{pH} = 6.10 + \log\frac{24}{1.20} = 6.10 + \log(20) = 6.10 + 1.301 = 7.40$$

**Why it matters in BME:** Every artificial kidney (dialyzer — Kolff 1943), every cardiopulmonary bypass circuit (Gibbon 1953), every extracorporeal membrane oxygenator (ECMO — Bartlett 1977) must maintain the [HCO₃⁻]/[CO₂] ratio within ±10% of the physiological value. The Stewart approach (Stewart 1983) provides an alternative strong-ion model, but for clinical decision-making, Henderson–Hasselbalch remains the lingua franca (Baron 2009).

---

### Mental Model 2: The Electronegativity-Driven Polarity Spectrum
**心智模型二：由電負度驅動的極性光譜**

Pauling (1932) established an electronegativity (χ) scale that governs bond polarity and intermolecular interactions. The numerical difference Δχ predicts bond character:

$$\Delta\chi < 0.5 \Rightarrow \text{Nonpolar covalent};\quad 0.5 < \Delta\chi < 1.7 \Rightarrow \text{Polar covalent};\quad \Delta\chi > 1.7 \Rightarrow \text{Ionic}$$

| Bond | χ₁ | χ₂ | Δχ | Character |
|------|------|------|------|-----------|
| C–C | 2.55 | 2.55 | 0.00 | Nonpolar |
| C–H | 2.55 | 2.20 | 0.35 | Nonpolar |
| C–O | 2.55 | 3.44 | 0.89 | Polar covalent |
| O–H | 3.44 | 2.20 | 1.24 | Polar covalent |
| Na–Cl | 0.93 | 3.16 | 2.23 | Ionic |

**Biological consequence:** The same scale explains water's hydrogen bonding, the hydrophobic effect (Tanford 1973), micelle formation (Hartley 1936), and the partition coefficient (LogP) that governs drug absorption (Lipinski 1997). For BME biomaterial design, surface χ determines protein adsorption and cell adhesion (Ratner 2004).

---

### Mental Model 3: The Hierarchical Levels of Protein Structure
**心智模型三：蛋白質結構的階層模型**

Protein architecture follows four ordered levels (Linderstrøm-Lang 1951; Anfinsen 1972):

| Level | Bonds/Forces | Example |
|-------|--------------|---------|
| **Primary (1°)** | Covalent peptide bonds (~350 kJ/mol) | Insulin sequence (Sanger 1955) |
| **Secondary (2°)** | H-bonds (~20 kJ/mol) | α-helix (Pauling 1951), β-sheet |
| **Tertiary (3°)** | Disulfide, ionic, hydrophobic, H-bonds | Myoglobin (Kendrew 1958) |
| **Quaternary (4°)** | Subunit interfaces | Hemoglobin tetramer (Perutz 1962) |

**Anfinsen's thermodynamic hypothesis (1972):** The native 3D structure is the global free-energy minimum under physiological conditions. This is why denaturation (e.g., by urea, heat, pH) is reversible — a foundational principle for BME applications such as protein-based biosensors and enzyme engineering.

$$\Delta G_{\text{folding}} = \Delta H - T\Delta S < 0$$

The hydrophobic effect drives folding by increasing entropy of released water (Kauzmann 1959; Dill 1990).

---

### Mental Model 4: Water as the Universal Biological Solvent
**心智模型四：水作為通用生物溶劑**

Water's anomalous properties arise from its H-bond network (Eisenberg & Kauzmann 1969):

$$c_p^{\text{water}} = 4.18 \;\text{J g}^{-1}\text{K}^{-1} \quad (\text{vs. ethanol: } 2.43)$$

$$\Delta H_{\text{vap}} = 44 \;\text{kJ/mol at } 100°\text{C}$$

$$\rho_{\text{ice}} = 0.917 \;\text{g/cm}^3 < \rho_{\text{water}} = 1.000 \;\text{g/cm}^3$$

**The hydrogen-bond network** is responsible for four biological "miracles": (1) high specific heat → thermal buffering for homeostasis; (2) ice floats → aquatic life survives under frozen ponds; (3) high heat of vaporization → evaporative cooling (sweating); (4) high dielectric constant (ε ≈ 78.5) → dissolves ionic species, weakening ionic bonds by factor 1/ε (Born 1920).

**BME relevance:** Tissue engineering scaffolds (Langer & Vacanti 1993) are designed with controlled hydrophilicity/hydrophobicity to regulate cell attachment, and the dielectric properties of water govern MRI contrast (Bloch 1946; Lauterbur 1973).

---

### Mental Model 5: The Drug Bioavailability Rules (Lipinski's Rule of Five)
**心智模型五：藥物生體可用率規則（Lipinski 五規則）**

Lipinski et al. (1997) at Pfizer analyzed >2,200 drug candidates and derived empirical cutoffs predicting oral bioavailability. A drug is likely absorbed if it satisfies **at least two** of:

$$\text{MW} \leq 500 \;\text{Da},\quad \log P \leq 5,\quad \text{H-bond donors} \leq 5,\quad \text{H-bond acceptors} \leq 10$$

(All multiples of five → "Rule of Five".) Veber (2002) added: **rotatable bonds ≤ 10**, **TPSA ≤ 140 Å²**. These rules are foundational to pharmaceutical BME: they predict intestinal absorption via the rule-of-five's passive-diffusion assumption, and predict bioavailability of nanocarrier-delivered drugs.

**Example violation (Quiz Q13):** MW = 800 Da, LogP = 6 → violates both MW and LogP → poor oral bioavailability.

---

# ⚔️ 3DG — Three Fundamental Disagreements
## 學術界三大爭議

---

### Disagreement 1: Henderson–Hasselbalch vs. Stewart's Strong-Ion Model
**爭議一：Henderson–Hasselbalch 與 Stewart 強離子模型之爭**

| Aspect | **Position A: Henderson–Hasselbalch** | **Position B: Stewart (1983) Strong-Ion Model** |
|--------|---------------------------------------|-------------------------------------------------|
| **Proponents** | Clinical chemistry tradition; Boron & Boulpaep (2017) text | Peter Stewart (1983); Kellum (2005); Story (2016) |
| **View** | pH and [HCO₃⁻] are independent variables to be measured and titrated | pH is a *dependent* variable determined by SID, A_tot, PCO₂ |
| **Mathematics** | $\text{pH} = 6.1 + \log\frac{[\text{HCO}_3^-]}{0.0301 \cdot P_{\text{CO}_2}}$ | $\text{pH}^3 + \text{(SID constants)} = \text{polymath equation}$ |
| **Strength** | Simple; widely taught; bedside-friendly | Mechanistically rigorous; explains non-bicarbonate effects |
| **Weakness** | Treats pH as causal, not resultant | Mathematically complex; not yet bedside-friendly |

**The tension:** Proponents of Stewart argue that the Henderson–Hasselbalch framework cannot explain phenomena like hyperchloremic acidosis from 0.9% saline (Kellum 2005), because changes in [Cl⁻] alter pH *without* changing [HCO₃�]. Defenders (Kurtz 2014; Berend 2014) argue Stewart's model is just a rearrangement of the same equilibria and adds mathematical burden without clinical benefit. The 2014 "great acid-base debate" (BJA editorial, Currell & Golland 2014) remains unresolved: ICU physicians often use Stewart for complex cases, while general practice still relies on Henderson–Hasselbalch.

---

### Disagreement 2: The "pH Stat" vs. "Alpha Stat" Strategy in Cardiopulmonary Bypass
**爭議二：體外循環中 pH-stat 與 α-stat 策略之爭**

| Aspect | **Position A: pH-stat** | **Position B: α-stat** |
|--------|-------------------------|--------------------------|
| **Definition** | Maintain pH = 7.40 at *actual* (37 °C) temperature; add CO₂ to cooled blood | Maintain pH = 7.40 at *37 °C* regardless of actual temperature; do NOT add CO₂ |
| **Proponents** | Swan (1985); Tallman (1992) | Rahn (1971); Bekeris (1979) |
| **Rationale** | Maintains constant [H⁺] for enzyme activity | Preserves protein ionization (α) state by allowing pH to rise with cooling as in nature |
| **Used in** | Pediatric CPB, deep hypothermic circulatory arrest | Adult CPB, most cardiac surgery |

**The tension:** At 18 °C (typical deep hypothermia), the *true* neutral pH of water is 7.94 (Reeves 1976). The two strategies produce a 0.5-unit pH difference at the same temperature. pH-stat proponents (Swan 1985) argue increased cerebral blood flow from added CO₂ improves uniform cooling. α-stat proponents argue that enzyme ionization is conserved across species (ectotherms vs. endotherms) by this method (Reeves 1972). The 1995 multi-center trial (Laffey 1995) and subsequent meta-analyses have not decisively settled the matter; practice remains institution-dependent.

---

### Disagreement 3: Hydrophobic Effect — Enthalpy or Entropy Driven?
**爭議三：疏水效應——焓驅動還是�驅動？**

| Aspect | **Position A: Classical entropy view (Kauzmann 1959; Tanford 1973; Dill 1990)** | **Position B: Enthalpy + entropy (sharp-kink model, Garde 1996; Hummer 1998)** |
|--------|------------------------------------------------------------------------------|---------------------------------------------------------------------------------|
| **Mechanism** | Water forms clathrate cages around nonpolar solutes → loss of entropy → driven by ΔS | Hydrophobic hydration has both ΔH and ΔS components; large solutes (≥1 nm) are entropy-driven, small solutes are enthalpy-driven |
| **Quantitative** | $\Delta G_{\text{hyd}} = \Delta H_{\text{hyd}} - T\Delta S_{\text{hyd}}$, with $-T\Delta S \gg \Delta H$ | $\Delta G = \Delta H + \Delta S$ terms of comparable magnitude |
| **Evidence** | Heat capacity signatures, cold denaturation of proteins | Molecular dynamics simulations (Garde 1996; Hummer 1998); NMR spectroscopy |
| **Implication** | Protein folding driven by entropy release when hydrophobic groups cluster | For large solutes, both enthalpy and entropy contribute (a "paradigm shift" Lum-Erlandsson 2008) |

**The tension:** Classical biochemistry texts teach that folding is entropy-driven. Modern MD simulations (Garde 1996, Hummer 1998, Chandler 2005) show that for small molecules like methane, hydrophobic solvation is enthalpy-driven, while only for larger surfaces does entropy dominate. This matters for BME: protein-folding predictions (used in drug design) require correct ΔG decomposition.

---

# ❓ 10Q — Ten Probing Questions with Detailed Answers
## 十個深入探討問題

---

### Q1. Why is the pKa of carbonic acid (6.10) so much lower than the pKa of a typical carboxylic acid (≈4.7), and how does this relate to the efficiency of the bicarbonate buffer at physiological pH?

**Answer (≥10 lines):**

The pKa of carbonic acid is anomalously elevated to 6.10 (in plasma at 37 °C, 5% CO₂) compared to a typical carboxylic acid pKa ≈ 4.7 (e.g., acetic acid pKa = 4.76). The reason is that "carbonic acid" H₂CO₃ is in equilibrium with dissolved CO₂; the apparent pKa reflects the true hydration equilibrium CO₂ + H₂O ⇌ H₂CO₃ ⇌ H⁺ + HCO₃⁻. The first step (hydration) is slow (k ≈ 0.03 s⁻¹) and is catalyzed by carbonic anhydrase (Meldrum & Roughton 1933), which accelerates the rate by ~10⁶-fold. The actual pKa of H₂CO₃ (the unhydrated species) is 3.6, but because the dominant species is CO₂ (which is *not* an acid), the *apparent* pKa is shifted upward. This shift is biologically fortuitous: the bicarbonate system has its maximum buffering capacity when pH = pKa ± 1 (i.e., 5.1–7.1), so pKa = 6.10 places the buffer near the bottom of its useful range — but the system has an additional, unique feature: **it is open**. The lungs continuously remove CO₂, preventing equilibrium saturation, while the kidneys regenerate HCO₃⁻. This openness means the effective buffering capacity at pH 7.4 is far greater than a closed buffer would have. Indeed, calculations using the Van Slyke equation (Van Slyke 1922) show that plasma bicarbonate has effective buffer capacity β ≈ 30 mmol/L per pH unit at pH 7.4, despite being at pH = pKa + 1.3. Other biological buffers (phosphate, proteins) cannot match this dynamic, open-system capacity, which is why bicarbonate dominates blood buffering (≈65% of total).

---

### Q2. Explain how Anfinsen's ribonuclease refolding experiment (1972) demonstrated the thermodynamic hypothesis and discuss at least two modern caveats that complicate the simple "sequence = structure" view.

**Answer (≥10 lines):**

Christian Anfinsen (1972) reduced and denatured ribonuclease A (124 amino acids, 4 disulfide bonds) using 8 M urea and β-mercaptoethanol. Upon removal of denaturant and oxidation, ribonuclease spontaneously regained full enzymatic activity — proving that all information needed for the native fold resides in the primary sequence. Anfinsen received the 1972 Nobel Prize for this work, establishing the **thermodynamic hypothesis**: native protein structure corresponds to the global free-energy minimum under physiological conditions. The mathematical statement is:

$$\Delta G_{\text{fold}} = G_{\text{native}} - G_{\text{denatured}} \ll 0$$

Modern caveats, however, significantly modify the simple view:

1. **Chaperones (Horwich 1989; Hartl 1994):** Many real proteins in vivo require ATP-dependent chaperones (GroEL/GroES, Hsp70) to fold efficiently — particularly large proteins where off-pathway aggregation competes with folding. Levinthal's paradox (1969) — that a 100-residue protein would need >10¹⁴³ years to sample conformations — suggests that *in vivo*, folding is kinetically assisted, even if thermodynamically determined.

2. **Intrinsically disordered proteins (Dunker 2001; Wright & Dyson 1999):** A large fraction (30–40%) of eukaryotic proteins contain disordered regions that lack stable 3D structure yet have function. These contradict the strict "one sequence → one structure" view; instead, the free-energy landscape is broad and shallow.

3. **Prions (Prusiner 1982):** Prion proteins exist in multiple conformations (PrPᶜ vs. PrPˢᶜ) with the same sequence but different folds, and the conformations can be self-propagating — a kinetic, not thermodynamic, phenomenon.

4. **Cotranslational folding (Fedorov & Baldwin 1997):** Many proteins begin folding on the ribosome before the chain is complete, and the N-terminal context can alter folding pathways.

Thus Anfinsen's hypothesis remains valid as a *boundary condition*, but BME applications (e.g., recombinant protein production for therapeutics) must account for these complexities.

---

### Q3. Derive Winter's formula for respiratory compensation in metabolic acidosis from first principles, and explain why compensation is "incomplete" in chronic metabolic acidosis.

**Answer (≥10 lines):**

Winter's formula (1973), for expected PCO₂ in metabolic acidosis:

$$P_{\text{CO}_2}^{\text{expected}} = 1.5 \times [\text{HCO}_3^-] + 8 \pm 2 \;\text{mmHg}$$

Empirical derivation: in pure metabolic acidosis, hyperventilation (Kussmaul respiration, Kussmaul 1874) reduces PCO₂ as a compensatory response. The relationship is linear because alveolar ventilation $\dot{V}_A$ is proportional to the metabolic acid load. Starting from the Henderson–Hasselbalch equation and applying the alveolar gas equation:

$$\text{pH} = 6.10 + \log\frac{[\text{HCO}_3^-]}{0.0301 \cdot P_{\text{CO}_2}}$$

If [HCO₃�] falls by Δ[HCO₃⁻] (e.g., from 24 to 12 mM, a fall of 12), the body compensates by reducing PCO₂ from 40 to ~26 mmHg. The empirical slope (1.5 mmHg per mM HCO₃�) reflects the body's ventilatory response to metabolic acid, but it is **incomplete** because compensation cannot fully restore pH to 7.40. Plugging the example into Winter's formula: PCO₂ = 1.5(12) + 8 = 26 mmHg. Then pH = 6.10 + log(12 / (0.0301 × 26)) = 6.10 + log(15.34) = 6.10 + 1.186 = 7.29 (still acidemic).

The compensation is incomplete because of the physiology of chemoreception: peripheral chemoreceptors (carotid bodies — Comroe 1937) respond to [H⁺] with a sigmoidal, saturating response, and central chemoreceptors (medulla) require time to detect CSF pH changes. The kidney *metabolic* compensation (renal regeneration of HCO₃⁻ over 3–5 days) is what ultimately restores pH, but it is much slower than respiratory compensation (minutes to hours). In chronic metabolic acidosis (e.g., renal tubular acidosis), the kidney's compensation is by definition impaired, leading to *worse* acidemia than Winter's formula would predict alone.

---

### Q4. Explain why Lipinski's Rule of Five (1997) fails to predict oral bioavailability for many modern drug classes, and discuss at least three alternative frameworks.

**Answer (≥10 lines):**

Lipinski et al. (1997) at Pfizer derived the Rule of Five from an empirical analysis of 2,245 drug candidates in the company's chemical library that had reached Phase II clinical trials. The rules (MW ≤ 500, LogP ≤ 5, HBD ≤ 5, HBA ≤ 10) describe the physicochemical space where passive intestinal absorption is likely, assuming diffusion across a lipid bilayer (Artursson & Karlsson 1991). However, **the rules fail for many modern drug classes**:

1. **Biologics (antibodies, peptides):** Monoclonal antibodies (MW ~150,000 Da) are absorbed via transcytosis, not passive diffusion (Roopenian & Akilesh 2007). They are administered IV/SC, not orally, but the framework cannot predict this because it was built on small molecules.

2. **Active transport substrates:** Many drugs are absorbed via specific transporters (e.g., PepT1 for dipeptides — Fei 1994). Lipinski's rules *under*-predict their bioavailability.

3. **Efflux-pumped drugs:** P-glycoprotein (P-gp, ABCB1 — Juliano & Ling 1976) effluxes many lipophilic compounds back into the gut lumen. The rules do not account for active efflux and *over*-predict bioavailability.

4. **Prodrugs:** Masked polar compounds (e.g., valacyclovir — Beauchamp 1992) are absorbed and then cleaved to active drug; the rules apply to the prodrug, not the active form.

Alternative frameworks:
- **Veber's rules (2002):** Rotatable bonds ≤ 10, TPSA ≤ 140 Å² — capture molecular flexibility and polarity more accurately.
- **Egan's egg model (2000):** LogP vs. TPSA plot identifies absorption "egg" region.
- **Bergström's BCS classification (Björkman 1995; Amidon 1995):** Biopharmaceutics Classification System ranks drugs by solubility and permeability.
- **PAMPA (Parallel Artificial Membrane Permeability Assay, Kansy 1994):** Experimental in vitro surrogate.
- **BDE (Biopharmaceutics Drug Disposition Classification System, Wu & Benet 2005):** Refines BCS to predict transporter effects.

Modern BME drug delivery uses Lipinski as a starting filter but supplements it with experimental permeability, transporter, and metabolism data.

---

### Q5. Using the principles of hydrogen bonding and entropy, explain why phospholipids spontaneously form bilayers in aqueous environments, and discuss how this relates to the fluid mosaic model.

**Answer (≥10 lines):**

Phospholipids (e.g., phosphatidylcholine) are **amphipathic** molecules with a polar head (phosphate + choline) and two nonpolar hydrocarbon tails (~16 carbons). When placed in water, individual phospholipids are energetically unfavorable because their hydrophobic tails disrupt the hydrogen-bond network of water, forcing water molecules into ordered clathrate-like cages (Frank & Evans 1945; Tanford 1973). This ordering reduces water entropy significantly. To minimize this entropic penalty, phospholipids aggregate with tails sequestered in the interior, away from water. There are two aggregation geometries: **micelles** (tails inside, single layer) for single-tailed lipids (e.g., detergents) and **lipids with two tails** form **bilayers** because the geometry of a cylindrical molecule (head area ≈ tail area) favors a planar arrangement. The critical packing parameter (Israelachvili 1976; Mitchell 1976) $v/(a_0 \cdot l_c)$, where $v$ = tail volume, $a_0$ = head area, $l_c$ = tail length, predicts: if $v/(a_0 l_c) < 1/3$ → micelle; ≈ 1/2 → bilayer; > 1 → inverted micelle. Phospholipids sit near 1/2.

Singer & Nicolson (1972) proposed the **Fluid Mosaic Model**: the bilayer is a two-dimensional fluid in which lipids and proteins diffuse laterally. The fluid property arises because the lipid melting temperature is below physiological temperature (e.g., DMPC melts at 23 °C; DOPC at -17 °C), with cholesterol modulating fluidity (Yeagle 1985). Membrane proteins (Perdue 1978) are "mosaic" inclusions that can be peripheral (cytoplasmic, dissociable) or integral (spanning). The model explains lateral diffusion rates measured by FRAP (Frye & Edidin 1970), cell fusion experiments, and forms the basis for understanding signal transduction, transport, and BME applications such as liposomal drug delivery (Bangham 1965; Gregoriadis 1976).

---

### Q6. Calculate the buffer capacity of a phosphate buffer (pH 7.4, total phosphate 0.1 M) and explain why phosphate is a poor blood buffer but excellent for laboratory use.

**Answer (≥10 lines):**

The buffer capacity equation (Van Slyke 1922) is:

$$\beta = 2.303 \cdot C_T \cdot \frac{K_a [\text{H}^+]}{(K_a + [\text{H}^+])^2}$$

For the second dissociation of phosphoric acid H₂PO₄⁻ ⇌ H⁺ + HPO₄²⁻ (pKa₂ = 7.21):

- $C_T = 0.1$ M
- $K_a = 10^{-7.21} = 6.17 \times 10^{-8}$
- $[\text{H}^+] = 10^{-7.4} = 3.98 \times 10^{-8}$

$$\beta = 2.303 \times 0.1 \times \frac{6.17 \times 10^{-8} \times 3.98 \times 10^{-8}}{(6.17 \times 10^{-8} + 3.98 \times 10^{-8})^2}$$

Numerator: $2.303 \times 0.1 \times (6.17 \times 10^{-8} \times 3.98 \times 10^{-8}) = 2.303 \times 0.1 \times 2.456 \times 10^{-15} = 5.66 \times 10^{-16}$

Denominator: $(1.015 \times 10^{-7})^2 = 1.030 \times 10^{-14}$

$$\beta \approx 5.49 \times 10^{-2} \;\text{mol/L per pH unit} = 0.055 \;\text{M}$$

Comparison with physiological bicarbonate: $\beta_{\text{bicarb}} \approx 30$ mmol/L/pH = 0.030 M.

**Why phosphate is poor in blood:** Blood phosphate concentration is only ~1 mM (mostly intracellular, ~1 mM extracellular), versus bicarbonate at 24 mM. Although phosphate's pKa = 7.21 is closer to 7.4 than bicarbonate's 6.10, the **concentration** of phosphate is too low to provide significant buffering capacity. Phosphate's role is primarily intracellular.

**Why phosphate is excellent in the lab:** The pKa₂ (7.21) is conveniently close to physiological pH (7.4), making phosphate the buffer of choice for cell culture media, enzyme assays, and protein purification. Its second pKa is also temperature-insensitive (ΔpKa/ΔT ≈ -0.0028/°C; compared to bicarbonate ΔpKa/ΔT ≈ -0.005/°C), and it does not absorb UV (unlike Tris, which has pKa ≈ 8.06 but is UV-absorbing).

---

### Q7. Discuss the structural basis of DNA double-helix stability. Why are G-C pairs (3 H-bonds) more stable than A-T pairs (2 H-bonds), and how does this explain the denaturation temperature (Tm) dependence on GC content?

**Answer (≥10 lines):**

Watson & Crick (1953), building on Chargaff's rules (Chargaff 1950) and Franklin's X-ray diffraction (Franklin & Gosling 1953), proposed the double-helix model. The two strands are held together by **Watson–Crick hydrogen bonds** between complementary bases: A pairs with T via 2 H-bonds (N6-H···O4, N1···N3-H), and G pairs with C via 3 H-bonds (O6···H-N4, N1···H-N3, N2-H···O2). The H-bond distances are 2.8–3.0 Å. The base-pair stacking interactions along the helical axis (ΔG ≈ -10 to -20 kJ/mol per stacking, much larger than individual H-bonds!) provide the **major** thermodynamic contribution to helix stability (Crothers & Zimm 1964; Yakovchuk 2006).

The Gibbs free energy of duplex formation per base pair is approximately:

$$\Delta G = \Delta H - T\Delta S$$

Marmur–Doty relation (Marmur & Doty 1962):

$$T_m = 81.5 + 16.6 \log_{10}[\text{Na}^+] + 0.41\%\text{GC} - 600/N$$

where N = chain length. For long DNA in 1 M Na⁺, $T_m \approx 69.3 + 0.41\%\text{GC}$, so a sequence with 50% GC melts at ~90 °C and one with 0% GC at ~69 °C.

**Mechanism:** GC base pairs have one additional H-bond (ΔG ≈ -2 to -5 kJ/mol extra) plus stronger stacking interactions (due to the larger polarizable surface of purine–pyrimidine pairs and the additional polar group). Both factors increase the energy barrier to denaturation, raising T_m. Biologically, thermophilic bacteria (e.g., *Thermus aquaticus* — Brock & Freeze 1969, source of Taq polymerase) have GC-rich genomes and modified bases (e.g., 2,6-diaminopurine) that raise T_m well above 70 °C, allowing DNA stability at high temperatures. BME applications exploit this: PCR primer design (Mullis 1985) uses GC content to set annealing temperatures.

---

### Q8. Explain the chemical basis for the unique property of water that ice floats. How is this relevant to engineering design of cryopreservation protocols for biological tissues?

**Answer (≥10 lines):**

Most substances are denser in solid form than in liquid form because molecules pack more tightly in the crystalline state. Water is the anomalously opposite: $\rho_{\text{ice}} = 0.917$ g/cm³ versus $\rho_{\text{water}} = 1.000$ g/cm³ at 0 °C (or 0.9998 at 4 °C, where water is densest). The explanation lies in the **directionality of hydrogen bonds** (Eisenberg & Kauzmann 1969). Each water molecule can donate and accept 2 H-bonds, forming a tetrahedral network. In liquid water, this network is partially broken, allowing molecules to pack slightly closer than in ice. In ice, every H-bond is intact and *oriented*, forcing an open hexagonal lattice with large voids (Pauling 1935). The O-O distance in ice is 2.76 Å, slightly longer than the 2.85 Å nearest-neighbor distance in liquid water (Soper 2013). The result: ice expands by ~9% in volume compared to water.

**Biological consequence:** Because ice floats, ponds freeze from the top down, insulating the liquid water below and allowing aquatic life to survive under frozen surfaces. If ice sank, entire bodies of water would freeze solid in cold climates, eliminating aquatic ecosystems (a likely factor in why Earth is habitable — Franks 1982).

**BME relevance — cryopreservation:** Cryopreservation of tissues, cells (e.g., oocytes — Trounson & Mohr 1983), and organs (e.g., vitrification — Fahy 1984) aims to avoid ice crystal formation, which causes mechanical damage to membranes (Mazur 1984, two-factor hypothesis: cooling rate too fast → intracellular ice; too slow → osmotic dehydration). Cryoprotective agents (CPAs) such as dimethyl sulfoxide (DMSO, pKa = 35; Lovelock & Bishop 1959) and ethylene glycol lower the freezing point and promote **vitrification** (glass formation without crystallization). The physics of water is thus the foundation of fertility clinics, organ banking, and tissue engineering.

---

### Q9. Compare and contrast the bonding in NaCl, in the C-O bond of carbon monoxide, and in the Watson-Crick base pair G-C. Use specific numerical values of electronegativity, bond length, and bond energy.

**Answer (≥10 lines):**

**NaCl (ionic bond):** Sodium donates its 3s¹ electron to chlorine, forming Na⁺ and Cl⁻. Ionic bond strength in vacuum is high (ΔH_lattice ≈ 787 kJ/mol for NaCl — Born & Mayer 1932), but in water (ε = 78.5) the effective strength drops by 1/ε to ~10 kJ/mol (Born 1920). Bond length: 2.36 Å (X-ray, Bragg 1913). Na-Cl bond is fully ionic (Δχ = 2.23, well above 1.7 threshold).

**C≡O in carbon monoxide:** CO has a triple bond (1 σ + 2 π), bond length 1.128 Å (Herzberg 1950), and bond energy 1072 kJ/mol — the strongest known chemical bond. Despite Δχ(O-C) = 0.89 (between C and O, with O = 3.44, C = 2.55), the bond is essentially nonpolar because of a subtle quantum effect: the carbon end carries a small negative formal charge due to π-back-donation from oxygen lone pair into the empty C π* orbital (Pauling 1939; Purcell & Kotz 1977). CO is therefore a nonpolar covalent bond with very high strength. This is why CO is poisonous: it binds hemoglobin with ~200× the affinity of O₂, displacing O₂ (Haldane 1895; Douglas 1912).

**G-C Watson-Crick base pair (noncovalent):** Three H-bonds. Bond energies: each H-bond in DNA is ~5–10 kJ/mol in the pairing geometry (Yakovchuk 2006), so total ~20 kJ/mol for the GC pair (versus ~13 kJ/mol for AT with 2 H-bonds). Bond lengths: N-H···O ≈ 2.9 Å, N-H···N ≈ 3.0 Å (Saenger 1984). The H-bonds are highly directional; the angle must be near-linear (180° ± 20°) for optimum strength (Jeffrey 1997). Stacking interactions between adjacent base pairs are ~10–20 kJ/mol per stack (Yakovchuk 2006), so the total duplex stability is dominated by stacking, not by H-bonds. This explains why single H-bond mismatches can still leave duplexes moderately stable — the mismatch disrupts H-bonding but not stacking.

**Comparison summary:**

| Bond | Type | Strength (kJ/mol) | Length (Å) |
|------|------|-------------------|------------|
| Na–Cl | Ionic | ~787 (vacuum), ~10 (water) | 2.36 |
| C≡O | Nonpolar covalent | 1072 | 1.128 |
| G–C (3 H-bonds) | Hydrogen | ~20 | ~2.9 |

---

### Q10. Describe the molecular mechanism by which carbonic anhydrase accelerates CO₂ hydration by 10⁶- to 10⁷-fold, and discuss its relevance to acid-base balance in BME applications (dialysis, ECMO, oxygenators).

**Answer (≥10 lines):**

Carbonic anhydrase (CA) is a zinc metalloenzyme (Keilin & Mann 1940), with a Zn²⁺ ion coordinated to three histidine residues (His94, His96, His119 in human CA II — Lindskog 1997). The catalytic mechanism (Lindskog 1971; Silverman & Lindskog 1988):

1. **Zn-OH deprotonation:** At physiological pH, the Zn-bound water (pKa reduced from 14 to ~7 by Zn²⁺ coordination — Lindskog 1971) loses a proton to generate a Zn-OH⁻ nucleophile.

2. **CO₂ attack:** The hydroxide attacks the electrophilic carbon of CO₂, forming Zn-bound bicarbonate (Zn-O-C(=O)-OH).

3. **Bicarbonate release:** The bicarbonate is displaced by a new water molecule, regenerating Zn-H₂O.

The overall reaction:

$$\text{CO}_2 + \text{H}_2\text{O} \xrightleftharpoons[k_{-1}]{k_1} \text{H}_2\text{CO}_3 \xrightleftharpoons[k_{-2}]{k_2} \text{H}^+ + \text{HCO}_3^-$$

The uncatalyzed rate $k_1 \approx 0.03$ s⁻¹, while CA-accelerated $k_{\text{cat}}/K_M \approx 10⁸$ M�¹s⁻¹ for CA II (Khalifah 1971). The 10⁶- to 10⁷-fold rate enhancement is essential: without CA, tissue CO₂ would not equilibrate fast enough with blood HCO₃⁻, and gas exchange would be rate-limited.

**BME applications:**
- **Dialysis (Kolff 1943; Henderson 1977):** Modern dialyzers use acetate or bicarbonate in the dialysate. Acetate metabolism generates HCO₃⁻; however, critically ill patients cannot metabolize acetate quickly, leading to "acetate intolerance" and hemodynamic instability (Schiffl 1983). Bicarbonate dialysate is preferred, but it requires CO₂ partial pressure control to prevent CaCO₃ precipitation. CA-coated hollow-fiber membranes (Broun 1990) accelerate CO₂-HCO₃⁻ equilibration and improve buffer capacity.
- **ECMO (Bartlett 1977) and cardiopulmonary bypass (Gibbon 1953):** Oxygenators must offload CO₂ while oxygenating blood. Membrane oxygenators (Kolobow 1969) use silicone or polymethylpentene membranes; the CO₂ transfer rate is ~10× slower than O₂ transfer (because CO₂ has higher solubility, the gradient is smaller). CA immobilization on membrane surfaces increases the effective transfer rate.
- **Implantable biosensors (Wang 2008):** CA-based sensors detect CO₂ in blood or tissue, used for respiratory monitoring.
- **Drug development:** CA inhibitors (acetazolamide — Maren 1967) treat glaucoma, altitude sickness, and metabolic alkalosis.

Thus a single enzyme illustrates the seamless integration of physical chemistry, molecular biology, and biomedical engineering.

---

# 📖 5DD — Five Deep Dives (Bilingual 中英對照)
## 五個深度探討：雙語對照

---

## Deep Dive 1: The Hydrogen Bond — A Quantum-Cultural History
## 深度探討一：氫鍵的量子與文化歷史

### English Version

The hydrogen bond (H-bond) sits at the intersection of chemistry, biology, and physics, with a conceptual history spanning over a century. **Latimer & Rodebush (1920)** coined the term in a *Journal of American Chemical Society* paper, recognizing that water's anomalous properties required a special "hydrogen-bridge" between oxygen atoms. **Pauling (1939)** systematized H-bonding in *The Nature of the Chemical Bond*, the most influential chemistry text of the 20th century, quantifying the directionality, distance (typically 2.5–3.2 Å), and energy (5–40 kJ/mol). **Jeffrey (1997)** categorized H-bonds into strong, moderate, and weak types, with a continuum from covalent (short, strong) to electrostatic (long, weak). Modern **DFT calculations** (Grabowski 2001) have refined our understanding of H-bond cooperativity (multiple H-bonds enhancing each other, ~10–20% per bond).

**Structural biology landmarks:**
- **Watson & Crick (1953)** showed DNA H-bonds (A-T, G-C) carry genetic information.
- **Pauling, Corey & Branson (1951)** proposed the α-helix and β-sheet of proteins, with H-bonds stabilizing secondary structure.
- **Perutz (1962)** confirmed H-bonds in hemoglobin.
- **Wüthrich (1986, 2002 Nobel)** used H-bond NMR restraints to determine protein structures in solution.

**Modern debates:** Whether short H-bonds (< 2.5 Å) have covalent character is unresolved. **Low-energy H-bonds** in enzyme active sites (e.g., serine proteases — Blow 1969) may transition toward covalent bond character, with potential energy surfaces showing low-barrier H-bonds (LBHB, Frey 1994; Cleland & Kreevoy 1994). This has implications for enzyme catalysis and drug design.

### 中文版本

�鍵 (Hydrogen bond) 跨越化學、生物學與物理學，其概念史已逾百年。**Latimer 與 Rodebush (1920)** 在《美國化學會期刊》中首次提出此術語，認識到水的異常性質需要氧原子之間的「�橋」來解釋。**Pauling (1939)** 在其二十世紀最具影響力的化學著作《化學鍵的本質》中系統化氫鍵，定量其方向性 (directionality)、鍵長 (2.5–3.2 Å) 與能量 (5–40 kJ/mol)。**Jeffrey (1997)** 將氫鍵分類為強、中、弱三類，呈現從共價性 (短而強) 到靜電性 (長而弱) 的連續光譜。現代**密度泛函理論 (DFT) 計算** (Grabowski 2001) 進一步釐清氫鍵協同效應 (cooperativity)，即多重氫鍵彼此相互增強，每個鍵增強約 10–20%。

**結構生物學里程碑：**
- **Watson 與 Crick (1953)** 證明 DNA 氫鍵 (A-T、G-C) 承載遺傳訊息。
- **Pauling, Corey 與 Branson (1951)** 提出蛋白質的 α 螺旋與 β 摺片，由氫鍵穩定二級結構。
- **Perutz (1962)** 確認血紅素中的氫鍵。
- **Wüthrich (1986, 2002 年諾貝爾獎)** 使用氫鍵 NMR 約束測定溶液中蛋白質結構。

**現代爭議：** 短氫鍵 (< 2.5 Å) 是否具有共價性仍未定論。**低能氫鍵** 出現在酶活性位 (如絲氨酸蛋白酶 — Blow 1969)，可能�向共價鍵性質，勢能面呈現低障氫鍵 (Low-Barrier Hydrogen Bond, LBHB, Frey 1994; Cleland & Kreevoy 1994)，對酶催化與藥物設計有重要意義。

---

## Deep Dive 2: Water's Anomalous Properties and the Hydrophobic Effect
## 深度探討二：水的異常性質與疏水效應

### English Version

Water has at least 16 anomalous properties compared to "normal" hydrogen-bonded liquids like H₂S or HF (Franks 1972). Most relevant to BME:

1. **Density maximum at 4 °C** (not at freezing point). Below 4 °C, water expands slightly as it cools, due to second-nearest-neighbor H-bond restructuring. This drives autumnal lake turnover and prevents thermal shock to aquatic life.

2. **High surface tension (γ = 71.97 mN/m at 25 °C; Pallas & Pethica 1985).** Higher than almost any nonmetallic liquid; enables capillary rise (Jurin 1718; Lucas 1928 — Washburn equation), critical for plant biology and microfluidic BME devices (Whitesides 2001).

3. **High dielectric constant (ε = 78.5 at 25 °C; Malmberg & Maryott 1956).** Coulomb's law in vacuum: $F = kq_1 q_2 / r^2$. In water: $F = kq_1 q_2 / (\varepsilon r^2)$. The factor 1/78.5 weakens ionic interactions and makes water the "universal solvent" for ions.

4. **High heat capacity, enthalpy of vaporization, and thermal conductivity** — all consequences of the H-bond network.

**The Hydrophobic Effect** (Kauzmann 1959; Tanford 1973): When nonpolar solutes (e.g., hexane) are introduced into water, water forms clathrate-like cages around them, reducing water entropy. The **positive ΔG** of solvation arises from $-T\Delta S$, not from $\Delta H$ (which can be slightly negative). The resulting driving force for nonpolar groups to associate (releasing water molecules and increasing entropy) is what stabilizes:
- Lipid bilayers (Tanford 1973; Israelachvili 1976)
- Protein tertiary structure (Dill 1990)
- Micelles and emulsions

**BME applications:**
- **Microfluidics (Whitesides 2001; Quake 2003):** Droplet microfluidics uses hydrophobic/hydrophilic patterning.
- **Drug delivery (Torchilin 2005):** Liposomes exploit hydrophobic effect to self-assemble.
- **Tissue engineering (Langer & Vacanti 1993; Ratner 2004):** Surface chemistry dictates cell adhesion.

### 中文版本

水相較於「正常」氫鍵液體 (如 H₂S 或 HF) 至少具有 16 項異常性質 (Franks 1972)。與 BME 最相關者包括：

1. **4 °C 時密度最大** (非在凝固點)。低於 4 °C 時，水隨冷卻略微膨脹，源於第二最近鄰氫鍵重組。此現象驅動秋季湖泊翻轉，並避免水生生物遭受溫度衝擊。

2. **高表面張力** (25 °C 時 γ = 71.97 mN/m；Pallas & Pethica 1985)。高於幾乎所有非金屬液體；引發毛細上升 (Jurin 1718；Lucas 1928 — Washburn equation)，對植物生物學與微流體 BME 裝置 (Whitesides 2001) 至關重要。

3. **高介電常數** (25 °C 時 ε = 78.5；Malmberg & Maryott 1956)。庫侖定律在真空中為 $F = kq_1 q_2 / r^2$；在水中為 $F = kq_1 q_2 / (\varepsilon r^2)$。1/78.5 的衰減因子削弱離子交互作用，使水成為離子的「通用溶劑」。

4. **高比熱容、汽化焓與熱導率** — 均為氫鍵網絡之結果。

**疏水效應** (Kauzmann 1959; Tanford 1973)：當非極性溶質 (如己烷) 進入水中，水在其周圍形成似籠狀結構，降低水熵。溶解作用的**正 ΔG** 源自 $-T\Delta S$，而非 $\Delta H$ (後者可能略為負值)。非極性基團因而傾向聚集 (釋出水分子、增加熵)，此驅動力穩定：
- 脂質雙層 (Tanford 1973; Israelachvili 1976)
- 蛋白質三級結構 (Dill 1990)
- 微胞與乳化液

**BME 應用：**
- **微流體** (Whitesides 2001; Quake 2003)：液滴微流體利用疏水/親水圖案化。
- **藥物傳遞** (Torchilin 2005)：微脂體利用疏水效應自組裝。
- **組織工程** (Langer & Vacanti 1993; Ratner 2004)：表面化學決定細胞貼附。

---

## Deep Dive 3: Acid-Base Physiology and Clinical Decision-Making
## 深度探討三：酸鹼生理學與臨床決策

### English Version

Clinical acid-base interpretation has three classical rules (Narins & Emmett 1980):

**Rule 1: Identify the primary disorder.**
- pH < 7.35 = acidemia → acidosis
- pH > 7.45 = alkalemia → alkalosis
- Look at PCO₂ and HCO₃⁻ to determine respiratory vs. metabolic

**Rule 2: Assess compensation.**
- Metabolic acidosis: expected PCO₂ = 1.5[HCO₃⁻] + 8 ± 2 (Winter 1973)
- Metabolic alkalosis: expected PCO₂ = 0.7[HCO₃⁻] + 20 ± 5
- Acute respiratory acidosis: HCO₃⁻ ↑ 1 mM per 10 mmHg ↑ PCO₂
- Chronic respiratory acidosis: HCO₃⁻ ↑ 3.5 mM per 10 mmHg ↑ PCO₂
- Acute respiratory alkalosis: HCO₃⁻ ↓ 2 mM per 10 mmHg ↓ PCO₂
- Chronic respiratory alkalosis: HCO₃⁻ ↓ 5 mM per 10 mmHg ↓ PCO₂

**Rule 3: Calculate the anion gap (AG).**
$$\text{AG} = [\text{Na}^+] - ([\text{Cl}^-] + [\text{HCO}_3^-])$$
- Normal = 12 ± 4 mEq/L
- Elevated AG (>16) → "MUDPILES": Methanol, Uremia, DKA, Propylene glycol, Iron/Isoniazid, Lactic acidosis, Ethylene glycol, Salicylates
- Normal AG (hyperchloremic) → diarrhea, RTA, ileostomy losses

**Case from Quiz:** Patient COPD with pH 7.32, PCO₂ 55, HCO₃⁻ 30 mM.
- Acidemia (pH < 7.35) + ↑PCO₂ → respiratory acidosis
- HCO₃⁻ = 30, expected for chronic: 24 + 0.35 × 15 = 29.25 ≈ 30 ✓ (compensation matches chronic)
- Diagnosis: Chronic respiratory acidosis with appropriate metabolic compensation

**Stewart vs. Henderson-Hasselbalch:** A modern intensivist might use Stewart's strong-ion difference (SID = [Na⁺] + [K⁺] + [Ca²�] + [Mg²⁺] − [Cl⁻] − [lactate]) to further characterize non-bicarbonate contributors. For instance, a 0.9% saline bolus reduces SID, causing hyperchloremic acidosis (Kellum 2005).

### 中文版本

臨床酸鹼判讀有三項經典規則 (Narins & Emmett 1980)：

**規則一：確認主要病變。**
- pH < 7.35 = 酸血症 → 酸中毒
- pH > 7.45 = 鹼血症 → �中毒
- 觀察 PCO₂ 與 HCO₃⁻ 決定呼吸性或代謝性

**規則二：評估代償。**
- 代謝性酸中毒：預期 PCO₂ = 1.5[HCO₃⁻] + 8 ± 2 (Winter 1973)
- 代謝性鹼中毒：預期 PCO₂ = 0.7[HCO₃⁻] + 20 ± 5
- 急性呼吸性酸中毒：HCO₃⁻ ↑ 1 mM / 每 10 mmHg ↑ PCO₂
- 慢性呼吸性酸中毒：HCO₃⁻ ↑ 3.5 mM / 每 10 mmHg ↑ PCO₂
- 急性呼吸性鹼中毒：HCO₃⁻ ↓ 2 mM / 每 10 mmHg ↓ PCO₂
- 慢性呼吸性鹼中毒：HCO₃� ↓ 5 mM / 每 10 mmHg ↓ PCO₂

**規則三：計算陰離子間隙 (AG)。**
$$\text{AG} = [\text{Na}^+] - ([\text{Cl}^-] + [\text{HCO}_3^-])$$
- 正常 = 12 ± 4 mEq/L
- AG 升高 (>16) → 「MUDPILES」：甲醇、尿毒症、糖尿病酮症中毒、丙二醇、鐵/異�鹼酸 (異菸鹼醯肼)、乳酸中毒、乙二醇、水楊酸 (Aspirin)
- AG 正常 (高氯性) → 腹瀉、腎小管酸中毒、腸造口流失

**測驗案例：** COPD 病人 pH 7.32、PCO₂ 55、HCO₃⁻ 30 mM。
- 酸血症 (pH < 7.35) + ↑PCO₂ → 呼吸性酸中毒
- HCO₃⁻ = 30，慢性預期值 = 24 + 0.35 × 15 = 29.25 ≈ 30 ✓ (代償符合慢性)
- 診斷：慢性呼吸性酸中毒合併適當代謝性代償

**Stewart 對 Henderson–Hasselbalch：** 現代重症醫師可能使用 Stewart 強離子差 (SID = [Na⁺] + [K⁺] + [Ca²⁺] + [Mg²⁺] − [Cl⁻] − [lactate]) 進一步分析非碳酸根貢獻。例如，0.9% 生理食鹽水灌注會降低 SID，造成高氯性酸中毒 (Kellum 2005)。

---

## Deep Dive 4: Protein Folding and Misfolding Diseases
## 深度探討四：蛋白質摺疊與錯摺疾病

### English Version

The **protein folding problem** — predicting a protein's 3D structure from its amino acid sequence — is one of the "grand challenges" of computational biology (Anfinsen 1972). The free-energy landscape theory (Bryngelson & Wolynes 1987; Dill 1990) models folding as a funnel, where the native state sits at the global free-energy minimum. Modern AI approaches (**AlphaFold2** — Jumper et al. 2021) achieved near-experimental accuracy at CASP14, revolutionizing structural biology.

**Misfolding diseases** arise when proteins fail to fold correctly or maintain their folded state:

| Disease | Protein | Year Discovered |
|---------|---------|------------------|
| **Sickle cell anemia** | Hemoglobin (E6V mutation) | Pauling 1949 (first "molecular disease") |
| **CJD / Prion** | PrP (PrPˢᶜ vs. PrPᶜ) | Prusiner 1982 (1997 Nobel) |
| **Alzheimer's** | Amyloid-β, Tau | Glenner & Wong 1984 |
| **Parkinson's** | α-synuclein | Polymeropoulos 1997 |
| **Type II Diabetes** | Amylin (IAPP) | Westermark 1987 |
| **Cystic Fibrosis** | CFTR (ΔF508) | Riordan 1989 |
| **Huntington's** | Huntingtin (polyQ) | HDCRG 1993 |

**Thermodynamic vs. kinetic models of misfolding:**
- **Thermodynamic:** A mildly destabilized mutation shifts the folding equilibrium so that the unfolded or partially folded state is populated. Sickle cell hemoglobin (HbS) polymerizes when deoxygenated (Pauling 1949).
- **Kinetic (amyloid):** Misfolding to a cross-β-sheet state is kinetically trapped; the amyloid fibril is a stable but off-pathway conformation (Dobson 1999).

**BME applications:**
- **Diagnostic biosensors:** Antibody-based detection of Aβ42 (Lue 1999) for early Alzheimer's
- **Therapeutics:** Small molecules (e.g., Tafamidis — Bulawa 2012) stabilize transthyretin tetramers against amyloid dissociation
- **Protein engineering:** Directed evolution (Arnold 1996) designs stable variants for therapeutics

### 中文版本

**蛋白質摺疊問題** — 由胺基酸序列預測蛋白質 3D 結構 — 是計算生物學的「重大挑戰」之一 (Anfinsen 1972)。自由能景觀理論 (Bryngelson & Wolynes 1987; Dill 1990) 將摺疊模型化為漏斗 (funnel)，原生態位於全局自由能最低點。現代 AI 方法 (**AlphaFold2** — Jumper et al. 2021) 在 CASP14 達到接近實驗準確度，徹底改變結構生物學。

**錯摺疾病** 因蛋白質未能正確摺疊或維持其摺疊態而起：

| 疾病 | 蛋白質 | 發現年份 |
|------|--------|----------|
| **鐮狀細胞貧血** | 血紅素 (E6V 突變) | Pauling 1949 (首個「分子疾病」) |
| **CJD / 普利昂** | PrP (PrPˢᶜ vs. PrPᶜ) | Prusiner 1982 (1997 諾貝爾獎) |
| **阿茲海默症** | 類澱粉 β (Aβ)、Tau | Glenner & Wong 1984 |
| **巴金森氏症** | α-突觸核蛋白 | Polymeropoulos 1997 |
| **第二型糖尿病** | 胰島澱粉樣多胜肽 (IAPP) | Westermark 1987 |
| **囊性纖維化** | CFTR (ΔF508) | Riordan 1989 |
| **亨丁頓舞蹈症** | Huntingtin (polyQ 擴增) | HDCRG 1993 |

**錯�的熱力學與動力學模型：**
- **熱力學：** 輕微失穩突變使摺疊平衡轉移，使未摺疊或部分�疊態被佔據。鐮狀細胞血紅素 (HbS) 在去氧時聚合 (Pauling 1949)。
- **動力學 (類澱粉)：** 錯摺至交互 β 摺片 (cross-β-sheet) 態為動力學陷阱 (kinetic trap)；類澱粉纖維為穩定但偏離正常途徑的構形 (Dobson 1999)。

**BME 應用：**
- **診斷生物感測器：** 抗體式 Aβ42 偵測 (Lue 1999) 用於早期阿茲海默症
- **治療：** 小分子 (如 Tafamidis — Bulawa 2012) 穩定 transthyretin 四聚體對抗類澱粉解離
- **蛋白質工程：** 定向演化 (Arnold 1996) 設計穩定變體用於治療

---

## Deep Dive 5: Drug Delivery — From Lipinski to Nanomedicine
## 深度探討五：藥物傳� — 從 Lipinski 到奈米醫學

### English Version

**Evolution of drug delivery paradigms:**
1. **Free drug (1900s–present):** Simple oral or IV administration; governed by Lipinski (1997) and BCS (Amidon 1995).
2. **Modified-release (1950s–):** Sustained, controlled release using polymer matrices (Higuchi 1961, t ∝ √t equation).
3. **Targeted delivery (1970s–):** Ligand-directed (e.g., folic acid receptor — Low 2008), antibody-drug conjugates (ADC — Senter 2001).
4. **Nanocarriers (1990s–present):** Liposomes (Bangham 1965), polymeric nanoparticles (Langer 1990), dendrimers (Tomalia 1985), quantum dots (Bruchez 1998; Chan & Nie 1998).

**Key equations:**
- **Higuchi equation:** $Q = \sqrt{D \cdot C_s \cdot (2A - C_s) \cdot t / V}$ (Higuchi 1961)
- **First-order release:** $M_t / M_\infty = 1 - e^{-kt}$
- **Zero-order release (ideal):** $M_t / M_\infty = kt$

**Examples of FDA-approved nanomedicines:**
- **Doxil® (1995):** PEGylated liposomal doxorubicin; half-life 73 hr (vs. 10 min free)
- **Abraxane® (2005):** Albumin-bound paclitaxel nanoparticles
- **Onpattro® (2018):** siRNA lipid nanoparticle (patisiran — first FDA-approved siRNA drug)
- **Comirnaty® (2020):** mRNA lipid nanoparticle vaccine (Pfizer-BioNTech COVID-19)
- **BNT162b2 mRNA encoding SARS-CoV-2 spike protein delivered via ionizable LNPs**

**Lipid Nanoparticle (LNP) breakthrough:** The key innovation is the **ionizable cationic lipid** (e.g., MC3, ALC-0315 — Maier 2013; Pardi 2015) that is neutral at pH 7.4 (reducing toxicity) but positively charged at endosomal pH (≈5), promoting membrane disruption and cytoplasmic delivery of mRNA. This solves the "endosomal escape problem" that previously limited nucleic acid delivery.

**BME impact:** Nanomedicine has transformed cancer therapy, vaccine technology, and gene therapy. The mRNA vaccine platform (Polack 2020) demonstrated that nanocarriers can deliver fragile, large (MW ~1 MDa), negatively charged mRNA into cells to produce protein antigens — a feat impossible with free mRNA.

### 中文版本

**藥物傳遞範式的演化：**
1. **自由藥物 (1900s 至今)：** 簡單口服或靜脈給藥；受 Lipinski (1997) 與 BCS (Amidon 1995) 規範。
2. **控釋劑型 (1950s–)：** 使用聚合物基質的持續、可控釋放 (Higuchi 1961，t ∝ √t 方程式)。
3. **標靶傳遞 (1970s–)：** 配體導向 (例如葉酸受體 — Low 2008)、抗體藥物複合體 (ADC — Senter 2001)。
4. **奈米載體 (1990s 至今)：** 微脂體 (Bangham 1965)、聚合物奈米粒 (Langer 1990)、樹狀高分子 (Tomalia 1985)、量子點 (Bruchez 1998; Chan & Nie 1998)。

**關鍵方程式：**
- **Higuchi 方程式：** $Q = \sqrt{D \cdot C_s \cdot (2A - C_s) \cdot t / V}$ (Higuchi 1961)
- **一階釋放：** $M_t / M_\infty = 1 - e^{-kt}$
- **零階釋放 (理想)：** $M_t / M_\infty = kt$

**FDA 核准之奈米藥物實例：**
- **Doxil® (1995)：** PEG 化微脂體 doxorubicin；半衰期 73 小時 (自由藥物僅 10 分鐘)
- **Abraxane® (2005)：** 白蛋白結合 paclitaxel 奈米粒
- **Onpattro® (2018)：** siRNA 脂質奈米粒 (patisiran — 首個 FDA 核准 siRNA 藥物)
- **Comirnaty® (2020)：** mRNA 脂質奈米粒疫苗 (Pfizer-BioNTech COVID-19)
- **BNT162b2 mRNA 編碼 SARS-CoV-2 棘蛋白，透過可離子化 LNP 傳遞**

**脂質奈米粒 (LNP) 突破：** 關鍵創新為**可離子化陽離子脂質** (如 MC3、ALC-0315 — Maier 2013; Pardi 2015)，在 pH 7.4 時為中性 (降低毒性)，但在內體 pH (≈5) 時帶正電，破壞膜並促進 mRNA 進入細胞質。此技術解決了「內體逃逸問題」，此問題過去限制核酸遞送。

**BME 影響：** 奈米醫學已轉變癌症治療、疫苗技術與基因治療。mRNA 疫苗平台 (Polack 2020) 證明奈米載體能將脆弱、大型 (MW ~1 MDa)、帶負電之 mRNA 遞送進細胞以產生蛋白質抗原 — 此為自由 mRNA 無法達成之壯舉。

---

# 📝 10SL — Ten Self-Test Solutions
## 十題自我測驗解答

---

### Self-Test Q1
**Q:** A buffer solution contains 0.20 M acetic acid (pKa = 4.74) and 0.30 M sodium acetate. Calculate the pH, then calculate the pH after adding 10 mL of 1.0 M HCl to 100 mL of this buffer.

**A:** Initial pH using Henderson–Hasselbalch:
$$\text{pH} = 4.74 + \log\frac{0.30}{0.20} = 4.74 + \log(1.5) = 4.74 + 0.176 = 4.92$$

After adding 10 mL of 1.0 M HCl = 0.010 mol H⁺. This converts 0.010 mol of acetate (A⁻) to acetic acid (HA).
- New [A⁻] = (0.030 − 0.010) / 0.110 L = 0.020 / 0.110 = 0.1818 M
- New [HA] = (0.020 + 0.010) / 0.110 L = 0.030 / 0.110 = 0.2727 M
$$\text{pH}_{\text{new}} = 4.74 + \log\frac{0.1818}{0.2727} = 4.74 + \log(0.667) = 4.74 - 0.176 = 4.56$$

ΔpH = 4.56 − 4.92 = −0.36. Compare to adding 10 mL HCl to 100 mL pure water: pH goes from 7 to ~2 — a Δ of 5 units. Buffer reduces change by ~14×.

---

### Self-Test Q2
**Q:** Define electronegativity and calculate Δχ for the bond P-O in ATP. Use values: χ_P = 2.19, χ_O = 3.44. Is this bond polar covalent? Predict its dipole moment qualitatively.

**A:** Electronegativity (Pauling 1932) is the ability of an atom in a chemical bond to attract electrons to itself. It is dimensionless on the Pauling scale (F = 4.0 reference).

$$\Delta\chi_{P-O} = 3.44 - 2.19 = 1.25$$

Since 0.5 < 1.25 < 1.7, the P-O bond is **polar covalent**, with the O atom bearing partial negative charge (δ⁻) and P bearing partial positive charge (δ⁺). The P=O double bond in ATP's phosphate groups has Δχ similar to P-O and even more polar character. Dipole moment μ ≈ 1.5–2.5 D (Debye) for P-O single bonds (CRC Handbook), making phosphates highly reactive in hydrolysis reactions and explaining why ATP hydrolysis releases ~30.5 kJ/mol of free energy (Lipmann 1941) under standard conditions (with physiological ΔG ≈ −50 to −60 kJ/mol).

---

### Self-Test Q3
**Q:** Describe, with a diagram, the difference between a hydrogen bond and a covalent bond in water, and explain why water has a high boiling point relative to H₂S despite a lower molecular weight.

**A:** Water (MW = 18) has bp = 100 °C. H₂S (MW = 34) has bp = −60 °C. The difference is the hydrogen bond.

Covalent bond O-H in water: ~463 kJ/mol, intramolecular, distance 0.96 Å, requires breaking to decompose water. Each water molecule has 2 O-H covalent bonds, contributing to its tetrahedral geometry.

Hydrogen bond in water: ~20 kJ/mol, intermolecular (between molecules), distance ~1.8 Å between O of one molecule and H of another, angle near 180°. Average ~3.5 H-bonds per water molecule in liquid (Soper 2013).

To vaporize water, all H-bonds must be broken (ΔH_vap = 44 kJ/mol). For H₂S, only weak van der Waals forces must be overcome (ΔH_vap = 19 kJ/mol), so its boiling point is far lower. The O-H covalent bond remains intact during phase changes — it is the H-bond that breaks.

---

### Self-Test Q4
**Q:** The peptide bond has partial double-bond character. Draw the resonance structures and explain the biological consequence for protein folding.

**A:** The peptide bond (Pauling & Corey 1951) is the amide linkage −C(=O)−N(H)− between amino acids. Two resonance structures:

**Structure A:** O=C–N–H (neutral, single bond C-N, double bond C=O)
**Structure B:** O⁻–C=N⁺–H (charge-separated, double bond C=N, single bond C-O with negative O)

The hybrid has ~40% double-bond character in C-N (Pauling 1939; Wenger 2000). Consequences:
- **Planarity:** The C-N bond cannot rotate freely; the six atoms O=C-N-H-Cα-Cα' lie in a plane.
- **Restricted conformation:** The only peptide backbone torsion angles are φ (phi, around N-Cα) and ψ (psi, around Cα-C). Ramachandran plots (Ramachandran 1963) show allowed regions.
- **cis/trans isomerism:** ~99.95% of peptide bonds are trans (more stable by ~8 kJ/mol); cis bonds occur mainly before proline residues (~6%) and cause rate-limiting steps in folding (Schmid 1991, peptidyl-prolyl isomerase).
- **IR spectroscopy signature:** Amide I band ~1650 cm⁻¹ (C=O stretch); Amide II ~1550 cm⁻¹ (N-H bend).

---

### Self-Test Q5
**Q:** Define pKa and explain why the pKa of the α-carboxyl group (~2.0) is much lower than the pKa of the α-amino group (~9.0) in free amino acids.

**A:** pKa = −log₁₀(Ka), where Ka is the acid dissociation constant.

For α-COOH: the conjugate base is COO⁻, stabilized by resonance across two equivalent oxygens. The electron-withdrawing effect of the α-NH₃⁺ also stabilizes the COO⁻ through inductive effect (-I effect). Hence pKa ~ 2 (lower = stronger acid, more dissociation).

For α-NH₃⁺: the conjugate base is NH₂ (neutral), which lacks resonance stabilization. The lone pair on nitrogen is localized; protonation is favorable because the positive charge is stabilized by the α-COO⁻ group (-I effect again). Hence pKa ~ 9 (higher = weaker acid, less tendency to dissociate).

**Physiological zwitterion:** At pH 7.4, free amino acids exist as ⁺H₃N-CHR-COO⁻, because the pH is above α-COOH pKa (proton dissociated → COO⁻) and below α-NH₃⁺ pKa (still protonated → NH₃⁺).

**Titration curve:** Plots of pH vs. added acid/base show two clear inflection points at pKa₁ (≈2) and pKa₂ (≈9). The isoelectric point pI = (pKa₁ + pKa₂)/2 for neutral amino acids (e.g., glycine pI = (2.34 + 9.60)/2 = 5.97).

---

### Self-Test Q6
**Q:** A patient has a respiratory alkalosis. ABG: pH = 7.55, PCO₂ = 30 mmHg,