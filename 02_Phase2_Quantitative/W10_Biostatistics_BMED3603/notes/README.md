# Week 10 Notes — Biostatistics: Hypothesis Testing & ANOVA (BMED3603)

## 問題 1：5 個核心心智模型

### 1. 假設檢定：統計推斷的框架 — Hypothesis Testing: Framework for Statistical Inference

**核心概念**：假設檢定是一種系統化的方法，用於根據樣本數據判斷關於總體的假設是否成立。它將不確定性量化為概率。

**數學表示**：
```latex
基本框架：
H₀: μ = μ₀ (零假設 — 通常表示無效應或無差異)
H₁: μ ≠ μ₀ (對立假設 — 表示有效應或有差異)

或者單尾：
H₀: μ ≤ μ₀
H₁: μ > μ₀

決策規則：
- 如果 p-value < α，拒絕 H₀
- 如果 p-value ≥ α，不能拒絕 H₀
```

**兩類錯誤**：
```latex
Type I Error (α): 當H₀為真時拒絕H₀
                  P(reject H₀ | H₀ true) = α
                  
Type II Error (β): 當H₁為真時接受H₀
                  P(fail to reject H₀ | H₁ true) = β

Power (檢定力): 1 - β = P(reject H₀ | H₁ true)
              當H₁為真時正確拒絕H₀的概率
```

**BME臨床試驗中的應用**：
| 試驗階段 | 典型α | β | 樣本量 | 目的 |
|---------|-------|---|--------|------|
| Phase I | 0.05-0.10 | 0.20 | 20-100 | 安全性 |
| Phase II | 0.05 | 0.20 | 100-300 | 有效性 |
| Phase III | 0.025-0.05 | 0.10-0.20 | 1000-5000 | 確認 |

**學者貢獻**：
- Fisher (1925) — 顯著性檢定框架
- Neyman & Pearson (1933) — 假設檢定理論
- Student (1908) — t分佈與小樣本理論

**深度問題**：為什麼α=0.05成為標準？p<0.05意味著什麼？它不等於「H₀為真的概率」。

---

### 2. t檢定：三種變體 — t-Test: Three Flavors

**核心概念**：t檢定用於比較樣本均值與總體均值，或比較兩個樣本均值。它是小樣本（n<30）且總體標準差未知時的首選方法。

**數學表示**：
```latex
單一樣本t檢定：
t = (x̄ - μ₀) / (s/√n)
df = n - 1

獨立兩樣本t檢定（假設等變異數）：
t = (x̄₁ - x̄₂) / √(s_p²(1/n₁ + 1/n₂))
其中 s_p² = [(n₁-1)s₁² + (n₂-1)s₂²] / (n₁ + n₂ - 2)
df = n₁ + n₂ - 2

Welch's t檢定（不等變異數）：
t = (x̄₁ - x̄₂) / √(s₁²/n₁ + s₂²/n₂)
df ≈ (s₁²/n₁ + s₂²/n₂)² / [(s₁²/n₁)²/(n₁-1) + (s₂²/n₂)²/(n₂-1)]

配對樣本t檢定：
t = d̄ / (s_d/√n)
df = n - 1
其中 d̄ = mean of differences, s_d = SD of differences
```

**選擇指南**：
```python
# 什麼時候用什麼t檢定？
def choose_ttest(data_type):
    if data_type == "one_group_vs_value":
        return "One-sample t-test"
    elif data_type == "two_independent_groups":
        if equal_variance_assumption_holds:
            return "Independent two-sample t-test"
        else:
            return "Welch's t-test"
    elif data_type == "paired_measurements":
        return "Paired t-test"
```

**BME應用**：
```python
import numpy as np
from scipy import stats

# BME Example: 比較兩種血壓藥物的效果
# 藥物A組
drug_A = np.array([125, 130, 128, 135, 132, 127, 131, 129, 133, 126])
# 藥物B組
drug_B = np.array([118, 122, 115, 120, 117, 119, 121, 116, 123, 118])

# 獨立兩樣本t檢定
t_stat, p_value = stats.ttest_ind(drug_A, drug_B)
print(f"t-statistic: {t_stat:.4f}")
print(f"p-value: {p_value:.6f}")

# 描述性統計
print(f"\nDrug A: mean={np.mean(drug_A):.1f}mmHg, SD={np.std(drug_A, ddof=1):.1f}")
print(f"Drug B: mean={np.mean(drug_B):.1f}mmHg, SD={np.std(drug_B, ddof=1):.1f}")
print(f"Difference: {np.mean(drug_A) - np.mean(drug_B):.1f}mmHg")

# 臨床解釋
alpha = 0.05
if p_value < alpha:
    print(f"\nConclusion: Reject H₀ (p={p_value:.4f} < {alpha})")
    print("Drug B significantly lowers blood pressure compared to Drug A")
else:
    print(f"\nConclusion: Fail to reject H₀ (p={p_value:.4f} ≥ {alpha})")
    print("No significant difference between drugs at α=0.05")
```

