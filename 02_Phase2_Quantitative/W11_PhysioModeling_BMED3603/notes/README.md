# Week 11 Notes — Physiological Modeling (ODEs) (BMED3603)

## 問題 1：5 個核心心智模型

### 1. 生理建模的RC電路類比 — RC Circuit Analogy for Physiological Systems

**核心概念**：一階微分方程廣泛存在於生理系統中，RC電路提供了一個直觀的類比，幫助理解藥物動力學、神經元膜電位和心血管動力學。

**數學表示**：
```latex
一階線性ODE標準形式：
τ(dy/dt) + y = K · x(t)

其中：
- τ：時間常數（time constant）
- K：靜態增益（steady-state gain）
- y：輸出（響應）
- x：輸入（刺激）

解：
y(t) = y(0)e^{-t/τ} + K(1-e^{-t/τ})x(t) [強制響應]
     = y_homogeneous + y_particular
```

**RC電路類比**：
```latex
電路方程：RC(dV/dt) + V = V_in
        τ = RC, K = 1

生理類比：
- V → 藥物濃度、膜電位、血壓
- V_in → 劑量輸入、電流輸入、心輸出
- RC → 清除率、分佈容積、順應性

物理意義：
- τ越大：系統響應越慢（長記憶）
- K越大：系統增益越高（敏感）
```

**Python模擬**：
```python
import numpy as np
import matplotlib.pyplot as plt

# 一階系統的階躍響應
def first_order_step(t, tau, K, y0=0):
    """一階系統對單位階躍輸入的響應"""
    return y0 * np.exp(-t/tau) + K * (1 - np.exp(-t/tau))

# 藥物動力學示例
# 口服給藥後血漿藥物濃度
tau = 2.0  # 小時
K = 100     #  steady-state concentration (μg/mL)
t = np.linspace(0, 10, 500)

# 不同時間常數的響應
plt.figure(figsize=(12, 5))
for tau_val in [1, 2, 4]:
    y = first_order_step(t, tau_val, K)
    plt.plot(t, y, label=f'τ = {tau_val} hr')

plt.xlabel('Time (hours)')
plt.ylabel('Drug Concentration (μg/mL)')
plt.title('First-Order Pharmacokinetics: Step Response')
plt.legend()
plt.grid(True)

# 標註半衰期
t_half = tau * np.log(2)
y_half = K * (1 - np.exp(-t_half/tau))
plt.axhline(y=K/2, color='k', linestyle=':', alpha=0.5)
plt.axvline(x=t_half, color='k', linestyle=':', alpha=0.5)
plt.annotate(f't_1/2 = {t_half:.1f}hr', xy=(t_half, K/2), 
             xytext=(t_half+1, K/2+10), fontsize=10)
plt.show()

print(f"Half-life: t_1/2 = τ·ln(2) = {tau*np.log(2):.2f} hours")
print(f"Steady-state achieved at ~4τ = {4*tau:.1f} hours")
```

**學者貢獻**：
- Teorell (1937) — 藥物動力學的先驅
- Widmark & Tandberg (1924) — 單室模型

**深度問題**：為什麼一階動力學如此常見？這個「一指數衰減」的生物學基礎是什麼？

---

### 2. 二階系統：生理振盪 — Second-Order Systems: Physiological Oscillations

**核心概念**：二階微分方程描述具有慣性和阻尼的系統，如血壓波動、心率變異性和前庭-眼反射。

**數學表示**：
```latex
標準二階系統：
d²y/dt² + 2ζω_n(dy/dt) + ω_n²y = ω_n²x(t)

其中：
- ω_n：自然頻率（undamped natural frequency）
- ζ：阻尼比（damping ratio）

特征方程：s² + 2ζω_n s + ω_n² = 0
特征根：s = -ζω_n ± ω_n√(ζ²-1)

三種情況：
1. ζ > 1（過阻尼）：兩個實根，無振盪
2. ζ = 1（臨界阻尼）：兩個相等實根
3. ζ < 1（欠阻尼）：共軛複根，衰減振盪
```

**欠阻尼響應**：
```latex
ζ < 1時：
s = -σ ± jω_d
其中：
- σ = ζω_n（衰減率）
- ω_d = ω_n√(1-ζ²)（阻尼自然頻率）

響應：
y(t) = A·e^{-σt}·cos(ω_dt + φ)

特性：
- 振盪頻率：ω_d < ω_n
- 衰減率：σ
- 峰值時間：t_p = π/ω_d
- 超調量：MP = exp(-ζπ/√(1-ζ²)) × 100%
```

