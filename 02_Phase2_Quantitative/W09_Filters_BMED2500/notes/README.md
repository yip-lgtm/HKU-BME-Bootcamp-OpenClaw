# Week 9 Notes — FIR/IIR Filters & Signal Processing Tools (BMED2500)

## 問題 1：5 個核心心智模型

### 1. FIR vs IIR：兩種濾波器哲學 — FIR vs IIR: Two Filter Philosophies

**核心概念**：數字濾波器分為有限脈衝響應（FIR）和無限脈衝響應（IIR）兩大類。FIR總是穩定且可實現線性相位，但計算量大；IIR計算效率高，但相位特性差且可能不穩定。

**數學表示**：
```latex
FIR濾波器：H(z) = Σ_{k=0}^{M} b_k z^{-k}
            = b₀ + b₁z^{-1} + ... + b_Mz^{-M}
           脈衝響應：h[n] = b_n, 對 n > M, h[n] = 0

IIR濾波器：H(z) = Σ_{k=0}^{M} b_k z^{-k} / (1 + Σ_{k=1}^{N} a_k z^{-k})
           脈衝響應：h[n] 一般無限長
```

**比較表**：
| 特性 | FIR | IIR |
|------|-----|-----|
| 脈衝響應 | 有限長度M+1 | 無限長度 |
| 穩定性 | 總是穩定 | 可能不穩定 |
| 相位 | 可嚴格線性 | 非線性 |
| 計算效率 | O(M) 乘法/輸出 | O(N) 乘法/輸出 |
| 設計方法 | 窗口法、Parks-McClellan | 模擬濾波器轉換 |
| 頻率選擇 | 需高階數 | 階數低 |

**BME應用選擇原則**：
- **需要線性相位**（波形保真）：ECG QRS檢測、語音處理 → FIR
- **計算資源受限**：嵌入式系統、實時處理 → IIR
- **需要陡峭過渡帶**：心電圖基線漂移移除 → IIR

**學者貢獻**：
- Parks-McClellan (1972) — 最優等紋波FIR設計
- Butterworth (1930) — 最大平坦幅度濾波器

**深度問題**：為什麼FIR總是穩定的？這個「有限」意味著什麼？

---

### 2. 窗口函數：頻譜洩漏的控制 — Window Functions: Spectral Leakage Control

**核心概念**：將無限長的理想脈衝響應截斷為有限長度會導致Gibbs現象和頻譜洩漏。窗口函數通過加權截斷來控制這些效應。

**數學表示**：
```latex
理想低通：h_d[n] = (sin(ω_c n))/πn,  for all n

加窗設計：h[n] = h_d[n] · w[n],  n = 0, 1, ..., M

頻譜效應：H(e^{jω}) = (1/2π)H_d(e^{jω}) * W(e^{jω})
其中 * 是卷積
```

**常用窗口函數**：

| 窗口 | w[n] | 主瓣寬度 | 峰值旁瓣 (dB) |
|------|------|---------|--------------|
| 矩形 | 1 | 4π/(M+1) | -13.3 |
| Hann | 0.5(1-cos(2πn/M)) | 8π/(M+1) | -31.5 |
| Hamming | 0.54-0.46cos(2πn/M) | 8π/(M+1) | -41.9 |
| Blackman | 0.42-0.5cos+0.08cos(4πn/M) | 12π/(M+1) | -58.1 |
| Kaiser | I₀(β√(1-(2n/M)²))/I₀(β) | 可變 | 可變 |

