# Week 10 Deliverables — Biostatistics (BMED3603)

## Deliverable 1: Statistical Analysis Report

- **Type**: Data Analysis Report
- **Deadline**: End of Week 10 (Sunday, 23:59 HKT)
- **Submission**: PDF + .ipynb

### Dataset
You will receive a simulated clinical dataset containing:
- Patient demographics: age, sex, BMI
- Vital signs: systolic BP, diastolic BP, heart rate
- Lab results: cholesterol, fasting glucose
- Treatment assignment: Group A, B, or C

### Required Analysis

**Part A — Descriptive Statistics (15 marks)**
1. Compute descriptive statistics for all numeric variables (mean, SD, median, IQR, range, 95% CI)
2. Create histograms with normal overlay for key variables
3. Test normality using Shapiro-Wilk test
4. Create box plots comparing groups
5. Identify any outliers (values > 3 SD from mean)

**Part B — Hypothesis Testing (20 marks)**
1. Test whether mean SBP differs from population norm (120 mmHg) using one-sample t-test
2. Test whether Group A and Group B differ in SBP using independent two-sample t-test
3. Test whether treatment reduces SBP using paired t-test (before/after)
4. For each test: state H₀/H₁, compute t-statistic, p-value, 95% CI, effect size (Cohen's d)
5. Correctly interpret each result in clinical terms

**Part C — ANOVA (15 marks)**
1. Conduct one-way ANOVA comparing SBP across all three treatment groups
2. Compute η² and ω² (effect size)
3. Perform post-hoc Tukey HSD to identify which groups differ
4. Verify ANOVA assumptions: normality (per group), homogeneity of variance (Levene's test)
5. If assumptions are violated, perform Kruskal-Wallis test and report results

### Rubric

- [ ] Part A: Descriptive stats complete with CIs (15 pts)
- [ ] Part B: All three t-tests with correct interpretation (20 pts)
- [ ] Part C: ANOVA + post-hoc + assumptions (15 pts)

**Total: 50 marks**

---

## Deliverable 2: Clinical Trial Design

- **Type**: Study Design + Power Analysis
- **Deadline**: End of Week 10

### Task
Design a clinical trial to compare a new hypertension drug with a standard treatment.

### Required Components

1. **Trial Design (10 marks)**
   - Specify: parallel group, crossover, or cluster design
   - Define primary outcome (SBP change from baseline)
   - Define inclusion/exclusion criteria
   - Sample size calculation with justification

2. **Statistical Analysis Plan (10 marks)**
   - Primary analysis: t-test or ANOVA?
   - Secondary analyses: subgroup analyses, regression adjustment
   - How will you handle missing data?
   - Multiple comparison corrections

3. **Power Analysis (10 marks)**
   - Effect size: What is the minimum clinically meaningful difference?
   - Calculate required sample size for 80% power at α = 0.05
   - How does sample size change for 90% power?
   - Sensitivity analysis: how does required N change if SD is larger than expected?

4. **Report (10 marks)**
   - Write a 2-page statistical analysis plan
   - Include: hypothesis, sample size, power, analysis methods

### Rubric

- [ ] Trial design with justification (10 pts)
- [ ] Statistical analysis plan (10 pts)
- [ ] Power analysis with calculations (10 pts)
- [ ] Written report (10 pts)

**Total: 40 marks**

---

## Deliverable 3: Regression Modeling Challenge (Bonus)

Build the best predictive model for a clinical outcome using the provided dataset.

**Criteria**: R², adjusted R², residual diagnostics, clinical interpretability

**Bonus 5 points** for best model.

---

## Submission Instructions

- `[StudentID]_W10_[DeliverableName].[ext]`
- All code must be reproducible

**Maintainer**: BME Bootcamp Agent | **Week 10** | **BMED3603: Biostatistics**
