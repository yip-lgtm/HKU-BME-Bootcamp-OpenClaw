# Week 21 Code Lab — Advanced Biomedical Signals (BMED4504)

## Advanced Signal Processing Lab

> **Dependencies**: `numpy`, `scipy`, `matplotlib`, `PyWavelets` (`pip install PyWavelets`)  
> **Estimated time**: 4-5 hours  
> **Learning goal**: Implement wavelet transforms, ICA, adaptive filters for biomedical signals

---

## Lab Setup

```bash
pip install numpy scipy matplotlib PyWavelets
```

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy import signal, stats
from scipy.fft import fft, ifft, fftfreq
import pywt
plt.style.use('seaborn-v0_8-whitegrid')
```

---

## Exercise 1: Wavelet Transform for EEG Analysis

```python
"""
W21_CodeLab_Ex1_Wavelet_EEG.py
================================
Apply CWT and DWT to EEG signal analysis.
Compare different wavelets and scales.
"""

def generate_eeg_like(t, fs=256):
    """
    Generate synthetic EEG-like signal with alpha rhythm and artifacts.
    """
    # Alpha rhythm (8-12 Hz)
    alpha = np.sin(2*np.pi*10*t) * 0.5
    
    # Eye blink artifact (0.5-3 Hz)
    blink_times = [2, 5, 10, 15]
    eye_blink = np.zeros_like(t)
    for bt in blink_times:
        idx = int(bt * fs)
        if idx < len(eye_blink):
            eye_blink[idx:idx+int(0.1*fs)] = np.exp(-np.linspace(0, 10, int(0.1*fs)))
    
    # Muscle artifact (high frequency)
    np.random.seed(42)
    muscle = np.random.randn(len(t)) * 0.2
    muscle_filt = np.convolve(muscle, np.hanning(50)/50, mode='same')
    
    # Combine
    eeg = alpha + eye_blink * 3 + muscle_filt
    return eeg

def plot_cwt_scalogram(data, fs, wavelet='morl', title='Scalogram'):
    """
    Compute and plot CWT scalogram.
    """
    # Scales for 1-30 Hz coverage
    scales = np.arange(1, 64)
    
    # CWT
    coeffs, freqs = pywt.cwt(data, scales, wavelet, 1/fs)
    
    fig, axes = plt.subplots(2, 1, figsize=(12, 8))
    
    # Time-domain signal
    t = np.arange(len(data)) / fs
    ax1 = axes[0]
    ax1.plot(t, data, 'b-', lw=0.5)
    ax1.set_xlabel('Time (s)')
    ax1.set_ylabel('Amplitude (μV)')
    ax1.set_title('EEG-like Signal')
    ax1.grid(True, alpha=0.3)
    
    # Scalogram (power)
    ax2 = axes[1]
    im = ax2.imshow(np.abs(coeffs)**2, extent=[t[0], t[-1], freqs[-1], freqs[0]],
                    aspect='auto', cmap='jet', interpolation='bilinear')
    ax2.set_xlabel('Time (s)')
    ax2.set_ylabel('Frequency (Hz)')
    ax2.set_title(title)
    plt.colorbar(im, ax=ax2, label='Power')
    
    plt.tight_layout()
    return coeffs, freqs

# Generate signal
fs = 256  # Hz
t = np.arange(0, 20, 1/fs)  # 20 seconds
eeg = generate_eeg_like(t, fs)

# Compare different wavelets
wavelets = ['morl', 'gaus8', 'mexh', 'cmor1-1.5']
fig, axes = plt.subplots(2, 2, figsize=(14, 10))
axes = axes.flatten()

for ax, wlet in zip(axes, wavelets):
    scales = np.arange(1, 64)
    coeffs, freqs = pywt.cwt(eeg, scales, wlet, 1/fs)
    
    t_axis = np.arange(len(eeg)) / fs
    im = ax.imshow(np.abs(coeffs)**2, extent=[t_axis[0], t_axis[-1], freqs[-1], freqs[0]],
                   aspect='auto', cmap='jet')
    ax.set_title(f'Wavelet: {wlet}')
    ax.set_xlabel('Time (s)')
    ax.set_ylabel('Frequency (Hz)')
    plt.colorbar(im, ax=ax)