**BME應用：血壓動力學**：
```python
import numpy as np
import matplotlib.pyplot as plt

def second_order_step(t, wn, zeta, K=1, y0=0, dy0=0):
    """二階系統的階躍響應"""
    sigma = zeta * wn
    wd = wn * np.sqrt(1 - zeta**2)
    
    if zeta < 1:  # Underdamped
        # Initial conditions
        A = np.sqrt(((y0 - K)**2 + ((sigma*(y0-K) + dy0)/wd)**2))
        phi = np.arctan2(-sigma*(y0-K) - dy0, (y0-K)*wd)
        
        y = K * (1 - 1/np.sqrt(1-zeta**2) * np.exp(-sigma*t) * 
                np.cos(wd*t - phi - np.arccos(zeta)))
        return y
    
    elif zeta == 1:  # Critically damped
        y = K * (1 - (1 + wn*t) * np.exp(-wn*t))
        return y
    
    else:  # Overdamped
        r1 = -sigma + np.sqrt(sigma**2 - wn**2)
        r2 = -sigma - np.sqrt(sigma**2 - wn**2)
        C1 = (y0 - K - r2*(y0-K + dy0)/r1) / (r1 - r2)
        C2 = -(y0 - K - r1*(y0-K + dy0)/r2) / (r1 - r2)
        y = K + C1*np.exp(r1*t) + C2*np.exp(r2*t)
        return y

t = np.linspace(0, 15, 500)
wn = 2.0  # Natural frequency (rad/s)

plt.figure(figsize=(12, 6))
for zeta in [0.2, 0.5, 0.7, 1.0, 2.0]:
    if zeta < 1:
        y = second_order_step(t, wn, zeta)
        label = f'ζ={zeta} (underdamped)'
    elif zeta == 1:
        y = second_order_step(t, wn, zeta)
        label = f'ζ={zeta} (critical)'
    else:
        y = second_order_step(t, wn, zeta)
        label = f'ζ={zeta} (overdamped)'
    plt.plot(t, y, label=label)

plt.xlabel('Time (s)')
plt.ylabel('Response')
plt.title('Second-Order System: Effect of Damping Ratio')
plt.legend()
plt.grid(True)

# 標註超調量
mp = np.exp(-0.5*np.pi/np.sqrt(1-0.5**2))
print(f"Overshoot for ζ=0.5: {mp*100:.1f}%")
plt.show()
```

**學者貢獻**：
- Windkessel (1899) — 心血管系統三元件模型
- Guyton (1972) — 血流動力學

**深度問題**：為什麼生理振盪通常是欠阻尼的？適度的振盪對生理功能有什麼意義？

---

### 3. Hodgkin-Huxley模型：神經計算的基石 — Hodgkin-Huxley: Foundation of Neural Computation

**核心概念**：Hodgkin-Huxley模型用四個離子通道和四個門控變量精確描述了槍烏賊巨軸突的動作電位，開創了定量神經科學。

**數學表示**：
```latex
膜電位方程：
C(dV/dt) = I - I_Na - I_K - I_L

離子電流：
I_Na = g_Na·m³h·(V - V_Na)    # 鈉離子電流
I_K = g_K·n⁴·(V - V_K)        # 鉀離子電流
I_L = g_L·(V - V_L)           # 漏電流

門控動力學：
dm/dt = α_m(V)(1-m) - β_m(V)m
dh/dt = α_h(V)(1-h) - β_h(V)h
dn/dt = α_n(V)(1-n) - β_n(V)n

平衡電位：
V_Na = +60 mV（細胞外高鈉）
V_K = -90 mV（細胞外高鉀）
V_L = -70 mV（漏電位）
```

**門控變量的意義**：
```latex
m：鈉離子激活門（快速）
  - m³表示三個m門必須同時開放
  
h：鈉離子失活門（較慢）
  - h表示失活狀態
  
n：鉀離子激活門（最慢）
  - n⁴表示四個n門必須同時開放

物理意義：
- m快速開放觸發去極化
- h延遲關閉維持去極化
- n緩慢開放導致復極化
```

