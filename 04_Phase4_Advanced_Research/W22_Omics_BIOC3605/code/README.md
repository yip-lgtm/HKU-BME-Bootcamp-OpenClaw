# Week 22 Code Lab — Omics: Genomics and Proteomics (BIOC3605)

## Bioinformatics Computational Lab

> **Dependencies**: `numpy`, `scipy`, `pandas`, `matplotlib`  
> **Estimated time**: 4-5 hours  
> **Learning goal**: Sequence alignment, RNA-seq analysis, GWAS simulation, proteomics

---

## Lab Setup

```bash
pip install numpy scipy pandas matplotlib biopython
```

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy import stats, special
from Bio import pairwise2, SeqIO
from collections import Counter
plt.style.use('seaborn-v0_8-whitegrid')
```

---

## Exercise 1: Sequence Alignment — Needleman-Wunsch & Smith-Waterman

```python
"""
W22_CodeLab_Ex1_SequenceAlignment.py
======================================
Implement global (Needleman-Wunsch) and local (Smith-Waterman) alignment.
"""

def needleman_wunsch(seq1, seq2, match=1, mismatch=-1, gap=-2):
    """
    Needleman-Wunsch global sequence alignment.
    
    Returns: alignment score, aligned sequences
    """
    m, n = len(seq1), len(seq2)
    
    # Initialize scoring matrix
    score = np.zeros((m+1, n+1))
    
    # Initialize first row and column (gap penalties)
    for i in range(1, m+1):
        score[i][0] = score[i-1][0] + gap
    for j in range(1, n+1):
        score[0][j] = score[0][j-1] + gap
    
    # Fill the scoring matrix
    for i in range(1, m+1):
        for j in range(1, n+1):
            match_score = match if seq1[i-1] == seq2[j-1] else mismatch
            diag = score[i-1][j-1] + match_score
            up = score[i-1][j] + gap
            left = score[i][j-1] + gap
            score[i][j] = max(diag, up, left)
    
    # Traceback
    align1, align2 = [], []
    i, j = m, n
    
    while i > 0 or j > 0:
        if i > 0 and j > 0:
            match_score = match if seq1[i-1] == seq2[j-1] else mismatch
            if score[i][j] == score[i-1][j-1] + match_score:
                align1.append(seq1[i-1])
                align2.append(seq2[j-1])
                i -= 1
                j -= 1
            elif score[i][j] == score[i-1][j] + gap:
                align1.append(seq1[i-1])
                align2.append('-')
                i -= 1
            else:
                align1.append('-')
                align2.append(seq2[j-1])
                j -= 1
        elif i > 0:
            align1.append(seq1[i-1])
            align2.append('-')
            i -= 1
        else:
            align1.append('-')
            align2.append(seq2[j-1])
            j -= 1
    
    return score[m][n], ''.join(reversed(align1)), ''.join(reversed(align2))

def smith_waterman(seq1, seq2, match=1, mismatch=-1, gap=-2):
    """
    Smith-Waterman local sequence alignment.
    
    Returns: best alignment score, aligned subsequences
    """
    m, n = len(seq1), len(seq2)
    
    # Initialize scoring matrix
    H = np.zeros((m+1, n+1))
    
    best_score = 0
    best_pos = (0, 0)
    
    # Fill the scoring matrix
    for i in range(1, m+1):
        for j in range(1, n+1):
            match_score = match if seq1[i-1] == seq2[j-1] else mismatch
            diag = H[i-1][j-1] + match_score
            up = H[i-1][j] + gap
            left = H[i][j-1] + gap
            
            H[i][j] = max(0, diag, up, left)
            
            if H[i][j] > best_score:
                best_score = H[i][j]
                best_pos = (i, j)
    
    # Traceback from best position
    align1, align2 = [], []
    i, j = best_pos
    
    while H[i][j] > 0:
        match_score = match if seq1[i-1] == seq2[j-1] else mismatch
        if H[i][j] == H[i-1][j-1] + match_score:
            align1.append(seq1[i-1])
            align2.append(seq2[j-1])
            i -= 1
            j -= 1
        elif H[i][j] == H[i-1][j] + gap:
            align1.append(seq1[i-1])
            align2.append('-')
            i -= 1
        else:
            align1.append('-')
            align2.append(seq2[j-1])
            j -= 1
    
    return best_score, ''.join(reversed(align1)), ''.join(reversed(align2))

