# Week 10 Readings — Biostatistics: Hypothesis Testing & ANOVA (BMED3603)

## Required Readings

### Primary Textbooks
- **Bland, M.** (2015) *An Introduction to Medical Statistics*, 4th ed. Oxford University Press — Chapters 7-13
- **Motulsky, H.** (2018) *Intuitive Biostatistics: A Nonmathematical Guide to Statistical Thinking*, 4th ed. Oxford University Press
- **Pagano, M. & Gauvreau, K.** (2018) *Principles of Biostatistics*, 2nd ed. CRC Press — Chapters 9-14

### HKU Course Materials
- **BMED3603** — Biostatistics for Biomedical Engineers (HKU SBME)
- Topics: Hypothesis testing, t-tests, ANOVA, non-parametric methods, power analysis

### Key Papers / Historical References
1. **Student (Gosset, W.S.)** (1908) "The Probable Error of a Mean" — *Biometrika*, 6(1), 1-25 — The t-distribution
2. **Fisher, R.A.** (1925) *Statistical Methods for Research Workers* — Foundation of modern statistics
3. **Neyman, J. & Pearson, E.S.** (1933) "On the Problem of the Most Efficient Tests of Statistical Hypotheses" — *Philosophical Transactions of the Royal Society A*, 231, 289-337
4. **Cohen, J.** (1992) "A Power Primer" — *Psychological Bulletin*, 112(1), 155-159 — Effect size and power analysis

### Online Resources
- Khan Academy: **Statistics & Probability** — https://www.khanacademy.org/math/statistics-probability
- StatQuest (Josh Starmer): **Hypothesis Testing, t-tests, ANOVA** — YouTube video series
- G*Power: **Power Analysis Software** — Free download for sample size calculation
- GraphPad Prism: **Statistical Guides** — Practical biomedical statistics
- OpenStax: **Introductory Statistics** — Free online textbook

## Pre-Week Objectives (BMED3603 Week 10)
By the end of Week 10, students should be able to:
1. Formulate null (H₀) and alternative (H₁) hypotheses correctly
2. Distinguish Type I error (α) from Type II error (β)
3. Perform one-sample, two-sample, and paired t-tests
4. Conduct one-way and two-way ANOVA with multiple comparisons
5. Apply non-parametric tests: Mann-Whitney U, Wilcoxon signed-rank, Kruskal-Wallis
6. Calculate statistical power and perform power analysis
7. Interpret effect sizes using Cohen's conventions
8. Report results following APA format and clinical trial standards

## Mathematical Foundations

### Core Equations (Week 10)

#### Hypothesis Testing
1. **Type I Error**: α = P(reject H₀ | H₀ true), typically set at 0.05
2. **Type II Error**: β = P(fail to reject H₀ | H₁ true)
3. **Power**: 1 - β = P(reject H₀ | H₁ true), typically ≥ 0.80

#### t-Tests
4. **One-sample t-test**: t = (x̄ - μ₀) / (s/√n), df = n-1
5. **Two-sample t-test (independent)**: t = (x̄₁ - x̄₂) / √(s₁²/n₁ + s₂²/n₂)
6. **Paired t-test**: t = d̄ / (s_d/√n), df = n-1
7. **Pooled variance (equal variances)**: s²_p = [(n₁-1)s₁² + (n₂-1)s₂²] / (n₁+n₂-2)

#### ANOVA
8. **F-statistic**: F = MS_between / MS_within = (SS_between/df_between) / (SS_within/df_within)
9. **One-way ANOVA df**: df_between = k-1, df_within = N-k
10. **Two-way ANOVA**: Main effects + interaction, each with its own F-test

#### Non-Parametric Tests
11. **Mann-Whitney U**: U = n₁n₂ + n₁(n₁+1)/2 - R₁
12. **Wilcoxon signed-rank**: T = min(T⁺, T⁻)
13. **Kruskal-Wallis H**: H = [12/(N(N+1))] ΣRᵢ²/nᵢ - 3(N+1)

#### Effect Size Measures
14. **Cohen's d**: d = (x̄₁ - x̄₂) / s_pooled
15. **Eta-squared (η²)**: η² = SS_between / SS_total
16. **Pearson's r**: r = √(t²/(t² + df))

### Statistical Significance Thresholds

| Test | Threshold (α) | Convention |
|------|---------------|------------|
| Standard | 0.05 | Significant (*) |
| Strict | 0.01 | Highly Significant (**) |
| Very strict | 0.001 | Extremely Significant (***) |
| Clinical trials | 0.025-0.05 | Per FDA/EMA guidelines |
| Multiple comparisons | Bonferroni: α/m | Family-wise error rate |

### Cohen's Effect Size Conventions

| Effect Size | Small | Medium | Large |
|-------------|-------|--------|-------|
| Cohen's d | 0.2 | 0.5 | 0.8 |
| r (correlation) | 0.1 | 0.3 | 0.5 |
| η² (ANOVA) | 0.01 | 0.06 | 0.14 |
| f (Cohen's f) | 0.1 | 0.25 | 0.4 |

### BME Clinical Trial Statistics
- **Phase I trials**: n = 20-100, safety endpoints
- **Phase II trials**: n = 100-300, efficacy endpoints
- **Phase III trials**: n = 1000-5000, randomized controlled trials
- **Adaptive designs**: Sample size re-estimation, n = function of interim results

## Assessment Alignment
- BMED3603 Problem Set 1 (Week 11): Hypothesis testing problems
- BMED3603 Quiz 1 (Week 11): t-tests, ANOVA interpretation
- HKU SBME Lab 1: Statistical analysis of medical device performance data

## Key Concepts for Deep Dive
1. **p-value misinterpretation** — p ≠ probability that H₀ is true
2. **Multiple comparison correction** — Bonferroni, Tukey HSD, FDR
3. **Statistical vs Clinical Significance** — Large n can detect tiny effects
4. **Assumptions of Parametric Tests** — Normality, independence, equal variance
5. **Sample Size Justification** — FDA requirements for medical device trials

## Supplementary Reading
- **Altman, D.G.** (1991) *Practical Statistics for Medical Research* — Chapman & Hall
- **Wassertheil-Smoller, S. & Kim, M.** (2015) *Biostatistics and Epidemiology* — Springer
- **Hulley, S.B. et al.** (2013) *Designing Clinical Research*, 4th ed. Lippincott Williams & Wilkins

## Reflection Questions
1. Why is Type I error rate (α) set at 0.05? What are the consequences of changing it?
2. When should you use non-parametric tests instead of parametric tests?
3. What is the difference between one-tailed and two-tailed tests? When is each appropriate?
4. How does sample size affect statistical power? What is the minimum sample for adequate power?
5. Why do clinical trials often use α = 0.025 instead of 0.05 for primary endpoints?
