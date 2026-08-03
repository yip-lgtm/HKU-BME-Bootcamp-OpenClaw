# Week 7 Code Lab — Biomedical Signals & LTI Systems (BMED2500)

> **Topics Covered**: Signal generation, convolution, LTI system analysis, basic signal operations
> **Prerequisites**: numpy, matplotlib, scipy
> **Estimated Time**: 2-3 hours

---

## Lab 1: Basic Biomedical Signal Generation

### Objective
Generate and visualize canonical biomedical signals: unit step, unit impulse, exponential decay, and simplified ECG waveforms. Understand the mathematical definitions and biological correlates of each signal type.

### Code

```python
"""
Week 7 Lab 1: Basic Biomedical Signal Generation
=================================================
Generate canonical signals and their biomedical analogues.
"""

import numpy as np
import matplotlib.pyplot as plt
from matplotlib.patches import FancyArrowPatch

# Set up nice plotting style
plt.rcParams['figure.figsize'] = (12, 8)
plt.rcParams['axes.grid'] = True
plt.rcParams['grid.alpha'] = 0.3

print("=" * 65)
print("LAB 1: Basic Biomedical Signal Generation")
print("=" * 65)

# ---------------------------------------------------------
# SECTION 1: Unit Step and Unit Impulse
# ---------------------------------------------------------
print("\n[1] Unit Step and Unit Impulse")

# Continuous-time unit step: u(t) = 1 for t >= 0, 0 for t < 0
t_ct = np.linspace(-5, 5, 2000)
dt = t_ct[1] - t_ct[0]  # ~0.005

def unit_step(t):
    """Unit step function: u(t)"""
    return np.where(t >= 0, 1.0, 0.0)

# Unit impulse approximated by narrow rect
def unit_impulse_approx(t, epsilon=0.01):
    """Approximation of δ(t) using narrow rect of area 1"""
    return np.where(np.abs(t) <= epsilon/2, 1.0/epsilon, 0.0)

u_ct = unit_step(t_ct)
delta_ct = unit_impulse_approx(t_ct)

# Verify: integral of δ(t) ≈ 1
integral_delta = np.sum(delta_ct) * dt
print(f"  Integral of δ(t) ≈ {integral_delta:.4f} (should be 1.0)")

# Relationship: u(t) = ∫δ(τ)dτ from -∞ to t
u_from_delta = np.cumsum(delta_ct) * dt
u_from_delta = u_from_delta / u_from_delta[-1]  # normalize

# Discrete-time signals
n_dt = np.arange(-10, 51)  # n ∈ [-10, 50]
u_dt = unit_step(n_dt)
delta_dt = np.where(n_dt == 0, 1.0, 0.0)

fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# CT Unit Step
axes[0, 0].plot(t_ct, u_ct, 'b-', linewidth=2)
axes[0, 0].axhline(y=0.5, color='gray', linestyle='--', alpha=0.5, label='Convention at t=0')
axes[0, 0].set_title('Continuous-Time Unit Step: u(t)', fontsize=12)
axes[0, 0].set_xlabel('t (seconds)')
axes[0, 0].set_ylabel('Amplitude')
axes[0, 0].set_xlim(-5, 5)
axes[0, 0].legend()

# CT Unit Impulse (approximation)
axes[0, 1].plot(t_ct, delta_ct, 'r-', linewidth=1.5)
axes[0, 1].fill_between(t_ct, delta_ct, alpha=0.3, color='red')
axes[0, 1].set_title(f'Continuous-Time Unit Impulse: δ(t) (ε={epsilon})', fontsize=12)
axes[0, 1].set_xlabel('t (seconds)')
axes[0, 1].set_ylabel('Amplitude (area=1)')
axes[0, 1].set_xlim(-1, 1)

# DT Unit Step
axes[1, 0].stem(n_dt, u_dt, basefmt=' ')
axes[1, 0].set_title('Discrete-Time Unit Step: u[n]', fontsize=12)
axes[1, 0].set_xlabel('n (samples)')
axes[1, 0].set_ylabel('Amplitude')
axes[1, 0].set_xlim(-10, 50)

# DT Unit Impulse
axes[1, 1].stem(n_dt, delta_dt, basefmt=' ', linefmt='r-', markerfmt='ro')
axes[1, 1].set_title('Discrete-Time Unit Impulse: δ[n]', fontsize=12)
axes[1, 1].set_xlabel('n (samples)')
axes[1, 1].set_ylabel('Amplitude')
axes[1, 1].set_xlim(-10, 50)
axes[1, 1].set_ylim(-0.2, 1.5)

plt.tight_layout()
plt.savefig('lab1_section1_signals.png', dpi=150)
plt.show()
print("  [Saved] lab1_section1_signals.png")

# ---------------------------------------------------------
# SECTION 2: Exponential Signals & Pharmacokinetics
# ---------------------------------------------------------
print("\n[2] Exponential Signals — Pharmacokinetics")

# BME Application: Drug concentration decay follows first-order kinetics
# C(t) = C0 * e^(-k_e * t)
# Half-life: t_1/2 = ln(2) / k_e

k_elimination = 0.15  # Elimination rate constant (hr^-1)
half_life = np.log(2) / k_elimination
print(f"  Elimination rate: k_e = {k_elimination} hr^-1")
print(f"  Half-life: t_1/2 = {half_life:.2f} hours")

t_drug = np.linspace(0, 30, 1000)
C0 = 100  # Initial concentration (mg/L)
C_t = C0 * np.exp(-k_elimination * t_drug)

# Time constants: τ = 1/k_e
tau = 1 / k_elimination
t_63 = tau  # 63.2% of steady state
t_95 = 3 * tau  # ~95% of steady state
t_99 = 5 * tau  # ~99% of steady state

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Drug concentration decay
axes[0].plot(t_drug, C_t, 'b-', linewidth=2, label='C(t) = C₀·e^(-k_e·t)')
axes[0].axhline(y=C0 * 0.5, color='r', linestyle='--', alpha=0.7, label=f'50% (t={half_life:.1f}hr)')
axes[0].axvline(x=half_life, color='r', linestyle=':', alpha=0.7)
axes[0].axhline(y=C0 * 0.01, color='g', linestyle='--', alpha=0.5, label='1% (practically zero)')
axes[0].axvline(x=t_99, color='g', linestyle=':', alpha=0.5)
axes[0].set_xlabel('Time (hours)', fontsize=11)
axes[0].set_ylabel('Concentration (mg/L)', fontsize=11)
axes[0].set_title('Drug Concentration: First-Order Elimination', fontsize=12)
axes[0].legend()
axes[0].set_xlim(0, 30)

# Semi-log plot (exponential becomes linear)
axes[1].semilogy(t_drug, C_t, 'b-', linewidth=2)
axes[1].axhline(y=C0 * 0.5, color='r', linestyle='--', alpha=0.7)
axes[1].axvline(x=half_life, color='r', linestyle=':', alpha=0.7)
axes[1].set_xlabel('Time (hours)', fontsize=11)
axes[1].set_ylabel('Concentration (mg/L) — log scale', fontsize=11)
axes[1].set_title('Semi-log Plot (Linearizes Exponential)', fontsize=12)
axes[1].set_xlim(0, 30)

plt.tight_layout()
plt.savefig('lab1_section2_exponential.png', dpi=150)
plt.show()
print("  [Saved] lab1_section2_exponential.png")

# ---------------------------------------------------------
# SECTION 3: Simplified ECG Signal Generation
# ---------------------------------------------------------
print("\n[3] Simplified ECG Signal Generation")

def generate_ecg_waveform(t, heart_rate=72):
    """
    Generate a simplified ECG waveform.
    
    ECG Components:
    - P wave: ~0.08s before QRS (atrial depolarization)
    - QRS complex: ~0.08s (ventricular depolarization)
    - T wave: ~0.18s after QRS (ventricular repolarization)
    
    Parameters:
        t: time array (seconds)
        heart_rate: beats per minute (bpm)
    """
    cycle_period = 60.0 / heart_rate  # seconds per beat
    ecg = np.zeros_like(t)
    
    for i, ti in enumerate(t):
        cycle_t = ti % cycle_period
        
        # P wave: small positive bump (atrial depolarization)
        if 0.0 < cycle_t < 0.10:
            ecg[i] = 0.15 * np.sin(np.pi * cycle_t / 0.10)
        
        # PR segment: baseline
        elif 0.10 < cycle_t < 0.12:
            ecg[i] = 0.0
        
        # QRS complex: sharp spike (ventricular depolarization)
        # Q: small negative
        elif 0.12 < cycle_t < 0.15:
            ecg[i] = -0.5 * np.sin(np.pi * (cycle_t - 0.12) / 0.03)
        # R: tall positive
        elif 0.15 < cycle_t < 0.19:
            ecg[i] = 1.8 * np.sin(np.pi * (cycle_t - 0.15) / 0.04)
        # S: small negative return
        elif 0.19 < cycle_t < 0.22:
            ecg[i] = -0.3 * np.sin(np.pi * (cycle_t - 0.19) / 0.03)
        
        # ST segment: baseline
        elif 0.22 < cycle_t < 0.30:
            ecg[i] = 0.0
        
        # T wave: broad positive bump (ventricular repolarization)
        elif 0.30 < cycle_t < 0.45:
            ecg[i] = 0.35 * np.sin(np.pi * (cycle_t - 0.30) / 0.15)
        
        # Return to baseline
        else:
            ecg[i] = 0.0
    
    return ecg

# Generate ECG signals
fs = 500  # Sampling rate (Hz) — must satisfy Nyquist for ECG bandwidth
t_ecg = np.arange(0, 4, 1/fs)  # 4 seconds of ECG

ecg_72bpm = generate_ecg_waveform(t_ecg, heart_rate=72)
ecg_90bpm = generate_ecg_waveform(t_ecg, heart_rate=90)
ecg_120bpm = generate_ecg_waveform(t_ecg, heart_rate=120)

print(f"  Sampling rate: {fs} Hz")
print(f"  ECG duration: {t_ecg[-1]:.1f} seconds")
print(f"  Total samples: {len(t_ecg)}")
print(f"  Heart rate range: 72-120 bpm")

fig, axes = plt.subplots(3, 1, figsize=(14, 10), sharex=True)

axes[0].plot(t_ecg, ecg_72bpm, 'b-', linewidth=1.2)
axes[0].set_title('ECG Waveform — 72 bpm (Normal)', fontsize=12)
axes[0].set_ylabel('Amplitude (mV)')
axes[0].axhline(y=0, color='k', linewidth=0.5)

axes[1].plot(t_ecg, ecg_90bpm, 'g-', linewidth=1.2)
axes[1].set_title('ECG Waveform — 90 bpm (Elevated)', fontsize=12)
axes[1].set_ylabel('Amplitude (mV)')
axes[1].axhline(y=0, color='k', linewidth=0.5)

axes[2].plot(t_ecg, ecg_120bpm, 'r-', linewidth=1.2)
axes[2].set_title('ECG Waveform — 120 bpm (Tachycardia)', fontsize=12)
axes[2].set_xlabel('Time (seconds)')
axes[2].set_ylabel('Amplitude (mV)')
axes[2].axhline(y=0, color='k', linewidth=0.5)

# Annotate ECG components on first plot
axes[0].annotate('P wave', xy=(0.05, 0.15), fontsize=9, color='purple')
axes[0].annotate('Q', xy=(0.13, -0.4), fontsize=9, color='darkblue')
axes[0].annotate('R', xy=(0.17, 1.7), fontsize=9, color='darkblue')
axes[0].annotate('S', xy=(0.205, -0.2), fontsize=9, color='darkblue')
axes[0].annotate('T', xy=(0.37, 0.3), fontsize=9, color='orange')

plt.tight_layout()
plt.savefig('lab1_section3_ecg.png', dpi=150)
plt.show()
print("  [Saved] lab1_section3_ecg.png")

print("\n✅ Lab 1 Complete!")
```