**Python實現**：
```python
import numpy as np
import matplotlib.pyplot as plt
from scipy import signal

# 設計參數
fs = 1000
fc = 100  # 截止頻率 Hz
M = 51    # 濾波器長度

# 理想脈衝響應
n = np.arange(-M//2, M//2 + 1)
h_ideal = np.sinc(2 * fc/fs * n)

# 不同窗口
w_rect = np.ones_like(n)
w_hann = np.hanning(M)
w_hamm = np.hamming(M)
w_black = np.blackman(M)

# 加窗
h_rect = h_ideal * w_rect
h_hann = h_ideal * w_hann
h_hamm = h_ideal * w_hamm
h_black = h_ideal * w_black

# 頻率響應
w, H_rect = signal.freqz(h_rect, worN=1024)
w, H_hann = signal.freqz(h_hann, worN=1024)
w, H_hamm = signal.freqz(h_hamm, worN=1024)
w, H_black = signal.freqz(h_black, worN=1024)

plt.figure(figsize=(12, 8))
plt.subplot(2, 1, 1)
plt.plot(w*fs/(2*np.pi), 20*np.log10(np.abs(H_rect)), label='Rectangular')
plt.plot(w*fs/(2*np.pi), 20*np.log10(np.abs(H_hann)), label='Hann')
plt.plot(w*fs/(2*np.pi), 20*np.log10(np.abs(H_hamm)), label='Hamming')
plt.plot(w*fs/(2*np.pi), 20*np.log10(np.abs(H_black)), label='Blackman')
plt.axvline(x=fc, color='k', linestyle='--', alpha=0.5)
plt.axhline(y=-3, color='r', linestyle='--', alpha=0.5)
plt.xlabel('Frequency (Hz)')
plt.ylabel('Magnitude (dB)')
plt.title('Window Comparison: FIR Lowpass Filters')
plt.legend()
plt.grid(True)
plt.xlim(0, 300)

plt.subplot(2, 1, 2)
plt.plot(n, w_rect, label='Rectangular')
plt.plot(n, w_hann, label='Hann')
plt.plot(n, w_hamm, label='Hamming')
plt.plot(n, w_black, label='Blackman')
plt.xlabel('Sample index')
plt.ylabel('Amplitude')
plt.title('Window Functions')
plt.legend()
plt.grid(True)
plt.show()
```

**學者貢獻**：
- Blackman & Tukey (1958) — 功率譜估計的窗口技術
- Kaiser (1974) — 可調參數窗口

**深度問題**：為什麼窗口函數能減少頻譜洩漏？主瓣寬度和旁瓣水平之間有什麼權衡？

---

### 3. IIR濾波器：模擬原型的數字化 — IIR Filters: Digitizing Analog Prototypes

**核心概念**：IIR濾波器設計通常從模擬濾波器（Butterworth、Chebyshev、橢圓）開始，然後使用雙線性變換轉換到數字域。

**模擬濾波器類型**：

| 類型 | 幅度響應 | 紋波位置 | 階躍響應 |
|------|---------|---------|---------|
| Butterworth | 最大平坦 | 無紋波 | 振鈴輕微 |
| Chebyshev I | 等紋波 | 通帶 | 振鈴中等 |
| Chebyshev II | 等紋波 | 阻帶 | 振鈴輕微 |
| 橢圓 | 等紋波 | 兩者都有 | 振鈴最重 |

**Butterworth幅度響應**：
```latex
|H(jω)|² = 1 / [1 + (ω/ω_c)^{2n}]

特性：
- ω=0 時 |H|=1（最大平坦）
- ω=ω_c 時 |H|=1/√2（-3dB）
- n越大，過渡越陡
```

**雙線性變換**：
```latex
s = (2/T_s) · (1 - z^{-1}) / (1 + z^{-1})

或者：z = (1 + sT_s/2) / (1 - sT_s/2)

映射關係：
- s平面左半平面 → z平面單位圓內
- s平面虛軸 → z平面單位圓
- s平面右半平面 → z平面單位圓外
```

**頻率預畸變**：
```latex
因為 tan(ωT_s/2) ≠ ωT_s/2（非線性映射）

預畸變公式：ω_a = (2/T_s) · tan(ω_d T_s/2)

其中：
- ω_a 是模擬頻率
- ω_d 是期望的數字頻率
```