def simple_blast_like(query, database, match=1, mismatch=-1, gap=-2):
    """
    Simplified BLAST-like local alignment search.
    Returns top hits with E-value approximation.
    """
    hits = []
    for db_seq in database:
        score, align_q, align_db = smith_waterman(query, db_seq, match, mismatch, gap)
        if score > 0:
            # Simplified E-value calculation
            m, n = len(query), len(db_seq)
            # E-value ≈ m * n * e^(-λS) approximation
            lambda_param = 0.3  # typical for DNA
            K = 0.1
            e_value = K * m * n * np.exp(-lambda_param * score)
            hits.append({
                'subject': db_seq[:20] + '...',
                'score': score,
                'e_value': e_value,
                'identity': sum(a == b for a, b in zip(align_q, align_db)) / len(align_q) * 100
            })
    
    # Sort by score
    hits.sort(key=lambda x: x['score'], reverse=True)
    return hits

# Test sequences
seq1 = "ATCGATCGATCG"
seq2 = "AT-GATC-ATCG"

print("="*60)
print("SEQUENCE ALIGNMENT")
print("="*60)

# Global alignment
score, align1, align2 = needleman_wunsch(seq1, seq2)
print(f"\nGlobal Alignment (Needleman-Wunsch):")
print(f"  Score: {score}")
print(f"  Seq1:  {align1}")
print(f"  Seq2:  {align2}")

# Local alignment
seq3 = "ATCGATCG"
seq4 = "GATCGATC"
score, align1, align2 = smith_waterman(seq3, seq4)
print(f"\nLocal Alignment (Smith-Waterman):")
print(f"  Score: {score}")
print(f"  Seq1:  {align1}")
print(f"  Seq2:  {align2}")

# BLAST-like search
database = [
    "ATCGATCGATCG",
    "GATCGATCGATC",
    "AAAAAAAAAAAA",
    "GCGCGCGCGCGC",
    "ATCGATCGATCGATCG"
]
query = "ATCGATCG"
hits = simple_blast_like(query, database)
print(f"\nBLAST-like Search (query: {query}):")
for hit in hits[:5]:
    print(f"  {hit['subject']}: score={hit['score']}, E={hit['e_value']:.2e}, identity={hit['identity']:.1f}%")

# Alignment visualization
def visualize_alignment(align1, align2, width=60):
    """Visualize alignment with matching bars."""
    matches = sum(a == b for a, b in zip(align1, align2) if a != '-' and b != '-')
    print(f"\nAlignment visualization ({matches}/{len([a for a in align1 if a != '-'])} matches):")
    for i in range(0, len(align1), width):
        print(f"  Seq1: {align1[i:i+width]}")
        print(f"        " + ''.join(['|' if a == b else ' ' for a, b in zip(align1[i:i+width], align2[i:i+width])]))
        print(f"  Seq2: {align2[i:i+width]}")

visualize_alignment(align1, align2)
```

---

## Exercise 2: RNA-seq Expression Analysis

```python
"""
W22_CodeLab_Ex2_RNASeq.py
============================
Simulate and analyze RNA-seq expression data.
Calculate TPM, FPKM, and differential expression.
"""

def calculate_tpm(counts, gene_lengths):
    """
    Calculate TPM (Transcripts Per Million).
    
    TPM_i = (R_i / L_i) / sum_j(R_j / L_j) × 10^6
    """
    # RPK (reads per kilobase)
    rpk = counts / gene_lengths
    
    # Sum of RPK (scaling factor)
    scaling_factor = rpk.sum()
    
    # TPM
    tpm = (rpk / scaling_factor) * 1e6
    
    return tpm