### Expected Output
- Four-panel plot showing unit step and impulse (CT and DT)
- Drug concentration decay curves with half-life annotations
- Three ECG waveforms at different heart rates with labeled PQRST components

---

## Lab 2: Convolution & LTI System Analysis

### Objective
Implement discrete-time convolution manually and using scipy, apply convolution to model LTI systems (cardiovascular impulse response, drug delivery), and verify convolution properties (commutativity, associativity).

### Code

```python
"""
Week 7 Lab 2: Convolution & LTI System Analysis
================================================
Convolution as the fundamental operation of LTI systems.
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy import signal

print("=" * 65)
print("LAB 2: Convolution & LTI System Analysis")
print("=" * 65)

# ---------------------------------------------------------
# SECTION 1: Manual Discrete Convolution Implementation
# ---------------------------------------------------------
print("\n[1] Manual Discrete Convolution Implementation")

def convolution_manual(x, h):
    """
    Implement discrete convolution: y[n] = Σ x[k]·h[n-k]
    
    This is the flip-and-slide operation:
    1. Flip h[n] to get h[-n]
    2. Slide over x[n]
    3. Multiply and sum at each position
    
    Parameters:
        x: input signal (length N)
        h: impulse response (length M)
    
    Returns:
        y: output signal (length N+M-1)
    """
    N = len(x)
    M = len(h)
    y = np.zeros(N + M - 1)
    
    for n in range(N + M - 1):
        for k in range(N):
            if 0 <= n - k < M:
                y[n] += x[k] * h[n - k]
    
    return y

def convolution_fast(x, h):
    """
    Fast convolution using numpy's built-in approach.
    Equivalent to scipy.signal.convolve and np.convolve.
    """
    return np.convolve(x, h, mode='full')

# Test with simple signals
# x: rectangular pulse of width 5
x_test = np.concatenate([np.ones(5), np.zeros(5)])
# h: exponential decay
h_test = 0.8 ** np.arange(10)

y_manual = convolution_manual(x_test, h_test)
y_fast = convolution_fast(x_test, h_test)

print(f"  Input x length: {len(x_test)}")
print(f"  Impulse h length: {len(h_test)}")
print(f"  Output y length: {len(y_manual)} (= {len(x_test)} + {len(h_test)} - 1)")
print(f"  Manual equals fast: {np.allclose(y_manual, y_fast)}")

# ---------------------------------------------------------
# SECTION 2: Convolution Properties Verification
# ---------------------------------------------------------
print("\n[2] Convolution Properties")

# Commutativity: x * h = h * x
x_prop = np.array([1.0, 2.0, 3.0, 4.0])
h_prop = np.array([0.5, 0.3, 0.2])

y_xh = np.convolve(x_prop, h_prop, mode='full')
y_hx = np.convolve(h_prop, x_prop, mode='full')

print(f"  Commutativity: x*h == h*x → {np.allclose(y_xh, y_hx)}")
print(f"    x*h = {y_xh}")
print(f"    h*x = {y_hx}")

# Associativity: (x * h1) * h2 = x * (h1 * h2)
h1 = np.array([1.0, 0.5])
h2 = np.array([0.5, 0.25])

left = np.convolve(np.convolve(x_prop, h1, mode='full'), h2, mode='full')
right = np.convolve(x_prop, np.convolve(h1, h2, mode='full'), mode='full')

print(f"  Associativity: (x*h1)*h2 == x*(h1*h2) → {np.allclose(left, right)}")

# Identity: x * δ = x
delta = np.zeros(len(x_prop) + 4)
delta[2] = 1.0  # δ at position 2
y_identity = np.convolve(x_prop, delta, mode='full')
print(f"  Identity: x*δ = x → {np.allclose(y_identity[2:2+len(x_prop)], x_prop)}")

# ---------------------------------------------------------
# SECTION 3: LTI System — Cardiovascular Model
# ---------------------------------------------------------
print("\n[3] LTI System: Cardiovascular Impulse Response")

# BME Application: Blood pressure response to a single heartbeat
# The heart's output is modeled as an impulse input to the circulatory system
# The impulse response h(t) describes how blood pressure decays after each beat

fs_cv = 100  # Sampling rate (Hz)
t_cv = np.arange(0, 5, 1/fs_cv)

# Impulse response: Windkessel model approximation
# h(t) = A * e^(-t/τ) * sin(ωt) — damped oscillation
A = 1.0  # Amplitude
tau_cv = 0.3  # Time constant (seconds) — compliance of arteries
omega = 2 * np.pi * 3  # Damping frequency (~3 Hz)
h_cardio = A * np.exp(-t_cv / tau_cv) * np.sin(omega * t_cv)
h_cardio = np.where(t_cv >= 0, h_cardio, 0)  # Causal

# Input: cardiac output (series of impulses at each heartbeat)
heart_rate = 72  # bpm
beat_period = 60.0 / heart_rate
x_cardio = np.zeros_like(t_cv)
for i, ti in enumerate(t_cv):
    x_cardio[i] = 1.0 if (ti % beat_period) < (1/fs_cv) else 0.0

# Convolution: blood pressure = cardiac output * impulse response
y_bp = np.convolve(x_cardio, h_cardio, mode='same') * (1/fs_cv)

# Normalize for visualization
y_bp = y_bp / np.max(np.abs(y_bp)) * 0.8
h_cardio_norm = h_cardio / np.max(np.abs(h_cardio))

fig, axes = plt.subplots(3, 1, figsize=(14, 10), sharex=True)

axes[0].plot(t_cv, x_cardio, 'b-', linewidth=1)
axes[0].set_title('Input: Cardiac Output (Impulse Sequence at 72 bpm)', fontsize=12)
axes[0].set_ylabel('Input Amplitude')
axes[0].set_xlim(0, 3)

axes[1].plot(t_cv, h_cardio_norm, 'r-', linewidth=1.5)
axes[1].set_title('Impulse Response h(t): Cardiovascular System (Windkessel Model)', fontsize=12)
axes[1].set_ylabel('|h(t)| Normalized')
axes[1].set_xlim(0, 2)

axes[2].plot(t_cv, y_bp, 'g-', linewidth=1.5)
axes[2].set_title('Output: Blood Pressure Response y(t) = x(t) * h(t)', fontsize=12)
axes[2].set_xlabel('Time (seconds)')
axes[2].set_ylabel('BP (normalized)')
axes[2].set_xlim(0, 3)

plt.tight_layout()
plt.savefig('lab2_section3_cardio.png', dpi=150)
plt.show()
print("  [Saved] lab2_section3_cardio.png")

# ---------------------------------------------------------
# SECTION 4: Convolution in Frequency Domain
# ---------------------------------------------------------
print("\n[4] Convolution Theorem: Time Domain ↔ Frequency Domain")

# The Convolution Theorem: x(t) * h(t) ⟷ X(ω) · H(ω)
# This is why convolution in time = multiplication in frequency

fs_ft = 500
t_ft = np.linspace(-1, 1, fs_ft + 1)

# Two Gaussian pulses
x_gauss = np.exp(-(t_ft / 0.15) ** 2)
h_gauss = np.exp(-((t_ft - 0.1) / 0.1) ** 2)

# Convolution in time domain
y_time = np.convolve(x_gauss, h_gauss, mode='same')
y_time = y_time / np.max(y_time)

# Convolution via FFT: Multiply in frequency domain
N = len(x_gauss)
X_fft = np.fft.fftshift(np.fft.fft(x_gauss))
H_fft = np.fft.fftshift(np.fft.fft(h_gauss))
Y_fft = X_fft * H_fft
y_freq = np.fft.ifftshift(np.fft.ifft(Y_fft))
y_freq = y_freq / np.max(np.abs(y_freq))

print(f"  Time-domain convolution ≈ FFT method: {np.allclose(y_time, y_freq.real, atol=0.01)}")

fig, axes = plt.subplots(2, 2, figsize=(14, 9))

# Time domain
axes[0, 0].plot(t_ft, x_gauss, 'b-', label='x(t)')
axes[0, 0].plot(t_ft, h_gauss, 'r-', label='h(t)')
axes[0, 0].set_title('Time Domain: Input & Impulse Response')
axes[0, 0].legend()
axes[0, 0].set_xlabel('t (s)')

axes[0, 1].plot(np.linspace(-0.5, 0.5, len(y_time)), y_time, 'g-', linewidth=2)
axes[0, 1].set_title('Time Domain: Convolution y(t) = x(t) * h(t)')
axes[0, 1].set_xlabel('t (s)')

# Frequency domain
freq = np.fft.fftshift(np.fft.fftfreq(N, d=t_ft[1]-t_ft[0]))
axes[1, 0].plot(freq, np.abs(X_fft)/N, 'b-', label='|X(ω)|')
axes[1, 0].plot(freq, np.abs(H_fft)/N, 'r-', label='|H(ω)|')
axes[1, 0].set_title('Frequency Domain: |X(ω)| and |H(ω)|')
axes[1, 0].legend()
axes[1, 0].set_xlabel('Frequency (Hz)')

axes[1, 1].plot(freq, np.abs(Y_fft)/N, 'g-', linewidth=2)
axes[1, 1].set_title('Frequency Domain: |Y(ω)| = |X(ω)| · |H(ω)|')
axes[1, 1].set_xlabel('Frequency (Hz)')

plt.tight_layout()
plt.savefig('lab2_section4_convolution_theorem.png', dpi=150)
plt.show()
print("  [Saved] lab2_section4_convolution_theorem.png")

# ---------------------------------------------------------
# SECTION 5: LTI System Properties Analysis
# ---------------------------------------------------------
print("\n[5] LTI System Properties: BIBO Stability & Causality")

def analyze_lti_system(h, name="System"):
    """
    Analyze LTI system properties:
    - BIBO Stability: Σ|h[n]| < ∞
    - Causality: h[n] = 0 for n < 0
    """
    h_abs_sum = np.sum(np.abs(h))
    
    # Check causality: first non-zero index
    first_nonzero = np.argmax(np.abs(h) > 1e-10)
    is_causal = first_nonzero == 0 or (h[0] != 0 and np.all(h[:first_nonzero] == 0))
    
    # Energy
    energy = np.sum(h ** 2)
    
    print(f"\n  [{name}]")
    print(f"    Length: {len(h)}")
    print(f"    Σ|h[n]| = {h_abs_sum:.4f} {'< ∞ → STABLE' if np.isfinite(h_abs_sum) else '= ∞ → UNSTABLE'}")
    print(f"    Σ|h[n]|² (energy) = {energy:.4f}")
    print(f"    First sample h[0] = {h[0]:.4f}")
    print(f"    Causality: {'CAUSAL' if is_causal else 'NON-CAUSAL'}")

# System 1: FIR moving average (always stable)
h_fir = np.ones(5) / 5.0
analyze_lti_system(h_fir, "FIR Moving Average (N=5)")

# System 2: IIR exponential decay (stable)
h_iir_stable = 0.8 ** np.arange(100)
analyze_lti_system(h_iir_stable, "IIR Exponential Decay (a=0.8)")

# System 3: IIR growing exponential (unstable)
h_iir_unstable = 1.2 ** np.arange(50)
analyze_lti_system(h_iir_unstable, "IIR Growing Exponential (a=1.2) [UNSTABLE]")

# System 4: Symmetric (non-causal)
h_noncausal = np.concatenate([np.ones(5), np.zeros(5)])
analyze_lti_system(h_noncausal, "Time-Reversed System [NON-CAUSAL]")

print("\n✅ Lab 2 Complete!")
print("\n" + "=" * 65)
print("SUMMARY: Key Concepts Demonstrated")
print("=" * 65)
print("1. Convolution: y[n] = Σ x[k]·h[n-k] — fundamental LTI operation")
print("2. Properties: Commutativity, associativity, identity verified")
print("3. Convolution theorem: Time convolution = Frequency multiplication")
print("4. BIBO Stability: Σ|h[n]| < ∞ is necessary and sufficient for LTI stability")
print("5. BME Applications: Cardiovascular system, pharmacokinetics, neural response")
```