**Python實現**：
```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.integrate import odeint

def hodgkin_huxley(Y, t, I_stim=10, g_Na=120, g_K=36, g_L=0.3,
                   E_Na=60, E_K=-90, E_L=-70, C=1):
    """Hodgkin-Huxley模型"""
    V, m, h, n = Y
    
    # 門控動力學（經驗公式）
    alpha_m = 0.1 * (V + 40) / (1 - np.exp(-(V + 40) / 10))
    beta_m = 4 * np.exp(-(V + 65) / 18)
    alpha_h = 0.07 * np.exp(-(V + 65) / 20)
    beta_h = 1 / (1 + np.exp(-(V + 35) / 10))
    alpha_n = 0.01 * (V + 55) / (1 - np.exp(-(V + 55) / 10))
    beta_n = 0.125 * np.exp(-(V + 65) / 80)
    
    # 離子電流
    I_Na = g_Na * m**3 * h * (V - E_Na)
    I_K = g_K * n**4 * (V - E_K)
    I_L = g_L * (V - E_L)
    
    # 微分方程
    dVdt = (I_stim - I_Na - I_K - I_L) / C
    dmdt = alpha_m * (1 - m) - beta_m * m
    dhdt = alpha_h * (1 - h) - beta_h * h
    dndt = alpha_n * (1 - n) - beta_n * n
    
    return [dVdt, dmdt, dhdt, dndt]

# 模擬
t = np.linspace(0, 100, 5000)

# 初始條件：V=-70mV rest, m,h,n steady-state
V0 = -70
m0 = 0.05
h0 = 0.6
n0 = 0.32
Y0 = [V0, m0, h0, n0]

# 注入電流：10μA/cm² 刺激
sol = odeint(hodgkin_huxley, Y0, t, args=(10,))

V = sol[:, 0]
m = sol[:, 1]
h = sol[:, 2]
n = sol[:, 3]

# 繪圖
fig, axes = plt.subplots(3, 1, figsize=(12, 8))

axes[0].plot(t, V, 'b-', linewidth=1.5)
axes[0].set_ylabel('Membrane Potential (mV)')
axes[0].set_title('Hodgkin-Huxley Action Potential')
axes[0].axhline(y=-70, color='k', linestyle=':', alpha=0.5)
axes[0].grid(True)

axes[1].plot(t, m, 'r-', label='m (Na+ activation)', linewidth=1)
axes[1].plot(t, h, 'g-', label='h (Na+ inactivation)', linewidth=1)
axes[1].plot(t, n, 'b-', label='n (K+ activation)', linewidth=1)
axes[1].set_ylabel('Gating Variables')
axes[1].set_title('Gating Variables')
axes[1].legend()
axes[1].grid(True)

axes[2].plot(t, 120*m**3*h*(V-60), 'r-', label='I_Na', linewidth=1)
axes[2].plot(t, 36*n**4*(V+90), 'b-', label='I_K', linewidth=1)
axes[2].set_xlabel('Time (ms)')
axes[2].set_ylabel('Current (μA/cm²)')
axes[2].set_title('Ion Currents')
axes[2].legend()
axes[2].grid(True)

plt.tight_layout()
plt.show()

print(f"Resting potential: {V[0]:.1f} mV")
print(f"Peak potential: {np.max(V):.1f} mV")
print(f"Action potential width: ~2 ms")
```

**學者貢獻**：
- Hodgkin & Huxley (1952) — Nobel Prize 1963
- 實驗數據來自槍烏賊巨軸突

**深度問題**：為什麼門控變量是m³h和n⁴而不是簡單的線性組合？這個非線性有什麼生理意義？

---

### 4. 藥物動力學：單室vs雙室模型 — Pharmacokinetics: One vs Two Compartments

**核心概念**：藥物動力學模型描述藥物在體內的吸收、分佈、代謝和排泄過程。單室模型適用於快速平衡的藥物，雙室模型適用於分佈不均勻的藥物。

**單室模型**：
```latex
一階消除：
dC/dt = -k_e·C

解：C(t) = C₀·e^{-k_e·t}

參數：
- k_e：消除速率常數 (hr⁻¹)
- t_1/2 = ln(2)/k_e ≈ 0.693/k_e（半衰期）
- CL = k_e·V_d（清除率）
- V_d：表現分佈容積

口服給藥：
C(t) = (F·D·k_a)/(V_d(k_a-k_e)) · (e^{-k_e·t} - e^{-k_a·t})

其中：
- F：生物可用度
- D：劑量
- k_a：吸收速率常數
```

**雙室模型**：
```latex
中央室（血漿）和周邊室（組織）：
dC_c/dt = -k_e·C_c - k_12·C_c + k_21·C_p
dC_p/dt = k_12·C_c - k_21·C_p

其中：
- C_c：中央室濃度
- C_p：周邊室濃度
- k_12, k_21：室間轉運速率
- k_e：中央室消除速率

解：雙指數函數
C_c(t) = A·e^{-αt} + B·e^{-βt}

其中：
- α：分佈相（快速）
- β：消除相（緩慢）
- A, B：由初始條件決定的常數
```

