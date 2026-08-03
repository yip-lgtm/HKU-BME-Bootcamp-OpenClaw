# Week 10 Self-Quiz — Biostatistics (BMED3603)

## Multiple Choice (10 questions, 2 points each = 20 points)

**1. A clinical trial reports p = 0.03 for the primary outcome. The correct interpretation is:**

- A. There is a 3% chance that the null hypothesis is true
- B. There is a 3% chance of observing such extreme results if H₀ is true
- C. The treatment effect is 3% larger in the treatment group
- D. The result is clinically significant

**Answer**: B | **Explanation**: The p-value is the probability of observing data at least as extreme as what was observed, assuming the null hypothesis is TRUE. It does NOT tell us the probability that H₀ is true (that requires Bayesian methods). It also does NOT measure clinical significance — statistical and clinical significance are different concepts.

---

**2. A researcher wants to compare blood pressure across 4 treatment groups. Which test is appropriate?**

- A. Four separate t-tests
- B. One-way ANOVA
- C. Paired t-test
- D. Chi-square test

**Answer**: B | **Explanation**: When comparing means across more than 2 groups, one-way ANOVA is appropriate. Four separate t-tests would inflate Type I error rate (family-wise error rate ≈ 4 × 0.05 = 0.20). ANOVA controls Type I error by testing all groups simultaneously. If ANOVA is significant, post-hoc tests (like Tukey HSD) identify which specific pairs differ.

---

**3. Which of the following describes the relationship between Type I and Type II errors?**

- A. As α increases, β (Type II error) decreases
- B. As α increases, β increases
- C. α and β are independent
- D. Type I error is always more serious than Type II error

**Answer**: A | **Explanation**: There's a fundamental tradeoff: reducing Type I error (α) increases Type II error (β) and vice versa, for fixed sample size. This is because we're shifting the decision threshold — making it harder to reject H₀ (lower α) means we also miss more true effects (higher β). Increasing sample size can reduce both α and β simultaneously.

---

**4. In simple linear regression, R² = 0.64. The correct interpretation is:**

- A. 64% of the data points lie on the regression line
- B. 64% of the variance in Y is explained by X
- C. The correlation coefficient is 0.64
- D. The residual standard error is 0.64

**Answer**: B | **Explanation**: R² (coefficient of determination) = 1 − SS_res/SS_tot = proportion of variance in Y explained by the regression model (the independent variable X). R² = r² (Pearson correlation squared) only in simple linear regression. R² ≠ correlation coefficient directly (sign is lost in squaring).

---

**5. A 95% confidence interval for the mean difference between two treatments is [−2.5, 5.3] mmHg. Which conclusion is correct?**

- A. The treatments are equivalent (CI includes 0)
- B. Treatment B is superior (CI is mostly positive)
- C. There is a statistically significant difference at α = 0.05
- D. Treatment A is better (CI includes negative values)

**Answer**: A | **Explanation**: Since the 95% CI includes 0, we cannot reject H₀ at α = 0.05. The difference is not statistically significant. Note: "statistically significant" and "CI includes 0" are equivalent statements for two-sided tests. However, the CI provides more information — it tells us the range of plausible effect sizes.

---

**6. The Levene's test for equality of variances gives p = 0.008. Which test should be used?**

- A. Standard independent t-test (equal variances assumed)
- B. Welch's t-test (unequal variances)
- C. Paired t-test
- D. One-sample t-test

**Answer**: B | **Explanation**: Levene's test p = 0.008 < 0.05 means we reject the null hypothesis of equal variances. When variances are unequal, Welch's t-test (also called unequal-variance t-test) should be used. It adjusts the degrees of freedom and standard error calculation to account for the inequality.

---

**7. In a two-way ANOVA, the interaction term is significant (p < 0.001). This means:**

- A. The main effects of both factors are significant
- B. The effect of one factor depends on the level of the other factor
- C. The data are not normally distributed
- D. The sample sizes are unequal

**Answer**: B | **Explanation**: A significant interaction means the effect of Factor A on the outcome depends on which level of Factor B is present. The simple main effects must be interpreted with caution. For example, a drug might be effective at low dose but ineffective (or harmful) at high dose — the dose effect depends on the drug type.

---

**8. The power of a statistical test is defined as:**

- A. P(reject H₀ | H₀ is true)
- B. P(reject H₀ | H₁ is true)
- C. P(fail to reject H₀ | H₁ is true)
- D. P(H₀ is true)

**Answer**: B | **Explanation**: Statistical power = 1 − β = P(reject H₀ | H₁ is true). It is the probability of correctly rejecting a false null hypothesis (detecting a true effect). Option A is Type I error (α). Option C is Type II error (β). Power depends on effect size, sample size, α, and variability.

---

**9. A researcher finds that Pearson r = −0.72 between exercise frequency and BMI. Which is TRUE?**

