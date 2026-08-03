# Week 8 Code Lab — Fourier Transform & Spectrum Analysis (BMED2500)

> **Topics Covered**: CTFT/DTFT, DFT/FFT, sampling theorem, spectrogram/STFT, spectral analysis of biomedical signals
> **Prerequisites**: numpy, matplotlib, scipy
> **Estimated Time**: 2-3 hours

---

## Lab 1: FFT Implementation & Spectral Analysis of ECG

### Objective
Implement the DFT and FFT algorithm from scratch, analyze the frequency spectrum of ECG signals, and understand the relationship between time-domain and frequency-domain representations. Compare manual DFT, numpy FFT, and scipy FFT implementations.

### Code

```python
"""
Week 8 Lab 1: FFT Implementation & ECG Spectral Analysis
=========================================================
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy import signal
import time

print("=" * 65)
print("LAB 1: FFT Implementation & Spectral Analysis")
print("=" * 65)

# ---------------------------------------------------------
# SECTION 1: Manual DFT Implementation (Educational)
# ---------------------------------------------------------
print("\n[1] Manual DFT vs numpy.fft Performance Comparison")

def manual_dft(x):
    """
    Manual N-point DFT implementation: O(N²) complexity
    X[k] = Σ_{n=0}^{N-1} x[n] · e^{-j2πkn/N}
    """
    N = len(x)
    X = np.zeros(N, dtype=complex)
    for k in range(N):
        for n in range(N):
            X[k] += x[n] * np.exp(-1j * 2 * np.pi * k * n / N)
    return X

def manual_fft(x):
    """
    Radix-2 Cooley-Tukey FFT: O(N log₂ N) complexity
    Requires N = 2^m (power of 2)
    """
    N = len(x)
    if N <= 1:
        return x
    if N % 2 != 0:
        raise ValueError(f"FFT size {N} must be power of 2")
    
    # Separate even and odd
    even = manual_fft(x[::2])
    odd = manual_fft(x[1::2])
    
    # Twiddle factors
    factor = np.exp(-2j * np.pi * np.arange(N // 2) / N)
    
    # Butterfly operations
    combined = np.zeros(N, dtype=complex)
    combined[:N//2] = even + factor * odd
    combined[N//2:] = even - factor * odd
    
    return combined

# Test correctness
np.random.seed(0)
x_test = np.random.randn(256)
x_test += 0.5 * np.sin(2 * np.pi * 50 * np.arange(256) / 256)

manual_result = manual_dft(x_test)
fft_result = np.fft.fft(x_test)
manual_fft_result = manual_fft(x_test)

error_manual = np.max(np.abs(manual_result - fft_result))
error_radix2 = np.max(np.abs(manual_fft_result - fft_result))

print(f"  Maximum error (manual DFT vs np.fft): {error_manual:.2e}")
print(f"  Maximum error (radix-2 FFT vs np.fft): {error_radix2:.2e}")
print(f"  ✓ Manual FFT correct: {error_radix2 < 1e-10}")

# Performance comparison
sizes = [128, 256, 512, 1024, 2048, 4096]
print("\n  Performance Comparison (single run):")
print(f"  {'N':>6} | {'Manual DFT (s)':>16} | {'Radix-2 FFT (s)':>16} | {'np.fft (s)':>12} | Speedup")
print("  " + "-" * 75)

for N in sizes:
    x_perf = np.random.randn(N) + np.sin(2*np.pi*10*np.arange(N)/N)
    
    t0 = time.time(); manual_dft(x_perf); t_manual = time.time() - t0
    t0 = time.time(); manual_fft(x_perf); t_radix2 = time.time() - t0
    t0 = time.time(); np.fft.fft(x_perf); t_numpy = time.time() - t0
    
    speedup = t_manual / t_radix2 if t_radix2 > 0 else float('inf')
    print(f"  {N:>6} | {t_manual:>16.6f} | {t_radix2:>16.6f} | {t_numpy:>12.6f} | {speedup:>7.1f}x")

# ---------------------------------------------------------
# SECTION 2: ECG Spectral Analysis
# ---------------------------------------------------------
print("\n[2] ECG Spectral Analysis — Frequency Content of Cardiac Signals")

# Generate ECG signal
fs_ecg = 500  # Hz
t_ecg = np.arange(0, 5, 1/fs_ecg)  # 5 seconds

def generate_ecg_full(t, heart_rate=72):
    """Generate full ECG with P, QRS, T waves"""
    cycle_period = 60.0 / heart_rate
    ecg = np.zeros_like(t)
    
    for i, ti in enumerate(t):
        ct = ti % cycle_period
        
        # P wave: 0.05-0.15s (atrial depolarization, 1-5 Hz)
        if 0.05 < ct < 0.15:
            ecg[i] = 0.15 * np.sin(np.pi * (ct - 0.05) / 0.10)
        
        # QRS: 0.15-0.22s (ventricular depolarization, 10-40 Hz)
        elif 0.15 < ct < 0.18:
            ecg[i] = -0.4 * np.sin(np.pi * (ct - 0.15) / 0.03)
        elif 0.18 < ct < 0.22:
            ecg[i] = 1.5 * np.sin(np.pi * (ct - 0.18) / 0.04)
        elif 0.22 < ct < 0.25:
            ecg[i] = -0.2 * np.sin(np.pi * (ct - 0.22) / 0.03)
        
        # T wave: 0.30-0.45s (ventricular repolarization, 1-8 Hz)
        elif 0.30 < ct < 0.45:
            ecg[i] = 0.30 * np.sin(np.pi * (ct - 0.30) / 0.15)
    
    return ecg

# Generate multiple ECG cycles
np.random.seed(42)
ecg_clean = generate_ecg_full(t_ecg, heart_rate=72)
ecg_noisy = ecg_clean + 0.1 * np.random.randn(len(t_ecg))  # Add noise

# Compute FFT
N = len(ecg_noisy)
freqs = np.fft.fftfreq(N, 1/fs_ecg)
fft_clean = np.fft.fft(ecg_clean)
fft_noisy = np.fft.fft(ecg_noisy)

# Get positive frequencies only
pos_mask = freqs >= 0
freqs_pos = freqs[pos_mask]
mag_clean = np.abs(fft_clean[pos_mask]) * 2 / N  # ×2 for single-sided
mag_noisy = np.abs(fft_noisy[pos_mask]) * 2 / N

# Identify dominant frequencies
peak_indices, _ = signal.find_peaks(mag_clean[1:], height=0.01, distance=10)
peak_indices += 1  # offset for the [1:] slice

print(f"  Signal duration: {N/fs_ecg:.1f} seconds")
print(f"  Frequency resolution: Δf = {fs_ecg/N:.3f} Hz")
print(f"  Nyquist frequency: {fs_ecg/2:.0f} Hz")
print(f"\n  Dominant frequency components:")
for idx in peak_indices[:8]:
    if freqs_pos[idx] < fs_ecg/2:
        print(f"    f = {freqs_pos[idx]:.1f} Hz, |X| = {mag_clean[idx]:.4f}")

fig, axes = plt.subplots(2, 2, figsize=(15, 11))

# ECG time domain
axes[0, 0].plot(t_ecg, ecg_clean, 'b-', linewidth=1, label='Clean')
axes[0, 0].plot(t_ecg, ecg_noisy, 'g-', alpha=0.5, linewidth=0.8, label='Noisy')
axes[0, 0].set_title('ECG Signal: Time Domain', fontsize=12)
axes[0, 0].set_xlabel('Time (s)')
axes[0, 0].set_ylabel('Amplitude (mV)')
axes[0, 0].legend()
axes[0, 0].set_xlim(0, 2)

# ECG frequency domain
axes[0, 1].plot(freqs_pos, mag_clean, 'b-', linewidth=1.5, label='Clean')
axes[0, 1].plot(freqs_pos, mag_noisy, 'g-', alpha=0.5, linewidth=0.8, label='Noisy')
axes[0, 1].set_title('ECG Signal: Frequency Spectrum (FFT)', fontsize=12)
axes[0, 1].set_xlabel('Frequency (Hz)')
axes[0, 1].set_ylabel('|X(f)|')
axes[0, 1].set_xlim(0, 60)
axes[0, 1].legend()

# Frequency bands annotation
bands = {'P wave': (1, 10, 'lightblue'), 'QRS': (10, 40, 'lightcoral'), 'T wave': (1, 8, 'lightyellow')}
for band, (f_low, f_high, color) in bands.items():
    axes[0, 1].axvspan(f_low, f_high, alpha=0.15, color=color, label=band)
axes[0, 1].legend()

# Log-scale spectrum
axes[1, 0].semilogy(freqs_pos[1:], mag_noisy[1:], 'g-', linewidth=1)
axes[1, 0].set_title('ECG Power Spectrum (Log Scale)', fontsize=12)
axes[1, 0].set_xlabel('Frequency (Hz)')
axes[1, 0].set_ylabel('|X(f)| (log)')
axes[1, 0].set_xlim(0, 100)

# Zoom on QRS region
qrs_mask = (freqs_pos >= 5) & (freqs_pos <= 50)
axes[1, 1].plot(freqs_pos[qrs_mask], mag_clean[qrs_mask], 'b-o', markersize=3, linewidth=1.5)
axes[1, 1].set_title('QRS Complex Frequency Band (5-50 Hz)', fontsize=12)
axes[1, 1].set_xlabel('Frequency (Hz)')
axes[1, 1].set_ylabel('|X(f)|')
axes[1, 1].set_xlim(5, 50)

plt.tight_layout()
plt.savefig('lab1_section2_ecg_spectrum.png', dpi=150)
plt.show()
print("  [Saved] lab1_section2_ecg_spectrum.png")

# ---------------------------------------------------------
# SECTION 3: Zero-Padding and Frequency Resolution
# ---------------------------------------------------------
print("\n[3] Zero-Padding and Frequency Resolution Enhancement")

# Original signal: 2 sinusoids
fs_res = 1000
t_res = np.arange(0, 0.1, 1/fs_res)  # 100 ms = 100 samples
x_res = 3 * np.sin(2*np.pi*100*t_res) + 1.5 * np.sin(2*np.pi*120*t_res)
N_orig = len(x_res)

# Zero-padded: append 900 zeros → N = 1000
x_zp = np.concatenate([x_res, np.zeros(900)])
N_zp = len(x_zp)

# FFT of both
X_orig = np.fft.fft(x_res)
X_zp = np.fft.fft(x_zp)
freqs_orig = np.fft.fftfreq(N_orig, 1/fs_res)
freqs_zp = np.fft.fftfreq(N_zp, 1/fs_res)

df_orig = fs_res / N_orig
df_zp = fs_res / N_zp

print(f"  Original: N = {N_orig}, Δf = {df_orig:.1f} Hz")
print(f"  Zero-padded: N = {N_zp}, Δf = {df_zp:.1f} Hz")
print(f"  Resolution improvement: {df_orig/df_zp:.0f}x")

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

pos_orig = freqs_orig >= 0
pos_zp = freqs_zp >= 0

axes[0].plot(freqs_orig[pos_orig], np.abs(X_orig[pos_orig]) * 2/N_orig, 'b-o', markersize=4)
axes[0].set_title(f'Without Zero-Padding: N={N_orig}, Δf={df_orig:.1f} Hz', fontsize=12)
axes[0].set_xlabel('Frequency (Hz)')
axes[0].set_ylabel('|X(f)|')
axes[0].set_xlim(0, 200)
axes[0].axvline(x=100, color='r', linestyle='--', alpha=0.5, label='True: 100 Hz')
axes[0].axvline(x=120, color='g', linestyle='--', alpha=0.5, label='True: 120 Hz')
axes[0].legend()

axes[1].plot(freqs_zp[pos_zp], np.abs(X_zp[pos_zp]) * 2/N_zp, 'r-o', markersize=2)
axes[1].set_title(f'With Zero-Padding: N={N_zp}, Δf={df_zp:.1f} Hz', fontsize=12)
axes[1].set_xlabel('Frequency (Hz)')
axes[1].set_ylabel('|X(f)|')
axes[1].set_xlim(0, 200)
axes[1].axvline(x=100, color='r', linestyle='--', alpha=0.5, label='True: 100 Hz')
axes[1].axvline(x=120, color='g', linestyle='--', alpha=0.5, label='True: 120 Hz')
axes[1].legend()

plt.tight_layout()
plt.savefig('lab1_section3_zero_pad.png', dpi=150)
plt.show()
print("  [Saved] lab1_section3_zero_pad.png")

print("\n✅ Lab 1 Complete!")
```