**學者貢獻**：
- Student (William Gosset) (1908) — 在吉尼斯啤酒廠工作時發明t分佈
- Gosset使用筆名"Student"發表

**深度問題**：Welch's t檢定 vs 標準t檢定什麼時候應該用？等變異數假設如何檢驗？

---

### 3. ANOVA：多組比較 — ANOVA: Comparing Multiple Groups

**核心概念**：ANOVA（變異數分析）用於比較三個或更多組的均值。它控制了多重比較的Type I錯誤膨脹問題。

**數學表示**：
```latex
單因子ANOVA：
H₀: μ₁ = μ₂ = ... = μ_k
H₁: 至少有一對均值不相等

F統計量：
F = MS_between / MS_within

其中：
SS_total = ΣΣ(x_ij - x̄)²
SS_between = Σn_i(x̄_i - x̄)²
SS_within = ΣΣ(x_ij - x̄_i)²

MS = SS / df

df_between = k - 1
df_within = N - k
df_total = N - 1
```

**變異數分解表**：
```latex
| Source      | SS          | df | MS         | F        |
|-------------|-------------|-----|------------|----------|
| Between     | SS_b        | k-1 | SS_b/df_b | MS_b/MS_w|
| Within      | SS_w        | N-k | SS_w/df_w |          |
| Total       | SS_t        | N-1 |            |          |
```

**雙因子ANOVA**：
```latex
雙因子模型：
Y_ijk = μ + α_i + β_j + (αβ)_ij + ε_ijk

主效應：
- α效應：行因素A的效應
- β效應：列因素B的效應

交互效應：
- (αβ)_ij：A和B的交互作用

F檢定：
F_A = MS_A / MS_within
F_B = MS_B / MS_within
F_AB = MS_AB / MS_within
```

**BME應用**：
```python
import numpy as np
import pandas as pd
from scipy import stats
import statsmodels.api as sm
from statsmodels.formula.api import ols

# BME Example: 三種鎮痛劑的效果比較
# 假設記錄了疼痛評分（0-10）

pain_scores = pd.DataFrame({
    'drug': ['A']*10 + ['B']*10 + ['C']*10,
    'score': [6.2, 5.8, 7.1, 6.5, 5.9, 6.3, 6.8, 5.7, 6.1, 6.4,
              4.2, 3.9, 4.5, 4.1, 3.8, 4.3, 4.0, 3.7, 4.4, 4.1,
              5.1, 5.3, 4.9, 5.5, 5.2, 4.8, 5.4, 5.0, 5.6, 5.2]
})

# 單因子ANOVA
model = ols('score ~ C(drug)', data=pain_scores).fit()
anova_table = sm.stats.anova_lm(model, typ=2)

print("ANOVA Table:")
print(anova_table)

# 事後檢驗（Tukey HSD）
from statsmodels.stats.multicomp import pairwise_tukeyhsd
posthoc = pairwise_tukeyhsd(pain_scores['score'], pain_scores['drug'])
print("\nTukey HSD Post-hoc Test:")
print(posthoc)
```

**學者貢獻**：
- Fisher (1922) — ANOVA的發明
- Fisher (1925) — 《研究工作者統計方法》

**深度問題**：為什麼不用多次t檢定？用三次t檢定比較四組會有什麼問題？

---

### 4. 非參數檢定：當假設不成立時 — Non-parametric Tests: When Assumptions Fail

**核心概念**：非參數檢定不依賴總體分佈的假設。當數據不服從正態分佈或樣本量太小時，非參數方法是更好的選擇。

**常用非參數檢定**：

| 情境 | 參數檢定 | 非參數檢定 |
|------|---------|-----------|
| 單組 vs 已知值 | 單樣本t | Wilcoxon signed-rank |
| 兩獨立組 | 獨立t | Mann-Whitney U |
| 兩配對組 | 配對t | Wilcoxon signed-rank |
| 多於兩組 | 單因子ANOVA | Kruskal-Wallis |
| 重複測量ANOVA | RM-ANOVA | Friedman |

**Mann-Whitney U檢定**：
```latex
U = n₁n₂ + n₁(n₁+1)/2 - R₁
其中 R₁ = 第一組的秩和

或者：
U' = n₁n₂ + n₂(n₂+1)/2 - R₂

U = min(U, U')

大樣本近似（n₁, n₂ ≥ 20）：
z = (U - μ_U) / σ_U
其中：
μ_U = n₁n₂/2
σ_U = √(n₁n₂(n₁+n₂+1)/12)
```

**Wilcoxon Signed-Rank檢定**：
```latex
1. 計算差值 d_i = x_i - y_i
2. 排除 d_i = 0 的對
3. 取 |d_i| 的秩
4. 計算正秩和 T⁺ 和負秩和 T⁻
5. T = min(T⁺, T⁻)

大樣本近似：
z = (T - μ_T) / σ_T
其中：
μ_T = n(n+1)/4
σ_T = √(n(n+1)(2n+1)/24)
```