def calculate_fpkm(counts, gene_lengths, total_reads):
    """
    Calculate FPKM (Fragments Per Kilobase per Million).
    
    FPKM = (10^3 × F) / (N × L)
    """
    # Total reads in millions
    N = total_reads / 1e6
    
    # FPKM
    fpkm = (counts * 1000) / (gene_lengths * N * 1e6)
    
    return fpkm

def differential_expression(counts_treatment, counts_control, 
                          gene_lengths, n_samples=3):
    """
    Simulate differential expression analysis (simplified DESeq2-like).
    """
    results = []
    
    for i, (gene, length) in enumerate(gene_lengths.items()):
        # Treatment counts (average)
        treat_mean = np.mean(counts_treatment[i])
        ctrl_mean = np.mean(counts_control[i])
        
        # Fold change
        if ctrl_mean > 0:
            fc = treat_mean / ctrl_mean
            log2fc = np.log2(fc) if fc > 0 else 0
        else:
            log2fc = 0
        
        # t-test
        t_stat, p_value = stats.ttest_ind(counts_treatment[i], counts_control[i])
        
        # Significance
        significant = p_value < 0.05 and abs(log2fc) > 1
        
        results.append({
            'gene': gene,
            'length': length,
            'treat_mean': treat_mean,
            'ctrl_mean': ctrl_mean,
            'fold_change': fc,
            'log2FC': log2fc,
            'p_value': p_value,
            'significant': significant
        })
    
    return pd.DataFrame(results)

# Simulate RNA-seq data
np.random.seed(42)

genes = ['GAPDH', 'ACTB', 'BRCA1', 'TP53', 'EGFR', 'VEGFA', 'IL6', 'TNF', 
         'IFNG', 'IL10', 'CDK4', 'MYC', 'KRAS', 'BRAF', 'PTEN']

gene_lengths = {
    'GAPDH': 1300, 'ACTB': 1800, 'BRCA1': 7092, 'TP53': 2512,
    'EGFR': 5022, 'VEGFA': 1716, 'IL6': 856, 'TNF': 1422,
    'IFNG': 1104, 'IL10': 1324, 'CDK4': 2066, 'MYC': 2391,
    'KRAS': 5678, 'BRAF': 2301, 'PTEN': 1215
}

# Simulate counts: treatment vs control
n_samples = 3
counts_treatment = {}
counts_control = {}

for i, gene in enumerate(genes):
    # Housekeeping genes (constant)
    if gene in ['GAPDH', 'ACTB']:
        base = 1000 + np.random.randint(-100, 100, n_samples)
    # Upregulated in treatment
    elif gene in ['BRCA1', 'TP53', 'EGFR']:
        base_t = 500 + np.random.randint(200, 400, n_samples)
        base_c = 200 + np.random.randint(-50, 50, n_samples)
    # Downregulated in treatment
    elif gene in ['VEGFA', 'IL6', 'TNF']:
        base_t = 100 + np.random.randint(-30, 30, n_samples)
        base_c = 400 + np.random.randint(50, 100, n_samples)
    # Not changed
    else:
        base_t = 300 + np.random.randint(-50, 50, n_samples)
        base_c = 300 + np.random.randint(-50, 50, n_samples)
    
    counts_treatment[i] = base_t
    counts_control[i] = base_c

# Calculate TPM for each sample
print("="*60)
print("RNA-SEQ EXPRESSION ANALYSIS")
print("="*60)

print("\n--- TPM Normalization (Sample 1) ---")
tpm_t1 = calculate_tpm(counts_treatment[0], gene_lengths)
tpm_c1 = calculate_tpm(counts_control[0], gene_lengths)

