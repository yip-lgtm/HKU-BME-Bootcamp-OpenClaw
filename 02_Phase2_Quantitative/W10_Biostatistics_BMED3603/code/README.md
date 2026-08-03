# Week 10 Code Lab — Biostatistics: Hypothesis Testing & ANOVA (BMED3603)

> **Topics Covered**: Descriptive statistics, t-tests, ANOVA, regression, confidence intervals, power analysis
> **Prerequisites**: numpy, scipy, matplotlib, statsmodels
> **Estimated Time**: 2-3 hours

---

## Lab 1: Descriptive Statistics & Hypothesis Testing

### Objective
Perform descriptive statistical analysis of clinical data, conduct one-sample, independent two-sample, and paired t-tests, compute confidence intervals, and interpret results in a clinical context.

### Code

```python
"""
Week 10 Lab 1: Descriptive Statistics & Hypothesis Testing
==========================================================
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy import stats
import warnings
warnings.filterwarnings('ignore')

print("=" * 65)
print("LAB 1: Descriptive Statistics & Hypothesis Testing")
print("=" * 65)

# ---------------------------------------------------------
# SECTION 1: Descriptive Statistics of Clinical Data
# ---------------------------------------------------------
print("\n[1] Descriptive Statistics — Blood Pressure Dataset")

# Simulated clinical data: systolic blood pressure (mmHg)
# Two treatment groups: Drug A (standard) vs Drug B (new)
np.random.seed(123)

# Drug A: standard treatment, mean=130, SD=12
drug_A = np.array([128, 135, 142, 118, 131, 145, 122, 138, 127, 133,
                   140, 119, 136, 129, 144, 124, 137, 130, 126, 141,
                   132, 139, 125, 134, 143, 120, 147, 123, 131, 115])

# Drug B: new treatment, mean=122, SD=10
drug_B = np.array([118, 125, 122, 115, 120, 128, 112, 127, 119, 124,
                   121, 114, 126, 118, 130, 116, 129, 122, 117, 133,
                   119, 124, 113, 121, 127, 111, 134, 115, 123, 110])

def descriptive_stats(data, name=""):
    """Compute comprehensive descriptive statistics"""
    n = len(data)
    mean = np.mean(data)
    median = np.median(data)
    std = np.std(data, ddof=1)  # sample SD
    sem = std / np.sqrt(n)  # standard error of mean
    var = np.var(data, ddof=1)
    min_val = np.min(data)
    max_val = np.max(data)
    q1 = np.percentile(data, 25)
    q3 = np.percentile(data, 75)
    iqr = q3 - q1
    skewness = stats.skew(data)
    kurtosis = stats.kurtosis(data)
    
    print(f"\n  [{name}]")
    print(f"    n = {n}")
    print(f"    Mean ± SD: {mean:.2f} ± {std:.2f} mmHg")
    print(f"    Median (Q2): {median:.1f} mmHg")
    print(f"    Range: [{min_val:.0f}, {max_val:.0f}] mmHg")
    print(f"    IQR: [{q1:.1f}, {q3:.1f}] ({iqr:.1f} mmHg)")
    print(f"    95% CI: [{mean - 1.96*sem:.2f}, {mean + 1.96*sem:.2f}] mmHg")
    print(f"    Standard Error: {sem:.3f} mmHg")
    print(f"    Skewness: {skewness:.3f} ({'right' if skewness > 0.5 else 'left' if skewness < -0.5 else 'approximately symmetric'})")
    print(f"    Kurtosis: {kurtosis:.3f} ({'heavy-tailed' if kurtosis > 1 else 'light-tailed' if kurtosis < -1 else 'normal'})")
    
    return {'n': n, 'mean': mean, 'std': std, 'median': median, 'sem': sem,
            'q1': q1, 'q3': q3, 'skewness': skewness, 'kurtosis': kurtosis}

stats_A = descriptive_stats(drug_A, "Drug A (Standard)")
stats_B = descriptive_stats(drug_B, "Drug B (New)")

# ---------------------------------------------------------
# SECTION 2: Normality Testing
# ---------------------------------------------------------
print("\n[2] Normality Testing — Should We Use Parametric Tests?")

for name, data in [("Drug A", drug_A), ("Drug B", drug_B)]:
    # Shapiro-Wilk test (best for n < 50)
    shapiro_stat, shapiro_p = stats.shapiro(data)
    
    # D'Agostino-Pearson test
    dagostino_stat, dagostino_p = stats.normaltest(data)
    
    # Anderson-Darling test
    anderson_result = stats.anderson(data, dist='norm')
    
    print(f"\n  [{name}]:")
    print(f"    Shapiro-Wilk: W = {shapiro_stat:.4f}, p = {shapiro_p:.4f}")
    print(f"    D'Agostino: K² = {dagostino_stat:.4f}, p = {dagostino_p:.4f}")
    print(f"    Anderson-Darling: statistic = {anderson_result.statistic:.4f}")
    print(f"    Anderson-Darling critical values: {anderson_result.critical_values}")
    
    # Decision
    alpha = 0.05
    if shapiro_p < alpha:
        print(f"    ⚠️ Shapiro-Wilk p < 0.05 → Reject H₀: Data is NOT normally distributed")
    else:
        print(f"    ✓ Shapiro-Wilk p ≥ 0.05 → Fail to reject H₀: Data appears normal")

# ---------------------------------------------------------
# SECTION 3: One-Sample t-test
# ---------------------------------------------------------
print("\n[3] One-Sample t-test — Is Drug A Mean Different from Population Norm?")

# Population norm for systolic BP: μ₀ = 130 mmHg
mu_0 = 130
t_stat_1samp, p_val_1samp = stats.ttest_1samp(drug_A, mu_0)

n_A = len(drug_A)
mean_A = np.mean(drug_A)
std_A = np.std(drug_A, ddof=1)
df_1samp = n_A - 1
se_A = std_A / np.sqrt(n_A)
t_critical = stats.t.ppf(0.975, df_1samp)  # two-tailed, α=0.05
ci_1samp = (mean_A - t_critical * se_A, mean_A + t_critical * se_A)

print(f"  H₀: μ = {mu_0} mmHg (population norm)")
print(f"  H₁: μ ≠ {mu_0} mmHg (two-tailed)")
print(f"  n = {n_A}, df = {df_1samp}")
print(f"  Sample mean = {mean_A:.2f} mmHg")
print(f"  t-statistic = {t_stat_1samp:.4f}")
print(f"  p-value = {p_val_1samp:.6f}")
print(f"  t-critical (±) = ±{t_critical:.4f}")
print(f"  95% CI: [{ci_1samp[0]:.2f}, {ci_1samp[1]:.2f}] mmHg")

alpha = 0.05
if p_val_1samp < alpha:
    print(f"  Conclusion: p = {p_val_1samp:.4f} < 0.05 → Reject H₀")
    print(f"  Drug A significantly lowers BP compared to population norm (130 mmHg)")
else:
    print(f"  Conclusion: p = {p_val_1samp:.4f} ≥ 0.05 → Fail to reject H₀")
    print(f"  No significant difference from population norm")

# Effect size (Cohen's d)
cohens_d = (mean_A - mu_0) / std_A
print(f"\n  Effect size (Cohen's d): {cohens_d:.3f}")
print(f"    |d| < 0.2 → negligible, 0.2 ≤ |d| < 0.5 → small, 0.5 ≤ |d| < 0.8 → medium, |d| ≥ 0.8 → large")

# ---------------------------------------------------------
# SECTION 4: Independent Two-Sample t-test
# ---------------------------------------------------------
print("\n[4] Independent Two-Sample t-test — Drug A vs Drug B")

# First check equal variance assumption
levene_stat, levene_p = stats.levene(drug_A, drug_B)
print(f"  Levene's test for equal variances:")
print(f"    W = {levene_stat:.4f}, p = {levene_p:.4f}")

if levene_p < 0.05:
    print(f"  ⚠️ Unequal variances → Use Welch's t-test")
    use_welch = True
else:
    print(f"  ✓ Equal variances assumed → Use standard t-test")
    use_welch = False

# Welch's t-test (unequal variances)
t_stat_welch, p_val_welch = stats.ttest_ind(drug_A, drug_B, equal_var=False)
# Standard t-test (equal variances)
t_stat_std, p_val_std = stats.ttest_ind(drug_A, drug_B, equal_var=True)

print(f"\n  Standard t-test (equal var): t = {t_stat_std:.4f}, p = {p_val_std:.6f}")
print(f"  Welch's t-test (unequal var): t = {t_stat_welch:.4f}, p = {p_val_welch:.6f}")

# Confidence interval for difference in means
mean_diff = np.mean(drug_A) - np.mean(drug_B)
se_diff = np.sqrt(std_A**2/n_A + np.std(drug_B, ddof=1)**2/len(drug_B))
df_welch = (std_A**2/n_A + np.std(drug_B,ddof=1)**2/len(drug_B))**2 / \
           ((std_A**2/n_A)**2/(n_A-1) + (np.std(drug_B,ddof=1)**2/len(drug_B))**2/(len(drug_B)-1))
t_crit_diff = stats.t.ppf(0.975, df_welch)
ci_diff = (mean_diff - t_crit_diff * se_diff, mean_diff + t_crit_diff * se_diff)

print(f"\n  Mean difference (Drug A − Drug B): {mean_diff:.2f} mmHg")
print(f"  95% CI for difference: [{ci_diff[0]:.2f}, {ci_diff[1]:.2f}] mmHg")
print(f"  df (Welch) ≈ {df_welch:.1f}")

alpha = 0.05
if p_val_welch < alpha:
    print(f"  Conclusion: p = {p_val_welch:.6f} < 0.05 → Reject H₀")
    print(f"  Drug B significantly lowers BP compared to Drug A")
else:
    print(f"  Conclusion: Fail to reject H₀")

# Effect size: Cohen's d for independent groups
pooled_std = np.sqrt(((n_A-1)*std_A**2 + (len(drug_B)-1)*np.std(drug_B,ddof=1)**2) / (n_A+len(drug_B)-2))
cohens_d_independent = mean_diff / pooled_std
print(f"  Cohen's d: {cohens_d_independent:.3f}")

# ---------------------------------------------------------
# SECTION 5: Paired t-test — Before/After Treatment
# ---------------------------------------------------------
print("\n[5] Paired t-test — Before vs After Treatment (Drug B)")

# Simulate paired data: same patients measured before and after treatment
np.random.seed(456)
n_paired = 25
before = drug_B[:n_paired]
# After treatment: reduce by 5-15 mmHg with some noise
after = before - 8 - 3 * np.random.randn(n_paired)

# Paired differences
differences = after - before
d_mean = np.mean(differences)
d_std = np.std(differences, ddof=1)
d_se = d_std / np.sqrt(n_paired)
df_paired = n_paired - 1

t_stat_paired, p_val_paired = stats.ttest_1samp(differences, 0)
ci_paired = (d_mean - stats.t.ppf(0.975, df_paired) * d_se,
             d_mean + stats.t.ppf(0.975, df_paired) * d_se)

print(f"  n = {n_paired} paired observations")
print(f"  Mean difference (after − before): {d_mean:.2f} mmHg")
print(f"  SD of differences: {d_std:.2f} mmHg")
print(f"  95% CI for mean difference: [{ci_paired[0]:.2f}, {ci_paired[1]:.2f}] mmHg")
print(f"  t-statistic = {t_stat_paired:.4f}, df = {df_paired}")
print(f"  p-value = {p_val_paired:.8f}")

# Effect size: Cohen's d for paired
cohens_d_paired = d_mean / d_std
print(f"  Cohen's d (paired): {cohens_d_paired:.3f}")

# ---------------------------------------------------------
# SECTION 6: Visualization
# ---------------------------------------------------------
fig, axes = plt.subplots(2, 3, figsize=(16, 10))

# Histograms with normal overlay
for i, (name, data) in enumerate([("Drug A", drug_A), ("Drug B", drug_B)]):
    ax = axes[0, i]
    ax.hist(data, bins=12, density=True, alpha=0.7, color=['blue', 'green'][i], edgecolor='black')
    
    # Normal distribution overlay
    x = np.linspace(data.min()-5, data.max()+5, 100)
    ax.plot(x, stats.norm.pdf(x, np.mean(data), np.std(data, ddof=1)),
            'r-', linewidth=2, label='Normal fit')
    
    ax.set_title(f'{name}: μ={np.mean(data):.1f}, σ={np.std(data,ddof=1):.1f}', fontsize=12)
    ax.set_xlabel('Systolic BP (mmHg)')
    ax.set_ylabel('Density')
    ax.legend()

# Box plots
axes[0, 2].boxplot([drug_A, drug_B], labels=['Drug A', 'Drug B'], patch_artist=True,
                    boxprops=dict(facecolor='lightblue', alpha=0.7))
axes[0, 2].scatter([1]*len(drug_A), drug_A, alpha=0.3, s=20, color='blue')
axes[0, 2].scatter([2]*len(drug_B), drug_B, alpha=0.3, s=20, color='green')
axes[0, 2].axhline(y=130, color='r', linestyle='--', label='Population norm (130)')
axes[0, 2].set_title('Box Plot: Drug A vs Drug B', fontsize=12)
axes[0, 2].set_ylabel('Systolic BP (mmHg)')
axes[0, 2].legend()

# QQ plots
for i, (name, data) in enumerate([("Drug A", drug_A), ("Drug B", drug_B)]):
    ax = axes[1, i]
    stats.probplot(data, dist="norm", plot=ax)
    ax.set_title(f'Q-Q Plot: {name}', fontsize=12)
    ax.grid(True, alpha=0.3)

# Before/After paired plot
axes[1, 2].scatter(before, after, alpha=0.6, s=50, c='purple')
ax_range = [min(before.min(), after.min())-3, max(before.max(), after.max())+3]
axes[1, 2].plot(ax_range, ax_range, 'k--', linewidth=1.5, label='y=x (no change)')
# Add lines for each point
for i in range(n_paired):
    axes[1, 2].plot([before[i], before[i]], [before[i], after[i]], 'b-', alpha=0.3)
axes[1, 2].set_xlabel('Before Treatment (mmHg)')
axes[1, 2].set_ylabel('After Treatment (mmHg)')
axes[1, 2].set_title(f'Before vs After (Drug B): mean diff={d_mean:.1f} mmHg', fontsize=12)
axes[1, 2].legend()
axes[1, 2].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('lab1_stats.png', dpi=150)
plt.show()
print("  [Saved] lab1_stats.png")

print("\n✅ Lab 1 Complete!")
```