**Kruskal-Wallis H檢定**：
```latex
H = [12/(N(N+1))] · ΣR_i²/n_i - 3(N+1)

其中：
R_i = 第i組的秩和
n_i = 第i組的樣本量
N = 總樣本量

df = k - 1

當存在tie時：
H_adj = H / [1 - Σ(t_j³ - t_j)/(N³ - N)]
```

**Python實現**：
```python
import numpy as np
from scipy import stats

# 非正態數據示例
# 假設這是偏態的分佈數據
group1 = np.random.exponential(scale=2, size=30)
group2 = np.random.exponential(scale=3, size=30)
group3 = np.random.exponential(scale=2.5, size=30)

# 檢驗正態性
for i, g in enumerate([group1, group2, group3], 1):
    stat, p = stats.shapiro(g)
    print(f"Group {i}: Shapiro-Wilk p={p:.4f} {'(non-normal)' if p<0.05 else '(normal)'}")

# 使用Kruskal-Wallis（非參數ANOVA）
H, p_value = stats.kruskal(group1, group2, group3)
print(f"\nKruskal-Wallis: H={H:.4f}, p={p_value:.4f}")

# 如果顯著，進行事後檢驗
from scikit_posthocs import posthoc_dunn
posthoc = posthoc_dunn([group1, group2, group3], p_adjust='bonferroni')
print("\nDunn's Post-hoc Test:")
print(posthoc)
```

**學者貢獻**：
- Mann & Whitney (1947) — Mann-Whitney U檢定
- Wilcoxon (1945) — Wilcoxon signed-rank檢定
- Kruskal & Wallis (1952) — Kruskal-Wallis檢定

**深度問題**：非參數檢定的效力（power）通常比參數檢定低，什麼時候這個代價值得？

---

### 5. 統計檢定力分析：樣本量計算 — Power Analysis: Sample Size Calculation

**核心概念**：檢定力分析確定檢測特定效應大小所需的樣本量。這是臨床試驗設計的關鍵步驟。

**數學表示**：
```latex
檢定力 = 1 - β = P(reject H₀ | H₁ is true)

效應大小 (Effect Size):
- Cohen's d = (μ₁ - μ₂) / σ_pooled
- r = √(t² / (t² + df))
- η² = SS_between / SS_total

Cohen's d 效應大小標準：
- 小效應：d = 0.2
- 中效應：d = 0.5
- 大效應：d = 0.8

樣本量公式（兩組t檢定）：
n = 2 · [(z_α/2 + z_β) / d]²
```

**樣本量計算**：
```python
from statsmodels.stats.power import TTestIndPower, TTestPower

# 兩組獨立t檢定的檢定力分析
analysis = TTestIndPower()

# 計算所需樣本量
effect_sizes = [0.2, 0.5, 0.8]  # 小、中、大效應
alpha = 0.05
power = 0.80

print("Sample size per group for 80% power:")
print("-" * 40)
for d in effect_sizes:
    n = analysis.solve_power(effect_size=d, alpha=alpha, power=power, ratio=1)
    print(f"d={d} (Cohen): n = {n:.1f} per group")

# 檢定力曲線
import matplotlib.pyplot as plt

sample_sizes = np.arange(10, 200, 5)
powers = [analysis.power(effect_size=0.5, nobs=n, alpha=0.05, ratio=1) 
          for n in sample_sizes]

plt.figure(figsize=(10, 6))
plt.plot(sample_sizes, powers, 'b-', linewidth=2)
plt.axhline(y=0.80, color='r', linestyle='--', label='80% power')
plt.axhline(y=0.90, color='g', linestyle='--', label='90% power')
plt.xlabel('Sample Size per Group')
plt.ylabel('Statistical Power')
plt.title('Power vs Sample Size (d=0.5, α=0.05)')
plt.legend()
plt.grid(True)
plt.show()
```

**BME臨床試驗樣本量**：
```python
# 典型BME臨床試驗樣本量估算
# 假設：兩組比較，終點是血壓變化

def calculate_sample_size(delta, sigma, alpha=0.05, power=0.80):
    """
    delta: 期望差異
    sigma: 標準差
    """
    from scipy import stats
    
    # 雙側檢定
    z_alpha = stats.norm.ppf(1 - alpha/2)
    z_beta = stats.norm.ppf(power)
    
    d = delta / sigma  # Cohen's d
    n = 2 * ((z_alpha + z_beta) / d)**2
    
    return n, d

# 示例
print("Clinical Trial Sample Size Calculation:")
print("-" * 50)
scenarios = [
    ("BP Drug Trial", 5, 10),    # 5mmHg差異, 10mmHg SD
    ("ECG Biomarker", 0.1, 0.2), # 0.1mV差異, 0.2mV SD
    ("EEG Power", 2, 5),         # 2μV²差異, 5μV² SD
]

for name, delta, sigma in scenarios:
    n, d = calculate_sample_size(delta, sigma)
    print(f"{name}: n = {n:.0f} per group (d = {d:.2f})")
```