tpm_df = pd.DataFrame({
    'Gene': genes,
    'Length': [gene_lengths[g] for g in genes],
    'Treatment_TPM': tpm_t1,
    'Control_TPM': tpm_c1
})
print(tpm_df.to_string(index=False))

# Differential expression
print("\n--- Differential Expression Analysis ---")
results_df = differential_expression(
    [counts_treatment[i] for i in range(len(genes))],
    [counts_control[i] for i in range(len(genes))],
    gene_lengths
)

# Apply FDR correction
from scipy.stats import false_discovery_control
results_df['p_adjusted'] = false_discovery_control(results_df['p_value'])
results_df['DE'] = (results_df['p_adjusted'] < 0.05) & (abs(results_df['log2FC']) > 1)

print(results_df[['gene', 'log2FC', 'p_value', 'p_adjusted', 'DE']].to_string(index=False))

# Volcano plot
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Volcano plot
ax1 = axes[0]
colors = ['red' if row['DE'] else 'gray' for _, row in results_df.iterrows()]
ax1.scatter(results_df['log2FC'], -np.log10(results_df['p_value']), 
           c=colors, alpha=0.7, s=50)
ax1.axhline(-np.log10(0.05), color='blue', ls='--', lw=2)
ax1.axvline(-1, color='green', ls='--', lw=1)
ax1.axvline(1, color='green', ls='--', lw=1)

# Label significant genes
for _, row in results_df[results_df['DE']].iterrows():
    ax1.annotate(row['gene'], (row['log2FC'], -np.log10(row['p_value'])),
                fontsize=9, fontweight='bold')

ax1.set_xlabel('log2 Fold Change', fontsize=12)
ax1.set_ylabel('-log10(p-value)', fontsize=12)
ax1.set_title('Volcano Plot: Differential Expression', fontsize=14)
ax1.grid(True, alpha=0.3)

# Bar plot of top DEGs
ax2 = axes[1]
top_de = results_df.nlargest(6, 'log2FC')
ax2.barh(top_de['gene'], top_de['log2FC'], color='steelblue')
ax2.set_xlabel('log2 Fold Change', fontsize=12)
ax2.set_title('Top Upregulated Genes', fontsize=14)
ax2.grid(True, alpha=0.3, axis='x')

plt.tight_layout()
plt.savefig('W22_rnaseq_de.png', dpi=150)
plt.show()

print(f"\n=== Summary ===")
print(f"Total genes tested: {len(results_df)}")
print(f"Upregulated (padj < 0.05, log2FC > 1): {(results_df['log2FC'] > 1).sum()}")
print(f"Downregulated (padj < 0.05, log2FC < -1): {(results_df['log2FC'] < -1).sum()}")
```

---

## Exercise 3: GWAS Simulation

```python
"""
W22_CodeLab_Ex3_GWAS.py
==========================
Simulate GWAS data and perform association analysis.
"""

def simulate_gwas_data(n_cases=5000, n_controls=5000, n_snps=10000, 
                       causal_snps=None, n_causal=5):
    """
    Simulate GWAS case-control data.
    
    Returns: genotypes (0/1/2), phenotype, SNP IDs
    """
    np.random.seed(42)
    
    # Generate random genotypes (MAF ~ 0.3-0.5)
    maf = np.random.uniform(0.3, 0.5, n_snps)
    
    # Hardy-Weinberg equilibrium
    p = maf
    q = 1 - p
    geno_probs = np.array([p**2, 2*p*q, q**2])  # AA, Aa, aa
    
    genotypes = np.zeros((n_cases + n_controls, n_snps), dtype=int)
    
    for j in range(n_snps):
        for i in range(n_cases + n_controls):
            genotypes[i, j] = np.random.choice([0, 1, 2], p=geno_probs[j])
    
    # Phenotype
    disease = np.zeros(n_cases + n_controls, dtype=int)
    disease[:n_cases] = 1
    
    # Add causal SNP effects
    if causal_snps is None:
        causal_snps = np.random.choice(n_snps, n_causal, replace=False)
    
    effect_sizes = np.random.uniform(0.3, 0.8, n_causal)
    
    # Add polygenic signal
    for i in range(n_cases + n_controls):
        genetic_score = np.sum(genotypes[i, causal_snps] * effect_sizes)
        prob = 1 / (1 + np.exp(-(genetic_score - 2)))
        disease[i] = np.random.choice([0, 1], p=[1-prob, prob])
    
    # Shuffle
    shuffle_idx = np.random.permutation(n_cases + n_controls)
    genotypes = genotypes[shuffle_idx]
    disease = disease[shuffle_idx]
    
    return genotypes, disease, causal_snps, effect_sizes