**Python實現**：
```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.integrate import odeint

# 單室模型
def one_compartment(Y, t, k_e, k_a=0, F=1, dose=100, V_d=50):
    """單室藥物動力學模型"""
    C = Y
    
    if k_a > 0:  # Oral dosing
        # Absorption and elimination
        dCdt = k_a * F * dose/V_d - k_e * C
    else:  # IV bolus
        dCdt = -k_e * C
    
    return dCdt

# IV給藥模擬
k_e = 0.2  # hr⁻¹ (半衰期 ~3.5 hr)
t = np.linspace(0, 24, 500)

C0 = 100 / 50  # 100mg dose, 50L V_d
C = C0 * np.exp(-k_e * t)

plt.figure(figsize=(12, 8))

# 單室
plt.subplot(2, 2, 1)
plt.semilogy(t, C, 'b-', linewidth=2)
plt.xlabel('Time (hours)')
plt.ylabel('Concentration (mg/L)')
plt.title('One-Compartment: IV Bolus')
plt.grid(True)

# 雙室
def two_compartment(Y, t, k_e, k_12, k_21):
    """雙室藥物動力學模型"""
    C_c, C_p = Y
    
    dCc_dt = -k_e*C_c - k_12*C_c + k_21*C_p
    dCp_dt = k_12*C_c - k_21*C_p
    
    return [dCc_dt, dCp_dt]

# 雙室參數
k_e = 0.1
k_12 = 0.2
k_21 = 0.05

# 初始條件：IV給藥只進入中央室
C_c0 = 2.0  # mg/L
C_p0 = 0.5  # mg/L

sol = odeint(two_compartment, [C_c0, C_p0], t, args=(k_e, k_12, k_21))

plt.subplot(2, 2, 2)
plt.semilogy(t, sol[:, 0], 'b-', label='Central (blood)', linewidth=2)
plt.semilogy(t, sol[:, 1], 'r-', label='Peripheral (tissue)', linewidth=2)
plt.xlabel('Time (hours)')
plt.ylabel('Concentration (mg/L)')
plt.title('Two-Compartment: IV Bolus')
plt.legend()
plt.grid(True)

# 對數-線性圖
plt.subplot(2, 2, 3)
plt.plot(t, sol[:, 0], 'b-', linewidth=2)
plt.xlabel('Time (hours)')
plt.ylabel('Central Compartment (mg/L)')
plt.title('Two-Compartment: Log-Linear Plot')
plt.grid(True)
plt.yscale('log')

# 雙室模型示意圖
plt.subplot(2, 2, 4)
plt.text(0.5, 0.8, 'Two-Compartment Model:', fontsize=12, fontweight='bold', 
         transform=plt.gca().transAxes)
plt.text(0.5, 0.6, '• Central: Blood, plasma\n• Peripheral: Tissues\n• k_12, k_21: Transfer rates\n• k_e: Elimination rate',
         fontsize=10, transform=plt.gca().transAxes, va='top')
plt.axis('off')

plt.tight_layout()
plt.show()

print(f"\nOne-compartment half-life: {np.log(2)/k_e:.1f} hours")
```

**學者貢獻**：
- Teorell (1937) — 藥物動力學的數學基礎
- Gibaldi & Perrier (1982) — 現代藥物動力學

**深度問題**：為什麼有些藥物需要雙室模型？什麼時候單室模型足夠？

---

### 5. Windkessel模型：心血管建模 — Windkessel: Cardiovascular System Model

**核心概念**：Windkessel模型用於描述心臟搏出時主動脈的血流動力學，是理解血壓波形和心血管疾病的重要工具。

**二元件Windkessel**：
```latex
模型：電阻R和順應性C串聯
方程：C(dP/dt) + P/R = Q(t)

其中：
- P：血壓
- Q：血流量
- R：系統血管阻力
- C：總順應性

脈搏波速：
PWV = √(C/ρ) · √(dP/dV)

其中：
- ρ：血液密度
- C：順應性
```

**三元件Windkessel**：
```latex
增加特性阻抗Z_c：
Z_c = ρ · PWV / A

方程：
C(dP/dt) + P/R = Q(t) + C·Z_c(dQ/dt)

優點：
- 更好地擬合早期收縮波形
- 考慮主動脈的慣性效應
```