### Expected Output
- FFT performance comparison table (Naive vs Radix-2 vs np.fft)
- ECG spectrum with labeled P-wave, QRS, T-wave frequency bands
- Zero-padding demonstration showing improved frequency resolution

---

## Lab 2: STFT — Time-Frequency Analysis of EEG

### Objective
Implement the Short-Time Fourier Transform (STFT) for time-frequency analysis, apply it to EEG signals to detect event-related changes, and compare different window lengths and overlap settings.

### Code

```python
"""
Week 8 Lab 2: STFT & Time-Frequency Analysis of EEG
======================================================
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy import signal
from scipy.signal import spectrogram, windows

print("=" * 65)
print("LAB 2: STFT — Time-Frequency Analysis of EEG")
print("=" * 65)

# ---------------------------------------------------------
# SECTION 1: STFT Implementation
# ---------------------------------------------------------
print("\n[1] STFT Implementation from Scratch")

def stft(x, fs, nperseg=256, noverlap=None, nfft=None, window='hann'):
    """
    Short-Time Fourier Transform (STFT)
    
    Computes the spectrogram: sliding-window FFT
    
    Parameters:
        x: input signal
        fs: sampling frequency
        nperseg: window length (samples)
        noverlap: overlap between windows (samples)
        nfft: FFT length
        window: window function name
    
    Returns:
        f: frequency array
        t: time array
        Zxx: STFT complex coefficients
    """
    if noverlap is None:
        noverlap = nperseg // 2
    
    if nfft is None:
        nfft = nperseg
    
    # Window function
    win = windows.get_window(window, nperseg)
    
    # Step size
    step = nperseg - noverlap
    
    # Number of time frames
    n_frames = 1 + (len(x) - nperseg) // step
    
    # Compute STFT
    Zxx = np.zeros((nfft // 2 + 1, n_frames), dtype=complex)
    t = np.zeros(n_frames)
    
    for i in range(n_frames):
        start = i * step
        end = start + nperseg
        segment = x[start:end] * win  # Apply window
        Zxx[:, i] = np.fft.rfft(segment, n=nfft)
        t[i] = (start + nperseg // 2) / fs
    
    f = np.fft.rfftfreq(nfft, 1/fs)
    
    return f, t, Zxx

# ---------------------------------------------------------
# SECTION 2: EEG Simulation — Alpha Rhythm
# ---------------------------------------------------------
print("\n[2] EEG Alpha Rhythm Simulation & Time-Frequency Analysis")

# EEG frequency bands
bands = {
    'Delta': (0.5, 4),
    'Theta': (4, 8),
    'Alpha': (8, 13),
    'Beta': (13, 30),
    'Gamma': (30, 100)
}

# Simulate EEG with event-related changes
fs_eeg = 250  # Hz (standard EEG sampling rate)
t_eeg = np.arange(0, 10, 1/fs_eeg)  # 10 seconds

np.random.seed(99)

# Base EEG: superposition of rhythmic components
eeg = (0.3 * np.random.randn(len(t_eeg)) +
       0.5 * np.sin(2*np.pi*10*t_eeg) +  # Alpha (10 Hz)
       0.2 * np.sin(2*np.pi*20*t_eeg) +  # Beta (20 Hz)
       0.1 * np.sin(2*np.pi*4*t_eeg))    # Theta (4 Hz)

# Add event at t=3s: Alpha burst (eyes closed simulation)
event_start = int(3 * fs_eeg)
event_end = int(5 * fs_eeg)
alpha_burst = 2.0 * np.sin(2*np.pi*10*np.arange(event_end-event_start)/fs_eeg)
eeg[event_start:event_end] += alpha_burst
eeg[event_start:event_end] += 0.2 * np.random.randn(len(alpha_burst))

print(f"  Sampling rate: {fs_eeg} Hz")
print(f"  Duration: {len(t_eeg)/fs_eeg:.0f} seconds")
print(f"  Alpha event: 3-5 seconds")
print(f"  EEG bands: Delta(0.5-4Hz), Theta(4-8Hz), Alpha(8-13Hz), Beta(13-30Hz)")

# Compute STFT with different window lengths
nperseg_short = 64    # 64 samples = 256 ms → poor freq resolution, good time resolution
nperseg_long = 256    # 256 samples = 1024 ms → good freq resolution, poor time resolution
nperseg_medium = 128  # 128 samples = 512 ms → balanced

_, t_short, Z_short = stft(eeg, fs_eeg, nperseg=nperseg_short, noverlap=nperseg_short//4)
_, t_long, Z_long = stft(eeg, fs_eeg, nperseg=nperseg_long, noverlap=nperseg_long//4)
_, t_med, Z_med = stft(eeg, fs_eeg, nperseg=nperseg_medium, noverlap=nperseg_medium//4)

# Power spectrogram
P_short = np.abs(Z_short)**2
P_long = np.abs(Z_long)**2
P_med = np.abs(Z_med)**2

# Frequency resolution for each
df_short = fs_eeg / nperseg_short
df_long = fs_eeg / nperseg_long
df_t_short = nperseg_short / fs_eeg  # time resolution
df_t_long = nperseg_long / fs_eeg

print(f"\n  STFT Parameters:")
print(f"    Short window (N={nperseg_short}): Δf={df_short:.1f}Hz, Δt={df_t_short*1000:.0f}ms")
print(f"    Long window (N={nperseg_long}): Δf={df_long:.1f}Hz, Δt={df_t_long*1000:.0f}ms")

fig, axes = plt.subplots(2, 3, figsize=(16, 10))

# EEG time series
axes[0, 0].plot(t_eeg, eeg, 'b-', linewidth=0.8)
axes[0, 0].axvspan(3, 5, alpha=0.2, color='yellow', label='Alpha burst')
axes[0, 0].set_title('EEG Time Series (10s)', fontsize=11)
axes[0, 0].set_xlabel('Time (s)')
axes[0, 0].set_ylabel('Amplitude (μV)')
axes[0, 0].legend()

# Short window spectrogram (good time resolution)
im0 = axes[0, 1].imshow(10*np.log10(P_short + 1e-10), aspect='auto', origin='lower',
                         extent=[t_short[0], t_short[-1], 0, fs_eeg/2],
                         cmap='viridis', vmin=-40, vmax=20)
axes[0, 1].set_title(f'STFT Short Window (N={nperseg_short}, Δf={df_short:.1f}Hz)', fontsize=11)
axes[0, 1].set_xlabel('Time (s)')
axes[0, 1].set_ylabel('Frequency (Hz)')
axes[0, 1].set_ylim(0, 40)
plt.colorbar(im0, ax=axes[0, 1], label='Power (dB)')

# Long window spectrogram (good frequency resolution)
im1 = axes[0, 2].imshow(10*np.log10(P_long + 1e-10), aspect='auto', origin='lower',
                         extent=[t_long[0], t_long[-1], 0, fs_eeg/2],
                         cmap='viridis', vmin=-40, vmax=20)
axes[0, 2].set_title(f'STFT Long Window (N={nperseg_long}, Δf={df_long:.1f}Hz)', fontsize=11)
axes[0, 2].set_xlabel('Time (s)')
axes[0, 2].set_ylabel('Frequency (Hz)')
axes[0, 2].set_ylim(0, 40)
plt.colorbar(im1, ax=axes[0, 2], label='Power (dB)')

# Average power spectrum during rest vs event
freq_med = np.fft.rfftfreq(nperseg_medium, 1/fs_eeg)

# Rest period: 0-2.5s, Event period: 3.5-4.5s
rest_idx = (t_med >= 0) & (t_med <= 2.5)
event_idx = (t_med >= 3.5) & (t_med <= 4.5)

P_rest = np.mean(P_med[:, rest_idx], axis=1)
P_event = np.mean(P_med[:, event_idx], axis=1)

axes[1, 0].plot(freq_med, 10*np.log10(P_rest + 1e-10), 'b-', linewidth=2, label='Rest (0-2.5s)')
axes[1, 0].plot(freq_med, 10*np.log10(P_event + 1e-10), 'r-', linewidth=2, label='Event (3.5-4.5s)')
axes[1, 0].set_title('Power Spectral Density: Rest vs Alpha Burst', fontsize=11)
axes[1, 0].set_xlabel('Frequency (Hz)')
axes[1, 0].set_ylabel('Power (dB)')
axes[1, 0].set_xlim(0, 30)
axes[1, 0].legend()
axes[1, 0].grid(True, alpha=0.3)

# Alpha band power over time
alpha_low, alpha_high = 8, 13
alpha_idx = (freq_med >= alpha_low) & (freq_med <= alpha_high)
alpha_power = 10 * np.log10(np.mean(P_med[alpha_idx, :], axis=0) + 1e-10)

axes[1, 1].plot(t_med, alpha_power, 'b-', linewidth=1.5)
axes[1, 1].axvspan(3, 5, alpha=0.2, color='yellow')
axes[1, 1].set_title('Alpha Band Power (8-13 Hz) Over Time', fontsize=11)
axes[1, 1].set_xlabel('Time (s)')
axes[1, 1].set_ylabel('Power (dB)')
axes[1, 1].set_xlim(0, 10)

# Comparison with scipy.signal.spectrogram
f_scipy, t_scipy, Sxx = signal.spectrogram(eeg, fs_eeg, nperseg=128, noverlap=96)
axes[1, 2].imshow(10*np.log10(Sxx + 1e-10), aspect='auto', origin='lower',
                   extent=[t_scipy[0], t_scipy[-1], f_scipy[0], f_scipy[-1]],
                   cmap='magma', vmin=-40, vmax=20)
axes[1, 2].set_title('scipy.signal.spectrogram (nperseg=128)', fontsize=11)
axes[1, 2].set_xlabel('Time (s)')
axes[1, 2].set_ylabel('Frequency (Hz)')
axes[1, 2].set_ylim(0, 40)

plt.tight_layout()
plt.savefig('lab2_section2_eeg_stft.png', dpi=150)
plt.show()
print("  [Saved] lab2_section2_eeg_stft.png")

# ---------------------------------------------------------
# SECTION 3: Aliasing Demonstration
# ---------------------------------------------------------
print("\n[3] Aliasing Demonstration — Why Nyquist Matters")

fs_alias = 100  # Low sampling rate
t_alias = np.arange(0, 0.3, 1/fs_alias)

# High-frequency signal: f_actual = 40 Hz
f_actual = 40
x_actual = np.sin(2 * np.pi * f_actual * t_alias)

# Aliased frequency: f_alias = |f_actual − f_s| = |40 − 100| = 60 Hz would be wrong
# Actually: aliases fold back. With f_s = 100, f_max = 50.
# f_actual = 40 Hz < 50 Hz → should be correctly sampled
# Let's use f_actual = 70 Hz (> 50) → will alias
f_alias_test = 70  # Hz — above Nyquist!
x_alias_test = np.sin(2 * np.pi * f_alias_test * t_alias)

# Aliased frequency calculation
def alias_freq(f, fs):
    """Compute aliased frequency after sampling"""
    m = np.floor(f / (fs / 2))
    if m % 2 == 0:
        return f - m * fs / 2
    else:
        return (m + 1) * fs / 2 - f

f_aliased = alias_freq(f_alias_test, fs_alias)
print(f"  True frequency: {f_alias_test} Hz")
print(f"  Sampling rate: {fs_alias} Hz (Nyquist: {fs_alias/2} Hz)")
print(f"  Aliased frequency: {f_aliased} Hz")
print(f"  Measured period: {1/f_aliased*1000:.1f} ms (vs true: {1000/f_alias_test:.1f} ms)")

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

t_continuous = np.linspace(0, 0.15, 2000)
x_true = np.sin(2*np.pi*f_alias_test*t_continuous)

axes[0].plot(t_continuous * 1000, x_true, 'b-', linewidth=1, alpha=0.5, label=f'True: {f_alias_test} Hz')
axes[0].stem(t_alias * 1000, x_alias_test, linefmt='r-', markerfmt='ro', basefmt=' ', label='Sampled (f_s=100Hz)')
axes[0].set_title(f'Aliasing: True f={f_alias_test}Hz, f_s={fs_alias}Hz, Aliased≈{f_aliased:.0f}Hz', fontsize=12)
axes[0].set_xlabel('Time (ms)')
axes[0].set_ylabel('Amplitude')
axes[0].legend()

# Show the reconstructed (aliased) signal
t_rec = np.linspace(0, 0.3, 500)
x_rec = np.sin(2 * np.pi * f_aliased * t_rec)
axes[1].plot(t_alias * 1000, x_alias_test, 'ro', markersize=5, alpha=0.7, label='Sampled points')
axes[1].plot(t_rec * 1000, x_rec, 'g-', linewidth=2, alpha=0.7, label=f'Reconstructed: {f_aliased:.0f} Hz (WRONG!)')
axes[1].plot(t_continuous * 1000, x_true, 'b--', linewidth=1, alpha=0.5, label=f'True signal: {f_alias_test} Hz')
axes[1].set_title('Reconstructed vs True Signal', fontsize=12)
axes[1].set_xlabel('Time (ms)')
axes[1].set_ylabel('Amplitude')
axes[1].legend()

plt.tight_layout()
plt.savefig('lab2_section3_aliasing.png', dpi=150)
plt.show()
print("  [Saved] lab2_section3_aliasing.png")

print("\n✅ Lab 2 Complete!")
```