**學者貢獻**：
- Cohen (1988) — 《統計功效分析》
- Cohen (1992) — 功效分析手冊

**深度問題**：為什麼FDA要求在臨床試驗開始前就計算樣本量？這個數字如何影響試驗的成功概率？

---

## 問題 2：3 個根本分歧

### 分歧 1：p值 vs 信賴區間 — p-value vs Confidence Interval

**p值方法**：
- 優點：單一數字，决策清晰
- 缺點：只告訴顯著性，不告訴效應大小或實際意義

**信賴區間方法**：
- 優點：同時顯示效應大小和精確度
- 缺點：需要更多解釋

**現代觀點**：兩者都報告，區間提供更多資訊

```python
import numpy as np
from scipy import stats

# 同一數據的兩種分析
data = np.array([22, 24, 25, 23, 26, 24, 25, 23, 24, 25])
n = len(data)
mean = np.mean(data)
se = stats.sem(data)
alpha = 0.05

# p值方法
t_stat, p_value = stats.ttest_1samp(data, 20)  # vs H₀: μ=20
print(f"t = {t_stat:.3f}, p = {p_value:.4f}")

# 信賴區間方法
ci = stats.t.interval(1-alpha, df=n-1, loc=mean, scale=se)
print(f"95% CI: [{ci[0]:.2f}, {ci[1]:.2f}]")

print(f"\nInterpretation:")
print(f"- Mean: {mean:.2f}")
print(f"- CI does not include 20, consistent with p < 0.05")
```

---

### 分歧 2：單尾 vs 雙尾檢定 — One-tailed vs Two-tailed

**單尾支持者**：
- 當有明確的方向假設時
- 更敏感（檢定力提高）
- 例：只關心新藥是否比安慰劑好

**雙尾支持者**：
- 當沒有明確方向時
- 更加保守
- 例：發現意外不良反應

**準則**：除非有強烈、先驗的方向性假設，否則用雙尾檢定

---

### 分歧 3：顯著性 vs 臨床意義 — Statistical vs Clinical Significance

**問題核心**：即使p<0.05，效應可能太小而無臨床意義

```python
# 例子：超大的樣本量
large_n = 10000
diff = 0.5  # 0.5mmHg的血壓差異
sigma = 10

# 這會顯著（因為n大），但臨床無意義
t_stat = diff / (sigma / np.sqrt(large_n))
p_value = 2 * (1 - stats.t.cdf(abs(t_stat), df=large_n-2))
print(f"Large sample: t={t_stat:.2f}, p={p_value:.6f}")
print(f"Statistically significant but: Δ={diff}mmHg may not be clinically meaningful")

# 臨床意義需要同時考慮效應大小和實際應用價值
effect_size = diff / sigma
print(f"Cohen's d = {effect_size:.3f} (small effect)")
```

---

## 問題 3：10 個深度問題

### Q1: 為什麼p值不是「H₀為真的概率」？

**答案**：
```latex
p-value的定義：
p = P(觀察到比實際結果更極端的數據 | H₀為真)

它是一個條件概率：P(data | H₀)
而不是 P(H₀ | data)

後者需要貝葉斯方法：
P(H₀ | data) = P(data | H₀) · P(H₀) / P(data)

p值容易被誤解：
- p=0.03 ≠ 「有3%的概率H₀為真」
- p=0.03 ≠ 「有97%的概率效應是真實的」
```

---

### Q2: 多重比較問題如何解決？

**答案**：
```python
# Bonferroni校正
n_comparisons = 10
alpha_adjusted = 0.05 / n_comparisons
print(f"Bonferroni: α_adj = {alpha_adjusted:.5f}")

# 問題：當比較很多時，變得過於保守

# 其他方法：
# - Tukey HSD：用於ANOVA後的所有成對比較
# - Benjamini-Hochberg (FDR)：控制虛假發現率
# - Scheffé方法：最保守的校正

from statsmodels.stats.multitest import multipletests
p_values = np.array([0.001, 0.01, 0.03, 0.04, 0.10])

# FDR校正
reject, pvals_corrected, _, _ = multipletests(p_values, alpha=0.05, method='fdr_bh')
print(f"\nBenjamini-Hochberg FDR correction:")
for i, (p, r, pc) in enumerate(zip(p_values, reject, pvals_corrected)):
    print(f"  Test {i+1}: p={p:.3f}, reject={r}, p_adjusted={pc:.4f}")
```

---

### Q3: 配對t檢定 vs 獨立t檢定什麼時候用？

**答案**：
```python
# 配對設計：每個受試者測量兩次（前/後）或配對
# 例如：同一患者的左右手臂血壓

# 獨立設計：不同受試者分組
# 例如：比較兩組不同患者的結果

# 何時用配對：
# 1. 同一受試者的前/後測量
# 2. 配對的受試者（雙胞胎、年齡性別匹配）
# 3. 同一標本的重複測量

# 配對的優勢：
# - 減少個體差異的影響
# - 更高的檢定力（控制混淆變量）
# - 通常需要更少的受試者
```