**Python實現**：
```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.integrate import odeint

def windkessel_2element(Y, t, R, C, Q_func):
    """二元件Windkessel模型"""
    P = Y[0]
    Q_in = Q_func(t)
    dPdt = (Q_in - P/R) / C
    return [dPdt]

def windkessel_3element(Y, t, R1, R2, C, Q_func):
    """三元件Windkessel模型"""
    P, Q_out = Y
    Q_in = Q_func(t)
    # P近端 = Q_in*Z_c + P
    # Q_out = (P - P_distal)/R2
    # C(dP/dt) = Q_in - Q_out
    dPdt = (Q_in - Q_out - P/R2) / C
    dQoutdt = (P/R2 - Q_out) / (R1 * C)  # Simplified
    return [dPdt, dQoutdt]

# 模擬心臟週期
t = np.linspace(0, 1, 1000)  # 1秒心週期

# 心輸出波形（心室容積變化率）
def cardiac_output(t, HR=60):
    """心輸出波形"""
    cycle = t * HR / 60  # 0-1 within cycle
    if cycle < 0.1:
        return 0  # Isovolumetric contraction
    elif cycle < 0.3:
        return 300 * np.sin(np.pi * (cycle - 0.1) / 0.2)  # Systole
    else:
        return 0  # Diastole

# 參數
R = 1.0  # mmHg·s/mL (systemic resistance)
C = 1.5  # mL/mmHg (total compliance)

# 運行模擬
P0 = [80]  # Initial pressure (mmHg)
sol = odeint(windkessel_2element, P0, t, args=(R, C, cardiac_output))

plt.figure(figsize=(12, 8))

# 血壓波形
plt.subplot(2, 2, 1)
plt.plot(t, sol[:, 0], 'b-', linewidth=2)
plt.xlabel('Time (s)')
plt.ylabel('Blood Pressure (mmHg)')
plt.title('2-Element Windkessel: Blood Pressure')
plt.grid(True)

# 比較不同順應性
plt.subplot(2, 2, 2)
for C_val in [1.0, 1.5, 2.0]:
    sol = odeint(windkessel_2element, P0, t, args=(R, C_val, cardiac_output))
    plt.plot(t, sol[:, 0], label=f'C={C_val}', linewidth=2)
plt.xlabel('Time (s)')
plt.ylabel('Pressure (mmHg)')
plt.title('Effect of Compliance')
plt.legend()
plt.grid(True)

# 血壓-流量關係
plt.subplot(2, 2, 3)
Q = np.array([cardiac_output(ti) for ti in t])
plt.plot(t, Q, 'r-', label='Flow (Q)', linewidth=2)
plt.plot(t, sol[:, 0], 'b-', label='Pressure (P)', linewidth=2)
plt.xlabel('Time (s)')
plt.xlabel('Flow/Pressure')
plt.title('Pressure-Flow Relationship')
plt.legend()
plt.grid(True)

plt.tight_layout()
plt.show()

print(f"Systolic pressure: {np.max(sol):.1f} mmHg")
print(f"Diastolic pressure: {np.min(sol):.1f} mmHg")
print(f"Pulse pressure: {np.max(sol) - np.min(sol):.1f} mmHg")
```

**學者貢獻**：
- Otto Frank (1899) — Windkessel原理的數學基礎
- Westerhof et al. (1969) — 現代三元件模型

**深度問題**：為什麼三元件Windkessel比二元件更能模擬真實血壓波形？

---

## 問題 2：3 個根本分歧

### 分歧 1：機理模型 vs 經驗模型 — Mechanistic vs Empirical Models

**機理模型支持者觀點**：
- 基於物理/生理原理
- 可解釋性強
- 可預測干預效果
- 例：Hodgkin-Huxley、Windkessel

**經驗模型支持者觀點**：
- 簡單實用
- 適合數據驅動分析
- 不需要詳細生理知識
- 例：經驗公式、回歸模型

**實際答案**：視數據和目的而定。機理模型適合藥物開發和解釋性研究；經驗模型適合臨床預測和快速部署。

---

### 分歧 2：確定性 vs 隨機微分方程 — Deterministic vs Stochastic ODEs

**確定性模型**：
- 給定初始條件，結果確定
- 可重現
- 例：基本HH模型

**隨機模型**：
- 包含隨機項
- 捕捉生物變異
- 例：添加噪聲的離子通道

**實際答案**：生理系統本質上有隨機性，但確定性模型在群體層面往往足夠。藥物開發用確定性，臨床決策需考慮個體變異。