**Python實現**：
```python
import numpy as np
import matplotlib.pyplot as plt
from scipy import signal

fs = 1000  # 採樣率
fc = 100   # 截止頻率
order = 4

# 設計Butterworth低通
b_butt, a_butt = signal.butter(order, fc/(fs/2), btype='low')

# 設計Chebyshev Type I
b_cheb, a_cheb = signal.cheby1(order, 0.5, fc/(fs/2), btype='low')  # 0.5dB通帶紋波

# 設計Chebyshev Type II (阻帶紋波)
b_cheb2, a_cheb2 = signal.cheby2(order, 40, fc/(fs/2), btype='low')  # 40dB阻帶衰減

# 設計橢圓
b_ellip, a_ellip = signal.ellip(order, 0.5, 40, fc/(fs/2), btype='low')

# 頻率響應
w, H_butt = signal.freqz(b_butt, a_butt)
_, H_cheb = signal.freqz(b_cheb, a_cheb)
_, H_cheb2 = signal.freqz(b_cheb2, a_cheb2)
_, H_ellip = signal.freqz(b_ellip, a_ellip)

plt.figure(figsize=(12, 8))
plt.subplot(2, 1, 1)
plt.plot(w*fs/(2*np.pi), 20*np.log10(np.abs(H_butt)), label='Butterworth')
plt.plot(w*fs/(2*np.pi), 20*np.log10(np.abs(H_cheb)), label='Chebyshev I (0.5dB ripple)')
plt.plot(w*fs/(2*np.pi), 20*np.log10(np.abs(H_cheb2)), label='Chebyshev II (40dB stopband)')
plt.plot(w*fs/(2*np.pi), 20*np.log10(np.abs(H_ellip)), label='Elliptic (0.5dB/40dB)')
plt.axvline(x=fc, color='k', linestyle='--', alpha=0.5)
plt.axhline(y=-3, color='r', linestyle='--', alpha=0.5)
plt.xlabel('Frequency (Hz)')
plt.ylabel('Magnitude (dB)')
plt.title('IIR Filter Comparison: Lowpass fc=100Hz, Order=4')
plt.legend()
plt.grid(True)
plt.xlim(0, 400)
plt.ylim(-80, 5)

# 極零點分析
plt.subplot(2, 1, 2)
for name, b, a in [('Butter', b_butt, a_butt), ('Cheby1', b_cheb, a_cheb)]:
    z, p, k = signal.tf2zpk(b, a)
    plt.scatter(np.real(p), np.imag(p), marker='x', s=100, label=f'{name} poles')
    
unit_circle = plt.Circle((0, 0), 1, fill=False, color='k', linewidth=1)
plt.gca().add_patch(unit_circle)
plt.axhline(y=0, color='k', linewidth=0.5)
plt.axvline(x=0, color='k', linewidth=0.5)
plt.xlabel('Real')
plt.ylabel('Imaginary')
plt.title('Pole Locations (z-plane)')
plt.legend()
plt.grid(True)
plt.xlim(-2, 2)
plt.ylim(-2, 2)
plt.gca().set_aspect('equal')
plt.tight_layout()
plt.show()
```

**學者貢獻**：
- Butterworth (1930) — 最大平坦幅度設計
- Chebyshev (1899) — 切比雪夫多項式
- Parks-McClellan (1972) — 最優FIR設計

**深度問題**：雙線性變換為什麼是非線性映射？這種非線性如何影響頻率響應？

---

### 4. 線性相位：波形保真的關鍵 — Linear Phase: Waveform Fidelity

**核心概念**：線性相位濾波器對所有頻率分量引入相同的延遲，確保波形形狀不變。這對ECG、EEG等波形分析至關重要。

**數學表示**：
```latex
線性相位條件：∠H(e^{jω}) = -ωτ,  for all ω

群延遲：τ_g(ω) = -d∠H(ω)/dω = τ (常數)

相位延遲：τ_p(ω) = -∠H(ω)/ω = τ (常數)

對正弦輸入：y(t) = A·sin(ω(t-τ)) = A·sin(ωt - ωτ)
         = A·sin(ωt + ∠H(ω))
```

**FIR線性相位類型**：

| 類型 | M | 對稱性 | h[n] = h[M-n] | ω=π處零點 |
|------|---|--------|---------------|-----------|
| I | 偶數 | 偶對稱 | 是 | 可選 |
| II | 奇數 | 偶對稱 | 是 | 是 |
| III | 偶數 | 奇對稱 | 否 | 是 |
| IV | 奇數 | 奇對稱 | 否 | 否 |

**幅度響應對稱性**：
```latex
偶對稱脈衝響應（Type I, II）：
H(e^{jω}) = e^{-jωM/2} · [h[M/2] + 2Σ_{n=0}^{M/2-1} h[n]cos(ω(n-M/2))]
           = e^{-jωM/2} · A(ω)

幅度 A(ω) 是 cos 函數的疊加
```

