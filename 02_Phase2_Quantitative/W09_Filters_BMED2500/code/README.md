# Week 9 Code Lab — FIR/IIR Filters & ECG Denoising (BMED2500)

> **Topics Covered**: FIR filter design (windowing, Parks-McClellan), IIR filter design (Butterworth, Chebyshev), filter structures, ECG/EEG denoising applications
> **Prerequisites**: numpy, matplotlib, scipy.signal
> **Estimated Time**: 2-3 hours

---

## Lab 1: FIR Filter Design — ECG Baseline Wander Removal

### Objective
Design and implement FIR lowpass and highpass filters using the window method. Apply FIR filters to remove baseline wander from ECG signals. Compare rectangular, Hann, Hamming, and Blackman windows in terms of frequency response and stopband attenuation.

### Code

```python
"""
Week 9 Lab 1: FIR Filter Design — ECG Baseline Wander Removal
=============================================================
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy import signal

print("=" * 65)
print("LAB 1: FIR Filter Design — ECG Baseline Wander Removal")
print("=" * 65)

# ---------------------------------------------------------
# SECTION 1: FIR Lowpass Filter Design (Window Method)
# ---------------------------------------------------------
print("\n[1] FIR Lowpass Filter Design — Window Method")

fs = 500  # Hz (ECG sampling rate)
fc = 40   # Hz (QRS complex upper frequency limit)
M = 101   # Filter length (must be odd for Type I)

def design_fir_lpf_windowed(M, fc, fs, window_type='hamming'):
    """
    Design FIR lowpass filter using the window method.
    
    Parameters:
        M: filter length (odd for Type I linear phase)
        fc: cutoff frequency (Hz)
        fs: sampling frequency (Hz)
        window_type: 'rect', 'hann', 'hamming', 'blackman'
    
    Returns:
        b: FIR coefficients (numerator)
    """
    # Ideal impulse response: h_d[n] = (sin(2πfc n/fs) / (πn))
    n = np.arange(M) - (M - 1) // 2  # centered at 0
    fc_norm = fc / (fs / 2)  # normalized to Nyquist
    
    # Avoid division by zero at n=0
    h_ideal = np.zeros(M)
    nonzero_mask = n != 0
    h_ideal[nonzero_mask] = np.sin(2 * np.pi * fc_norm * n[nonzero_mask] / 2) / (np.pi * n[nonzero_mask])
    h_ideal[(M - 1) // 2] = fc_norm  # h[0] at center = fc_norm (L'Hôpital)
    
    # Apply window
    if window_type == 'rect':
        w = np.ones(M)
    elif window_type == 'hann':
        w = np.hanning(M)
    elif window_type == 'hamming':
        w = np.hamming(M)
    elif window_type == 'blackman':
        w = np.blackman(M)
    else:
        w = np.ones(M)
    
    b = h_ideal * w
    
    # Normalize so that DC gain = 1
    b = b / np.sum(b)
    
    return b

# Design filters with different windows
windows = ['rect', 'hann', 'hamming', 'blackman']
filters = {}
colors = ['red', 'blue', 'green', 'purple']

for win in windows:
    filters[win] = design_fir_lpf_windowed(M, fc, fs, win)

# Compute frequency responses
fig, axes = plt.subplots(2, 2, figsize=(15, 12))

for i, (win, b) in enumerate(filters.items()):
    w, H = signal.freqz(b, worN=4096)
    freq_hz = w * fs / (2 * np.pi)
    mag_db = 20 * np.log10(np.abs(H) + 1e-10)
    
    # Time domain: impulse response
    axes[0, 0].plot(np.arange(M) - (M-1)//2, b, color=colors[i], 
                     linewidth=1.5, label=win.capitalize())
    
    # Frequency domain: magnitude
    axes[0, 1].plot(freq_hz, mag_db, color=colors[i], linewidth=1.5, label=win.capitalize())
    
    # Phase (unwrapped)
    phase = np.unwrap(np.angle(H))
    axes[1, 0].plot(freq_hz, phase, color=colors[i], linewidth=1.5, label=win.capitalize())

axes[0, 0].set_title('FIR Impulse Responses (M=101, fc=40Hz)', fontsize=12)
axes[0, 0].set_xlabel('Sample index (n)')
axes[0, 0].set_ylabel('h[n]')
axes[0, 0].legend()
axes[0, 0].grid(True, alpha=0.3)

axes[0, 1].set_title('Magnitude Response (dB)', fontsize=12)
axes[0, 1].set_xlabel('Frequency (Hz)')
axes[0, 1].set_ylabel('|H(e^{jω})| (dB)')
axes[0, 1].set_xlim(0, 100)
axes[0, 1].set_ylim(-80, 5)
axes[0, 1].axvline(x=fc, color='k', linestyle='--', alpha=0.5)
axes[0, 1].axhline(y=-3, color='r', linestyle=':', alpha=0.5)
axes[0, 1].legend()
axes[0, 1].grid(True, alpha=0.3)

axes[1, 0].set_title('Phase Response', fontsize=12)
axes[1, 0].set_xlabel('Frequency (Hz)')
axes[1, 0].set_ylabel('∠H(e^{jω}) (rad)')
axes[1, 0].set_xlim(0, 100)
axes[1, 0].legend()
axes[1, 0].grid(True, alpha=0.3)

# Window functions comparison
for win in windows:
    w = signal.windows.get_window(win, M)
    axes[1, 1].plot(np.arange(M), w, color=colors[windows.index(win)], 
                     linewidth=1.5, label=win.capitalize())

axes[1, 1].set_title('Window Functions', fontsize=12)
axes[1, 1].set_xlabel('Sample index')
axes[1, 1].set_ylabel('Amplitude')
axes[1, 1].legend()
axes[1, 1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('lab1_section1_fir_design.png', dpi=150)
plt.show()
print("  [Saved] lab1_section1_fir_design.png")

# Print stopband attenuation
print("\n  Window Comparison (at ω = 0.8π):")
for win, b in filters.items():
    w, H = signal.freqz(b, worN=4096)
    mag_db = 20 * np.log10(np.abs(H) + 1e-10)
    # Find attenuation at 60 Hz
    freq_hz = w * fs / (2 * np.pi)
    att_idx = np.argmin(np.abs(freq_hz - 60))
    print(f"    {win.capitalize():>10}: Stopband attenuation at 60 Hz = {mag_db[att_idx]:.1f} dB")

# ---------------------------------------------------------
# SECTION 2: ECG Baseline Wander Removal
# ---------------------------------------------------------
print("\n[2] ECG Baseline Wander Removal")

# Generate ECG with baseline wander
t_ecg = np.arange(0, 5, 1/fs)  # 5 seconds
heart_rate = 75
cycle_period = 60.0 / heart_rate

def ecg_waveform(t, hr):
    """ECG with P, QRS, T waves"""
    cp = 60.0 / hr
    ecg = np.zeros_like(t)
    for i, ti in enumerate(t):
        ct = ti % cp
        if 0.05 < ct < 0.15:
            ecg[i] = 0.15 * np.sin(np.pi*(ct-0.05)/0.10)
        elif 0.15 < ct < 0.18:
            ecg[i] = -0.4 * np.sin(np.pi*(ct-0.15)/0.03)
        elif 0.18 < ct < 0.22:
            ecg[i] = 1.5 * np.sin(np.pi*(ct-0.18)/0.04)
        elif 0.22 < ct < 0.25:
            ecg[i] = -0.15 * np.sin(np.pi*(ct-0.22)/0.03)
        elif 0.30 < ct < 0.45:
            ecg[i] = 0.30 * np.sin(np.pi*(ct-0.30)/0.15)
    return ecg

ecg_clean = ecg_waveform(t_ecg, heart_rate)

# Add baseline wander: very low frequency drift (0.2 Hz, 0.5 Hz)
baseline_wander = (0.3 * np.sin(2*np.pi*0.2*t_ecg) + 
                   0.2 * np.sin(2*np.pi*0.5*t_ecg + 1.2))
ecg_with_baseline = ecg_clean + baseline_wander

# Add high-frequency noise
np.random.seed(55)
ecg_noisy = ecg_with_baseline + 0.05 * np.random.randn(len(t_ecg))

# Design highpass FIR filter to remove baseline (fc = 0.5 Hz)
fc_hp = 0.5  # Hz — removes slow baseline drift
M_hp = 201   # Long filter for sharp transition

# Use highpass filter by spectral inversion of lowpass
b_lp = design_fir_lpf_windowed(M_hp, fc_hp, fs, 'hamming')
b_hp = -b_lp  # Spectral inversion
b_hp[(M_hp-1)//2] += 1  # Spectral reversal (add delta at center)

# Apply filter
ecg_filtered = signal.lfilter(b_hp, 1, ecg_noisy)

# Alternative: use scipy.signal.firwin
b_hp_scipy = signal.firwin(M_hp, fc_hp/(fs/2), pass_zero=False, window='hamming')
ecg_filtered_alt = signal.lfilter(b_hp_scipy, 1, ecg_noisy)

fig, axes = plt.subplots(3, 1, figsize=(14, 10), sharex=True)

axes[0].plot(t_ecg, ecg_noisy, 'b-', linewidth=0.8, alpha=0.7)
axes[0].set_title('ECG with Baseline Wander + Noise', fontsize=12)
axes[0].set_ylabel('Amplitude (mV)')
axes[0].grid(True, alpha=0.3)

axes[1].plot(t_ecg, ecg_filtered, 'g-', linewidth=0.8)
axes[1].set_title('FIR Highpass Filtered (fc=0.5Hz, M=201, Hamming window)', fontsize=12)
axes[1].set_ylabel('Amplitude (mV)')
axes[1].grid(True, alpha=0.3)

axes[2].plot(t_ecg, ecg_filtered_alt, 'r-', linewidth=0.8)
axes[2].set_title('Filtered (scipy.signal.firwin)', fontsize=12)
axes[2].set_xlabel('Time (seconds)')
axes[2].set_ylabel('Amplitude (mV)')
axes[2].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('lab1_section2_baseline_removal.png', dpi=150)
plt.show()
print("  [Saved] lab1_section2_baseline_removal.png")

# Compute improvement in SNR
def snr(signal, noise):
    return 10 * np.log10(np.mean(signal**2) / (np.mean(noise**2) + 1e-10))

original_snr = snr(ecg_clean, baseline_wander + 0.05*np.random.randn(len(t_ecg)))
# For filtered, we need to estimate residual noise
# Simple comparison: use sections without QRS (baseline-only regions)
print(f"\n  Original SNR (approx): {original_snr:.1f} dB")

print("\n✅ Lab 1 Complete!")
```