---

### 分歧 3：解析解 vs 數值解 — Analytical vs Numerical Solutions

**解析解優勢**：
- 精確
- 揭示數學結構
- 便於理論分析

**數值解優勢**：
- 可處理複雜模型
- 任意初始條件
- 可加入非線性

**實際答案**：簡單模型追求解析解；複雜模型（真實生理系統）用數值解。

---

## 問題 3：10 個深度問題

### Q1: 什麼是時間常數τ的生理意義？

**答案**：
```latex
時間常數τ定義系統響應速度：

τ = 系統達到最終值的63.2%所需的時間

物理意義：
- τ越大：系統「記憶」越長
- τ越小：系統響應越快

常見生理τ值：
- 藥物消除：0.5-24 小時
- 神經膜電位：1-10 ms
- 血壓波：0.1-1 秒
- 溫度調節：分鐘級

τ與其他參數的關係：
- 一階RC：τ = RC
- 藥物動力學：τ = V_d / CL
```

---

### Q2: 如何從實驗數據估計ODE參數？

**答案**：
```python
# 參數估計方法
import numpy as np
from scipy.optimize import curve_fit

# 示例：估計藥物動力學參數
def one_compartment_model(t, C0, k_e):
    return C0 * np.exp(-k_e * t)

# 實驗數據
t_data = np.array([0, 1, 2, 4, 6, 8, 12])
C_data = np.array([10.0, 8.2, 6.7, 4.5, 3.0, 2.0, 0.9])

# 非線性最小二乘擬合
popt, pcov = curve_fit(one_compartment_model, t_data, C_data, 
                       p0=[10, 0.2], bounds=([0, 0], [20, 1]))
C0_fit, k_e_fit = popt

print(f"Estimated: C0 = {C0_fit:.2f} mg/L, ke = {k_e_fit:.3f} hr⁻¹")
print(f"Half-life: {np.log(2)/k_e_fit:.2f} hours")

# 残差分析
residuals = C_data - one_compartment_model(t_data, *popt)
print(f"RMSE: {np.sqrt(np.mean(residuals**2)):.3f}")
```

---

### Q3: 什麼是系統識別？如何應用於生理建模？

**答案**：
```python
# 系統識別流程
print("System Identification Steps:")
print("1. Collect input-output data")
print("2. Choose model structure (order, type)")
print("3. Estimate parameters (least squares, ML)")
print("4. Validate model (residuals, prediction)")
print("5. Simplify if needed")

# 示例：從血壓數據識別Windkessel參數
# 輸入：心輸出Q(t)
# 輸出：血壓P(t)
# 模型：R, C估計
```

---

### Q4: 為什麼HH模型用四個門控變量？

**答案**：
```latex
實驗觀察：
1. 鈉電流激活快、失活慢
2. 鉀電流激活慢、無失活
3. 需要三個變量描述鈉通道（m³h）
4. 需要一個變量描述鉀通道（n⁴）

非線性組合的生理意義：
- m³：需要三個門都開放才能導電
- n⁴：需要四個門都開放才能導電
- 這種合作性使通道更像「開關」

物理化學基礎：
- 蛋白質構象變化
- 多個亞基的協調運動
```

---

### Q5: 什麼是分佈容積V_d的意義？

**答案**：
```latex
V_d的定義：
V_d = Amount of drug / Plasma concentration

物理意義：
- 如果V_d等於體液總量 → 藥物均勻分佈
- 如果V_d > 體液總量 → 藥物在組織中蓄積
- 如果V_d < 血漿體積 → 藥物主要在血漿中

常見藥物的V_d：
- 華法林：V_d ≈ 10L（小）
- 咖啡因：V_d ≈ 40L（中等）
- 氯奎：V_d ≈ 1000L（大，在組織蓄積）

臨床意義：
- V_d影響負荷劑量計算
- V_d大 → 組織效果好
```

---

### Q6: 如何比較不同模型的擬合優劣？