**BME臨床意義**：
```python
import numpy as np
import matplotlib.pyplot as plt

# 模擬線性vs非線性相位對ECG的影響
fs = 500
t = np.arange(0, 1, 1/fs)

# 簡化ECG波形
ecg = np.zeros_like(t)
ecg[100:120] = np.sin(np.linspace(0, np.pi, 20))  # R峰
ecg[80:100] = -0.3 * np.sin(np.linspace(0, np.pi, 20))  # QRS

# 線性相位延遲：所有頻率延遲相同
delay_samples = 20
ecg_linear_phase = np.zeros_like(ecg)
ecg_linear_phase[delay_samples:] = ecg[:-delay_samples]

# 非線性相位（簡化模擬：低頻延遲多，高頻延遲少）
def apply_nonlinear_phase(signal, delay_low, delay_high, fs):
    # 簡化：通過頻域操作模擬
    X = np.fft.fft(signal)
    freqs = np.fft.fftfreq(len(signal), 1/fs)
    
    # 為不同頻率分配不同相位偏移
    phase_shift = np.linspace(delay_high, delay_low, len(freqs)) * 2 * np.pi * freqs
    X_shifted = X * np.exp(-1j * phase_shift)
    
    return np.fft.ifft(X_shifted).real

ecg_nonlinear = apply_nonlinear_phase(ecg, 25/fs, 15/fs, fs)

plt.figure(figsize=(12, 6))
plt.subplot(2, 2, 1)
plt.plot(t, ecg, label='Original')
plt.plot(t, ecg_linear_phase, '--', label='Linear phase (τ=40ms)')
plt.xlabel('Time (s)')
plt.title('Linear Phase: Waveform Preserved')
plt.legend()

plt.subplot(2, 2, 2)
plt.plot(t, ecg, label='Original')
plt.plot(t, ecg_nonlinear, '--', label='Nonlinear phase')
plt.xlabel('Time (s)')
plt.title('Nonlinear Phase: Waveform Distorted')
plt.legend()

plt.subplot(2, 2, 3)
plt.plot(t, ecg_linear_phase - ecg, label='Error (linear)')
plt.subplot(2, 2, 4)
plt.plot(t, ecg_nonlinear - ecg, label='Error (nonlinear)')
plt.xlabel('Time (s)')
plt.title('Distortion Error')
plt.tight_layout()
plt.show()
```

**學者貢獻**：
- Herrmann et al. (1971) — FIR線性相位設計
- Rabiner & Gold (1975) — 數字濾波器理論

**深度問題**：為什麼IIR濾波器不能實現嚴格的線性相位？如何近似線性相位？

---

### 5. 數字濾波器結構：實現的藝術 — Digital Filter Structures: Implementation

**核心概念**：同一個系統函數可以有多種等效的實現結構，不同結構有不同的數值特性、延遲和存儲需求。

**直接形式**：
```latex
直接形式I：
y[n] = b₀x[n] + b₁x[n-1] + ... + b_Mx[n-M]
       - a₁y[n-1] - ... - a_Ny[n-N]

直接形式II（規範形式）：
狀態變量數量 = max(M, N)
乘法和加法數量最少
```

**串聯形式（級聯）**：
```latex
H(z) = H₁(z) · H₂(z) · ... · H_K(z)

每個 H_k(z) 通常是二階（SOS）形式：
H_k(z) = (b₀ + b₁z^{-1} + b₂z^{-2}) / (1 + a₁z^{-1} + a₂z^{-2})

優點：
- 對量化誤差較魯棒
- 可獨立調整每個二階節
- 便於並行實現
```

**並聯形式**：
```latex
H(z) = G + Σ H_k(z)

每個 H_k(z) 也是二階形式

優點：
- 無反饋路徑 → 無振鈴
- 可獨立調整直流增益
```

**轉置形式**：
```latex
將直接形式的所有分支反轉：
- 加法器→分配器
- 延遲器→前向連接

數值特性更好
```

**Python實現**：
```python
import numpy as np
import matplotlib.pyplot as plt
from scipy import signal

# 設計一個四階濾波器
fs = 1000
fc = 100
b, a = signal.butter(4, fc/(fs/2), btype='low')

print("Transfer Function Coefficients:")
print(f"b = {b}")
print(f"a = {a}")

# 轉換為二階節（SOS）
sos = signal.tf2sos(b, a)
print(f"\nSecond-Order Sections (SOS):")
print(sos)

# 使用sosfilt（推薦）
t = np.arange(0, 0.1, 1/fs)
x = np.sin(2*np.pi*50*t) + 0.5*np.sin(2*np.pi*150*t) + 0.3*np.random.randn(len(t))
y = signal.sosfilt(sos, x)

plt.figure(figsize=(12, 6))
plt.subplot(2, 1, 1)
plt.plot(t, x, label='Input')
plt.plot(t, y, label='Filtered')
plt.xlabel('Time (s)')
plt.title('Filtered Signal using SOS')
plt.legend()

# 頻率響應
w, H = signal.sosfreqz(sos)
plt.subplot(2, 1, 2)
plt.plot(w*fs/(2*np.pi), 20*np.log10(np.abs(H)))
plt.xlabel('Frequency (Hz)')
plt.ylabel('Magnitude (dB)')
plt.title('SOS Frequency Response')
plt.grid(True)
plt.show()
```