plt.suptitle('CWT Scalograms: Different Wavelets', fontsize=14)
plt.tight_layout()
plt.savefig('W21_cwt_comparison.png', dpi=150)
plt.show()

# DWT decomposition
print("\n=== DWT Decomposition (5 levels) ===")
wavelet = 'db4'
coeffs = pywt.wavedec(eeg, wavelet, level=5)

print(f"Signal length: {len(eeg)}")
for i, c in enumerate(coeffs):
    level_name = 'Approximation' if i == 0 else f'Detail level {6-i}'
    freq_range = {
        0: f'0 - {fs/64:.1f} Hz',
        5: f'{fs/64:.1f} - {fs/32:.1f} Hz',
        4: f'{fs/32:.1f} - {fs/16:.1f} Hz',
        3: f'{fs/16:.1f} - {fs/8:.1f} Hz',
        2: f'{fs/8:.1f} - {fs/4:.1f} Hz',
        1: f'{fs/4:.1f} - {fs/2:.1f} Hz'
    }
    print(f"  Level {i} ({level_name}): {len(c)} samples, {freq_range.get(i, 'N/A')}")

# Reconstruct and denoise (thresholding)
def wavelet_denoise(signal_data, wavelet='db4', level=4, threshold_type='soft'):
    """Denoise signal using wavelet thresholding."""
    coeffs = pywt.wavedec(signal_data, wavelet, level=level)
    
    # Universal threshold (VisuShrink)
    sigma = np.median(np.abs(coeffs[-1])) / 0.6745
    threshold = sigma * np.sqrt(2 * np.log(len(signal_data)))
    
    # Apply threshold to detail coefficients only
    denoised_coeffs = [coeffs[0]]  # Keep approximation
    for c in coeffs[1:]:
        denoised_coeffs.append(pywt.threshold(c, threshold, mode=threshold_type))
    
    # Reconstruct
    return pywt.waverec(denoised_coeffs, wavelet)[:len(signal_data)]

eeg_denoised = wavelet_denoise(eeg, wavelet='db4', level=5)

# Plot comparison
fig, axes = plt.subplots(3, 1, figsize=(12, 10))
t_axis = np.arange(len(eeg)) / fs

ax1 = axes[0]
ax1.plot(t_axis, eeg, 'b-', lw=0.5, alpha=0.7)
ax1.set_title('Original EEG-like Signal')
ax1.set_xlabel('Time (s)')
ax1.set_ylabel('Amplitude')
ax1.grid(True, alpha=0.3)

ax2 = axes[1]
ax2.plot(t_axis, eeg_denoised, 'r-', lw=0.5)
ax2.set_title('Denoised EEG (Wavelet Thresholding)')
ax2.set_xlabel('Time (s)')
ax2.set_ylabel('Amplitude')
ax2.grid(True, alpha=0.3)

ax3 = axes[2]
ax3.plot(t_axis, eeg - eeg_denoised, 'k-', lw=0.5, alpha=0.7)
ax3.set_title('Removed Noise (Eye Blink + Muscle)')
ax3.set_xlabel('Time (s)')
ax3.set_ylabel('Amplitude')
ax3.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W21_wavelet_denoising.png', dpi=150)
plt.show()

# SNR improvement
def snr(signal, noise):
    return 10 * np.log10(np.var(signal) / np.var(noise))

snr_original = snr(alpha, eeg - alpha)
snr_denoised = snr(alpha, eeg_denoised - alpha)
print(f"\n=== Denoising Performance ===")
print(f"SNR before: {snr_original:.2f} dB")
print(f"SNR after: {snr_denoised:.2f} dB")
print(f"Improvement: {snr_denoised - snr_original:.2f} dB")
```

---

## Exercise 2: ICA for EEG Artifact Removal

```python
"""
W21_CodeLab_Ex2_ICA_EEG.py
============================
Implement FastICA for EEG artifact removal.
"""

