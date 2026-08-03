# Week 23 Code Lab — Capstone Sprint

## Capstone Project Implementation

> **Purpose**: Complete your capstone project implementation.  
> **Deliverable**: Working code/prototype for your chosen topic.

---

## Project Structure Template

```python
"""
[Project Title]
===============
[Brief description]

Author: [Your Name]
Date: [Week 23]
"""

# Import necessary libraries
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy import stats

# Set random seed for reproducibility
np.random.seed(42)

# ============================================================
# SECTION 1: DATA GENERATION / LOADING
# ============================================================
# [Your data generation or loading code]

# ============================================================
# SECTION 2: DATA PREPROCESSING
# ============================================================
# [Your preprocessing code]

# ============================================================
# SECTION 3: ANALYSIS / MAIN ALGORITHM
# ============================================================
# [Your core analysis code]

# ============================================================
# SECTION 4: RESULTS & VISUALIZATION
# ============================================================
# [Your visualization code]

# ============================================================
# SECTION 5: CONCLUSIONS
# ============================================================
# [Summary of findings]

print("="*60)
print("CAPSTONE PROJECT COMPLETE")
print("="*60)
```

---

## Common Capstone Project Templates

### Template 1: Tissue Engineering Scaffold Analysis

```python
"""
Capstone: Tissue Engineering Scaffold Analysis
===============================================
Analyze scaffold properties for bone regeneration.
"""

def analyze_scaffold():
    """Main analysis function."""
    # Parameters
    porosity = 0.70  # 70% porosity
    pore_size = 300e-6  # 300 μm
    material_modulus = 2e9  # 2 GPa (PLGA)
    
    # Calculate scaffold modulus (Gibson-Ashby)
    scaffold_modulus = material_modulus * (1 - porosity)**2
    
    # Results
    print(f"Scaffold Properties:")
    print(f"  Porosity: {porosity*100:.0f}%")
    print(f"  Pore size: {pore_size*1e6:.0f} μm")
    print(f"  Scaffold modulus: {scaffold_modulus/1e6:.1f} MPa")
    
    return scaffold_modulus

if __name__ == "__main__":
    analyze_scaffold()
```

### Template 2: Signal Processing Pipeline

```python
"""
Capstone: Biomedical Signal Processing Pipeline
================================================
Process EEG/ECG signals for clinical analysis.
"""

def signal_processing_pipeline(raw_signal, fs):
    """Main signal processing pipeline."""
    import numpy as np
    from scipy import signal
    
    # 1. Preprocessing
    # Bandpass filter
    nyquist = fs / 2
    low = 0.5 / nyquist
    high = 50 / nyquist
    b, a = signal.butter(4, [low, high], btype='band')
    filtered = signal.filtfilt(b, a, raw_signal)
    
    # 2. Feature extraction
    # Power spectral density
    freqs, psd = signal.welch(filtered, fs, nperseg=1024)
    
    # 3. Classification
    # Simple threshold-based
    alpha_power = np.sum(psd[(freqs > 8) & (freqs < 13)])
    beta_power = np.sum(psd[(freqs > 13) & (freqs < 30)])
    
    print(f"Alpha power: {alpha_power:.2f}")
    print(f"Beta power: {beta_power:.2f}")
    
    return filtered, freqs, psd

if __name__ == "__main__":
    print("Signal processing pipeline ready.")
```

### Template 3: Bioinformatics Analysis

```python
"""
Capstone: RNA-seq Differential Expression Analysis
===================================================
Identify DE genes from treatment vs. control.
"""

def differential_expression_analysis(counts_df, groups):
    """Main DE analysis."""
    from scipy import stats
    import numpy as np
    
    results = []
    
    for gene in counts_df.columns:
        control = counts_df.loc[groups == 'control', gene]
        treatment = counts_df.loc[groups == 'treatment', gene]
        
        # t-test
        t_stat, p_value = stats.ttest_ind(control, treatment)
        
        # Fold change
        fc = treatment.mean() / control.mean() if control.mean() > 0 else 0
        log2fc = np.log2(fc) if fc > 0 else 0
        
        results.append({
            'gene': gene,
            'log2FC': log2fc,
            'p_value': p_value,
            'significant': p_value < 0.05 and abs(log2fc) > 1
        })
    
    return pd.DataFrame(results)

if __name__ == "__main__":
    print("DE analysis pipeline ready.")
```

---

## Validation Framework

```python
"""
Validation Framework for Capstone Project
==========================================
Ensure results are robust and reproducible.
"""

def validate_results(your_results, expected_baseline=None):
    """
    Validate capstone results against known standards.
    """
    # 1. Internal consistency
    print("1. Internal Consistency Checks:")
    print(f"   - Data points: {len(your_results)}")
    print(f"   - Missing values: {your_results.isna().sum()}")
    
    # 2. Reproducibility (if applicable)
    print("\n2. Reproducibility:")
    print("   - Random seed set: Yes")
    print("   - Parameters documented: Yes")
    
    # 3. Comparison to literature (if baseline available)
    if expected_baseline is not None:
        print("\n3. Comparison to Literature:")
        diff = abs(your_results - expected_baseline) / expected_baseline * 100
        print(f"   - Deviation from expected: {diff:.1f}%")
    
    # 4. Statistical robustness
    print("\n4. Statistical Robustness:")
    print("   - Appropriate tests used: [Yes/No]")
    print("   - Effect sizes reported: [Yes/No]")
    
    return True

# Example usage
print("Validation framework ready.")
```

---

## Figure Generation Template

```python
"""
Figure Generation for Capstone Report
=====================================
Publication-quality figures.
"""

def generate_figures(data_dict, output_dir='figures/'):
    """Generate all figures for capstone report."""
    import os
    os.makedirs(output_dir, exist_ok=True)
    
    plt.style.use('seaborn-v0_8-whitegrid')
    
    # Figure 1: Main result
    fig1, ax1 = plt.subplots(figsize=(8, 6))
    # [Your plotting code]
    ax1.set_xlabel('X Label')
    ax1.set_ylabel('Y Label')
    ax1.set_title('Figure 1: Main Result')
    fig1.savefig(f'{output_dir}fig1_main_result.png', dpi=300, bbox_inches='tight')
    
    # Figure 2: Supplementary
    fig2, axes = plt.subplots(1, 2, figsize=(12, 5))
    # [Your plotting code]
    fig2.savefig(f'{output_dir}fig2_supplementary.png', dpi=300, bbox_inches='tight')
    
    plt.close('all')
    print(f"Figures saved to {output_dir}")
    
    return output_dir

if __name__ == "__main__":
    print("Figure generation ready.")
```

---

## Documentation Checklist

Before submission, verify:

- [ ] Code runs without errors
- [ ] All functions documented (docstrings)
- [ ] Variables named clearly
- [ ] Figures saved in high resolution (300 dpi)
- [ ] Output clearly labeled
- [ ] Results match written report

---

## Submission Instructions

Your Week 23 deliverable includes:

1. **Code** (`code/` folder):
   - Complete, runnable implementation
   - Clean, commented code
   - README with setup instructions

2. **Preliminary Results** (`deliverables/` folder):
   - Draft of results section
   - All figures with captions
   - Statistical analysis summary

3. **Presentation** (`deliverables/` folder):
   - Draft presentation slides
   - 15-minute talk outline

Submit all to the deliverables folder by Sunday midnight.