**學者貢獻**：
- Oppenheim & Schafer (1975) — 數字濾波器結構經典教材
- Jackson (1996) — 量化效應分析

**深度問題**：為什麼串聯形式比直接形式對量化誤差更魯棒？

---

## 問題 2：3 個根本分歧

### 分歧 1：FIR vs IIR — 設計的最終選擇

**FIR支持者觀點**：
- 嚴格線性相位，波形保真
- 總是穩定，無反饋路徑
- 可設計任意頻率響應
- 適合BME波形分析

**IIR支持者觀點**：
- 計算效率高（階數低）
- 可以直接繼承成熟的模擬濾波器設計
- 硬件實現簡單
- 適合嵌入式BME設備

**實際答案**：
- 波形分析（ECG、EEG）→ FIR
- 計算資源受限（可穿戴設備）→ IIR
- 需要陡峭過渡帶 → IIR或高階FIR

---

### 分歧 2：窗口法 vs Parks-McClellan — FIR設計方法

**窗口法支持者**：
- 簡單直觀
- 計算量小
- 可解釋性強
- 缺點：紋波不均勻

**Parks-McClellan支持者**：
- 最優等紋波設計
- 紋波在整個通帶/阻帶均勻分佈
- 對給定規格，階數最低
- 缺點：設計複雜

---

### 分歧 3：Butterworth vs Chebyshev vs 橢圓 — IIR原型選擇

**Butterworth**：
- 最大平坦通帶
- 過渡帶較緩
- 適合：高精度直流測量

**Chebyshev**：
- 通帶或阻帶有等紋波
- 過渡帶較陡
- 適合：需要銳截止的應用

**橢圓**：
- 通帶和阻帶都有等紋波
- 過渡帶最陡
- 適合：過渡帶寬度受限的情況

---

## 問題 3：10 個深度問題

### Q1: 為什麼FIR總是穩定的？

**答案**：
```latex
FIR脈衝響應：h[n] = 0 for n > M

穩定性條件：Σ|h[n]| < ∞

對於FIR：Σ_{n=0}^{M} |h[n]| ≤ (M+1) · max|h[n]| < ∞ ✓

任何有限長序列的絕對和必為有限

對比IIR：h[n]可能不衰減 → Σ|h[n]| = ∞ → 不穩定
```

---

### Q2: 什麼是Gibbs現象？如何緩解？

**答案**：
```latex
Gibbs現象：截斷脈衝響應導致間斷點附近振盪

原因：矩形窗口的頻譜是sinc函數，有無限延伸的旁瓣

緩解方法：
1. 使用緩慢衰減的窗口（Hann、Hamming、Blackman）
2. 增加濾波器階數（振盪區域變窄，但幅度不變）
3. 使用非矩形窗口（減少旁瓣）
4. 預畸變技術
```

---

### Q3: 雙線性變換為什麼需要頻率預畸變？

**答案**：
```latex
雙線性變換：s → (2/T_s)(1-z^{-1})/(1+z^{-1})

映射關係：ω_a = (2/T_s)tan(ω_d T_s/2)

問題：tan函數在高頻快速增長
- ω_d = 0.5ω_s → ω_a = ∞
- 導致數字頻率被「壓縮」到整個模擬頻率範圍

預畸變：ω_a = (2/T_s)tan(ω_d T_s/2)

設計時使用預畸變頻率，確保實際數字頻率符合要求
```

---

### Q4: 如何選擇窗口函數？

**答案**：
```python
# 窗口選擇指南
windows = {
    'rectangular': {'sidelobe': -13, 'mainlobe': 4, 'use': '短序列，高頻分辨率'},
    'hann': {'sidelobe': -31, 'mainlobe': 8, 'use': '一般用途'},
    'hamming': {'sidelobe': -41, 'mainlobe': 8, 'use': '語音處理'},
    'blackman': {'sidelobe': -58, 'mainlobe': 12, 'use': '高阻帶衰減'},
    'kaiser(β=8)': {'sidelobe': -50, 'mainlobe': '可變', 'use': '可調'},
}

# 選擇原則
if need_high_stopband_attenuation:
    use 'blackman' or 'kaiser'
elif need_low_mainlobe_width:
    use 'rectangular'
elif need_tradeoff:
    use 'hamming'
```

---

### Q5: FIR線性相位的代價是什麼？

**答案**：
```latex
線性相位FIR需要偶對稱或奇對稱脈衝響應

代價：
1. 階數增加：相比非線性相位FIR，可能需要2-3倍階數
2. 計算量增加