def fastica(X, n_components, max_iter=200, tol=1e-4):
    """
    FastICA algorithm (simplified implementation).
    
    Based on Hyvärinen & Oja (1999).
    """
    n, p = X.shape
    
    # Whiten data (PCA)
    X_centered = X - X.mean(axis=0)
    cov = np.cov(X_centered.T)
    eigenvalues, eigenvectors = np.linalg.eigh(cov)
    idx = np.argsort(eigenvalues)[::-1]
    eigenvalues = eigenvalues[idx]
    eigenvectors = eigenvectors[:, idx]
    
    # Keep top n_components
    V = eigenvectors[:, :n_components]
    D = np.diag(1.0 / np.sqrt(eigenvalues[:n_components]))
    X_white = X_centered @ V @ D
    
    # Initialize
    W = np.random.randn(n_components, n_components)
    
    for iteration in range(max_iter):
        # FastICA update
        W_new = X_white @ (X_white @ W).T**3 / n - 3 * W
        
        # Orthogonalization (Gram-Schmidt)
        W_new, _ = np.linalg.qr(W_new.T)
        W_new = W_new.T
        
        # Check convergence
        change = np.max(np.abs(np.abs(np.diag(W_new @ W.T)) - 1))
        
        if change < tol:
            print(f"Converged at iteration {iteration}")
            break
        
        W = W_new
    
    # Sources
    S = W @ X_white
    
    # Unmixing matrix in original space
    A = V @ D @ W.T
    
    return S, W, A

def generate_eeg_mixture(n_samples=2000, fs=256):
    """
    Generate synthetic EEG with eye blink artifact.
    """
    np.random.seed(42)
    t = np.arange(n_samples) / fs
    
    # Source 1: Alpha rhythm (8-12 Hz)
    s1 = np.sin(2*np.pi*10*t) * 2
    
    # Source 2: Eye blink (0.5-2 Hz)
    blink_times = [2, 5, 8, 12]
    s2 = np.zeros(n_samples)
    for bt in blink_times:
        idx = int(bt * fs)
        if idx < n_samples:
            s2[idx:idx+int(0.2*fs)] = 5 * np.exp(-np.linspace(0, 5, int(0.2*fs)))
    
    # Source 3: Muscle artifact (30-50 Hz)
    s3 = np.random.randn(n_samples) * 1.5
    s3 = signal.filtfilt(*signal.butter(4, [30, 50], btype='bandpass', fs=fs), s3)
    
    sources = np.vstack([s1, s2, s3])
    
    # Mixing matrix (4 channels from 3 sources)
    A = np.random.randn(4, 3) * 0.5 + np.eye(4, 3) * 2
    
    # Mix
    X = sources.T @ A.T + np.random.randn(n_samples, 4) * 0.1
    
    return sources, X, t, fs

# Generate data
sources, X, t, fs = generate_eeg_mixture()

# Add a 4th channel with strong line noise
X[:, 3] = sources[2] * 0.8 + 2 * np.sin(2*np.pi*60*t)  # 60 Hz line noise

# Apply ICA
n_components = 4
S_recovered, W, A = fastica(X, n_components)

# Plot results
fig, axes = plt.subplots(4, 2, figsize=(14, 12))

for i in range(4):
    # Original sources
    ax1 = axes[i, 0]
    ax1.plot(t, sources[i], 'b-', lw=0.5)
    ax1.set_ylabel(f'Source {i+1}')
    if i == 0:
        ax1.set_title('Original Sources')
    ax1.grid(True, alpha=0.3)
    
    # Recovered sources
    ax2 = axes[i, 1]
    ax2.plot(t, S_recovered[i], 'r-', lw=0.5)
    ax2.set_ylabel(f'IC {i+1}')
    if i == 0:
        ax2.set_title('Recovered Independent Components')
    ax2.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W21_ica_recovery.png', dpi=150)
plt.show()