### Expected Output
- Descriptive statistics table for both drug groups
- Normality test results (Shapiro-Wilk, D'Agostino)
- t-test results (one-sample, independent, paired) with p-values and confidence intervals
- Histograms with normal distribution overlay
- Box plots and Q-Q plots

---

## Lab 2: ANOVA — Comparing Multiple Groups

### Objective
Conduct one-way and two-way ANOVA to compare means across multiple groups, perform post-hoc tests (Tukey HSD, Bonferroni), and interpret interaction effects.

### Code

```python
"""
Week 10 Lab 2: ANOVA — Comparing Multiple Groups
================================================
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy import stats
import statsmodels.api as sm
from statsmodels.formula.api import ols
from statsmodels.stats.multicomp import pairwise_tukeyhsd
import warnings
warnings.filterwarnings('ignore')

print("=" * 65)
print("LAB 2: ANOVA — Comparing Multiple Groups")
print("=" * 65)

# ---------------------------------------------------------
# SECTION 1: One-Way ANOVA — Pain Relief Study
# ---------------------------------------------------------
print("\n[1] One-Way ANOVA — Comparing Three Analgesics")

# Pain scores (0-10 scale) for three treatments
# Treatment A: placebo
treatment_A = np.array([7.2, 6.8, 7.5, 6.9, 7.1, 7.3, 6.5, 7.4, 7.0, 6.7,
                        7.6, 6.4, 7.8, 6.6, 7.0])
# Treatment B: standard analgesic
treatment_B = np.array([4.2, 3.9, 4.5, 4.1, 3.8, 4.3, 4.0, 3.7, 4.4, 4.1,
                        3.6, 4.8, 3.5, 4.2, 3.9])
# Treatment C: new analgesic
treatment_C = np.array([2.1, 1.8, 2.4, 1.9, 2.2, 1.7, 2.5, 1.6, 2.3, 1.5,
                        2.0, 1.4, 2.6, 1.3, 1.9])

# Descriptive statistics
for name, data in [("Treatment A (Placebo)", treatment_A),
                    ("Treatment B (Standard)", treatment_B),
                    ("Treatment C (New)", treatment_C)]:
    print(f"\n  {name}:")
    print(f"    n = {len(data)}, Mean = {np.mean(data):.2f}, SD = {np.std(data, ddof=1):.2f}")

# One-way ANOVA using scipy
f_stat, p_val_anova = stats.f_oneway(treatment_A, treatment_B, treatment_C)
print(f"\n  One-Way ANOVA Results:")
print(f"    F-statistic = {f_stat:.4f}")
print(f"    p-value = {p_val_anova:.8f}")

# Effect size: eta-squared (η²)
# η² = SS_between / SS_total
all_data = np.concatenate([treatment_A, treatment_B, treatment_C])
grand_mean = np.mean(all_data)
n_total = len(all_data)

ss_total = np.sum((all_data - grand_mean)**2)
ss_between = (len(treatment_A)*(np.mean(treatment_A)-grand_mean)**2 +
              len(treatment_B)*(np.mean(treatment_B)-grand_mean)**2 +
              len(treatment_C)*(np.mean(treatment_C)-grand_mean)**2)
ss_within = ss_total - ss_between

eta_squared = ss_between / ss_total
omega_squared = (ss_between - (3-1)*ss_within/(n_total-3)) / (ss_total + ss_within/(n_total-3))

print(f"\n  Effect Size:")
print(f"    η² (eta-squared) = {eta_squared:.4f}")
print(f"    ω² (omega-squared) = {omega_squared:.4f}")
print(f"    Interpretation: {eta_squared*100:.1f}% of variance in pain scores is explained by treatment")

# Degrees of freedom
df_between = 3 - 1  # k - 1
df_within = n_total - 3  # N - k
df_total = n_total - 1  # N - 1

print(f"\n  ANOVA Table:")
print(f"    {'Source':<12} {'SS':>12} {'df':>6} {'MS':>12} {'F':>12}")
print(f"    {'-'*54}")
print(f"    {'Between':<12} {ss_between:>12.4f} {df_between:>6} {ss_between/df_between:>12.4f} {f_stat:>12.4f}")
print(f"    {'Within':<12} {ss_within:>12.4f} {df_within:>6} {ss_within/df_within:>12.4f} {'':>12}")
print(f"    {'Total':<12} {ss_total:>12.4f} {df_total:>6} {'':>12} {'':>12}")

# ---------------------------------------------------------
# SECTION 2: Post-Hoc Tests
# ---------------------------------------------------------
print("\n[2] Post-Hoc Tests — Which Groups Differ?")

# Combine data for post-hoc
all_pain = np.concatenate([treatment_A, treatment_B, treatment_C])
groups = np.concatenate([['A']*len(treatment_A), ['B']*len(treatment_B), ['C']*len(treatment_C)])

# Tukey HSD
tukey = pairwise_tukeyhsd(all_pain, groups, alpha=0.05)
print("\n  Tukey HSD Results:")
print(tukey.summary())

# Pairwise t-tests with Bonferroni correction
pairs = [('A', 'B'), ('A', 'C'), ('B', 'C')]
alpha = 0.05
n_comparisons = len(pairs)
bonferroni_alpha = alpha / n_comparisons

print(f"\n  Pairwise t-tests (Bonferroni-corrected α = {bonferroni_alpha:.4f}):")
for g1, g2 in pairs:
    data1 = treatment_A if g1 == 'A' else treatment_B if g1 == 'B' else treatment_C
    data2 = treatment_A if g2 == 'A' else treatment_B if g2 == 'B' else treatment_C
    t, p = stats.ttest_ind(data1, data2)
    sig = "***" if p < 0.001 else "**" if p < 0.01 else "*" if p < bonferroni_alpha else "ns"
    print(f"    {g1} vs {g2}: t = {t:.4f}, p = {p:.6f} {sig}")

# ---------------------------------------------------------
# SECTION 3: Two-Way ANOVA — Drug × Dosage Study
# ---------------------------------------------------------
print("\n[3] Two-Way ANOVA — Drug × Dosage Interaction")

# Simulated data: pain relief score = f(drug, dosage)
# 3 drugs × 2 dosages = 6 groups
np.random.seed(789)
n_per_cell = 10

# Drug A: weak, Drug B: moderate, Drug C: strong
drug_effect = {'A': 0, 'B': 2.5, 'C': 5.0}
# Low dose: small effect, High dose: larger effect
dose_effect = {'Low': 0, 'High': 3.0}
# Interaction: Drug C benefits more from high dose
interaction = {('C', 'High'): 2.0, ('A', 'High'): 0.5, ('B', 'High'): 1.0}

base_score = 6.0  # baseline pain score
data_2way = []

for drug in ['A', 'B', 'C']:
    for dose in ['Low', 'High']:
        mean = (base_score - drug_effect[drug] + 
                 (base_score - dose_effect[dose]) + 
                 base_score - interaction.get((drug, dose), 0))
        scores = mean + 0.8 * np.random.randn(n_per_cell)
        for s in scores:
            data_2way.append({'Drug': drug, 'Dose': dose, 'Score': s})

# Convert to arrays
drug_arr = np.array([d['Drug'] for d in data_2way])
dose_arr = np.array([d['Dose'] for d in data_2way])
score_arr = np.array([d['Score'] for d in data_2way])

# Two-way ANOVA using statsmodels
model = ols('Score ~ C(Drug) + C(Dose) + C(Drug):C(Dose)', data={'Score': score_arr, 
                                                                   'Drug': drug_arr, 
                                                                   'Dose': dose_arr}).fit()
anova_table = sm.stats.anova_lm(model, typ=2)

print("\n  Two-Way ANOVA Results:")
print(anova_table)

print("\n  Interpretation:")
print(f"    Drug effect: F = {anova_table.loc['C(Drug)', 'F']:.4f}, p = {anova_table.loc['C(Drug)', 'PR(>F)']:.6f}")
print(f"    Dose effect: F = {anova_table.loc['C(Dose)', 'F']:.4f}, p = {anova_table.loc['C(Dose)', 'PR(>F)']:.6f}")
print(f"    Interaction: F = {anova_table.loc['C(Drug):C(Dose)', 'F']:.4f}, p = {anova_table.loc['C(Drug):C(Dose)', 'PR(>F)']:.6f}")

# Interaction plot
fig, axes = plt.subplots(1, 3, figsize=(15, 5))

# Group means for interaction plot
means = {}
for drug in ['A', 'B', 'C']:
    for dose in ['Low', 'High']:
        mask = (drug_arr == drug) & (dose_arr == dose)
        means[(drug, dose)] = np.mean(score_arr[mask])

# Plot
for i, drug in enumerate(['A', 'B', 'C']):
    axes[i].plot(['Low', 'High'], [means[(drug, 'Low')], means[(drug, 'High')]], 
                 'o-', markersize=10, linewidth=2)
    axes[i].set_title(f'Drug {drug}', fontsize=12)
    axes[i].set_xlabel('Dosage')
    axes[i].set_ylabel('Pain Relief Score')
    axes[i].grid(True, alpha=0.3)

plt.suptitle('Interaction Plot: Drug × Dosage', fontsize=14, y=1.02)
plt.tight_layout()
plt.savefig('lab2_anova.png', dpi=150)
plt.show()
print("  [Saved] lab2_anova.png")

# ---------------------------------------------------------
# SECTION 4: Non-Parametric Alternative — Kruskal-Wallis
# ---------------------------------------------------------
print("\n[4] Non-Parametric Alternative — Kruskal-Wallis Test")

# Kruskal-Wallis (non-parametric alternative to one-way ANOVA)
h_stat, p_kw = stats.kruskal(treatment_A, treatment_B, treatment_C)
print(f"  Kruskal-Wallis H-test:")
print(f"    H = {h_stat:.4f}, p = {p_kw:.8f}")

# Mann-Whitney U for pairwise comparisons
print("\n  Pairwise Mann-Whitney U tests:")
for g1, g2, d1, d2 in [('A', 'B', treatment_A, treatment_B),
                         ('A', 'C', treatment_A, treatment_C),
                         ('B', 'C', treatment_B, treatment_C)]:
    u_stat, p_mw = stats.mannwhitneyu(d1, d2, alternative='two-sided')
    print(f"    {g1} vs {g2}: U = {u_stat:.1f}, p = {p_mw:.6f}")

print("\n✅ Lab 2 Complete!")
```

### Expected Output
- ANOVA table with F-statistic and p-value
- Post-hoc Tukey HSD results showing which groups differ
- Two-way ANOVA with interaction effect
- Interaction plots showing drug × dosage effects

---

## Lab 3: Regression & Power Analysis

### Objective
Perform linear regression, multiple regression, assess model assumptions (residuals, normality, homoscedasticity), and conduct power analysis for sample size determination.

### Code

```python
"""
Week 10 Lab 3: Regression Analysis & Power Analysis
===================================================
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy import stats
import statsmodels.api as sm
from statsmodels.stats.diagnostic import het_breuschpagan, lilliefors
import warnings
warnings.filterwarnings('ignore')

print("=" * 65)
print("LAB 3: Regression Analysis & Power Analysis")
print("=" * 65)

# ---------------------------------------------------------
# SECTION 1: Linear Regression — Weight vs Blood Pressure
# ---------------------------------------------------------
print("\n[1] Linear Regression — Body Weight vs Systolic BP")

np.random.seed(321)

# Simulated clinical data
n_patients = 50
age = np.random.normal(55, 10, n_patients)
weight = np.random.normal(75, 12, n_patients)  # kg
height = np.random.normal(170, 8, n_patients)   # cm

# True relationship: BP = 90 + 0.3*age + 0.5*weight + noise
bp_true = 90 + 0.3 * age + 0.5 * weight
bp = bp_true + 8 * np.random.randn(n_patients)

# Simple linear regression: BP ~ Weight
slope, intercept, r_value, p_value, std_err = stats.linregress(weight, bp)
r_squared = r_value**2

print(f"  Simple Linear Regression: BP = β₀ + β₁·Weight")
print(f"    β₀ (intercept) = {intercept:.4f}")
print(f"    β₁ (slope) = {slope:.4f}")
print(f"    SE(β₁) = {std_err:.4f}")
print(f"    t-statistic = {slope/std_err:.4f}")
print(f"    p-value = {p_value:.8f}")
print(f"    R² = {r_squared:.4f} ({r_squared*100:.1f}% of variance explained)")
print(f"    Pearson r = {r_value:.4f}")

# Regression equation
print(f"\n  Regression equation:")
print(f"    SBP = {intercept:.2f} + {slope:.4f} × Weight")
print(f"    Interpretation: Each 1 kg increase in weight → {slope:.3f} mmHg increase in SBP")

# ---------------------------------------------------------
# SECTION 2: Multiple Regression
# ---------------------------------------------------------
print("\n[2] Multiple Regression — BP ~ Age + Weight")

# Create design matrix
X = np.column_stack([np.ones(n_patients), age, weight])
y = bp

# OLS using numpy (manual)
beta = np.linalg.lstsq(X, y, rcond=None)[0]
y_pred = X @ beta
residuals = y - y_pred
ss_res = np.sum(residuals**2)
ss_tot = np.sum((y - np.mean(y))**2)
r_squared_multi = 1 - ss_res/ss_tot
r_squared_adj = 1 - (1-r_squared_multi)*(n_patients-1)/(n_patients-3)

# Standard errors of coefficients
mse = ss_res / (n_patients - 3)  # MSE with p=3 parameters
XtX_inv = np.linalg.inv(X.T @ X)
se_beta = np.sqrt(np.diag(XtX_inv) * mse)

t_stats = beta / se_beta
p_values = 2 * (1 - stats.t.cdf(np.abs(t_stats), df=n_patients-3))

print(f"  Multiple Regression: SBP = β₀ + β₁·Age + β₂·Weight")
print(f"    {'Variable':<12} {'Coefficient':>12} {'SE':>10} {'t-stat':>10} {'p-value':>12}")
print(f"    {'-'*56}")
for name, b, s, t, p in zip(['Intercept', 'Age', 'Weight'], beta, se_beta, t_stats, p_values):
    sig = '***' if p < 0.001 else '**' if p < 0.01 else '*' if p < 0.05 else ''
    print(f"    {name:<12} {b:>12.4f} {s:>10.4f} {t:>10.4f} {p:>12.6f} {sig}")

print(f"\n  R² = {r_squared_multi:.4f}")
print(f"  Adjusted R² = {r_squared_adj:.4f}")
print(f"  MSE = {mse:.4f}")
print(f"  RMSE = {np.sqrt(mse):.4f} mmHg")

# ---------------------------------------------------------
# SECTION 3: Regression Diagnostics
# ---------------------------------------------------------
print("\n[3] Regression Diagnostics")

# 1. Residual analysis
fig, axes = plt.subplots(2, 3, figsize=(16, 10))

# Fitted vs residuals
axes[0, 0].scatter(y_pred, residuals, alpha=0.6, s=50)
axes[0, 0].axhline(y=0, color='r', linestyle='--', linewidth=2)
axes[0, 0].set_xlabel('Fitted Values (mmHg)')
axes[0, 0].set_ylabel('Residuals (mmHg)')
axes[0, 0].set_title('Residuals vs Fitted (Homoscedasticity Check)', fontsize=11)

# Histogram of residuals
axes[0, 1].hist(residuals, bins=15, density=True, alpha=0.7, edgecolor='black')
x_norm = np.linspace(residuals.min(), residuals.max(), 100)
axes[0, 1].plot(x_norm, stats.norm.pdf(x_norm, 0, np.std(residuals)), 'r-', linewidth=2)
axes[0, 1].set_xlabel('Residual Value')
axes[0, 1].set_ylabel('Density')
axes[0, 1].set_title('Distribution of Residuals (Normality Check)', fontsize=11)

# Q-Q plot of residuals
stats.probplot(residuals, dist="norm", plot=axes[0, 2])
axes[0, 2].set_title('Q-Q Plot of Residuals', fontsize=11)

# Scale-Location plot (sqrt of standardized residuals)
std_resid = residuals / np.std(residuals)
axes[1, 0].scatter(y_pred, np.sqrt(np.abs(std_resid)), alpha=0.6, s=50)
axes[1, 0].set_xlabel('Fitted Values')
axes[1, 0].set_ylabel('√|Standardized Residuals|')
axes[1, 0].set_title('Scale-Location (Homoscedasticity Check)', fontsize=11)

# Residuals vs Weight
axes[1, 1].scatter(weight, residuals, alpha=0.6, s=50)
axes[1, 1].axhline(y=0, color='r', linestyle='--', linewidth=2)
axes[1, 1].set_xlabel('Weight (kg)')
axes[1, 1].set_ylabel('Residuals')
axes[1, 1].set_title('Residuals vs Weight', fontsize=11)

# Residuals vs Age
axes[1, 2].scatter(age, residuals, alpha=0.6, s=50)
axes[1, 2].axhline(y=0, color='r', linestyle='--', linewidth=2)
axes[1, 2].set_xlabel('Age (years)')
axes[1, 2].set_ylabel('Residuals')
axes[1, 2].set_title('Residuals vs Age', fontsize=11)

plt.tight_layout()
plt.savefig('lab3_regression_diagnostics.png', dpi=150)
plt.show()
print("  [Saved] lab3_regression_diagnostics.png")

# 2. Formal tests
# Shapiro-Wilk on residuals
shap_resid, p_shap = stats.shapiro(residuals)
print(f"\n  Shapiro-Wilk test for normality of residuals:")
print(f"    W = {shap_resid:.4f}, p = {p_shap:.4f}")
print(f"    {'✓ Residuals appear normal' if p_shap > 0.05 else '⚠️ Residuals NOT normal'}")

# Breusch-Pagan test for heteroscedasticity
bp_stat, bp_p, _, _ = het_breuschpagan(residuals, X[:, 1:])  # exclude intercept
print(f"\n  Breusch-Pagan test for heteroscedasticity:")
print(f"    LM = {bp_stat:.4f}, p = {bp_p:.4f}")
print(f"    {'✓ Homoscedasticity assumed' if bp_p > 0.05 else '⚠️ Heteroscedasticity detected'}")

# ---------------------------------------------------------
# SECTION 4: Power Analysis
# ---------------------------------------------------------
print("\n[4] Power Analysis — Sample Size Determination")

def power_ttest(n, delta, sigma, alpha=0.05, test='two-sided'):
    """
    Compute statistical power for a two-sample t-test.
    
    Power = P(reject H₀ | H₁ is true)
    """
    # Standard error
    se = sigma * np.sqrt(2/n)
    # Critical value
    z_alpha = stats.norm.ppf(1 - alpha/2) if test == 'two-sided' else stats.norm.ppf(1-alpha)
    # Non-centrality parameter
    ncp = delta / se
    # Power = P(|Z| > z_alpha | ncp)
    power = 1 - stats.norm.cdf(z_alpha - ncp) + stats.norm.cdf(-z_alpha - ncp)
    return power

def sample_size_ttest(delta, sigma, alpha=0.05, power=0.8, test='two-sided'):
    """
    Compute required sample size for given power.
    """
    z_alpha = stats.norm.ppf(1 - alpha/2) if test == 'two-sided' else stats.norm.ppf(1-alpha)
    z_beta = stats.norm.ppf(power)
    n = 2 * ((z_alpha + z_beta) * sigma / delta)**2
    return int(np.ceil(n))

# Clinical scenario: comparing two drugs
# Minimum clinically important difference: Δ = 5 mmHg
# Expected SD: σ = 12 mmHg (from pilot study)
delta_clinical = 5.0  # mmHg
sigma_clinical = 12.0  # mmHg

print(f"\n  Clinical Scenario: Comparing Drug A vs Drug B")
print(f"    Minimum clinically important difference: Δ = {delta_clinical} mmHg")
print(f"    Expected SD: σ = {sigma_clinical} mmHg")
print(f"    Significance level: α = 0.05 (two-tailed)")

for target_power in [0.80, 0.90, 0.95]:
    n_required = sample_size_ttest(delta_clinical, sigma_clinical, alpha=0.05, 
                                   power=target_power, test='two-sided')
    print(f"\n  For power = {target_power*100:.0f}%:")
    print(f"    Required n per group: {n_required}")
    print(f"    Total N: {2*n_required}")
    
    # Verify power
    actual_power = power_ttest(n_required, delta_clinical, sigma_clinical)
    print(f"    Actual power: {actual_power:.4f}")

# Power curve
ns = np.arange(5, 101, 1)
powers = [power_ttest(n, delta_clinical, sigma_clinical) for n in ns]

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

axes[0].plot(ns, powers, 'b-', linewidth=2)
axes[0].axhline(y=0.8, color='r', linestyle='--', label='Power = 0.8')
axes[0].axhline(y=0.9, color='orange', linestyle='--', label='Power = 0.9')
axes[0].axhline(y=0.95, color='green', linestyle='--', label='Power = 0.95')
n80 = sample_size_ttest(delta_clinical, sigma_clinical, power=0.8)
axes[0].axvline(x=n80, color='r', linestyle=':', alpha=0.7)
axes[0].scatter([n80], [0.8], color='r', s=100, zorder=5)
axes[0].annotate(f'n={n80}', xy=(n80, 0.8), xytext=(n80+5, 0.75), fontsize=10)
axes[0].set_xlabel('Sample Size per Group (n)')
axes[0].set_ylabel('Statistical Power')
axes[0].set_title('Power Curve: Effect of Sample Size', fontsize=12)
axes[0].legend()
axes[0].grid(True, alpha=0.3)

# Power vs effect size
effect_sizes = np.linspace(1, 15, 100)
ns_for_power = [power_ttest(30, d, sigma_clinical) for d in effect_sizes]
axes[1].plot(effect_sizes, ns_for_power, 'b-', linewidth=2)
axes[1].axhline(y=0.8, color='r', linestyle='--')
axes[1].axvline(x=delta_clinical, color='g', linestyle=':', alpha=0.7)
axes[1].annotate(f'Δ={delta_clinical}', xy=(delta_clinical, 0.8), xytext=(delta_clinical+1, 0.7), fontsize=10)
axes[1].set_xlabel('Effect Size (Δ in mmHg)')
axes[1].set_ylabel('Power (for n=30 per group)')
axes[1].set_title('Power vs Effect Size (n=30 per group)', fontsize=12)
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('lab3_power_analysis.png', dpi=150)
plt.show()
print("  [Saved] lab3_power_analysis.png")

print("\n✅ Lab 3 Complete!")
print("\n" + "=" * 65)
print("LAB SUMMARY: Week 10 Biostatistics")
print("=" * 65)
print("✅ Descriptive statistics: mean, SD, median, IQR, CI, skewness, kurtosis")
print("✅ Normality testing: Shapiro-Wilk, D'Agostino, Anderson-Darling")
print("✅ One-sample t-test: compare sample to population norm")
print("✅ Independent t-test: compare two independent groups")
print("✅ Paired t-test: before/after measurements")
print("✅ One-way ANOVA: compare k > 2 groups")
print("✅ Post-hoc tests: Tukey HSD, Bonferroni correction")
print("✅ Two-way ANOVA: interaction effects (Drug × Dosage)")
print("✅ Linear regression: BP ~ Weight")
print("✅ Multiple regression: BP ~ Age + Weight")
print("✅ Regression diagnostics: residuals, normality, homoscedasticity")
print("✅ Power analysis: sample size determination")
```

### Expected Output
- Regression diagnostics plots (residuals vs fitted, Q-Q plot, scale-location)
- Power curve showing required sample size for different effect sizes
- Shapiro-Wilk and Breusch-Pagan test results for model assumptions

---

## References

- Fisher (1925). *Statistical Methods for Research Workers*.
- Student (1908). *The Probable Error of a Mean*. Biometrika.
- Zar (2010). *Biostatistical Analysis*. 5th ed. Prentice Hall.
- Cohen (1988). *Statistical Power Analysis for the Behavioral Sciences*. 2nd ed.

**Maintainer**: BME Bootcamp Agent | **Week 10** | **BMED3603: Biostatistics**