- A. Higher BMI causes lower exercise frequency
- B. Exercise frequency explains 72% of variance in BMI
- C. There is a strong negative linear relationship
- D. The relationship is not statistically significant

**Answer**: C | **Explanation**: |r| = 0.72 is a strong correlation. The negative sign indicates an inverse relationship (more exercise → lower BMI). R² = 0.72² = 0.518, meaning ~52% of variance in BMI is explained by exercise — not 72%. Correlation does not imply causation (A is wrong). We can't determine significance without the sample size and p-value.

---

**10. In multiple regression with 3 predictors and n = 50, the adjusted R² = 0.55 while R² = 0.58. Why is adjusted R² lower?**

- A. The model is overfitted
- B. The predictors are highly correlated
- C. Adjusted R² always penalizes for the number of predictors
- D. The sample size is too small

**Answer**: C | **Explanation**: Adjusted R² = 1 − [(1−R²)(n−1)/(n−p−1)] where p = number of predictors. Adding more predictors always increases R² (never decreases), but adjusted R² penalizes for adding predictors that don't contribute substantially. The gap (0.58 − 0.55 = 0.03) indicates some of the R² increase is due to overfitting (adding predictors that don't meaningfully improve the model).

---

## Short Answer (5 questions, 4 points each = 20 points)

**1. A clinical trial compares a new drug (n = 30) vs placebo (n = 30). The result: t = 2.10, p = 0.04. The mean SBP reduction: Drug = 8.2 mmHg, Placebo = 3.1 mmHg. The researchers conclude the drug is effective. Is this conclusion justified? What additional information would you want?**

**Answer**: The result is statistically significant at α = 0.05 (p = 0.04 < 0.05), so the null hypothesis of no difference is rejected. However, several factors should be considered:

**Justification check**:
- Clinical significance: Mean difference = 8.2 − 3.1 = 5.1 mmHg. Is a 5 mmHg reduction clinically meaningful? This depends on clinical context (for hypertension, even 5 mmHg can be meaningful for population-level outcomes).
- Effect size: Cohen's d = (5.1) / pooled_SD. If pooled SD ≈ 10, then d ≈ 0.51 (medium effect).
- Power: Was the study adequately powered? What was the pre-study power calculation?