# Identify artifact components
print("\n=== Component Analysis ===")
for i in range(n_components):
    ic = S_recovered[i]
    
    # Frequency content
    freqs, psd = signal.welch(ic, fs, nperseg=1024)
    
    # Dominant frequency
    dom_freq = freqs[np.argmax(psd)]
    
    # Kurtosis (measure of non-Gaussianity)
    kurt = stats.kurtosis(ic)
    
    # Identify
    if dom_freq < 5:
        label = "Eye Blink Artifact"
    elif 25 < dom_freq < 60:
        label = "Muscle / Line Noise"
    elif 8 < dom_freq < 13:
        label = "Alpha Rhythm (Neural)"
    else:
        label = "Mixed / Unknown"
    
    print(f"IC{i+1}: dominant freq = {dom_freq:.1f} Hz, kurtosis = {kurt:.2f} → {label}")

print("\n=== Artifact Removal ===")
print("To remove artifacts, zero out artifact components and reconstruct:")
print("  X_clean = X @ pinv(A) with artifact rows zeroed")
```

---

## Exercise 3: LMS Adaptive Filter

```python
"""
W21_CodeLab_Ex3_LMS.py
========================
Implement LMS adaptive filter for noise cancellation.
"""

def lms_filter(x, d, M, mu, verbose=False):
    """
    LMS adaptive filter for noise cancellation.
    
    Parameters:
    -----------
    x : array
        Reference (noise) signal
    d : array
        Desired signal (primary input)
    M : int
        Filter length
    mu : float
        Step size
    
    Returns:
    --------
    y : array
        Filter output (estimated noise)
    e : array
        Error signal (cleaned signal)
    w : array
        Final filter coefficients
    """
    N = len(d)
    w = np.zeros(M)  # Filter coefficients
    y = np.zeros(N)  # Output
    e = np.zeros(N)  # Error
    
    for n in range(M, N-1):
        # Input vector (reference signal samples)
        x_n = x[n-M+1:n+1][::-1]
        
        # Filter output
        y[n] = np.dot(w, x_n)
        
        # Error (desired - estimated noise)
        e[n] = d[n] - y[n]
        
        # Update weights
        w = w + mu * e[n] * x_n
        
        if verbose and n % 1000 == 0:
            mse = np.mean(e[n-999:n]**2)
            print(f"n={n}, MSE={mse:.6f}, w[0]={w[0]:.4f}")
    
    return y, e, w

def generate_adaptive_filter_data(fs=1000):
    """
    Generate data for adaptive noise cancellation.
    """
    t = np.arange(0, 5, 1/fs)  # 5 seconds
    
    # Primary signal (clean ECG-like)
    np.random.seed(42)
    ecg = np.zeros_like(t)
    # R-peaks at 1 Hz
    for i, ti in enumerate(t):
        phase = (ti % 1.0)
        if phase < 0.05:
            ecg[i] = 2 * np.exp(-((phase - 0.025) / 0.01)**2)
        elif phase < 0.1:
            ecg[i] = -0.5 * np.exp(-((phase - 0.07) / 0.01)**2)
    
    # Reference noise (60 Hz power line)
    noise_ref = 0.3 * np.sin(2*np.pi*60*t)
    
    # Add noise to primary
    d = ecg + noise_ref + 0.1 * np.random.randn(len(t))
    
    return t, d, noise_ref, ecg

# Generate data
t, d, noise_ref, ecg_clean = generate_adaptive_filter_data()

# Apply LMS filter
M = 32  # Filter length
mu_values = [0.01, 0.05, 0.1]  # Step sizes

fig, axes = plt.subplots(3, 2, figsize=(14, 12))