---

### Q4: ANOVA的前提假設是什麼？如何檢驗？

**答案**：
```python
# ANOVA的前提假設
# 1. 獨立性：觀測值之間相互獨立
# 2. 常態性：每組的總體服從正態分佈
# 3. 方差同質性：各組的方差相等

# 檢驗方法
from scipy import stats
import numpy as np

# 示例數據
groups = [
    np.array([8.5, 9.2, 8.8, 9.1, 8.7]),
    np.array([7.8, 8.1, 7.5, 8.0, 7.9]),
    np.array([9.5, 9.8, 9.2, 9.6, 9.4])
]

# 1. 獨立性：通常由試驗設計保證

# 2. 常態性檢驗（Shapiro-Wilk）
print("Normality Tests (Shapiro-Wilk):")
for i, g in enumerate(groups):
    stat, p = stats.shapiro(g)
    print(f"  Group {i+1}: W={stat:.4f}, p={p:.4f}")

# 3. 方差同質性檢驗（Levene's test）
stat, p = stats.levene(*groups)
print(f"\nLevene's Test: W={stat:.4f}, p={p:.4f}")

# 如果假設不滿足：
# - 非正態：Kruskal-Wallis（非參數替代）
# - 方差異質：Welch's ANOVA或Games-Howell事後檢驗
```

---

### Q5: 效應大小的意義是什麼？

**答案**：
```python
import numpy as np

# Cohen's d 的意義
# d = 0.2：小效應（可能被忽略）
# d = 0.5：中效應（實際可見的差異）
# d = 0.8：大效應（明顯的差異）

# r (相關係數) 的意義
# r = 0.1：小相關（解釋1%的變異）
# r = 0.3：中相關（解釋9%的變異）
# r = 0.5：大相關（解釋25%的變異）

# η² (ANOVA效應大小)
# η² = 0.01：小效應
# η² = 0.06：中效應
# η² = 0.14：大效應

# 為什麼重要：
# - p值只告訴你顯不顯著
# - 效應大小告訴你實際影響有多大
# - 大樣本可能讓小效應也顯著

# 計算範例
n1, n2 = 50, 50
mean1, mean2 = 10.5, 10.0
var1, var2 = 4.0, 4.0

pooled_std = np.sqrt(((n1-1)*var1 + (n2-1)*var2) / (n1+n2-2))
d = (mean1 - mean2) / pooled_std
print(f"Cohen's d = {d:.2f}")

# 報告時同時報告
print("\nIn a paper, report both:")
print(f"t(98) = 2.50, p = .014, d = 0.50")
```

---

### Q6: 什麼是Type I錯誤率膨脹？如何避免？

**答案**：
```python
# Type I錯誤率膨脹
# 當進行多次檢定時，累積的虛假阳性概率增加

# 問題：
# 如果進行20次獨立的α=0.05檢定：
# P(at least one false positive) = 1 - (1-0.05)^20 ≈ 0.64

# 避免方法：
# 1. Bonferroni：α_adj = α / m
# 2. Tukey HSD：用於ANOVA成對比較
# 3. Scheffé：最保守，適用於所有可能比較
# 4. Benjamini-Hochberg (FDR)：控制虛假發現率

# 選擇原則：
# - 探索性研究：可用FDR
# - 驗證性研究：用Bonferroni或Tukey
```

---

### Q7: 樣本量計算中的效應大小如何估計？

**答案**：
```python
# 效應大小的估計方法
# 1. 前期研究：使用文獻中的效應大小
# 2. 試驗性研究：小型先導研究
# 3. 臨床重要性：由臨床專家決定
# 4.  convention：使用Cohen的標準值

# G*Power示例
print("Sample Size Estimation Approaches:")
print("-" * 50)
print("1. From literature: Look for effect sizes in similar studies")
print("2. From pilot study: Run a small study first (n=10-20)")
print("3. Clinically meaningful: Ask experts what difference matters")
print("4. Use Cohen's conventions as starting point")

# 估算策略
print("\nPractical Strategy:")
print("1. Start with Cohen's d = 0.5 (medium effect)")
print("2. If resources allow, target d = 0.3 (detect smaller effects)")
print("3. Report sensitivity analysis for different effect sizes")
```

---

### Q8: 配對設計的優勢是什麼？

**答案**：
```python
# 配對設計的優勢
# 1. 控制混淆變量（個體差異）
# 2. 提高檢定力（減少誤差變異）
# 3. 需要的總樣本量更少

# 例子：比較兩種ECG算法的準確度
# 獨立設計：招募100人用算法A，100人用算法B
# 配對設計：同一100人先用算法A再用算法B

# 配對設計的方差
# SD_diff = SD_individual × √(2 × (1-r))
# 其中r是相關係數

# 當r=0.5時：
# SD_diff = SD × √(2 × 0.5) = SD × 1
# 相比獨立的SD，減少了約30%

print("Paired Design Benefits:")
print(f"Variance reduction factor: √(1-r)")
print(f"For r=0.3: factor = {np.sqrt(1-0.3):.2f}")
print(f"For r=0.5: factor = {np.sqrt(1-0.5):.2f}")
print(f"For r=0.7: factor = {np.sqrt(1-0.7):.2f}")
```