def gwas_chi2_test(genotypes, phenotype):
    """
    Perform χ² association test for each SNP.
    """
    n_snps = genotypes.shape[1]
    p_values = np.zeros(n_snps)
    odds_ratios = np.zeros(n_snps)
    
    cases = phenotype == 1
    controls = phenotype == 0
    
    for j in range(n_snps):
        # Count alleles
        alleles_case = np.sum(genotypes[cases, j])
        alleles_ctrl = np.sum(genotypes[controls, j])
        
        # 2x3 contingency table
        table = np.array([
            [np.sum(genotypes[cases, j] == 0), np.sum(genotypes[cases, j] == 1), np.sum(genotypes[cases, j] == 2)],
            [np.sum(genotypes[controls, j] == 0), np.sum(genotypes[controls, j] == 1), np.sum(genotypes[controls, j] == 2)]
        ])
        
        # χ² test
        try:
            chi2, p = stats.chi2_contingency(table)[:2]
            p_values[j] = p
        except:
            p_values[j] = 1.0
        
        # Odds ratio (per allele)
        if alleles_ctrl > 0 and (len(genotypes[controls, j])*2 - alleles_ctrl) > 0:
            or_val = (alleles_case * (len(genotypes[controls, j])*2 - alleles_ctrl)) / \
                     (alleles_ctrl * (len(genotypes[cases, j])*2 - alleles_case))
            odds_ratios[j] = or_val if or_val > 0 else 1
    
    return p_values, odds_ratios

def bonferroni_correction(p_values, alpha=0.05):
    """Bonferroni correction for multiple testing."""
    n_tests = len(p_values)
    threshold = alpha / n_tests
    significant = p_values < threshold
    return significant, threshold

# Simulate GWAS data
print("="*60)
print("GWAS SIMULATION")
print("="*60)

genotypes, disease, causal_snps, effect_sizes = simulate_gwas_data(
    n_cases=2000, n_controls=2000, n_snps=1000, n_causal=10
)

print(f"\nSimulated data:")
print(f"  Cases: {np.sum(disease)}, Controls: {len(disease) - np.sum(disease)}")
print(f"  SNPs: {genotypes.shape[1]}")
print(f"  True causal SNPs: {causal_snps}")

# GWAS
p_values, odds_ratios = gwas_chi2_test(genotypes, disease)

# Apply Bonferroni correction
bonf_significant, bonf_threshold = bonferroni_correction(p_values)

# Apply FDR (Benjamini-Hochberg)
from scipy.stats import false_discovery_control
p_adjusted = false_discovery_control(p_values)
fdr_significant = p_adjusted < 0.05

print(f"\nBonferroni correction:")
print(f"  Threshold: p < {bonf_threshold:.2e}")
print(f"  Significant SNPs: {np.sum(bonf_significant)}")

# Find top hits
top_hits_idx = np.argsort(p_values)[:20]

# Manhattan plot simulation
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Manhattan-like plot (SNPs ordered by position)
ax1 = axes[0]
neg_log_p = -np.log10(p_values)
colors = ['red' if i in causal_snps else 'gray' for i in range(len(p_values))]
ax1.scatter(range(len(p_values)), neg_log_p, c=colors, alpha=0.5, s=20)
ax1.axhline(-np.log10(bonf_threshold), color='blue', ls='--', lw=2, 
            label=f'Bonferroni ({bonf_threshold:.1e})')
