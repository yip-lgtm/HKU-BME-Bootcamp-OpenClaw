# Week 22 Self-Quiz — Omics (BIOC3605)

> **Format**: Self-assessment | **Time**: ~60 minutes  

---

## Part I: Multiple Choice Questions (10 × 3 pts = 30 pts)

**Q1.** The human genome contains approximately:
- (A) 3.2 million base pairs  
- (B) 3.2 billion base pairs  
- (C) 20,000 base pairs  
- (D) 1 million base pairs  
- (E) 3.2 trillion base pairs

---

**Q2.** Which normalization method is preferred for comparing gene expression across samples in RNA-seq?
- (A) RPKM only  
- (B) FPKM only  
- (C) TPM  
- (D) Raw counts  
- (E) RMA

---

**Q3.** In BLAST, an E-value of 10⁻⁵⁰ indicates:
- (A) Not significant  
- (B) Weakly similar  
- (C) Highly significant (strong match)  
- (D) Identical sequences  
- (E) Cannot determine

---

**Q4.** The genome-wide significance threshold for GWAS (after multiple testing correction) is typically:
- (A) p < 0.05  
- (B) p < 0.01  
- (C) p < 5 × 10⁻⁸  
- (D) p < 10⁻²⁰  
- (E) p < 0.001

---

**Q5.** Which technique is used for protein identification in proteomics?
- (A) Western blotting  
- (B) ELISA  
- (C) Mass spectrometry  
- (D) Northern blotting  
- (E) qPCR

---

**Q6.** In RNA-seq, a gene with log2FC = 2 and p-adj < 0.05 indicates:
- (A) 2-fold downregulation  
- (B) 4-fold upregulation  
- (C) No change  
- (D) 50% downregulation  
- (E) Cannot interpret

---

**Q7.** Linkage disequilibrium (LD) in GWAS means:
- (A) Genes are linked to diseases  
- (B) Nearby SNPs are correlated and inherited together  
- (C) All SNPs are independent  
- (D) SNPs cause disease  
- (E) Linkage causes recombination

---

**Q8.** Which post-translational modification is most commonly analyzed in proteomics?
- (A) Glycation  
- (B) Phosphorylation  
- (C) Acetylation  
- (D) Ubiquitination  
- (E) Methylation

---

**Q9.** The central dogma describes the flow of genetic information from:
- (A) Protein → RNA → DNA  
- (B) DNA → Protein → RNA  
- (C) DNA → RNA → Protein  
- (D) RNA → DNA → Protein  
- (E) Protein → DNA → RNA

---

**Q10.** TMT (Tandem Mass Tag) labeling is used for:
- (A) DNA sequencing  
- (B) RNA quantification  
- (C) Protein quantification (multiplex)  
- (D) Metabolomics  
- (E) Epigenomics

---

## Part II: Short Answer Questions (5 × 6 pts = 30 pts)

**SQ1.** Explain the difference between global alignment (Needleman-Wunsch) and local alignment (Smith-Waterman). When would you use each?

---

**SQ2.** Why is TPM preferred over RPKM for comparing expression across samples? Include the mathematical reasoning.

---

**SQ3.** What is "missing heritability" in GWAS? What are some proposed explanations?

---

**SQ4.** In mass spectrometry-based proteomics, explain what MS/MS (tandem mass spectrometry) provides that MS alone cannot.

---

**SQ5.** In GO enrichment analysis, what does the hypergeometric test measure and why is multiple testing correction important?

---

## Part III: Problem-Solving (2 × 20 pts = 40 pts)

**PQ1.** [20 pts] **RNA-seq Expression Analysis**

You have RNA-seq data for a cancer study with treatment vs. control.

(a) [4 pts] A gene has 500 reads mapping to it in a sample with 10 million total mapped reads and gene length of 2000 bp. Calculate RPKM.

(b) [4 pts] If the control sample has TPM = 100 and treatment has TPM = 300, what is the log2 fold change?

(c) [4 pts] With p-value = 0.001, is this significant after FDR correction for 20,000 genes tested?

(d) [4 pts] If RNA-seq depth increases from 10M to 50M reads, how does this affect detection sensitivity?

(e) [4 pts] Design an experiment to validate the RNA-seq results at the protein level.

---

**PQ2.** [20 pts] **GWAS Analysis**

You perform a GWAS with 1 million SNPs in 5000 cases and 5000 controls.

(a) [4 pts] What is the Bonferroni-corrected significance threshold?

(b) [4 pts] A SNP has p = 10⁻⁶. Is this genome-wide significant?

(c) [4 pts] Explain what population stratification is and how it affects GWAS results.

(d) [4 pts] If LD between two SNPs is r² = 0.8, and SNP1 is genome-wide significant, what can you conclude about SNP2?

(e) [4 pts] Design a follow-up study to identify the causal variant among LD-correlated SNPs.

---

## Answer Key

### MCQ Answers

| Q | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | 3.2 billion bp (3.2 Gbp) |
| 2 | **C** | TPM normalizes for both gene length AND sequencing depth |
| 3 | **C** | E-value << 1 indicates highly significant match |
| 4 | **C** | 5 × 10⁻⁸ is standard GWAS threshold |
| 5 | **C** | Mass spectrometry identifies peptides by mass |
| 6 | **B** | log2FC = 2 → FC = 2² = 4-fold upregulation |
| 7 | **B** | LD: nearby SNPs correlate due to limited recombination |
| 8 | **B** | Phosphorylation is the most studied PTM |
| 9 | **C** | DNA → RNA → Protein (central dogma) |
| 10 | **C** | TMT is for multiplexed protein quantification |

### Short Answer Solutions

**SQ1**: Global aligns entire sequences; local aligns best-matching subsequences. Use global for similar-length, related sequences; local for finding domains/motifs.

**SQ2**: TPM first normalizes by gene length (RPK), then by sum of RPKs. RPKM normalizes by total reads first, making cross-sample comparison problematic.

**SQ3**: Missing heritability = SNP-based h² < twin-based H². Explanations: rare variants not captured, structural variants, gene-environment interactions, epistasis.

**SQ4**: MS/MS fragments peptides and measures fragment ion masses, allowing peptide sequence identification (not just mass).

**SQ5**: Hypergeometric test assesses overrepresentation of GO term genes in DE set. Multiple testing needed because thousands of GO terms tested.

### Problem Solutions

**PQ1**: (a) RPKM = 500/(2×10) = 25; (b) log2FC = log2(300/100) = log2(3) ≈ 1.58; (c) With 20,000 tests, p_adj = 0.001 × 20000 = 20 → Not significant; (d) More reads → better detection of low-expressed genes; (e) Western blot or targeted proteomics (SRM).

**PQ2**: (a) α = 0.05/1,000,000 = 5×10⁻⁸; (b) No, p=10⁻⁶ > 5×10⁻⁸; (c) Population stratification = ancestry differences between cases/controls causing spurious associations; mitigated by PCA; (d) SNP2 is likely also significant due to LD; (e) Fine-mapping, functional studies (CRISPR), allelic expression.

---

## Self-Scoring Guide

| Score | Grade | Interpretation |
|-------|-------|-----------------|
| 85-100 | A | Excellent |
| 70-84 | B | Good |
| 55-69 | C | Satisfactory |
| < 55 | D | Needs review |