results = {}
for ax_row, mu in zip(axes, mu_values):
    y, e, w = lms_filter(noise_ref, d, M, mu)
    results[mu] = (y, e, w)
    
    # Time domain
    ax_row[0].plot(t, e, 'b-', lw=0.5, alpha=0.8)
    ax_row[0].plot(t, ecg_clean, 'r--', lw=1, alpha=0.7, label='Clean ECG')
    ax_row[0].set_xlabel('Time (s)')
    ax_row[0].set_ylabel('Amplitude')
    ax_row[0].set_title(f'LMS (μ={mu}): Cleaned Signal')
    ax_row[0].legend()
    ax_row[0].grid(True, alpha=0.3)
    
    # Filter coefficients
    ax_row[1].plot(w, 'b-', lw=2)
    ax_row[1].set_xlabel('Tap')
    ax_row[1].set_ylabel('Weight')
    ax_row[1].set_title(f'LMS (μ={mu}): Final Filter Weights')
    ax_row[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W21_lms_adaptive.png', dpi=150)
plt.show()

# Performance comparison
print("\n=== LMS Performance vs. Step Size ===")
print(f"{'μ':<10} {'MSE (mean)':<15} {'MSE (final 1s)':<18} {'SNR Improvement'}")
print("-" * 60)

for mu, (y, e, w) in results.items():
    mse_mean = np.mean(e**2)
    mse_final = np.mean(e[-1000:]**2)
    
    # SNR before
    snr_before = 10 * np.log10(np.var(ecg_clean) / np.var(d - ecg_clean))
    
    # SNR after
    snr_after = 10 * np.log10(np.var(ecg_clean) / np.var(e - ecg_clean))
    
    print(f"{mu:<10.3f} {mse_mean:<15.6f} {mse_final:<18.6f} {snr_after - snr_before:.2f} dB")
```

---

## Exercise 4: Compressive Sensing Simulation

```python
"""
W21_CodeLab_Ex4_CompressedSensing.py
=====================================
Demonstrate compressive sensing with sparse signal reconstruction.
"""

def generate_sparse_signal(N=1024, K=10):
    """Generate K-sparse signal in wavelet domain."""
    np.random.seed(42)
    
    # Sparse in time (spikes)
    x = np.zeros(N)
    spike_indices = np.random.choice(N, K, replace=False)
    spike_values = np.random.randn(K) * np.random.uniform(0.5, 2, K)
    x[spike_indices] = spike_values
    
    # Or sparse in wavelet domain (more realistic)
    # x = pywt.idwt(coeffs_sparse, None, 'db4')
    
    return x

def compressed_measurement(x, M, seed=42):
    """
    Perform compressed sensing measurement.
    y = Φx
    """
    np.random.seed(seed)
    Phi = np.random.randn(M, len(x)) / np.sqrt(M)  # Gaussian random matrix
    y = Phi @ x
    return y, Phi

def omp_reconstruction(y, Phi, K, max_iter=None):
    """
    Orthogonal Matching Pursuit (OMP) for sparse reconstruction.
    """
    if max_iter is None:
        max_iter = K
    
    residual = y.copy()
    support = []
    x_hat = np.zeros(Phi.shape[1])
    
    for _ in range(max_iter):
        # Find atom most correlated with residual
        correlations = np.abs(Phi.T @ residual)
        correlations[support] = -np.inf  # Exclude selected
        
        idx = np.argmax(correlations)
        support.append(idx)
        
        # Solve least squares on support
        Phi_s = Phi[:, support]
        x_s = np.linalg.lstsq(Phi_s, y, rcond=None)[0]
        
        # Update residual
        residual = y - Phi_s @ x_s
        
        # Check convergence
        if np.linalg.norm(residual) < 1e-6:
            break
    
    x_hat[support] = x_s
    return x_hat, support

def l1_reconstruction(y, Phi, alpha=0.1):
    """
    L1 minimization (simplified via basis pursuit denoising).
    Uses iterative soft thresholding (ISTA).
    """
    from scipy.optimize import minimize
    
    def objective(x):
        return 0.5 * np.linalg.norm(y - Phi @ x)**2 + alpha * np.sum(np.abs(x))
    
    x0 = Phi.T @ y  # Initialize with pseudoinverse
    
    result = minimize(objective, x0, method='L-BFGS-B', 
                     options={'maxiter': 1000, 'disp': False})
    
    return result.x

# Main comparison
N = 1024  # Original signal length
M_values = [64, 128, 256, 512]  # Measurement counts
K_true = 15  # True sparsity

x_true = generate_sparse_signal(N, K_true)

print("="*60)
print("COMPRESSIVE SENSING RECONSTRUCTION")
print("="*60)
print(f"Original signal length: N = {N}")
print(f"True sparsity: K = {K_true}")
print(f"Compression ratio: {N/M_values[-1]:.1f}×")
print()

results_summary = []

for M in M_values:
    y, Phi = compressed_measurement(x_true, M)
    
    # OMP reconstruction
    x_omp, support = omp_reconstruction(y, Phi, K_true)
    mse_omp = np.mean((x_true - x_omp)**2)
    
    # L1 reconstruction
    x_l1 = l1_reconstruction(y, Phi, alpha=0.05)
    mse_l1 = np.mean((x_true - x_l1)**2)
    
    # Normalized reconstruction error
    nrmse_omp = np.sqrt(mse_omp / np.var(x_true)) * 100
    nrmse_l1 = np.sqrt(mse_l1 / np.var(x_true)) * 100
    
    print(f"M = {M:4d} ({M/N*100:.1f}%): OMP NRMSE = {nrmse_omp:6.2f}%, L1 NRMSE = {nrmse_l1:6.2f}%")
    
    results_summary.append({
        'M': M, 'mse_omp': mse_omp, 'mse_l1': mse_l1,
        'nrmse_omp': nrmse_omp, 'nrmse_l1': nrmse_l1
    })

# Plot results
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# Original signal
ax1 = axes[0, 0]
ax1.stem(x_true, linefmt='b-', markerfmt='bo', basefmt=' ')
ax1.set_title('Original Sparse Signal')
ax1.set_xlabel('Sample Index')
ax1.set_ylabel('Amplitude')
ax1.grid(True, alpha=0.3)

# Measurements
ax2 = axes[0, 1]
y, _ = compressed_measurement(x_true, 128)
ax2.stem(y, linefmt='r-', markerfmt='ro', basefmt=' ')
ax2.set_title('Compressed Measurements (M=128)')
ax2.set_xlabel('Measurement Index')
ax2.set_ylabel('Amplitude')
ax2.grid(True, alpha=0.3)

# Reconstructed signals
ax3 = axes[1, 0]
x_omp, _ = omp_reconstruction(y, Phi, K_true)
ax3.stem(x_omp, linefmt='g-', markerfmt='go', basefmt=' ', label='OMP')
ax3.set_title(f'OMP Reconstruction (NRMSE={results_summary[1]["nrmse_omp"]:.1f}%)')
ax3.set_xlabel('Sample Index')
ax3.set_ylabel('Amplitude')
ax3.grid(True, alpha=0.3)

ax4 = axes[1, 1]
x_l1 = l1_reconstruction(y, Phi, alpha=0.05)
ax4.stem(x_l1, linefmt='m-', markerfmt='mo', basefmt=' ')
ax4.set_title(f'L1 Reconstruction (NRMSE={results_summary[1]["nrmse_l1"]:.1f}%)')
ax4.set_xlabel('Sample Index')
ax4.set_ylabel('Amplitude')
ax4.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W21_compressive_sensing.png', dpi=150)
plt.show()

# Minimum measurements analysis
print("\n=== Minimum Measurements for Perfect Reconstruction ===")
for M in [32, 48, 64, 80, 96, 128]:
    y, Phi = compressed_measurement(x_true, M)
    x_omp, support = omp_reconstruction(y, Phi, K_true)
    nrmse = np.sqrt(np.mean((x_true - x_omp)**2) / np.var(x_true)) * 100
    status = "✓ Perfect" if nrmse < 1 else "✗"
    print(f"M = {M:3d}: NRMSE = {nrmse:6.2f}% {status}")
```

---

## Exercise 5: EEG Advanced Analysis Pipeline

```python
"""
W21_CodeLab_Ex5_EEG_Pipeline.py
================================
Complete EEG signal processing pipeline with all techniques.
"""

def generate_realistic_eeg(fs=256, duration=30):
    """
    Generate realistic EEG with multiple rhythms and artifacts.
    """
    np.random.seed(42)
    t = np.arange(0, duration, 1/fs)
    n = len(t)
    
    # EEG rhythms
    # Delta (0.5-4 Hz) - deep sleep
    delta = np.sin(2*np.pi*2*t) * 0.3 * np.random.uniform(0.5, 1.5)
    
    # Theta (4-8 Hz) - drowsiness
    theta = np.sin(2*np.pi*6*t) * 0.4 * np.random.uniform(0.5, 1.5)
    
    # Alpha (8-13 Hz) - relaxed eyes closed
    alpha = np.sin(2*np.pi*10*t) * 0.8 * (1 + 0.3*np.sin(2*np.pi*0.1*t))
    
    # Beta (13-30 Hz) - active thinking
    beta = np.sin(2*np.pi*20*t) * 0.3 * np.random.uniform(0.5, 1.5)
    
    # Combine rhythms
    eeg = delta + theta + alpha + beta
    
    # Add artifacts
    # Eye blinks (every 5 seconds)
    for bt in np.arange(5, duration, 5):
        idx = int(bt * fs)
        if idx < n:
            eeg[idx:idx+int(0.15*fs)] += 5 * np.exp(-np.linspace(0, 10, int(0.15*fs)))
    
    # Muscle artifacts (random)
    muscle_start = np.random.randint(n//4, n//2)
    eeg[muscle_start:muscle_start+int(2*fs)] += 2 * np.random.randn(int(2*fs))
    
    # Line noise (60 Hz)
    eeg += 0.2 * np.sin(2*np.pi*60*t)
    
    # Add channel dimension
    eeg = eeg.reshape(1, -1)
    
    return eeg, t, fs

def process_eeg_pipeline(eeg_raw, fs):
    """
    Complete EEG processing pipeline:
    1. Preprocessing (bandpass filter)
    2. ICA for artifact removal
    3. Wavelet denoising
    4. Time-frequency analysis
    """
    print("Step 1: Preprocessing (Bandpass 0.5-50 Hz)")
    # Bandpass filter
    b, a = signal.butter(4, [0.5, 50], btype='bandpass', fs=fs)
    eeg_filt = signal.filtfilt(b, a, eeg_raw[0])
    
    print("Step 2: Notch filter (60 Hz)")
    # Notch filter for line noise
    b, a = signal.iirnotch(60, 30, fs=fs)
    eeg_notch = signal.filtfilt(b, a, eeg_filt)
    
    print("Step 3: ICA for artifact removal")
    # Simple ICA (placeholder - in practice use sklearn.decomposition.FastICA)
    from scipy.linalg import svd
    X = np.vstack([eeg_notch, eeg_filt])  # 2-channel for demo
    X_centered = X - X.mean(axis=1, keepdims=True)
    U, S, Vt = svd(X_centered, full_matrices=False)
    
    # Keep top 2 components
    n_components = 2
    S_recovered = U[:n_components].T * S[:n_components]
    
    # Analyze components
    kurtoses = [stats.kurtosis(S_recovered[i]) for i in range(n_components)]
    
    print("Step 4: Wavelet denoising")
    eeg_denoised = wavelet_denoise_simple(eeg_notch, wavelet='db4', level=5)
    
    return eeg_filt, eeg_notch, eeg_denoised, S_recovered, kurtoses

def wavelet_denoise_simple(x, wavelet='db4', level=5):
    """Simple wavelet denoising."""
    coeffs = pywt.wavedec(x, wavelet, level=level)
    sigma = np.median(np.abs(coeffs[-1])) / 0.6745
    threshold = sigma * np.sqrt(2 * np.log(len(x)))
    denoised = pywt.threshold(coeffs, threshold, mode='soft')
    return pywt.waverec(denoised, wavelet)[:len(x)]

# Generate data
print("Generating realistic EEG data...")
eeg_raw, t, fs = generate_realistic_eeg(fs=256, duration=30)
print(f"Data shape: {eeg_raw.shape}, Duration: {len(t)/fs:.1f} s")

# Process
print("\n=== EEG Processing Pipeline ===")
eeg_filt, eeg_notch, eeg_denoised, ica_comps, kurtoses = process_eeg_pipeline(eeg_raw, fs)

# Time-frequency analysis
print("\nStep 5: Time-frequency analysis (CWT)")
scales = np.arange(1, 64)
coeffs, freqs = pywt.cwt(eeg_denoised, scales, 'morl', 1/fs)

# Plot
fig = plt.figure(figsize=(16, 12))

# Original
ax1 = plt.subplot(3, 2, 1)
ax1.plot(t, eeg_raw[0], 'b-', lw=0.3)
ax1.set_title('Raw EEG')
ax1.set_xlabel('Time (s)')
ax1.set_ylabel('Amplitude')
ax1.grid(True, alpha=0.3)

# Filtered
ax2 = plt.subplot(3, 2, 2)
ax2.plot(t, eeg_filt, 'g-', lw=0.3)
ax2.set_title('Bandpass + Notch Filtered')
ax2.set_xlabel('Time (s)')
ax2.set_ylabel('Amplitude')
ax2.grid(True, alpha=0.3)

# Denoised
ax3 = plt.subplot(3, 2, 3)
ax3.plot(t, eeg_denoised, 'r-', lw=0.3)
ax3.set_title('Wavelet Denoised')
ax3.set_xlabel('Time (s)')
ax3.set_ylabel('Amplitude')
ax3.grid(True, alpha=0.3)

# Spectrogram (STFT)
ax4 = plt.subplot(3, 2, 4)
f, t_spec, Sxx = signal.spectrogram(eeg_denoised, fs, nperseg=256, noverlap=128)
ax4.pcolormesh(t_spec, f, 10*np.log10(Sxx), cmap='jet', shading='gouraud')
ax4.set_ylim(0, 50)
ax4.set_title('STFT Spectrogram')
ax4.set_xlabel('Time (s)')
ax4.set_ylabel('Frequency (Hz)')
ax4.set_yscale('log')

# Scalogram (CWT)
ax5 = plt.subplot(3, 2, 5)
im = ax5.imshow(np.abs(coeffs)**2, extent=[t[0], t[-1], freqs[-1], freqs[0]],
                aspect='auto', cmap='jet')
ax5.set_ylim(0, 50)
ax5.set_title('CWT Scalogram')
ax5.set_xlabel('Time (s)')
ax5.set_ylabel('Frequency (Hz)')
plt.colorbar(im, ax=ax5)

# Power spectral density
ax6 = plt.subplot(3, 2, 6)
freqs_psd, psd = signal.welch(eeg_denoised, fs, nperseg=1024)
ax6.semilogy(freqs_psd, psd, 'b-', lw=1.5)
ax6.axvspan(0.5, 4, alpha=0.2, color='red', label='Delta')
ax6.axvspan(4, 8, alpha=0.2, color='orange', label='Theta')
ax6.axvspan(8, 13, alpha=0.2, color='green', label='Alpha')
ax6.axvspan(13, 30, alpha=0.2, color='blue', label='Beta')
ax6.set_xlabel('Frequency (Hz)')
ax6.set_ylabel('Power Spectral Density')
ax6.set_title('Power Spectrum')
ax6.legend()
ax6.grid(True, alpha=0.3)
ax6.set_xlim(0, 50)

plt.tight_layout()
plt.savefig('W21_eeg_pipeline.png', dpi=150)
plt.show()

print("\n=== Pipeline Summary ===")
print(f"ICA components: {len(ica_comps)}")
for i, k in enumerate(kurtoses):
    print(f"  Component {i+1}: kurtosis = {k:.2f}")
```

---

## Lab Report Checklist

| Task | Description | File |
|------|-------------|------|
| ☐ Ex 1 | Wavelet denoising comparison | `W21_cwt_comparison.png`, `W21_wavelet_denoising.png` |
| ☐ Ex 2 | ICA artifact removal | `W21_ica_recovery.png` |
| ☐ Ex 3 | LMS adaptive filtering | `W21_lms_adaptive.png` |
| ☐ Ex 4 | Compressive sensing | `W21_compressive_sensing.png` |
| ☐ Ex 5 | Complete EEG pipeline | `W21_eeg_pipeline.png` |

---

## Extension Challenge

**Challenge**: Implement a real-time adaptive noise cancellation system for a biomedical signal of your choice (ECG with baseline wander, or fetal ECG extraction). Compare LMS vs. RLS performance in terms of convergence speed and steady-state error.