ax1.axhline(-np.log10(1e-5), color='green', ls=':', lw=1.5, 
            label='Suggestive (1e-5)')
ax1.set_xlabel('SNP Index', fontsize=12)
ax1.set_ylabel('-log10(p-value)', fontsize=12)
ax1.set_title('GWAS Results (Manhattan-like)', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3)

# QQ plot
ax2 = axes[1]
expected_p = np.sort(stats.uniform.cdf(np.linspace(0, 1, len(p_values))))
observed_p = np.sort(p_values)
ax2.scatter(-np.log10(expected_p), -np.log10(observed_p), s=20, alpha=0.5)
ax2.plot([0, 5], [0, 5], 'r--', lw=2, label='Expected under null')
ax2.set_xlabel('Expected -log10(p)', fontsize=12)
ax2.set_ylabel('Observed -log10(p)', fontsize=12)
ax2.set_title('QQ Plot', fontsize=14)
ax2.legend()
ax2.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W22_gwas_results.png', dpi=150)
plt.show()

# Summary
print(f"\n=== Top 10 GWAS Hits ===")
top_idx = np.argsort(p_values)[:10]
for i, idx in enumerate(top_idx):
    status = "✓ CAUSAL" if idx in causal_snps else ""
    print(f"{i+1}. SNP_{idx}: p={p_values[idx]:.2e}, OR={odds_ratios[idx]:.3f} {status}")
```

---

## Exercise 4: Proteomics Quantification

```python
"""
W22_CodeLab_Ex4_Proteomics.py
================================
Simulate proteomics quantification data.
Compare label-free vs. TMT labeling approaches.
"""