**答案**：
```python
import numpy as np

# 模型選擇準則
def model_comparison(C_data, C_pred, n_params, n_data):
    # 殘差平方和
    RSS = np.sum((C_data - C_pred)**2)
    
    # 估計σ²
    sigma2 = RSS / (n_data - n_params)
    
    # AIC (Akaike Information Criterion)
    AIC = n_data * np.log(sigma2) + 2 * n_params
    
    # BIC (Bayesian Information Criterion)
    BIC = n_data * np.log(sigma2) + n_params * np.log(n_data)
    
    # R²
    TSS = np.sum((C_data - np.mean(C_data))**2)
    R2 = 1 - RSS/TSS
    
    # 調整R²
    R2_adj = 1 - (RSS/(n_data-n_params))/(TSS/(n_data-1))
    
    return {'RSS': RSS, 'AIC': AIC, 'BIC': BIC, 'R2': R2, 'R2_adj': R2_adj}

print("Model comparison metrics:")
print("- Lower AIC/BIC: Better")
print("- Higher R²: Better fit")
print("- R²_adj accounts for model complexity")
```

---

### Q7: 什麼是穩態和瞬態響應？

**答案**：
```latex
穩態響應（Steady-State）：
- 當t → ∞時的響應
- 對於階躍輸入：y_ss = K（常量）
- 系統已「穩定」

瞬態響應（Transient）：
- 達到穩態之前的響應
- 包含振盪（如果欠阻尼）
- 由系統極點決定

總響應 = 穩態 + 瞬態
y(t) = y_ss + y_transient(t)

例：一階系統
y(t) = K(1 - e^{-t/τ}) + y(0)e^{-t/τ}
     = K + [y(0) - K]e^{-t/τ}
     = 穩態 + 瞬態
```

---

### Q8: 如何處理生理模型中的非線性？

**答案**：
```python
# 非線性處理方法

# 1. 線性化（泰勒展開）
# 在工作點附近用線性近似

# 2. 數值積分
from scipy.integrate import ode45

def nonlinear_system(Y, t):
    V, w = Y
    # 非線性微分方程
    dVdt = -V**3 + w
    dwdt = -V
    return [dVdt, dwdt]

# 3. 非線性擬合
from scipy.optimize import curve_fit

def nonlinear_model(t, a, b, c):
    return a * np.exp(-b*t) + c*t**2
```

---

### Q9: 什麼是藥物的半衰期？如何影響給藥方案？

**答案**：
```latex
半衰期定義：
t_1/2 = ln(2)/k_e ≈ 0.693/k_e

意義：
- 藥物濃度降低50%所需的時間
- 與給藥頻率直接相關

給藥方案考慮：
- 頻繁給藥（t < t_1/2）：蓄積風險
- 延長給藥（t > t_1/2）：波動大
- 最佳：每個半衰期給藥一次

蓄積因子：
R = 1 / (1 - e^{-k_e·τ})
其中τ是給藥間隔

穩態濃度波動：
C_max / C_min = e^{k_e·τ}
```

---

### Q10: 如何驗證生理模型的臨床適用性？

**答案**：
```python
# 模型驗證步驟

# 1. 內部驗證
# - 擬合優度（R², RMSE）
# - 殘差分析
# - 交叉驗證

# 2. 外部驗證
# - 用新數據測試
# - 不同人群驗證

# 3. 臨床驗證
# - 預測與實際臨床結局比較
# - 敏感性分析

# 4. 不確定性量化
# - 參數置信區間
# - 預測區間

print("Model Validation Checklist:")
print("✓ Parameter estimation quality")
print("✓ Residual analysis")
print("✓ External dataset testing")
print("✓ Sensitivity analysis")
print("✓ Clinical outcome prediction")
```

---

# 核心概念深化（中英對照）

## 1. 微分方程基礎 / Differential Equations Foundation

### 1.1 Bilingual 概念對照

| 中文 | English | 中文解釋 | English Definition |
|------|---------|----------|---------------------|
| 微分方程 | Differential Equation | 含導數的方程 | Equation with derivatives |
| 常微分方程 | ODE | 只含一個變量的導數 | Single variable derivative |
| 偏微分方程 | PDE | 含多變量偏導數 | Partial derivatives |
| 初值問題 | Initial Value Problem | 給定初始條件 | IVP: y(0)=y₀ |
| 邊值問題 | Boundary Value Problem | 給定邊界條件 | BVP: y(a)=A, y(b)=B |

### 1.2 數值方法

```python
# 歐拉法
def euler_method(f, y0, t):
    y = np.zeros(len(t))
    y[0] = y0
    for i in range(1, len(t)):
        dt = t[i] - t[i-1]
        y[i] = y[i-1] + f(y[i-1], t[i-1]) * dt
    return y

# 龍格-庫塔法（RK4）
from scipy.integrate import solve_ivp

def rk4_system(t, y):
    return -y + np.sin(t)  # Example

sol = solve_ivp(rk4_system, [0, 10], [1], method='RK45', dense_output=True)
```