---

### Q9: 如何報告統計結果（APA格式）？

**答案**：
```python
# APA格式報告指南

# t檢定報告
print("t-Test Report (APA style):")
print("-" * 40)
print("There was a significant difference between")
print("Drug A (M=130.2, SD=5.1) and Drug B (M=118.4, SD=4.8);")
print("t(18) = 5.43, p < .001, d = 1.72")
print()
print("解釋：")
print("- t(18)：自由度")
print("- p < .001：p值（小於0.001）")
print("- d = 1.72：Cohen's d效應大小（大效應）")

# ANOVA報告
print("\n\nANOVA Report (APA style):")
print("-" * 40)
print("A one-way ANOVA revealed a significant effect of")
print("treatment on pain scores, F(2, 27) = 12.34, p < .001, η² = .48")
print()
print("Post-hoc tests indicated that Drug C (M=5.1) was")
print("significantly more effective than both Drug A (M=6.4, p=.012)")
print("and Drug B (M=4.2, p=.038).")
```

---

### Q10: 非劣效性檢定 vs 等效性檢定 vs 優效性檢定

**答案**：
```python
# 三種臨床試驗類型

# 1. 優效性檢定（Superiority）
# H₀: μ₁ - μ₂ ≤ δ
# H₁: μ₁ - μ₂ > δ
# 標準方法，用於新藥是否比現有藥物更好

# 2. 非劣效性檢定（Non-inferiority）
# H₀: μ₁ - μ₂ ≤ -δ  (A比B差至少δ)
# H₁: μ₁ - μ₂ > -δ  (A不比B差超過δ)
# 用於新藥是否「不差於」現有藥物

# 3. 等效性檢定（Equivalence）
# H₀: |μ₁ - μ₂| ≥ δ
# H₁: |μ₁ - μ₂| < δ
# 用於生物等效性研究

print("Clinical Trial Design Types:")
print("-" * 50)
print("Superiority:     H₀: A ≤ B,    H₁: A > B")
print("Non-inferiority: H₀: A < B-δ,  H₁: A ≥ B-δ")
print("Equivalence:     H₀: |A-B|≥δ,   H₁: |A-B|<δ")

print("\nBME Application:")
print("- Non-inferiority: New medical device as good as standard")
print("- Equivalence: Generic drug vs brand-name drug")
print("- Superiority: New treatment better than standard")
```

---

# 核心概念深化（中英對照）

## 1. 假設檢定框架 / Hypothesis Testing Framework

### 1.1 Bilingual 概念對照

| 中文 | English | 中文解釋 | English Definition |
|------|---------|----------|---------------------|
| 零假設 | Null Hypothesis (H₀) | 無效應或無差異 | Status quo assumption |
| 對立假設 | Alternative Hypothesis (H₁) | 有效應或有差異 | Research hypothesis |
| Type I 錯誤 | Type I Error | 假陽性 | Reject H₀ when true |
| Type II 錯誤 | Type II Error | 假陰性 | Fail to reject H₀ when false |
| 檢定力 | Statistical Power | 正確檢測效應的能力 | P(reject H₀ when H₁ true) |
| p值 | p-value | 觀測數據的極端程度 | P(data or more extreme \| H₀) |

### 1.2 關鍵推導

```latex
兩類錯誤的關係：
α = P(Type I) = P(reject H₀ | H₀ true)
β = P(Type II) = P(fail to reject H₀ | H₁ true)
Power = 1 - β

當樣本量增加時：
- α 不變（由研究者設定）
- β 減少
- Power 增加

當效應大小增加時：
- β 減少
- Power 增加
```

### 1.3 BME 應用

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy import stats

# Type I vs Type II Error Visualization
mu0, sigma = 0, 1
mu1 = 0.5  # 真實效應

x = np.linspace(-4, 4, 1000)
H0_dist = stats.norm.pdf(x, mu0, sigma)
H1_dist = stats.norm.pdf(x, mu1, sigma)

# 臨界值
alpha = 0.05
z_crit = stats.norm.ppf(1 - alpha)
x_crit = mu0 + z_crit * sigma

plt.figure(figsize=(12, 5))
plt.plot(x, H0_dist, 'b-', label=f'H₀: μ={mu0}')
plt.plot(x, H1_dist, 'r-', label=f'H₁: μ={mu1}')
plt.fill_between(x[x > x_crit], H0_dist[x > x_crit], alpha=0.3, color='blue', label=f'α = P(Type I)')
plt.fill_between(x[x < x_crit], H1_dist[x < x_crit], alpha=0.3, color='red', label=f'β = P(Type II)')
plt.axvline(x=x_crit, color='k', linestyle='--')
plt.xlabel('Test Statistic')
plt.ylabel('Probability Density')
plt.title('Type I and Type II Errors')
plt.legend()
plt.show()