def simulate_proteomics_data(n_proteins=500, n_samples=4):
    """
    Simulate proteomics expression data.
    """
    np.random.seed(42)
    
    proteins = [f"Protein_{i}" for i in range(n_proteins)]
    
    # Simulate expression levels (some upregulated, some downregulated)
    fold_changes = np.random.choice([0.5, 1, 1, 1, 2, 3], n_proteins)
    
    # Control samples
    base_intensity = np.random.lognormal(15, 2, n_proteins)  # log-normal distribution
    control = base_intensity * np.random.lognormal(0, 0.1, (n_samples//2, n_proteins))
    
    # Treatment samples
    treatment = base_intensity * fold_changes * np.random.lognormal(0, 0.1, (n_samples//2, n_proteins))
    
    return proteins, control, treatment

def label_free_quantification(control, treatment):
    """
    Label-free quantification using peak intensity.
    """
    # Mean intensity per protein
    ctrl_means = np.mean(control, axis=0)
    treat_means = np.mean(treatment, axis=0)
    
    # Ratio
    ratios = treat_means / ctrl_means
    
    return ratios

def tmt_quantification_simulation(control, treatment):
    """
    Simulate TMT 8-plex quantification.
    """
    # TMT channels: 4 control + 4 treatment
    # Control: 126, 127, 128, 129
    # Treatment: 130, 131, 132, 133
    
    # Add isobaric interference (typical ~10-20%)
    interference = 0.15
    
    tmt_control = control + control * np.random.normal(0, interference, control.shape)
    tmt_treatment = treatment + treatment * np.random.normal(0, interference, treatment.shape)
    
    # Normalize to total signal
    tmt_control = tmt_control / tmt_control.sum(axis=1, keepdims=True) * 1e6
    tmt_treatment = tmt_treatment / tmt_treatment.sum(axis=1, keepdims=True) * 1e6
    
    # Ratios
    ctrl_means = np.mean(tmt_control, axis=0)
    treat_means = np.mean(tmt_treatment, axis=0)
    ratios = treat_means / ctrl_means
    
    return ratios

# Simulate data
print("="*60)
print("PROTEOMICS QUANTIFICATION")
print("="*60)

proteins, control, treatment = simulate_proteomics_data(n_proteins=200, n_samples=4)

# LFQ
lfq_ratios = label_free_quantification(control, treatment)

# TMT
tmt_ratios = tmt_quantification_simulation(control, treatment)

# Compare
print(f"\n--- Label-Free vs. TMT Comparison ---")
comparison_df = pd.DataFrame({
    'Protein': proteins[:10],
    'LFQ_Ratio': lfq_ratios[:10],
    'TMT_Ratio': tmt_ratios[:10]
})
print(comparison_df.to_string(index=False))

# Scatter plot
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
ax1.scatter(np.log2(lfq_ratios), np.log2(tmt_ratios), alpha=0.5, s=20)
ax1.plot([-3, 3], [-3, 3], 'r--', lw=2, label='Perfect agreement')
ax1.set_xlabel('log2 Ratio (LFQ)', fontsize=12)
ax1.set_ylabel('log2 Ratio (TMT)', fontsize=12)
ax1.set_title('LFQ vs. TMT Quantification', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3)
ax1.set_xlim(-3, 3)
ax1.set_ylim(-3, 3)

# Distribution of ratios
ax2 = axes[1]
ax2.hist(np.log2(lfq_ratios), bins=50, alpha=0.5, label='LFQ', density=True)
ax2.hist(np.log2(tmt_ratios), bins=50, alpha=0.5, label='TMT', density=True)
ax2.axvline(0, color='black', ls='--', lw=1)
ax2.set_xlabel('log2 Ratio (Treatment/Control)', fontsize=12)
ax2.set_ylabel('Density', fontsize=12)
ax2.set_title('Ratio Distribution', fontsize=14)
ax2.legend()
ax2.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W22_proteomics.png', dpi=150)
plt.show()

# Correlation
corr = np.corrcoef(np.log2(lfq_ratios), np.log2(tmt_ratios))[0, 1]
print(f"\nCorrelation between LFQ and TMT: r = {corr:.4f}")
```

---

## Exercise 5: GO Enrichment Analysis

```python
"""
W22_CodeLab_Ex5_GOEnrichment.py
================================
Perform Gene Ontology enrichment analysis.
"""

def go_enrichment(de_genes, background_genes, go_terms, gene_to_go):
    """
    Perform GO enrichment using hypergeometric test.
    
    Hypergeometric test:
    P = 1 - Σ_{i=0}^{k-1} [C(M,i) × C(N-M, n-i)] / C(N,n)
    
    Where:
    N = total genes tested
    M = genes in GO term
    n = DE genes
    k = DE genes in GO term
    """
    N = len(background_genes)
    results = []
    
    for term, genes_in_term in go_terms.items():
        # Genes in term
        M = len(set(genes_in_term) & set(background_genes))
        
        # DE genes in term
        k = len(set(de_genes) & set(genes_in_term))
        
        # Total DE genes
        n = len(de_genes)
        
        if M > 0 and n > 0:
            # Hypergeometric p-value
            # scipy hypergeom: sf(k-1, M, N, n)
            p_value = stats.hypergeom.sf(k-1, N, M, n)
            
            # Fold enrichment
            observed = k / n if n > 0 else 0
            expected = M / N if N > 0 else 0
            fold_enrich = observed / expected if expected > 0 else 0
            
            results.append({
                'GO_term': term,
                'genes_in_term': M,
                'de_genes_in_term': k,
                'p_value': p_value,
                'fold_enrichment': fold_enrich
            })
    
    return pd.DataFrame(results)

# Simulate GO enrichment
print("="*60)
print("GO ENRICHMENT ANALYSIS")
print("="*60)

# Background genes
background = [f"Gene_{i}" for i in range(500)]

# Differentially expressed genes (subset)
de_genes = np.random.choice(background, 50, replace=False)

# GO terms
go_terms = {
    'GO:0006955': ['Gene_1', 'Gene_5', 'Gene_10', 'Gene_15', 'Gene_20', 'Gene_25', 'Gene_30'],
    'GO:0006915': ['Gene_2', 'Gene_6', 'Gene_11', 'Gene_16', 'Gene_21', 'Gene_26', 'Gene_31', 'Gene_36'],
    'GO:0010628': ['Gene_3', 'Gene_7', 'Gene_12', 'Gene_17', 'Gene_22', 'Gene_27', 'Gene_32', 'Gene_37', 'Gene_42'],
    'GO:0043066': ['Gene_4', 'Gene_8', 'Gene_13', 'Gene_18', 'Gene_23', 'Gene_28', 'Gene_33', 'Gene_38', 'Gene_43', 'Gene_48'],
    'GO:0045944': ['Gene_5', 'Gene_9', 'Gene_14', 'Gene_19', 'Gene_24', 'Gene_29', 'Gene_34', 'Gene_39', 'Gene_44', 'Gene_49'],
}

# Map genes to GO
gene_to_go = {}
for term, genes in go_terms.items():
    for gene in genes:
        if gene not in gene_to_go:
            gene_to_go[gene] = []
        gene_to_go[gene].append(term)

# Add DE genes to GO terms (so some are enriched)
de_subset = list(de_genes[:10])
for term in ['GO:0006915', 'GO:0043066']:
    for gene in de_subset[:3]:
        if gene in gene_to_go:
            gene_to_go[gene].append(term)
        else:
            gene_to_go[gene] = [term]

# Run enrichment
enrichment_df = go_enrichment(de_genes, background, go_terms, gene_to_go)

# Sort by p-value
enrichment_df = enrichment_df.sort_values('p_value')

print(f"\n--- GO Enrichment Results ---")
print(enrichment_df.to_string(index=False))

# Apply FDR
enrichment_df['p_adjusted'] = false_discovery_control(enrichment_df['p_value'])
enrichment_df['significant'] = enrichment_df['p_adjusted'] < 0.05

print(f"\n--- Significant GO Terms (FDR < 0.05) ---")
sig_terms = enrichment_df[enrichment_df['significant']]
print(sig_terms.to_string(index=False))

# Bar plot
fig, ax = plt.subplots(figsize=(10, 6))

# Top 10 terms
top = enrichment_df.nlargest(10, 'fold_enrichment')
colors = ['red' if p < 0.05 else 'gray' for p in top['p_adjusted']]

bars = ax.barh(range(len(top)), top['fold_enrichment'], color=colors)
ax.set_yticks(range(len(top)))
ax.set_yticklabels(top['GO_term'])
ax.set_xlabel('Fold Enrichment', fontsize=12)
ax.set_title('GO Enrichment Analysis', fontsize=14)
ax.axvline(1, color='black', ls='--', lw=1)
ax.grid(True, alpha=0.3, axis='x')

plt.tight_layout()
plt.savefig('W22_go_enrichment.png', dpi=150)
plt.show()
```

---

## Lab Report Checklist

| Task | Description | File |
|------|-------------|------|
| ☐ Ex 1 | Sequence alignment algorithms | Console output |
| ☐ Ex 2 | RNA-seq DE analysis | `W22_rnaseq_de.png` |
| ☐ Ex 3 | GWAS simulation | `W22_gwas_results.png` |
| ☐ Ex 4 | Proteomics quantification | `W22_proteomics.png` |
| ☐ Ex 5 | GO enrichment analysis | `W22_go_enrichment.png` |

---

## Extension Challenge

**Challenge**: Design a complete bioinformatics pipeline for analyzing a real RNA-seq dataset from GEO. Include: QC, trimming, alignment, quantification, DE analysis, and functional enrichment. Use real data from a public repository.