### Expected Output
- Spectrogram of EEG with alpha burst event (3-5 seconds)
- Comparison of short vs long window STFT
- Alpha band power over time showing event-related synchronization
- Aliasing demonstration: 70 Hz signal sampled at 100 Hz appears as 30 Hz

---

## Lab 3: Laplace Transform & System Analysis

### Objective
Implement the Laplace transform for system analysis, visualize pole-zero plots, analyze system stability in the s-plane, and apply the transfer function to model pharmacokinetic systems.

### Code

```python
"""
Week 8 Lab 3: Laplace Transform & s-plane Analysis
===================================================
"""

import numpy as np
import matplotlib.pyplot as plt
from matplotlib.patches import Circle, FancyArrowPatch
from scipy import signal

print("=" * 65)
print("LAB 3: Laplace Transform & s-plane System Analysis")
print("=" * 65)

# ---------------------------------------------------------
# SECTION 1: Laplace Transform of Common Signals
# ---------------------------------------------------------
print("\n[1] Laplace Transform of Common BME Signals")

def laplace_transform(f, t_max=10, num_points=1000):
    """
    Numerical Laplace transform: X(s) = ∫ x(t)e^{-st}dt
    Evaluate X(s) on the jω axis (frequency response)
    """
    t = np.linspace(0, t_max, num_points)
    dt = t[1] - t[0]
    omega = np.linspace(-100, 100, 500)
    
    X_s = np.zeros(len(omega), dtype=complex)
    for i, w in enumerate(omega):
        s = 1j * w
        X_s[i] = np.sum(f(t) * np.exp(-s * t)) * dt
    
    return omega, X_s

# (a) Unit step: u(t) → X(s) = 1/s
def step_response(t):
    return np.ones_like(t)

omega_step, X_step = laplace_transform(step_response, t_max=20)
X_step_analytical = 1 / (1j * omega_step + 1e-10)  # 1/(jω)

# (b) Exponential decay: e^{-at}u(t) → X(s) = 1/(s+a)
a = 2.0
def exp_decay(t):
    return np.exp(-a * t) * np.ones_like(t)

omega_exp, X_exp = laplace_transform(exp_decay, t_max=10)
X_exp_analytical = 1 / (1j * omega_exp + a)

# (c) Pharmacokinetics: C(t) = Dose/V_d · e^{-k_e·t}
k_e = 0.1  # hr^-1
dose = 100  # mg
V_d = 50   # L

def drug_concentration(t):
    return (dose / V_d) * np.exp(-k_e * t)

omega_drug, X_drug = laplace_transform(drug_concentration, t_max=50, num_points=2000)
X_drug_analytical = (dose / V_d) / (1j * omega_drug + k_e)

# Half-life in Laplace domain: pole at s = -k_e
t_half = np.log(2) / k_e
print(f"  Drug half-life: t_1/2 = ln(2)/k_e = {t_half:.2f} hours")
print(f"  Laplace pole: s = -{k_e} hr^-1")
print(f"  Residence time: τ = 1/k_e = {1/k_e:.1f} hours")

fig, axes = plt.subplots(2, 3, figsize=(16, 10))

# Time domain
t_plot = np.linspace(0, 10, 500)
axes[0, 0].plot(t_plot, step_response(t_plot), 'b-', linewidth=2)
axes[0, 0].set_title('Step Response: u(t)', fontsize=12)
axes[0, 0].set_xlabel('t (s)')

axes[0, 1].plot(t_plot, exp_decay(t_plot), 'r-', linewidth=2)
axes[0, 1].set_title(f'Exponential Decay: e^(-{a}t)·u(t)', fontsize=12)
axes[0, 1].set_xlabel('t (s)')

t_drug_plot = np.linspace(0, 48, 500)
axes[0, 2].plot(t_drug_plot, drug_concentration(t_drug_plot), 'g-', linewidth=2)
axes[0, 2].axhline(y=(dose/V_d)*0.5, color='k', linestyle='--', alpha=0.5)
axes[0, 2].axvline(x=t_half, color='k', linestyle=':', alpha=0.5)
axes[0, 2].annotate(f't_1/2={t_half:.1f}h', xy=(t_half, (dose/V_d)*0.5),
                    xytext=(t_half+5, (dose/V_d)*0.6), fontsize=10)
axes[0, 2].set_title('Drug Concentration: C(t) = Dose/V_d · e^(-k_e·t)', fontsize=12)
axes[0, 2].set_xlabel('Time (hours)')

# Frequency domain (magnitude)
axes[1, 0].plot(omega_step, np.abs(X_step), 'b-', linewidth=1.5, label='Numerical')
axes[1, 0].plot(omega_step, np.abs(X_step_analytical), 'r--', linewidth=1.5, label='Analytical')
axes[1, 0].set_title('Laplace of u(t): |1/jω|', fontsize=12)
axes[1, 0].set_xlabel('ω (rad/s)')
axes[1, 0].set_ylabel('|X(jω)|')
axes[1, 0].set_xlim(-50, 50)
axes[1, 0].legend()

axes[1, 1].plot(omega_exp, np.abs(X_exp), 'r-', linewidth=1.5, label='Numerical')
axes[1, 1].plot(omega_exp, np.abs(X_exp_analytical), 'b--', linewidth=1.5, label='Analytical')
axes[1, 1].set_title(f'Laplace of e^(-{a}t): |1/(jω+{a})|', fontsize=12)
axes[1, 1].set_xlabel('ω (rad/s)')
axes[1, 1].set_ylabel('|X(jω)|')
axes[1, 1].set_xlim(-10, 10)
axes[1, 1].legend()

axes[1, 2].plot(omega_drug, np.abs(X_drug), 'g-', linewidth=1.5, label='Numerical')
axes[1, 2].plot(omega_drug, np.abs(X_drug_analytical), 'r--', linewidth=1.5, label='Analytical')
axes[1, 2].axvline(x=-k_e, color='purple', linestyle=':', alpha=0.7, label=f'Pole at s={-k_e}')
axes[1, 2].set_title('Drug PK: |C(s)| = |Dose/(V_d(jω+k_e))|', fontsize=12)
axes[1, 2].set_xlabel('ω (rad/hr)')
axes[1, 2].set_ylabel('|C(jω)|')
axes[1, 2].set_xlim(-1, 1)
axes[1, 2].legend()

plt.tight_layout()
plt.savefig('lab3_section1_laplace.png', dpi=150)
plt.show()
print("  [Saved] lab3_section1_laplace.png")

# ---------------------------------------------------------
# SECTION 2: Pole-Zero Analysis in s-plane
# ---------------------------------------------------------
print("\n[2] Pole-Zero Analysis — s-plane Stability")

def plot_s_plane_zpk(z, p, k, ax, title="s-plane"):
    """Plot poles (×) and zeros (○) in the s-plane"""
    ax.set_aspect('equal')
    ax.axhline(y=0, color='k', linewidth=0.8)
    ax.axvline(x=0, color='k', linewidth=0.8)
    
    # Shade left/right half planes
    ax.fill_between([-10, 0], -10, 10, alpha=0.1, color='green', label='LHP (Stable)')
    ax.fill_between([0, 10], -10, 10, alpha=0.1, color='red', label='RHP (Unstable)')
    
    # Unit circle approximation (for reference)
    theta = np.linspace(0, 2*np.pi, 100)
    ax.plot(np.zeros(100), np.zeros(100), 'k-', linewidth=0.5)  # jω axis
    
    # Plot zeros
    if len(z) > 0:
        ax.scatter(np.real(z), np.imag(z), s=100, marker='o', 
                   facecolors='none', edgecolors='b', linewidths=2, label=f'Zeros ({len(z)})')
    
    # Plot poles
    if len(p) > 0:
        ax.scatter(np.real(p), np.imag(p), s=120, marker='x', 
                   color='r', linewidths=2.5, label=f'Poles ({len(p)})')
    
    ax.set_xlim(-5, 5)
    ax.set_ylim(-5, 5)
    ax.set_xlabel('σ (Real)')
    ax.set_ylabel('jω (Imaginary)')
    ax.set_title(title, fontsize=12)
    ax.grid(True, alpha=0.3)
    ax.legend(loc='upper right', fontsize=8)

# Systems to analyze
systems = [
    {"name": "1-compartment PK", "z": [], "p": [-0.1], "k": 1.0, "desc": "Single exponential decay"},
    {"name": "2-compartment PK", "z": [], "p": [-0.1, -0.5], "k": 1.0, "desc": "Two exponential decay modes"},
    {"name": "Underdamped 2nd order", "z": [], "p": [-2+3j, -2-3j], "k": 13, "desc": "Oscillatory response (e.g., blood pressure)"},
    {"name": "Unstable system", "z": [], "p": [0.5], "k": 1.0, "desc": "Growing exponential — UNSTABLE"},
]

fig, axes = plt.subplots(2, 2, figsize=(12, 12))
axes = axes.flatten()

for i, sys in enumerate(systems):
    z = np.array(sys["z"])
    p = np.array(sys["p"])
    
    # Check stability
    is_stable = np.all(np.real(p) < 0)
    
    plot_s_plane_zpk(z, p, sys["k"], axes[i], 
                     title=f"{sys['name']}: {'STABLE' if is_stable else 'UNSTABLE'}")
    
    # Time domain response
    t_sys = np.linspace(0, 10, 500)
    h_sys = np.zeros_like(t_sys)
    
    if len(p) == 1:
        h_sys = sys["k"] * np.exp(p[0] * t_sys) * np.heaviside(t_sys, 0)
    elif len(p) == 2:
        # Two-pole system
        if np.isreal(p[0]) and np.isreal(p[1]):
            # Real poles
            A1 = sys["k"] * (p[1] / (p[1] - p[0]))
            A2 = -sys["k"] * (p[0] / (p[1] - p[0]))
            h_sys = (A1 * np.exp(p[0]*t_sys) + A2 * np.exp(p[1]*t_sys)) * np.heaviside(t_sys, 0)
        else:
            # Complex conjugate poles: σ ± jω_d
            sigma = np.real(p[0])
            omega_d = np.imag(p[0])
            A = sys["k"] / np.sqrt(1 - (sigma/np.abs(p[0]))**2)
            h_sys = A * np.exp(sigma * t_sys) * np.sin(omega_d * t_sys + np.angle(p[0])) * np.heaviside(t_sys, 0)
    
    axes[i].text(-4.5, -4.5, f"{sys['desc']}\nh(t) = impulse response", 
                fontsize=8, style='italic', color='gray')
    
    print(f"  [{sys['name']}]: {'STABLE ✓' if is_stable else 'UNSTABLE ✗'}")
    print(f"    Poles: {p}")
    print(f"    {'LHP (all σ < 0)' if is_stable else 'RHP or on axis (UNSTABLE)'}")

plt.tight_layout()
plt.savefig('lab3_section2_splane.png', dpi=150)
plt.show()
print("  [Saved] lab3_section2_splane.png")

# ---------------------------------------------------------
# SECTION 3: Bode Plot — Frequency Response Analysis
# ---------------------------------------------------------
print("\n[3] Bode Plot — Pharmacokinetic System Frequency Response")

# Bode plot for the 1-compartment PK model
# H(s) = K / (τs + 1) = 1 / (10s + 1)  [τ = 10 hours]
tau_pk = 10.0  # hours
K_pk = 1.0

omega_bode = np.logspace(-3, 0, 500)  # 0.001 to 1 rad/hr
s_bode = 1j * omega_bode
H_bode = K_pk / (tau_pk * s_bode + 1)

mag_db = 20 * np.log10(np.abs(H_bode))
phase_deg = np.angle(H_bode, deg=True)

# Find bandwidth (where |H| drops to 1/√2 = -3dB)
bw_idx = np.argmin(np.abs(mag_db + 3))
bw = omega_bode[bw_idx]
print(f"  Bandwidth: ω_3dB = {bw:.4f} rad/hr = {bw*60/(2*np.pi):.4f} cycles/hr")
print(f"  This means the system responds to input changes slower than ~{1/tau_pk:.1f} rad/hr")

fig, axes = plt.subplots(2, 1, figsize=(12, 8))

axes[0].semilogx(omega_bode, mag_db, 'b-', linewidth=2)
axes[0].axhline(y=0, color='gray', linestyle='--', alpha=0.5)
axes[0].axhline(y=-3, color='r', linestyle='--', alpha=0.7, label='-3 dB')
axes[0].axvline(x=bw, color='r', linestyle=':', alpha=0.7, label=f'Bandwidth: {bw:.3f} rad/hr')
axes[0].set_ylabel('Magnitude (dB)')
axes[0].set_title('Bode Plot: 1-Compartment Pharmacokinetic System\nH(s) = 1/(τs+1), τ=10 hr', fontsize=12)
axes[0].set_xlim(1e-3, 1)
axes[0].set_ylim(-30, 5)
axes[0].legend()
axes[0].grid(True, which='both', alpha=0.3)

axes[1].semilogx(omega_bode, phase_deg, 'b-', linewidth=2)
axes[1].axhline(y=-90, color='gray', linestyle='--', alpha=0.5)
axes[1].axhline(y=-45, color='gray', linestyle='--', alpha=0.5)
axes[1].set_xlabel('Frequency ω (rad/hr)')
axes[1].set_ylabel('Phase (degrees)')
axes[1].set_title('Phase Response')
axes[1].set_xlim(1e-3, 1)
axes[1].set_ylim(-100, 10)
axes[1].grid(True, which='both', alpha=0.3)

plt.tight_layout()
plt.savefig('lab3_section3_bode.png', dpi=150)
plt.show()
print("  [Saved] lab3_section3_bode.png")

print("\n✅ Lab 3 Complete!")
print("\n" + "=" * 65)
print("LAB SUMMARY: Week 8 Fourier & Laplace Analysis")
print("=" * 65)
print("✅ Manual DFT implemented (O(N²))")
print("✅ Radix-2 FFT implemented (O(N log N))")
print("✅ ECG spectrum: P-wave (1-10Hz), QRS (10-40Hz), T-wave (1-8Hz)")
print("✅ Zero-padding: improved frequency resolution")
print("✅ STFT: time-frequency analysis of EEG alpha rhythm")
print("✅ Aliasing: 70Hz signal at f_s=100Hz → aliased as 30Hz")
print("✅ Laplace transform: numerical evaluation on jω axis")
print("✅ s-plane pole-zero plots: stability visualization")
print("✅ Bode plot: pharmacokinetic frequency response")
```

### Expected Output
- s-plane pole-zero plots showing stable vs unstable systems
- Bode plot of pharmacokinetic system with bandwidth annotation
- Time-domain impulse responses corresponding to pole locations

---

## References

- Oppenheim & Schafer (2010). *Discrete-Time Signal Processing*. 3rd ed. Prentice Hall.
- Proakis & Manolakis (2006). *Digital Signal Processing*. 4th ed. Prentice Hall.
- Rangayyan (2002). *Biomedical Signal Analysis*. IEEE Press.

**Maintainer**: BME Bootcamp Agent | **Week 8** | **BMED2500: Fourier & Sampling**