---

## 2. 藥物動力學 / Pharmacokinetics

### 2.1 Bilingual 概念對照

| 中文 | English | 中文解釋 | English Definition |
|------|---------|----------|---------------------|
| 清除率 | Clearance (CL) | 單位時間被清除的血漿體積 | Volume cleared per unit time |
| 分佈容積 | Volume of Distribution (V_d) | 理論上的藥物分佈空間 | Theoretical space |
| 半衰期 | Half-life (t_1/2) | 濃度降低50%的時間 | Time for 50% reduction |
| 生物可用度 | Bioavailability (F) | 進入體循環的藥物比例 | Fraction reaching systemic |

### 2.2 關鍵公式

```latex
基本關係：
V_d = Dose / C₀
CL = k_e · V_d
t_1/2 = ln(2) / k_e = 0.693 · V_d / CL

穩態濃度（重複給藥）：
C_ss,max = (F·D/V_d) / (1 - e^{-k_e·τ})
C_ss,min = C_ss,max · e^{-k_e·τ}
```

---

## 3. Hodgkin-Huxley模型 / Hodgkin-Huxley Model

### 3.1 Bilingual 概念對照

| 中文 | English | 中文解釋 | English Definition |
|------|---------|----------|---------------------|
| 膜電位 | Membrane Potential | 跨膜電壓差 | Voltage across membrane |
| 動作電位 | Action Potential | 神經信號脈衝 | Nerve impulse |
| 去極化 | Depolarization | 電位變正 | Becoming less negative |
| 復極化 | Repolarization | 恢復靜息電位 | Returning to rest |
| 超極化 | Hyperpolarization | 電位更負 | More negative than rest |

### 3.2 參數表

| 參數 | 符號 | 值 | 單位 |
|------|------|-----|------|
| 膜電容 | C | 1.0 | μF/cm² |
| 鈉電導 | g_Na | 120 | mS/cm² |
| 鉀電導 | g_K | 36 | mS/cm² |
| 漏電導 | g_L | 0.3 | mS/cm² |
| 鈉平衡電位 | E_Na | +60 | mV |
| 鉀平衡電位 | E_K | -90 | mV |
| 漏電位 | E_L | -70 | mV |

---

## 4. 心血管建模 / Cardiovascular Modeling

### 4.1 Bilingual 概念對照

| 中文 | English | 中文解釋 | English Definition |
|------|---------|----------|---------------------|
| 收縮壓 | Systolic Pressure | 心室收縮時血壓峰值 | Peak during contraction |
| 舒張壓 | Diastolic Pressure | 心室舒張時血壓最低值 | Minimum during relaxation |
| 脈搏壓 | Pulse Pressure | 收縮壓-舒張壓 | SP - DP |
| 順應性 | Compliance | 容積變化/壓力變化 | ΔV/ΔP |
| 阻力 | Resistance | 血流障礙 | R = ΔP/Q |

### 4.2 臨床相關

```python
# 臨床血壓解讀
def bp_interpretation(systolic, diastolic):
    """根據JNC 7分類"""
    if systolic < 120 and diastolic < 80:
        return "Normal"
    elif systolic < 140 or diastolic < 90:
        return "Prehypertension"
    elif systolic < 160 or diastolic < 100:
        return "Stage 1 Hypertension"
    else:
        return "Stage 2 Hypertension"

print(bp_interpretation(135, 85))  # Prehypertension
```

---

## 5. 模型驗證 / Model Validation

### 5.1 驗證方法

```python
# 交叉驗證
from sklearn.model_selection import cross_val_score
from sklearn.linear_model import LinearRegression

# 敏感性分析
def sensitivity_analysis(model, param_ranges):
    """計算參數敏感度"""
    # 計算偏導數
    # 使用有限差分
    pass
```

---

## 總結

### Week 11 核心要點

| 概念 | 關鍵方程式 | BME應用 |
|------|-----------|---------|
| 一階ODE | τ(dy/dt)+y = Kx(t) | 藥物動力學 |
| 二階ODE | s²+2ζωₙs+ωₙ²=0 | 心血管振盪 |
| Hodgkin-Huxley | C(dV/dt)=I-ΣI_ion | 神經元模型 |
| Windkessel | C(dP/dt)+P/R=Q | 血壓建模 |
| 數值方法 | RK4, odeint | 複雜系統求解 |

---

**Maintainer**: BME Bootcamp Agent | **Week 11** | **BMED3603: Physiological Modeling**