# 計算實際錯誤率
beta = stats.norm.cdf(x_crit, mu1, sigma)
power = 1 - beta
print(f"α = {alpha}")
print(f"β = {beta:.4f}")
print(f"Power (1-β) = {power:.4f}")
```

### 1.4 Deep Test Question

**Q**: 解釋為什麼不能同時最小化α和β

**Solution**:
```latex
α和β存在權衡關係：
1. 降低α（如從0.05到0.01）→ 臨界值移向右側
2. 臨界值右移 → β增加（接受更多H₀）
3. 因此Power降低

提高Power的方法：
1. 增加樣本量n
2. 增加效應大小
3. 提高α（但這增加了Type I錯誤風險）

結論：選擇α和n需要在錯誤風險和成本之間平衡
```

---

## 2. t檢定詳解 / t-Test Deep Dive

### 2.1 Bilingual 概念對照

| 中文 | English | 中文解釋 | English Definition |
|------|---------|----------|---------------------|
| 自由度 | Degrees of Freedom | 獨立估計量的數量 | n - 1 for sample variance |
| 樣本均值的標準誤 | Standard Error | 估計的精確度 | s/√n |
| pooled variance | 合併變異數 | 兩組變異數加權平均 | Weighted average of variances |

### 2.2 關鍵推導

```latex
獨立兩樣本t檢定的 pooled variance：
s_p² = [(n₁-1)s₁² + (n₂-1)s₂²] / (n₁ + n₂ - 2)

自由度：df = n₁ + n₂ - 2

Welch's t檢定的自由度：
df = (s₁²/n₁ + s₂²/n₂)² / [(s₁²/n₁)²/(n₁-1) + (s₂²/n₂)²/(n₂-1)]
```

### 2.3 BME 應用

```python
# BME Example: ECG信號處理前後的噪聲水平比較
pre_processing = np.array([15.2, 14.8, 16.1, 15.5, 14.9, 15.8, 15.3, 14.7, 16.0, 15.4])
post_processing = np.array([3.2, 2.8, 3.5, 3.1, 2.9, 3.4, 3.0, 2.7, 3.6, 3.2])

# 配對t檢定（因為是同一信號處理前後）
differences = pre_processing - post_processing
t_stat, p_value = stats.ttest_rel(pre_processing, post_processing)

print(f"Paired t-test:")
print(f"  Mean difference: {np.mean(differences):.2f} dB")
print(f"  t = {t_stat:.4f}, df = {len(differences)-1}, p = {p_value:.2e}")
print(f"  Significant noise reduction: {p_value < 0.05}")
```

---

## 3. ANOVA詳解 / ANOVA Deep Dive

### 3.1 Bilingual 概念對照

| 中文 | English | 中文解釋 | English Definition |
|------|---------|----------|---------------------|
| 組間變異 | Between-group SS | 組間差異 | Variation due to groups |
| 組內變異 | Within-group SS | 組內差異 | Random variation |
| 均方 | Mean Square | 變異數除以自由度 | SS/df |

### 3.2 關鍵推導

```latex
F統計量的推導：
F = MS_between / MS_within

當H₀為真時：
MS_between ≈ MS_within ≈ σ²
→ F ≈ 1

當H₁為真時：
MS_between > σ²
→ F > 1

F分佈：分子df=k-1，分母df=N-k
```

### 3.3 BME 應用

```python
# 多組BME設備精度比較
device_readings = pd.DataFrame({
    'device': ['A']*8 + ['B']*8 + ['C']*8,
    'error': [0.12, 0.15, 0.11, 0.14, 0.13, 0.10, 0.16, 0.12,
              0.08, 0.09, 0.07, 0.10, 0.08, 0.11, 0.09, 0.10,
              0.05, 0.06, 0.04, 0.07, 0.05, 0.06, 0.05, 0.06]
})

# ANOVA
model = ols('error ~ C(device)', data=device_readings).fit()
anova_table = sm.stats.anova_lm(model, typ=2)

print("ANOVA Results:")
print(anova_table)
print(f"\nF({int(anova_table['df']['C(device)'])}, {int(anova_table['df']['Residual'])}): "
      f"F = {anova_table['F']['C(device)']:.2f}, p = {anova_table['PR(>F)']['C(device)']:.4f}")
```

---

## 4. 非參數檢定 / Non-parametric Tests

### 4.1 Bilingual 概念對照

| 中文 | English | 中文解釋 | English Definition |
|------|---------|----------|---------------------|
| 秩 | Rank | 觀測值排序後的位置 | Position in sorted order |
| 秩和 | Rank Sum | 一組觀測值的秩總和 | Sum of ranks for a group |
| 正秩和 | Positive Rank Sum | 正差值的秩總和 | T⁺ |

### 4.2 Python實現

```python
# Mann-Whitney U 檢定
group1 = np.array([12, 15, 18, 14, 16])
group2 = np.array([20, 22, 25, 23, 21])