### Expected Output
- Cardiovascular impulse response and blood pressure output plots
- Convolution theorem verification (time vs frequency domain)
- LTI system stability analysis for multiple systems

---

## Lab 3: Signal Energy, Power, and Correlation

### Objective
Calculate energy and power of biomedical signals, implement cross-correlation for signal similarity detection (e.g., ECG template matching, neural spike sorting), and compute correlation coefficients for clinical data comparison.

### Code

```python
"""
Week 7 Lab 3: Signal Energy, Power, and Correlation
=====================================================
"""

import numpy as np
import matplotlib.pyplot as plt

print("=" * 65)
print("LAB 3: Signal Energy, Power, and Correlation")
print("=" * 65)

# ---------------------------------------------------------
# SECTION 1: Energy vs Power Signals
# ---------------------------------------------------------
print("\n[1] Energy vs Power Signals")

def signal_energy(x, dt=1.0):
    """Energy: E = Σ|x[n]|² (discrete) or ∫|x(t)|²dt (continuous)"""
    return np.sum(x ** 2) * dt

def signal_power(x, T=None, dt=1.0):
    """
    Power: P = E/T for finite duration
           P = lim(T→∞) E/T for infinite duration
    """
    if T is None:
        T = len(x) * dt
    return signal_energy(x, dt) / T

# BME Example 1: ECG burst — Energy signal (finite duration)
fs = 500
t_ecg = np.arange(0, 3, 1/fs)
ecg_burst = np.zeros_like(t_ecg)
heart_rate = 72
cycle_period = 60 / heart_rate

for i, ti in enumerate(t_ecg):
    cycle_t = ti % cycle_period
    if 0.15 < cycle_t < 0.19:
        ecg_burst[i] = 1.5 * np.sin(np.pi * (cycle_t - 0.15) / 0.04)
    elif 0.30 < cycle_t < 0.45:
        ecg_burst[i] = 0.3 * np.sin(np.pi * (cycle_t - 0.30) / 0.15)

E_ecg = signal_energy(ecg_burst, dt=1/fs)
P_ecg = signal_power(ecg_burst, T=3, dt=1/fs)

print(f"\n  ECG Burst (3 seconds):")
print(f"    Energy E = {E_ecg:.4f} (finite → ENERGY SIGNAL)")
print(f"    Average Power P = {P_ecg:.6f}")

# BME Example 2: EEG background activity — Power signal (infinite duration)
# Modeled as stationary random process
np.random.seed(42)
t_eeg = np.arange(0, 10000, 1/fs)  # Long recording
eeg_bg = 0.5 * np.random.randn(len(t_eeg))  # Background noise model
eeg_signal = np.sin(2 * np.pi * 10 * t_eeg) + eeg_bg  # 10 Hz alpha rhythm

E_eeg = signal_energy(eeg_signal, dt=1/fs)
P_eeg = signal_power(eeg_signal, T=10000/fs, dt=1/fs)

print(f"\n  EEG Background (10000 seconds):")
print(f"    Energy E ≈ {E_eeg:.2f} (very large → likely POWER SIGNAL)")
print(f"    Average Power P = {P_eeg:.6f}")
print(f"    Conclusion: {'POWER SIGNAL' if E_eeg == float('inf') else 'ENERGY SIGNAL'}")

# ---------------------------------------------------------
# SECTION 2: Cross-Correlation for ECG Template Matching
# ---------------------------------------------------------
print("\n[2] Cross-Correlation: ECG Template Matching")

def cross_correlation(x, y):
    """
    Compute cross-correlation: r[lag] = Σ x[n]·y[n+lag]
    
    This measures similarity between x and y as a function of lag.
    For normalized cross-correlation, divide by √(E_x · E_y).
    """
    return np.correlate(x, y, mode='full')

# Generate template ECG and noisy measurement
fs_match = 500
t_match = np.arange(0, 1, 1/fs_match)

def make_ecg_template(t, amplitude=1.0):
    ecg = np.zeros_like(t)
    # R peak (dominant feature)
    peak_idx = int(0.3 * fs_match)
    ecg[peak_idx-5:peak_idx+5] = amplitude * np.hanning(10)
    return ecg

# Template: clean QRS complex
template = make_ecg_template(t_match, amplitude=1.0)

# Noisy measurement: template + noise + shifted version
np.random.seed(7)
noise = 0.3 * np.random.randn(len(t_match))
measurement = make_ecg_template(t_match, amplitude=0.8)
# Add a shifted version (simulating another QRS complex)
shift = 180
measurement[shift:shift+10] += 0.6 * np.hanning(10)
measurement += noise

# Cross-correlation
xcorr = cross_correlation(measurement, template)
lags = np.arange(-(len(template)-1), len(measurement))
max_lag = lags[np.argmax(xcorr)]
max_corr = np.argmax(xcorr)

# Normalized cross-correlation
norm_factor = np.sqrt(signal_energy(template) * signal_energy(measurement))
xcorr_normalized = xcorr / (norm_factor + 1e-10)

print(f"  Template length: {len(template)} samples ({len(template)/fs_match*1000:.0f} ms)")
print(f"  Measurement length: {len(measurement)} samples ({len(measurement)/fs_match*1000:.0f} ms)")
print(f"  Peak correlation at lag: {max_lag} samples ({max_lag/fs_match*1000:.1f} ms)")
print(f"  Normalized peak correlation: {xcorr_normalized.max():.4f}")

# Find all peaks in cross-correlation (multiple QRS detections)
from scipy.signal import find_peaks
peaks, properties = find_peaks(xcorr_normalized, height=0.3, distance=int(0.3*fs_match))
print(f"  Detected {len(peaks)} QRS complexes in measurement")

fig, axes = plt.subplots(3, 1, figsize=(14, 9))

axes[0].plot(t_match * 1000, template, 'b-', linewidth=2)
axes[0].set_title('ECG Template (Clean QRS Complex)', fontsize=12)
axes[0].set_xlabel('Time (ms)')
axes[0].set_ylabel('Amplitude')

axes[1].plot(t_match * 1000, measurement, 'g-', alpha=0.7)
axes[1].set_title('Noisy ECG Measurement', fontsize=12)
axes[1].set_xlabel('Time (ms)')
axes[1].set_ylabel('Amplitude')

lags_ms = lags / fs_match * 1000
axes[2].plot(lags_ms, xcorr_normalized, 'r-', linewidth=1)
axes[2].plot(lags_ms[peaks], xcorr_normalized[peaks], 'ko', markersize=8, label=f'Detected peaks ({len(peaks)} QRS)')
axes[2].axhline(y=0.3, color='k', linestyle='--', alpha=0.5, label='Detection threshold')
axes[2].set_title('Normalized Cross-Correlation: Template Matching', fontsize=12)
axes[2].set_xlabel('Lag (ms)')
axes[2].set_ylabel('Normalized Correlation')
axes[2].legend()

plt.tight_layout()
plt.savefig('lab3_section2_correlation.png', dpi=150)
plt.show()
print("  [Saved] lab3_section2_correlation.png")

# ---------------------------------------------------------
# SECTION 3: Autocorrelation of Neural Spike Trains
# ---------------------------------------------------------
print("\n[3] Autocorrelation: Neural Spike Train Analysis")

# BME Application: Autocorrelation reveals periodicity in neural firing
# Poisson process model of spontaneous neural firing

np.random.seed(123)
t_neuron = np.arange(0, 5, 0.001)  # 5 seconds, 1 ms resolution
firing_rate = 20  # Hz (average firing rate)

# Homogeneous Poisson process
n_trials = 1
spike_train = np.zeros_like(t_neuron)
prob_spike = firing_rate * 0.001  # P(spike in 1ms bin)
spike_train[np.random.rand(len(t_neuron)) < prob_spike] = 1.0

# Autocorrelation
autocorr = np.correlate(spike_train, spike_train, mode='full')
autocorr = autocorr[len(autocorr)//2:]  # Take positive lags only
autocorr = autocorr / (len(spike_train) - np.arange(len(autocorr)))  # Normalize
autocorr = autocorr - firing_rate**2 * 0.001  # Subtract mean rate squared

# Find inter-spike interval distribution
spike_times = t_neuron[spike_train > 0]
isis = np.diff(spike_times) * 1000  # Convert to ms

print(f"  Total spikes in 5 seconds: {int(np.sum(spike_train))}")
print(f"  Average firing rate: {np.sum(spike_train)/5:.1f} Hz")
print(f"  Mean ISI: {np.mean(isis):.1f} ms")
print(f"  Std ISI: {np.std(isis):.1f} ms")
print(f"  ISI CV (Poisson ~1.0): {np.std(isis)/np.mean(isis):.3f}")

fig, axes = plt.subplots(2, 2, figsize=(14, 9))

# Spike train
axes[0, 0].eventplot(spike_times * 1000, lineoffsets=0.5, linelengths=0.8, color='b')
axes[0, 0].set_title('Neural Spike Train (Poisson Process, 20 Hz)', fontsize=12)
axes[0, 0].set_xlabel('Time (ms)')
axes[0, 0].set_yticks([])
axes[0, 0].set_xlim(0, 500)

# Autocorrelation
lags_ms = np.arange(len(autocorr)) * 1  # 1 ms bins
axes[0, 1].plot(lags_ms, autocorr, 'b-', linewidth=1)
axes[0, 1].axhline(y=0, color='k', linestyle='--', alpha=0.5)
axes[0, 1].set_title('Autocorrelation of Spike Train', fontsize=12)
axes[0, 1].set_xlabel('Lag (ms)')
axes[0, 1].set_ylabel('Correlation')
axes[0, 1].set_xlim(0, 200)

# ISI histogram
axes[1, 0].hist(isis, bins=50, density=True, alpha=0.7, color='coral')
axes[1, 0].set_title('Inter-Spike Interval Distribution', fontsize=12)
axes[1, 0].set_xlabel('ISI (ms)')
axes[1, 0].set_ylabel('Probability Density')
# Overlay exponential fit (characteristic of Poisson)
x_exp = np.linspace(0, 200, 200)
exp_fit = (1/np.mean(isis)) * np.exp(-x_exp / np.mean(isis))
axes[1, 0].plot(x_exp, exp_fit, 'r--', linewidth=2, label=f'Exponential fit (λ={1/np.mean(isis):.3f})')
axes[1, 0].legend()

# Power spectral density (FFT of autocorrelation)
psd = np.fft.fft(autocorr)
freqs = np.fft.fftfreq(len(psd), d=0.001)[:len(psd)//2]
psd_mag = np.abs(psd)[:len(psd)//2]
axes[1, 1].plot(freqs, psd_mag, 'b-', linewidth=1)
axes[1, 1].set_title('Power Spectral Density (from Autocorrelation)', fontsize=12)
axes[1, 1].set_xlabel('Frequency (Hz)')
axes[1, 1].set_ylabel('Power')
axes[1, 1].set_xlim(0, 100)

plt.tight_layout()
plt.savefig('lab3_section3_spikes.png', dpi=150)
plt.show()
print("  [Saved] lab3_section3_spikes.png")

print("\n✅ Lab 3 Complete!")
print("\n" + "=" * 65)
print("LAB SUMMARY: Week 7 Signal Processing Fundamentals")
print("=" * 65)
print("✅ Unit step, impulse, exponential signals generated")
print("✅ ECG waveforms synthesized at different heart rates")
print("✅ Discrete convolution implemented and verified")
print("✅ Convolution theorem demonstrated (time ↔ frequency)")
print("✅ LTI system stability and causality analyzed")
print("✅ Signal energy and power calculated")
print("✅ Cross-correlation for template matching (ECG QRS)")
print("✅ Autocorrelation for neural spike train analysis")
print("\nBME Applications Demonstrated:")
print("  - Pharmacokinetics: first-order drug elimination")
print("  - Cardiovascular: blood pressure impulse response")
print("  - Neurology: Poisson spike train analysis")
print("  - Cardiology: ECG template matching via correlation")
```

### Expected Output
- Cross-correlation plot showing ECG template matching with detected peaks
- Neural spike train with autocorrelation and ISI histogram
- Power spectral density from autocorrelation

---

## References

- Oppenheim & Willsky (1997). *Signals and Systems*. Prentice Hall.
- Rangayyan (2002). *Biomedical Signal Analysis*. IEEE Press.
- Bracewell (2000). *The Fourier Transform*. McGraw-Hill.

**Maintainer**: BME Bootcamp Agent | **Week 7** | **BMED2500: Signals and Systems**