**Additional information needed**:
1. Confidence interval for the mean difference
2. Effect size (Cohen's d) and its interpretation
3. Clinical significance threshold (is 5.1 mmHg clinically important?)
4. Adverse events/side effects comparison
5. Study duration and dropout rate
6. Whether the analysis was pre-specified or post-hoc
7. Any conflicts of interest

---

**2. Explain why multiple t-tests comparing more than 2 groups inflate Type I error, and how ANOVA addresses this problem.**

**Answer**: When testing k groups with all pairwise comparisons using t-tests:

**Problem**: Each individual test has Type I error rate α. For k independent tests, the family-wise error rate (FWER) = 1 − (1−α)^k.

Example: k = 4 groups → 6 pairwise comparisons
FWER = 1 − (1−0.05)⁶ = 1 − 0.74 = 0.26 (26%!)

This means there's a 26% chance of finding at least one false positive — far exceeding our intended 5% rate.

**ANOVA solution**: ANOVA tests all groups simultaneously with a single test:
H₀: μ₁ = μ₂ = ... = μ_k
H₁: at least one μᵢ differs

By testing all groups at once, the overall Type I error is controlled at α. If H₀ is rejected, we then use post-hoc tests (Tukey HSD, Bonferroni, etc.) with adjusted α levels to identify specific differences while maintaining the family-wise error rate.

---

**3. A regression model for predicting blood pressure has residuals that are not normally distributed (Shapiro-Wilk p < 0.01). What are the implications? What remedies are available?**

**Answer**: Non-normal residuals in linear regression have several implications:

**Implications**:
1. The t-tests and F-test may be invalid (their p-values rely on normality assumption)
2. Confidence intervals may be unreliable
3. Predictions may be biased, especially in the tails

**Remedies**:
1. **Transform the outcome variable**: Log, square root, or Box-Cox transformation of Y
2. **Use robust standard errors**: HC0, HC1, HC2, HC3 — these are robust to non-normality
3. **Use non-parametric methods**: Bootstrap confidence intervals (don't assume normality)
4. **Consider a different model**: If residuals are heavy-tailed, use quantile regression or robust regression (e.g., Huber regression)
5. **Check for outliers**: Extreme outliers can cause non-normality. Investigate and consider robust methods.
6. **Increase sample size**: By the Central Limit Theorem, with large n (n > 30-50), t-test F-test are robust to moderate non-normality

---

**4. A study reports: "The new diagnostic test has sensitivity = 0.92 and specificity = 0.88." Calculate the positive and negative predictive values if the disease prevalence is 5%. What do these values tell us?**

**Answer**: PPV and NPV require Bayes' theorem:

PPV = (Sensitivity × Prevalence) / [(Sensitivity × Prevalence) + (1−Specificity)(1−Prevalence)]
PPV = (0.92 × 0.05) / [(0.92 × 0.05) + (0.12 × 0.95)]
PPV = 0.046 / [0.046 + 0.114] = 0.046 / 0.160 = **0.287 = 28.7%**

NPV = (Specificity × (1−Prevalence)) / [(Specificity × (1−Prevalence)) + ((1−Sensitivity) × Prevalence)]
NPV = (0.88 × 0.95) / [(0.88 × 0.95) + (0.08 × 0.05)]
NPV = 0.836 / [0.836 + 0.004] = 0.836 / 0.840 = **0.995 = 99.5%**

**Interpretation**:
- PPV = 28.7%: Of all positive test results, only 28.7% actually have the disease. This is LOW because the disease is rare (5% prevalence).
- NPV = 99.5%: Of all negative test results, 99.5% truly don't have the disease. This is HIGH.

**Clinical implication**: In low-prevalence populations, even sensitive and specific tests have low PPV. A positive result requires confirmatory testing. This is why screening tests need very high sensitivity even at the cost of specificity.

---

**5. Calculate the sample size needed for a two-group comparison with the following parameters: Δ = 10 mmHg (minimum clinically meaningful difference), σ = 20 mmHg, α = 0.05 (two-tailed), Power = 0.80.**

**Answer**: Using the formula for two-sample t-test:

n = 2 × [(z_α/2 + z_β) × σ/Δ]²

Where:
- z_α/2 for two-tailed test at α=0.05: z_0.025 = 1.96
- z_β for power=0.80: z_0.80 = 0.84
- σ = 20 mmHg
- Δ = 10 mmHg

n = 2 × [(1.96 + 0.84) × 20/10]²
n = 2 × [(2.80) × 2]²
n = 2 × [5.6]²
n = 2 × 31.36
n = 62.72 ≈ **63 per group**

**Total N = 2 × 63 = 126 participants**

Interpretation: You need 63 participants per group (126 total) to have an 80% chance of detecting a true 10 mmHg difference as statistically significant at the 0.05 level.

If power increases to 0.90 (z_0.90 = 1.28):
n = 2 × [(1.96 + 1.28) × 2]² = 2 × [3.24 × 2]² = 2 × 42.1 ≈ **85 per group** (170 total)

---

## True or False (5 questions, 1 point each = 5 points)

**1. In multiple regression, adding more predictors always increases R².**
**Answer**: True. R² = 1 − SS_res/SS_tot. Adding predictors can only reduce SS_res (or keep it the same if the predictor adds no information). Therefore R² never decreases. However, adjusted R² can decrease if the added predictor doesn't justify its penalty.

---

**2. If p = 0.06, we should conclude the treatment is ineffective.**
**Answer**: False. p = 0.06 is not statistically significant at α = 0.05, but this does NOT mean the treatment is ineffective. It means we don't have sufficient evidence to reject H₀. The treatment could be effective — we just didn't detect it with statistical significance. The result should be reported as "not statistically significant," and the effect size should be reported to assess clinical relevance.

---

**3. Bonferroni correction is more conservative than Tukey HSD for post-hoc pairwise comparisons in ANOVA.**
**Answer**: True (generally). Bonferroni divides α by the number of comparisons: α_adj = α/k. Tukey HSD uses the studentized range distribution, which is less conservative for all pairwise comparisons. However, Tukey HSD is only valid for all pairwise comparisons — if you want to compare specific subsets, Bonferroni or Dunnett's test may be more appropriate.

---

**4. In regression, the residual standard error (RSE) represents the average prediction error.**
**Answer**: True. RSE = √(SS_res/(n−p−1)). It is the standard deviation of the residuals (prediction errors). Approximately 68% of residuals fall within ±1 RSE, and ~95% fall within ±2 RSE. RSE is in the same units as the outcome variable Y.

---

**5. Homoscedasticity means that the variance of residuals is constant across all levels of the predictor.**
**Answer**: True. Homoscedasticity (constant variance) is one of the key assumptions of linear regression. Heteroscedasticity (non-constant variance) causes standard errors to be biased, leading to unreliable hypothesis tests and confidence intervals. It can be detected with the Breusch-Pagan test or by examining residual plots ( residuals vs fitted values).

---

## Score Guide

| Score | Grade | Interpretation |
|-------|-------|---------------|
| 40–45 | A | Excellent! Ready for Week 11 (Physiological Modeling) |
| 32–39 | B | Good. Review missed questions. |
| 24–31 | C | Review Week 10 concepts before proceeding |
| < 24 | D | Re-do Week 10 code labs |

**Maintainer**: BME Bootcamp Agent | **Week 10** | **BMED3603: Biostatistics**