# 方法1：使用scipy
stat, p = stats.mannwhitneyu(group1, group2, alternative='two-sided')
print(f"Mann-Whitney U: U={stat:.1f}, p={p:.4f}")

# 方法2：手動計算
all_data = np.concatenate([group1, group2])
ranks = stats.rankdata(all_data)
R1 = np.sum(ranks[:len(group1)])
n1, n2 = len(group1), len(group2)
U1 = R1 - n1*(n1+1)/2
U2 = n1*n2 - U1
U = min(U1, U2)
print(f"Manual U: {U}")

# Wilcoxon signed-rank 檢定
before = np.array([100, 95, 102, 98, 101, 97, 99, 103])
after = np.array([90, 88, 92, 89, 91, 87, 90, 93])

stat, p = stats.wilcoxon(before - after)
print(f"\nWilcoxon: W={stat:.1f}, p={p:.4f}")
```

---

## 5. 功效分析 / Power Analysis

### 5.1 Bilingual 概念對照

| 中文 | English | 中文解釋 | English Definition |
|------|---------|----------|---------------------|
| 功效曲線 | Power Curve | 功效 vs 樣本量關係 | Power as function of n |
| 效應大小 | Effect Size | 實際差異的大小 | Magnitude of difference |
| 樣本量估算 | Sample Size Calculation | 確定所需n | Computing required n |

### 5.2 Python實現

```python
from statsmodels.stats.power import TTestIndPower, GofChisquarePower
import matplotlib.pyplot as plt

# 功效曲線
analysis = TTestIndPower()

sample_sizes = np.arange(5, 200, 5)
effect_sizes = [0.2, 0.5, 0.8]

plt.figure(figsize=(10, 6))
for d in effect_sizes:
    powers = [analysis.power(effect_size=d, nobs=n, alpha=0.05, ratio=1) 
              for n in sample_sizes]
    plt.plot(sample_sizes, powers, label=f'd={d}')

plt.axhline(y=0.80, color='k', linestyle='--', alpha=0.5)
plt.xlabel('Sample Size per Group')
plt.ylabel('Power')
plt.title('Power Analysis for Two-Sample t-Test')
plt.legend()
plt.grid(True)
plt.show()
```

---

## 深度自測問題詳解

### 1. 假設檢定步驟

**Q**: 描述完整的假設檢定步驟

**Solution**:
```latex
1. 陳述假設：
   - H₀: 具體的參數值或無差異
   - H₁: 研究假設

2. 選擇顯著性水平：
   - α = 0.05 (標準)
   - 單尾或雙尾？

3. 選擇檢定方法：
   - 根據數據類型和前提假設
   - t檢定、ANOVA、非參數等

4. 計算檢定統計量：
   - 根據公式計算

5. 確定p值或臨界值：
   - p < α → 拒絕H₀
   - p ≥ α → 不能拒絕H₀

6. 結論：
   - 統計結論
   - 臨床/實際意義
```

---

### 2. 配對vs獨立設計

**Q**: 解釋為什麼配對設計通常更有效

**Solution**:
```latex
配對設計減少變異：
1. 每個個體作為自己的對照
2. 個體間的穩定差異被控制
3. 殘差變異減少

數學表示：
獨立設計的標準誤：SE = √(σ₁²/n₁ + σ₂²/n₂)
配對設計的標準誤：SE_diff = √(σ_d²/n)

其中 σ_d² = σ₁² + σ₂² - 2ρσ₁σ₂
ρ 是相關係數

當 ρ > 0.5 時，配對設計的SE明顯較小
→ 更大的檢定力
```

---

### 3. ANOVA事後檢驗

**Q**: ANOVA顯著後為什麼需要事後檢驗？

**Solution**:
```latex
ANOVA告訴我們「至少有一對顯著不同」
但沒有說哪一對或多對

例如：組A、組B、組C
ANOVA顯著後：
- 可能是A vs B顯著
- 可能是A vs C顯著
- 可能是B vs C顯著
- 可能三對都顯著

事後檢驗控制多重比較：
- Tukey HSD：所有成對比較
- Bonferroni：簡單但保守
- Scheffé：最靈活也最保守
```

---

## 總結

### Week 10 核心要點

| 概念 | 關鍵公式 | BME應用 |
|------|---------|---------|
| 假設檢定 | H₀, H₁, α, β, Power | 臨床試驗設計 |
| t檢定 | t = (x̄-μ)/(s/√n) | 兩組均值比較 |
| ANOVA | F = MS_between/MS_within | 多組比較 |
| 非參數檢定 | U, H, T統計量 | 非正態數據 |
| 功效分析 | n = f(α, β, d) | 樣本量計算 |

---

**Maintainer**: BME Bootcamp Agent | **Week 10** | **BMED3603: Biostatistics**