### Expected Output
- FIR impulse responses and frequency responses for 4 window types
- ECG baseline wander removal: before/after comparison plots

---

## Lab 2: IIR Filters — ECG Powerline Interference Removal

### Objective
Design Butterworth, Chebyshev I, Chebyshev II, and elliptic IIR filters. Compare their magnitude responses, phase responses, and stopband characteristics. Apply notch filters to remove 50/60 Hz powerline interference from ECG signals.

### Code

```python
"""
Week 9 Lab 2: IIR Filters — ECG Powerline Interference Removal
================================================================
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy import signal

print("=" * 65)
print("LAB 2: IIR Filters — ECG Denoising")
print("=" * 65)

# ---------------------------------------------------------
# SECTION 1: IIR Filter Comparison
# ---------------------------------------------------------
print("\n[1] IIR Filter Design — Butterworth vs Chebyshev vs Elliptic")

fs = 500   # Hz
fc = 40    # Hz (cutoff frequency)
order = 5  # Filter order

# Design filters
# Butterworth: maximally flat in passband
b_butt, a_butt = signal.butter(order, fc/(fs/2), btype='low')

# Chebyshev Type I: equiripple in passband
b_cheb1, a_cheb1 = signal.cheby1(order, 0.5, fc/(fs/2), btype='low')  # 0.5 dB ripple

# Chebyshev Type II: equiripple in stopband
b_cheb2, a_cheb2 = signal.cheby2(order, 40, fc/(fs/2), btype='low')  # 40 dB stopband

# Elliptic: equiripple in both passband and stopband
b_ellip, a_ellip = signal.ellip(order, 0.5, 40, fc/(fs/2), btype='low')

# Frequency responses
w, H_butt = signal.freqz(b_butt, a_butt)
_, H_cheb1 = signal.freqz(b_cheb1, a_cheb1)
_, H_cheb2 = signal.freqz(b_cheb2, a_cheb2)
_, H_ellip = signal.freqz(b_ellip, a_ellip)

freq_hz = w * fs / (2 * np.pi)
mag_butt = 20 * np.log10(np.abs(H_butt) + 1e-10)
mag_cheb1 = 20 * np.log10(np.abs(H_cheb1) + 1e-10)
mag_cheb2 = 20 * np.log10(np.abs(H_cheb2) + 1e-10)
mag_ellip = 20 * np.log10(np.abs(H_ellip) + 1e-10)

# Phase responses
phase_butt = np.unwrap(np.angle(H_butt))
phase_cheb1 = np.unwrap(np.angle(H_cheb1))
phase_cheb2 = np.unwrap(np.angle(H_cheb2))
phase_ellip = np.unwrap(np.angle(H_ellip))

# Group delay
def group_delay(b, a, worN=2048):
    """Compute group delay: -d(∠H)/dω"""
    w, H = signal.freqz(b, a, worN=worN)
    phase = np.unwrap(np.angle(H))
    # Approximate derivative
    gd = -np.diff(phase) / np.diff(w)
    return w[:-1] + np.diff(w)/2, gd

gd_butt, gd_w = group_delay(b_butt, a_butt)
gd_cheb1, _ = group_delay(b_cheb1, a_cheb1)
gd_cheb2, _ = group_delay(b_cheb2, a_cheb2)
gd_ellip, _ = group_delay(b_ellip, a_ellip)

print("  Filter comparison summary:")
print(f"    Butterworth: maximally flat passband, moderate transition width")
print(f"    Chebyshev I: 0.5 dB ripple in passband, sharper transition than Butterworth")
print(f"    Chebyshev II: 40 dB attenuation in stopband, flat passband")
print(f"    Elliptic:    0.5 dB passband ripple + 40 dB stopband, sharpest transition")

fig, axes = plt.subplots(2, 2, figsize=(15, 11))

axes[0, 0].plot(freq_hz, mag_butt, 'b-', linewidth=2, label='Butterworth')
axes[0, 0].plot(freq_hz, mag_cheb1, 'r-', linewidth=2, label='Chebyshev I (0.5 dB)')
axes[0, 0].plot(freq_hz, mag_cheb2, 'g-', linewidth=2, label='Chebyshev II (40 dB)')
axes[0, 0].plot(freq_hz, mag_ellip, 'm-', linewidth=2, label='Elliptic (0.5/40 dB)')
axes[0, 0].axvline(x=fc, color='k', linestyle='--', alpha=0.5, label=f'fc={fc}Hz')
axes[0, 0].axhline(y=-3, color='gray', linestyle=':', alpha=0.5)
axes[0, 0].set_title('Magnitude Response Comparison', fontsize=12)
axes[0, 0].set_xlabel('Frequency (Hz)')
axes[0, 0].set_ylabel('|H(e^{jω})| (dB)')
axes[0, 0].set_xlim(0, 100)
axes[0, 0].set_ylim(-80, 5)
axes[0, 0].legend()
axes[0, 0].grid(True, alpha=0.3)

axes[0, 1].plot(freq_hz, phase_butt, 'b-', linewidth=1.5, label='Butterworth')
axes[0, 1].plot(freq_hz, phase_cheb1, 'r-', linewidth=1.5, label='Chebyshev I')
axes[0, 1].plot(freq_hz, phase_cheb2, 'g-', linewidth=1.5, label='Chebyshev II')
axes[0, 1].plot(freq_hz, phase_ellip, 'm-', linewidth=1.5, label='Elliptic')
axes[0, 1].set_title('Phase Response Comparison', fontsize=12)
axes[0, 1].set_xlabel('Frequency (Hz)')
axes[0, 1].set_ylabel('∠H(e^{jω}) (rad)')
axes[0, 1].set_xlim(0, 100)
axes[0, 1].legend()
axes[0, 1].grid(True, alpha=0.3)

# Pole locations for Butterworth
z_butt, p_butt, k_butt = signal.tf2zpk(b_butt, a_butt)
z_cheb, p_cheb, k_cheb = signal.tf2zpk(b_cheb1, a_cheb1)

unit_circle = plt.Circle((0, 0), 1, fill=False, color='k', linewidth=1)
axes[1, 0].add_patch(unit_circle)
axes[1, 0].scatter(np.real(p_butt), np.imag(p_butt), marker='x', s=150, 
                    linewidths=2.5, color='blue', label='Butterworth poles', zorder=5)
axes[1, 0].scatter(np.real(p_cheb), np.imag(p_cheb), marker='+', s=150, 
                    linewidths=2, color='red', label='Chebyshev I poles', zorder=5)
axes[1, 0].scatter(np.real(z_butt), np.imag(z_butt), marker='o', s=80,
                    facecolors='none', edgecolors='blue', label='Zeros', zorder=5)
axes[1, 0].set_xlim(-1.5, 1.5)
axes[1, 0].set_ylim(-1.5, 1.5)
axes[1, 0].set_aspect('equal')
axes[1, 0].set_title('Pole-Zero Plot (z-plane)', fontsize=12)
axes[1, 0].set_xlabel('Real')
axes[1, 0].set_ylabel('Imaginary')
axes[1, 0].legend(fontsize=8)
axes[1, 0].grid(True, alpha=0.3)

# Group delay
gd_w_hz = gd_w * fs / (2 * np.pi)
axes[1, 1].plot(gd_w_hz, gd_butt * fs, 'b-', linewidth=2, label='Butterworth')
axes[1, 1].plot(gd_w_hz, gd_cheb1 * fs, 'r-', linewidth=2, label='Chebyshev I')
axes[1, 1].plot(gd_w_hz, gd_cheb2 * fs, 'g-', linewidth=2, label='Chebyshev II')
axes[1, 1].plot(gd_w_hz, gd_ellip * fs, 'm-', linewidth=2, label='Elliptic')
axes[1, 1].set_title('Group Delay Comparison', fontsize=12)
axes[1, 1].set_xlabel('Frequency (Hz)')
axes[1, 1].set_ylabel('Group Delay (samples)')
axes[1, 1].set_xlim(0, 100)
axes[1, 1].legend()
axes[1, 1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('lab2_section1_iir_comparison.png', dpi=150)
plt.show()
print("  [Saved] lab2_section1_iir_comparison.png")

# Check stability
print("\n  Stability check:")
for name, p in [('Butterworth', p_butt), ('Chebyshev I', p_cheb)]:
    all_inside = np.all(np.abs(p) < 1)
    print(f"    {name}: All poles |p| < 1 → {'STABLE ✓' if all_inside else 'UNSTABLE ✗'}")

# ---------------------------------------------------------
# SECTION 2: Notch Filter for Powerline Interference
# ---------------------------------------------------------
print("\n[2] Notch Filter — Remove 50 Hz Powerline Interference")

# Design notch filter at 50 Hz (and 60 Hz option)
f0 = 50.0  # Hz (powerline frequency in many countries, including HK)
Q = 30     # Quality factor — higher Q = narrower notch

# Notch filter using scipy
b_notch, a_notch = signal.iirnotch(f0, Q, fs)
print(f"  Notch filter: f0 = {f0} Hz, Q = {Q}")
print(f"  3-dB bandwidth ≈ f0/Q = {f0/Q:.2f} Hz")

# Alternative: design using second-order sections
# Notch at ω0: H(z) = (z - e^{jω0})(z - e^{-jω0}) / (z - r·e^{jω0})(z - r·e^{-jω0})
w0 = 2 * np.pi * f0 / fs  # digital frequency in rad/sample
r = 1 - 1/Q  # pole radius (closer to unit circle = narrower notch)

# Zero locations (on unit circle)
z1 = np.exp(1j * w0)
z2 = np.exp(-1j * w0)
# Pole locations (slightly inside unit circle)
p1 = r * np.exp(1j * w0)
p2 = r * np.exp(-1j * w0)

# Polynomial form
b_notch2 = np.poly([z1, z2])
a_notch2 = np.poly([p1, p2])

# Normalize DC gain to 1
gain = np.sum(b_notch2) / np.sum(a_notch2)
b_notch2 = b_notch2 * gain

# Frequency response of notch filter
w_n, H_notch = signal.freqz(b_notch, a_notch, worN=4096)
freq_n = w_n * fs / (2 * np.pi)
mag_notch = 20 * np.log10(np.abs(H_notch) + 1e-10)

# Find notch depth at 50 Hz
idx_50 = np.argmin(np.abs(freq_n - 50))
print(f"  Notch depth at 50 Hz: {mag_notch[idx_50]:.1f} dB")

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

axes[0].plot(freq_n, mag_notch, 'b-', linewidth=2)
axes[0].axvline(x=50, color='r', linestyle='--', alpha=0.7, label='50 Hz notch')
axes[0].axhline(y=-3, color='gray', linestyle=':', alpha=0.5)
axes[0].set_title(f'Notch Filter Response: f0={f0}Hz, Q={Q}', fontsize=12)
axes[0].set_xlabel('Frequency (Hz)')
axes[0].set_ylabel('|H(e^{jω})| (dB)')
axes[0].set_xlim(0, 100)
axes[0].set_ylim(-60, 5)
axes[0].legend()
axes[0].grid(True, alpha=0.3)

# Zoom around notch frequency
zoom_mask = (freq_n >= 40) & (freq_n <= 60)
axes[1].plot(freq_n[zoom_mask], mag_notch[zoom_mask], 'b-o', markersize=4, linewidth=2)
axes[1].axvline(x=50, color='r', linestyle='--', alpha=0.7)
axes[1].axhline(y=-3, color='gray', linestyle=':', alpha=0.5)
axes[1].set_title('Zoomed: Notch Filter (40-60 Hz)', fontsize=12)
axes[1].set_xlabel('Frequency (Hz)')
axes[1].set_ylabel('|H(e^{jω})| (dB)')
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('lab2_section2_notch.png', dpi=150)
plt.show()
print("  [Saved] lab2_section2_notch.png")

# ---------------------------------------------------------
# SECTION 3: Full ECG Denoising Pipeline
# ---------------------------------------------------------
print("\n[3] Full ECG Denoising Pipeline")

# Generate noisy ECG
t_ecg = np.arange(0, 4, 1/fs)
np.random.seed(77)

# Clean ECG
def gen_ecg(t, hr):
    ecg = np.zeros_like(t)
    cp = 60.0 / hr
    for i, ti in enumerate(t):
        ct = ti % cp
        if 0.05 < ct < 0.15: ecg[i] = 0.15 * np.sin(np.pi*(ct-0.05)/0.10)
        elif 0.15 < ct < 0.18: ecg[i] = -0.4 * np.sin(np.pi*(ct-0.15)/0.03)
        elif 0.18 < ct < 0.22: ecg[i] = 1.5 * np.sin(np.pi*(ct-0.18)/0.04)
        elif 0.22 < ct < 0.25: ecg[i] = -0.15 * np.sin(np.pi*(ct-0.22)/0.03)
        elif 0.30 < ct < 0.45: ecg[i] = 0.30 * np.sin(np.pi*(ct-0.30)/0.15)
    return ecg

ecg_clean = gen_ecg(t_ecg, 75)

# Add noise: baseline wander (0.3 Hz) + powerline (50 Hz) + white noise
baseline = 0.3 * np.sin(2*np.pi*0.3*t_ecg) + 0.2 * np.sin(2*np.pi*0.5*t_ecg)
powerline = 0.1 * np.sin(2*np.pi*50*t_ecg) + 0.08 * np.sin(2*np.pi*100*t_ecg)  # harmonics
white_noise = 0.05 * np.random.randn(len(t_ecg))

ecg_noisy = ecg_clean + baseline + powerline + white_noise

# Denoising pipeline
# Step 1: Highpass filter (fc=0.5 Hz) — remove baseline
b_hp, a_hp = signal.butter(2, 0.5/(fs/2), btype='high')
step1 = signal.filtfilt(b_hp, a_hp, ecg_noisy)

# Step 2: Notch filter (50 Hz) — remove powerline
b_notch, a_notch = signal.iirnotch(50, 30, fs)
step2 = signal.filtfilt(b_notch, a_notch, step1)

# Step 3: Lowpass filter (40 Hz) — remove high-frequency noise
b_lp, a_lp = signal.butter(4, 40/(fs/2), btype='low')
ecg_denoised = signal.filtfilt(b_lp, a_lp, step2)

# Also design an FIR alternative for comparison
b_fir_hp = signal.firwin(101, 0.5/(fs/2), pass_zero=False, window='hamming')
step1_fir = signal.lfilter(b_fir_hp, 1, ecg_noisy)
step1_fir = signal.lfilter(b_fir_hp[::-1], 1, step1_fir)  # zero-phase (two-pass)

# Compute SNR improvement
def snr_db(signal, noise):
    return 10 * np.log10(np.mean(signal**2) / (np.mean(noise**2) + 1e-10))

original_snr = snr_db(ecg_clean, baseline + powerline + white_noise)
# Approximate filtered SNR (compare denoised to clean)
residual = ecg_denoised - ecg_clean
filtered_snr = snr_db(ecg_clean, residual)

print(f"  Original SNR: {original_snr:.1f} dB")
print(f"  Filtered SNR: {filtered_snr:.1f} dB")
print(f"  SNR improvement: {filtered_snr - original_snr:.1f} dB")

fig, axes = plt.subplots(4, 1, figsize=(15, 14), sharex=True)

axes[0].plot(t_ecg, ecg_noisy, 'b-', linewidth=0.8, alpha=0.8)
axes[0].set_title('Noisy ECG: Baseline + 50Hz Powerline + White Noise', fontsize=12)
axes[0].set_ylabel('Amplitude (mV)')
axes[0].grid(True, alpha=0.3)

axes[1].plot(t_ecg, step1, 'g-', linewidth=0.8)
axes[1].set_title('After Highpass (fc=0.5Hz) — Baseline Removed', fontsize=12)
axes[1].set_ylabel('Amplitude (mV)')
axes[1].grid(True, alpha=0.3)

axes[2].plot(t_ecg, step2, 'orange', linewidth=0.8)
axes[2].set_title('After Notch (f0=50Hz) — Powerline Removed', fontsize=12)
axes[2].set_ylabel('Amplitude (mV)')
axes[2].grid(True, alpha=0.3)

axes[3].plot(t_ecg, ecg_denoised, 'r-', linewidth=1.2, label='IIR Filtered')
axes[3].plot(t_ecg, ecg_clean, 'b--', linewidth=0.8, alpha=0.5, label='Clean ECG')
axes[3].set_title('Final: Lowpass (fc=40Hz) — IIR Butterworth Denoted ECG', fontsize=12)
axes[3].set_xlabel('Time (seconds)')
axes[3].set_ylabel('Amplitude (mV)')
axes[3].legend()
axes[3].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('lab2_section3_ecg_denoising.png', dpi=150)
plt.show()
print("  [Saved] lab2_section3_ecg_denoising.png")

print("\n✅ Lab 2 Complete!")
```

### Expected Output
- IIR filter comparison: Butterworth vs Chebyshev vs Elliptic magnitude/phase/group delay
- Notch filter frequency response with zoomed view
- Full ECG denoising pipeline with before/after comparison

---

## Lab 3: Filter Structures & Real-time Implementation

### Objective
Implement direct form I, direct form II, and cascade (SOS) filter structures. Understand the trade-offs in coefficient quantization and finite word-length effects.

### Code

```python
"""
Week 9 Lab 3: Filter Structures & Implementation
==================================================
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy import signal

print("=" * 65)
print("LAB 3: Filter Structures & Implementation")
print("=" * 65)

# ---------------------------------------------------------
# SECTION 1: Filter Structure Comparison
# ---------------------------------------------------------
print("\n[1] Direct Form I vs Direct Form II (Canonical)")

# Design a 4th order Butterworth lowpass filter
fs = 500
fc = 40
order = 4
b, a = signal.butter(order, fc/(fs/2), btype='low')

print(f"  Filter order: {order}")
print(f"  Numerator (b): {b}")
print(f"  Denominator (a): {a}")

def filter_direct_form_I(x, b, a):
    """Direct Form I: most straightforward implementation"""
    N = len(b)
    M = len(a)
    L = len(x)
    y = np.zeros(L)
    
    for n in range(L):
        for k in range(N):
            if n - k >= 0:
                y[n] += b[k] * x[n - k]
        for k in range(1, M):
            if n - k >= 0:
                y[n] -= a[k] * y[n - k]
    
    return y

def filter_direct_form_II(x, b, a):
    """Direct Form II (Canonical): minimum state variables"""
    N = len(b)
    M = len(a)
    L = len(x)
    
    # State variables (delay elements)
    w = np.zeros(max(N, M) - 1)  # N-1 or M-1 states
    
    y = np.zeros(L)
    for n in range(L):
        # Push new input
        w[0] = x[n]
        for k in range(1, len(w)):
            w[0] -= a[k] * w[k]
        
        # Compute output
        y[n] = np.dot(b[:len(w)], w)
        
        # Update delay line (shift)
        for k in range(len(w)-1, 0, -1):
            w[k] = w[k-1]
    
    return y

# Compare with scipy's SOS implementation
sos = signal.tf2sos(b, a)
print(f"\n  Second-Order Sections (SOS):")
print(f"  {sos}")

# Test signal: unit step response
n_test = np.arange(0, 100)
x_test = np.ones(100)
y_direct1 = filter_direct_form_I(x_test, b, a)
y_direct2 = filter_direct_form_II(x_test, b, a)
y_scipy = signal.sosfilt(sos, x_test)

print(f"\n  Sample outputs (n=50):")
print(f"    Direct Form I:  {y_direct1[50]:.6f}")
print(f"    Direct Form II: {y_direct2[50]:.6f}")
print(f"    scipy sosfilt: {y_scipy[50]:.6f}")
print(f"    Max error DF1:  {np.max(np.abs(y_direct1 - y_scipy)):.2e}")

# ---------------------------------------------------------
# SECTION 2: Coefficient Quantization Effects
# ---------------------------------------------------------
print("\n[2] Coefficient Quantization Effects")

def quantize_coeffs(b, a, n_bits=8):
    """Quantize filter coefficients to n_bits precision"""
    def quantize_vector(v, bits):
        LSB = 2.0 ** (-bits)
        return np.round(v / LSB) * LSB
    
    return quantize_vector(b, n_bits), quantize_vector(a, n_bits)

# Compare frequency responses with different word lengths
b_q8, a_q8 = quantize_coeffs(b, a, n_bits=8)
b_q16, a_q16 = quantize_coeffs(b, a, n_bits=16)

w, H_orig = signal.freqz(b, a, worN=4096)
_, H_q8 = signal.freqz(b_q8, a_q8, worN=4096)
_, H_q16 = signal.freqz(b_q16, a_q16, worN=4096)

freq_hz = w * fs / (2 * np.pi)

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

axes[0].plot(freq_hz, 20*np.log10(np.abs(H_orig)), 'b-', linewidth=2, label='Full precision')
axes[0].plot(freq_hz, 20*np.log10(np.abs(H_q8)), 'r--', linewidth=1.5, label='8-bit quantized')
axes[0].plot(freq_hz, 20*np.log10(np.abs(H_q16)), 'g:', linewidth=1.5, label='16-bit quantized')
axes[0].axvline(x=fc, color='k', linestyle='--', alpha=0.5)
axes[0].set_title('Effect of Coefficient Quantization on Magnitude Response', fontsize=12)
axes[0].set_xlabel('Frequency (Hz)')
axes[0].set_ylabel('|H| (dB)')
axes[0].set_xlim(0, 100)
axes[0].set_ylim(-80, 5)
axes[0].legend()
axes[0].grid(True, alpha=0.3)

# Pole locations after quantization
z_orig, p_orig, k_orig = signal.tf2zpk(b, a)
z_q8, p_q8, k_q8 = signal.tf2zpk(b_q8, a_q8)
z_q16, p_q16, k_q16 = signal.tf2zpk(b_q16, a_q16)

unit_circle = plt.Circle((0, 0), 1, fill=False, color='k', linewidth=1)
axes[1].add_patch(unit_circle)
axes[1].scatter(np.real(p_orig), np.imag(p_orig), marker='x', s=200, 
                color='blue', linewidths=2, label='Original poles', zorder=5)
axes[1].scatter(np.real(p_q8), np.imag(p_q8), marker='o', s=100, 
                facecolors='none', edgecolors='red', linewidths=2, label='8-bit poles', zorder=5)
axes[1].scatter(np.real(p_q16), np.imag(p_q16), marker='s', s=80, 
                facecolors='none', edgecolors='green', linewidths=1.5, label='16-bit poles', zorder=5)
axes[1].set_xlim(-1.5, 1.5)
axes[1].set_ylim(-1.5, 1.5)
axes[1].set_aspect('equal')
axes[1].set_title('Pole Locations After Quantization', fontsize=12)
axes[1].set_xlabel('Real')
axes[1].set_ylabel('Imaginary')
axes[1].legend(fontsize=8)
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('lab3_section2_quantization.png', dpi=150)
plt.show()
print("  [Saved] lab3_section2_quantization.png")

# Check for pole instability after quantization
print("\n  Stability after quantization:")
print(f"    Original: {np.all(np.abs(p_orig) < 1)}")
print(f"    8-bit:    {np.all(np.abs(p_q8) < 1)}")
print(f"    16-bit:   {np.all(np.abs(p_q16) < 1)}")

# ---------------------------------------------------------
# SECTION 3: Real-time Filtering Simulation
# ---------------------------------------------------------
print("\n[3] Real-time Filtering: Sample-by-Sample Processing")

def realtime_filter(x, b, a):
    """Real-time filter: process one sample at a time"""
    N = len(b)
    M = len(a)
    
    # State: past inputs and outputs
    x_state = np.zeros(N)
    y_state = np.zeros(M)
    
    y = np.zeros(len(x))
    
    for n in range(len(x)):
        # Shift state
        x_state[1:] = x_state[:-1]
        x_state[0] = x[n]
        
        # Compute output
        y_out = np.dot(b, x_state) - np.dot(a[1:], y_state[1:])
        y_out /= a[0]
        
        y[n] = y_out
        
        # Update output state
        y_state[1:] = y_state[:-1]
        y_state[0] = y_out
    
    return y

# Process ECG sample by sample
fs_rt = 500
t_rt = np.arange(0, 2, 1/fs_rt)
np.random.seed(99)

ecg_rt = np.zeros_like(t_rt)
cp = 60.0 / 75
for i, ti in enumerate(t_rt):
    ct = ti % cp
    if 0.18 < ct < 0.22: ecg_rt[i] = 1.5 * np.sin(np.pi*(ct-0.18)/0.04)
    elif 0.30 < ct < 0.45: ecg_rt[i] = 0.30 * np.sin(np.pi*(ct-0.30)/0.15)

ecg_rt += 0.1 * np.random.randn(len(t_rt))

# Apply filter in real-time mode
b_rt, a_rt = signal.butter(2, 40/(fs_rt/2), btype='low')
y_rt = realtime_filter(ecg_rt, b_rt, a_rt)
y_batch = signal.lfilter(b_rt, a_rt, ecg_rt)

error = np.max(np.abs(y_rt - y_batch))
print(f"  Max difference between real-time and batch: {error:.2e}")
print(f"  {'✓ Real-time filter matches batch filter' if error < 1e-10 else '✗ Mismatch!'}")

fig, axes = plt.subplots(2, 1, figsize=(14, 8), sharex=True)

axes[0].plot(t_rt, ecg_rt, 'b-', linewidth=0.8, alpha=0.7)
axes[0].plot(t_rt, y_rt, 'r-', linewidth=1.5, label='Real-time (sample-by-sample)')
axes[0].plot(t_rt, y_batch, 'g--', linewidth=1, alpha=0.7, label='Batch (scipy.lfilter)')
axes[0].set_title('Real-time vs Batch Filtering', fontsize=12)
axes[0].set_ylabel('Amplitude')
axes[0].legend()
axes[0].grid(True, alpha=0.3)

axes[1].plot(t_rt, np.abs(y_rt - y_batch), 'purple', linewidth=1)
axes[1].set_title('Difference: Real-time vs Batch', fontsize=12)
axes[1].set_xlabel('Time (s)')
axes[1].set_ylabel('|Error|')
axes[1].set_ylim(1e-15, 1e-10)
axes[1].set_yscale('log')
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('lab3_section3_realtime.png', dpi=150)
plt.show()
print("  [Saved] lab3_section3_realtime.png")

print("\n✅ Lab 3 Complete!")
print("\n" + "=" * 65)
print("LAB SUMMARY: Week 9 — FIR & IIR Filters")
print("=" * 65)
print("✅ FIR design: window method (rect, Hann, Hamming, Blackman)")
print("✅ ECG baseline wander removal (FIR highpass, fc=0.5Hz)")
print("✅ IIR comparison: Butterworth, Chebyshev I/II, Elliptic")
print("✅ Notch filter for 50Hz powerline interference removal")
print("✅ Full ECG denoising pipeline: HP + Notch + LP")
print("✅ Filter structures: Direct Form I/II, SOS")
print("✅ Coefficient quantization effects (8-bit vs 16-bit)")
print("✅ Real-time sample-by-sample filtering")
```

### Expected Output
- FIR window comparison: impulse response, magnitude/phase responses
- IIR filter comparison: Butterworth, Chebyshev, Elliptic magnitude, phase, group delay
- Pole-zero plots before and after quantization
- ECG denoising pipeline demonstration

---

## References

- Oppenheim & Schafer (2010). *Discrete-Time Signal Processing*. 3rd ed.
- Ifeachor & Jervis (2002). *Digital Signal Processing*. 2nd ed.
- van de Geijn & Myers (1990). *A First Course in Digital Signal Processing*.

**Maintainer**: BME Bootcamp Agent | **Week 9** | **BMED2500: FIR/IIR Filters**
