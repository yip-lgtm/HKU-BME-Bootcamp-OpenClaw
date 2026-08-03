# Week 18 Code Lab — Medical Imaging II (MRI, Nuclear & Optical Imaging)
## Python Computational Medical Imaging Lab

> **Dependencies**: `numpy`, `scipy`, `matplotlib`, `pandas`  
> **Estimated time**: 3-4 hours  
> **Learning goal**: Simulate MRI Bloch equation, PET SUV calculation, OCT depth profiling

---

## Exercise 1: MRI Bloch Equation Simulation

```python
"""
W18_CodeLab_Ex1_BlochEquation.py
===================================
Simulate the Bloch equations for MRI relaxation.
Model T1 recovery and T2 decay.
Simulate spin echo sequence.
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy.fft import fft, fftfreq, fftshift

# =============================================================
# PARAMETERS: Tissue T1 and T2 values at 1.5T
# =============================================================
tissues = {
    'CSF':          {'T1': 3000, 'T2': 1500, 'rho': 1.0},
    'Gray Matter':  {'T1': 950,  'T2': 100,  'rho': 0.95},
    'White Matter': {'T1': 600,  'T2': 70,   'rho': 0.85},
    'Muscle':       {'T1': 900,  'T2': 40,   'rho': 1.0},
    'Fat':          {'T1': 250,  'T2': 80,   'rho': 0.9},
}

# =============================================================
# BLOCH EQUATIONS
# =============================================================
def T1_recovery(t, T1, M0=1.0, M_init=0.0):
    """Longitudinal relaxation: M_z(t) = M0 - (M0 - M_init) * exp(-t/T1)"""
    return M0 - (M0 - M_init) * np.exp(-t / T1)

def T2_decay(t, T2, M_xy0=1.0):
    """Transverse relaxation: M_xy(t) = M_xy0 * exp(-t/T2)"""
    return M_xy0 * np.exp(-t / T2)

def T2star_decay(t, T2, T2star_factor=0.7, M_xy0=1.0):
    """Effective T2* decay (faster due to field inhomogeneity)"""
    T2star = T2 * T2star_factor
    return M_xy0 * np.exp(-t / T2star)

# =============================================================
# SPIN ECHO SIMULATION
# =============================================================
def spin_echo(t_array, TE, T2, M_xy0=1.0, gamma_B1=1.0):
    """
    Simulate spin echo: 90° → wait τ → 180° → wait τ → echo at TE=2τ
    At TE, dephased spins refocus → echo amplitude = M_xy0 * exp(-TE/T2)
    """
    M_xy = np.zeros_like(t_array)
    N = len(t_array)
    tau = TE / 2.0
    
    # Find indices
    idx_90 = 0
    idx_180 = np.argmin(np.abs(t_array - tau))
    idx_echo = np.argmin(np.abs(t_array - TE))
    
    # Before 90°: equilibrium
    M_xy[t_array < 0] = 0  # No signal at equilibrium
    
    # After 90°, before 180°: T2* dephasing (fast)
    mask_before180 = (t_array >= 0) & (t_array < tau)
    t_before = t_array[mask_before180]
    M_xy[mask_before180] = M_xy0 * np.exp(-t_before / (T2 * 0.6))
    
    # After 180°, dephasing reverses → M_xy builds up to echo
    mask_after180 = t_array >= tau
    t_after = t_array[mask_after180] - tau  # Time since 180°
    t_total = t_array[mask_after180]  # Time since 90°
    
    # Simplified: M_xy recovers to T2 envelope at echo
    M_xy[mask_after180] = M_xy0 * np.exp(-t_total / T2) * gamma_B1
    
    return M_xy

# =============================================================
# SIMULATION: T1 and T2 relaxation
# =============================================================
t_T1 = np.linspace(0, 5000, 500)  # 0-5 seconds
t_T2 = np.linspace(0, 500, 500)   # 0-500 ms

fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# =============================================================
# 1. T1 RECOVERY
# =============================================================
ax1 = axes[0, 0]
colors_t = {'CSF': 'blue', 'Gray Matter': 'red', 'White Matter': 'orange', 
            'Muscle': 'green', 'Fat': 'brown'}

for tissue, params in tissues.items():
    T1 = params['T1']
    M_z = T1_recovery(t_T1, T1, M0=1.0, M_init=0.0)
    ax1.plot(t_T1, M_z, color=colors_t[tissue], lw=2.5, label=f'{tissue} (T1={T1} ms)')

ax1.axhline(1.0, color='gray', ls='--', lw=1, label='M₀ (fully recovered)')
ax1.axhline(0.63, color='gray', ls=':', lw=1, label='63% at t=T1')
ax1.set_xlabel('Time (ms)', fontsize=12)
ax1.set_ylabel('M_z / M₀', fontsize=12)
ax1.set_title('T1 Recovery (Longitudinal Relaxation)\nM_z(t) = M₀(1 - e^(-t/T1))', fontsize=14)
ax1.legend(fontsize=9)
ax1.grid(True, alpha=0.3)
ax1.set_xlim(0, 5000)

# =============================================================
# 2. T2 DECAY
# =============================================================
ax2 = axes[0, 1]
t_T2_plot = np.linspace(0, 300, 300)

for tissue, params in tissues.items():
    T2 = params['T2']
    M_xy = T2_decay(t_T2_plot, T2, M_xy0=1.0)
    M_xy_star = T2star_decay(t_T2_plot, T2, T2star_factor=0.5, M_xy0=1.0)
    if tissue in ['CSF', 'Gray Matter', 'Fat']:
        ax2.plot(t_T2_plot, M_xy, color=colors_t[tissue], lw=2.5, label=f'{tissue} T2={T2} ms')
        ax2.plot(t_T2_plot, M_xy_star, color=colors_t[tissue], lw=1.5, ls='--', alpha=0.5)

ax2.set_xlabel('Time (ms)', fontsize=12)
ax2.set_ylabel('M_xy / M₀', fontsize=12)
ax2.set_title('T2 Decay (Transverse Relaxation)\nM_xy(t) = M₀e^(-t/T2)', fontsize=14)
ax2.legend(fontsize=9)
ax2.grid(True, alpha=0.3)
ax2.set_xlim(0, 300)

# =============================================================
# 3. SPIN ECHO at different TEs
# =============================================================
ax3 = axes[1, 0]
t_se = np.linspace(-50, 500, 550)

for T2, color, name in [(70, 'orange', 'White Matter'), (100, 'red', 'Gray Matter'), (1500, 'blue', 'CSF')]:
    for TE, ls in [(20, '-'), (80, '--'), (150, ':')]:
        M_se = spin_echo(t_se, TE, T2, M_xy0=1.0)
        ax3.plot(t_se, M_se, color=color, lw=1.5, ls=ls, 
                 label=f'{name}, TE={TE} ms' if T2 == 70 else None)

ax3.axvline(0, color='green', ls='-', lw=2, label='90° pulse')
ax3.axvline(40, color='purple', ls='--', lw=1.5, label='180° pulse (τ=20 ms)')
ax3.axvline(80, color='purple', ls='--', lw=1.5)
ax3.set_xlabel('Time (ms)', fontsize=12)
ax3.set_ylabel('M_xy / M₀', fontsize=12)
ax3.set_title('Spin Echo: M_xy vs. Time\nEcho at TE=2τ, Amplitude ∝ e^(-TE/T2)', fontsize=14)
ax3.legend(fontsize=7, ncol=2)
ax3.grid(True, alpha=0.3)

# =============================================================
# 4. T1-WEIGHTED vs T2-WEIGHTED contrast
# =============================================================
ax4 = axes[1, 1]
# T1W: TR=500ms, TE=20ms
# T2W: TR=3000ms, TE=80ms
TR_T1W, TE_T1W = 500e-3, 20e-3   # seconds
TR_T2W, TE_T2W = 3000e-3, 80e-3  # seconds

tissue_names = list(tissues.keys())
T1_vals = [tissues[t]['T1']/1000 for t in tissue_names]   # seconds
T2_vals = [tissues[t]['T2']/1000 for t in tissue_names]   # seconds

# Simplified signal model (GRE)
def signal_gre(TR, TE, T1, T2, alpha_deg=90):
    alpha = np.radians(alpha_deg)
    E1 = np.exp(-TR/T1)
    E2 = np.exp(-TE/T2)
    S = np.sin(alpha) * (1 - E1) / (1 - E1 * np.cos(alpha)) * E2
    return np.maximum(S, 0)

S_T1W = [signal_gre(TR_T1W, TE_T1W, T1, T2) for T1, T2 in zip(T1_vals, T2_vals)]
S_T2W = [signal_gre(TR_T2W, TE_T2W, T1, T2) for T1, T2 in zip(T1_vals, T2_vals)]

x_pos = np.arange(len(tissue_names))
width = 0.35
bars1 = ax4.bar(x_pos - width/2, S_T1W, width, label=f'T1W (TR={TR_T1W:.0f}s, TE={int(TE_T1W*1000)}ms)', color='royalblue', alpha=0.8)
bars2 = ax4.bar(x_pos + width/2, S_T2W, width, label=f'T2W (TR={TR_T2W:.0f}s, TE={int(TE_T2W*1000)}ms)', color='forestgreen', alpha=0.8)
ax4.set_xticks(x_pos)
ax4.set_xticklabels(tissue_names, rotation=30, ha='right', fontsize=9)
ax4.set_ylabel('Signal (normalized)', fontsize=12)
ax4.set_title('T1W vs T2W Signal Contrast\nAcross Tissue Types', fontsize=14)
ax4.legend(fontsize=9)
ax4.grid(True, alpha=0.3, axis='y')

print("=== Signal Contrast Analysis ===")
print(f"T1W: Fat (T1={tissues['Fat']['T1']}ms) is brightest → {S_T1W[-1]:.3f}")
print(f"T2W: CSF (T1={tissues['CSF']['T1']}ms, T2={tissues['CSF']['T2']}ms) is brightest → {S_T2W[0]:.3f}")

plt.tight_layout()
plt.savefig('W18_mri_bloch.png', dpi=150)
plt.show()
```

---

## Exercise 2: PET SUV Calculation and Kinetic Analysis

```python
"""
W18_CodeLab_Ex2_PET_SUV.py
=============================
Calculate standardized uptake value (SUV) from PET data.
Analyze tumor metabolic activity.
Simulate time-activity curves.
"""

import numpy as np
import matplotlib.pyplot as plt
import pandas as pd

# =============================================================
# PET TRACERS AND PARAMETERS
# =============================================================
tracers = {
    '¹⁸F-FDG':    {'halflife': 110,   'E': 'Glucose metabolism', 'injected_MBq': 370, 'W_typical': 70},
    '¹¹C-raclopride': {'halflife': 20,   'E': 'Dopamine D2 receptor', 'injected_MBQ': 370, 'W_typical': 70},
    '⁶⁸Ga-DOTATATE': {'halflife': 68,   'E': 'Somatostatin receptor', 'injected_MBq': 150, 'W_typical': 70},
}

# =============================================================
# SUV CALCULATION
# =============================================================
def SUV(activity_kBqg, injected_MBq, weight_g):
    """
    Calculate SUV.
    activity_kBqg: tissue activity concentration in kBq/g
    injected_MBq: injected dose in MBq
    weight_g: patient weight in g
    
    SUV = r_tissue / (D_inj / W)
    """
    return activity_kBqg / (injected_MBq / weight_g)

def decay_corrected_SUV(activity_kBqg, injected_MBq, weight_g, 
                        inj_time_min, scan_time_min, halflife_min):
    """
    Decay-corrected SUV accounting for radioactive decay.
    """
    # Decay factor
    delta_t = scan_time_min - inj_time_min
    decay_factor = 2 ** (-delta_t / halflife_min)
    # Corrected activity
    activity_corrected = activity_kBqg / decay_factor
    return SUV(activity_corrected, injected_MBq, weight_g)

# =============================================================
# TUMOR ANALYSIS
# =============================================================
# Simulated tumor PET data
tumors = {
    'Patient 1': {'SUV_max': 18.5, 'SUV_mean': 12.3, 'volume_mL': 45, 'location': 'Lung (NSCLC)'},
    'Patient 2': {'SUV_max': 6.2,  'SUV_mean': 4.8,  'volume_mL': 120, 'location': 'Lung (benign)'},
    'Patient 3': {'SUV_max': 8.1,  'SUV_mean': 6.5,  'volume_mL': 22, 'location': 'Breast Ca'},
    'Patient 4': {'SUV_max': 22.0, 'SUV_mean': 15.1, 'volume_mL': 8,  'location': 'Lymphoma'},
    'Patient 5': {'SUV_max': 2.8,  'SUV_mean': 2.1,  'volume_mL': 60, 'location': 'Inflammatory'},
}

# =============================================================
# PLOT: SUV analysis
# =============================================================
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

ax1 = axes[0, 0]
patients = list(tumors.keys())
SUV_max = [tumors[p]['SUV_max'] for p in patients]
SUV_mean = [tumors[p]['SUV_mean'] for p in patients]

x_pos = np.arange(len(patients))
width = 0.35
ax1.bar(x_pos - width/2, SUV_max, width, label='SUV_max', color='royalblue', alpha=0.8)
ax1.bar(x_pos + width/2, SUV_mean, width, label='SUV_mean', color='forestgreen', alpha=0.8)
ax1.axhline(2.5, color='orange', ls='--', lw=2, label='SUV=2.5 (threshold)')
ax1.axhline(4.0, color='red', ls='--', lw=2, label='SUV=4.0 (high suspicion)')
ax1.set_xticks(x_pos)
ax1.set_xticklabels(patients, rotation=30, ha='right', fontsize=9)
ax1.set_ylabel('SUV', fontsize=12)
ax1.set_title('PET SUV Analysis:\n¹⁸F-FDG Uptake by Tumor', fontsize=14)
ax1.legend(fontsize=9)
ax1.grid(True, alpha=0.3, axis='y')

# Classify
print("\n=== PET Tumor Classification ===")
for p, data in tumors.items():
    suv = data['SUV_max']
    if suv < 2.5:
        status = "Likely benign"
    elif suv < 4.0:
        status = "Indeterminate"
    else:
        status = "Suspicious for malignancy"
    print(f"  {p} ({data['location']}): SUV_max={suv:.1f} → {status}")

# =============================================================
# 2. TIME-ACTIVITY CURVE (TAC)
# =============================================================
ax2 = axes[0, 1]
t = np.linspace(0, 180, 200)  # 0-180 min

# FDG biodistribution: blood pool, muscle, liver, tumor
def blood_activity(t, A=100):
    """Blood pool FDG activity (normalized)"""
    return A * np.exp(-0.015 * t)  # Fast clearance

def muscle_activity(t, A=30):
    """Muscle FDG activity"""
    peak = 30  # min
    return A * t/peak * np.exp(1 - t/peak)

def liver_activity(t, A=60):
    """Liver FDG activity"""
    peak = 60
    return A * (1 - np.exp(-t/20)) * np.exp(-(t-peak)**2/500)

def tumor_activity(t, A=80, k_in=0.05, k_out=0.002):
    """Tumor FDG: irreversible uptake + slow washout"""
    return A * (1 - np.exp(-k_in * t)) * np.exp(-k_out * t)

ax2.plot(t, blood_activity(t), 'r-', lw=2, label='Blood pool')
ax2.plot(t, muscle_activity(t), 'g-', lw=2, label='Muscle')
ax2.plot(t, liver_activity(t), 'orange', lw=2, label='Liver')
ax2.plot(t, tumor_activity(t), 'b-', lw=2.5, label='Tumor (high uptake)')

ax2.axvline(60, color='gray', ls='--', lw=1.5, label='Scan time (60 min)')
ax2.set_xlabel('Time after injection (min)', fontsize=12)
ax2.set_ylabel('Activity (normalized)', fontsize=12)
ax2.set_title('FDG Time-Activity Curves\n(Biodistribution)', fontsize=14)
ax2.legend(fontsize=9)
ax2.grid(True, alpha=0.3)
ax2.set_xlim(0, 180)

# =============================================================
# 3. SUV vs. INJECTED DOSE / WEIGHT
# =============================================================
ax3 = axes[1, 0]
weights = np.linspace(40, 120, 100)  # 40-120 kg
injected = 370.0  # MBq FDG

# For same tissue activity (100 kBq/g)
tissue_activity = 100.0  # kBq/g
SUV_vs_weight = tissue_activity / (injected / (weights * 1000))  # weight in g

ax3.plot(weights, SUV_vs_weight, 'b-', lw=2.5)
ax3.axhline(2.5, color='orange', ls='--', lw=1.5)
ax3.axhline(4.0, color='red', ls='--', lw=1.5)
ax3.fill_between(weights, 0, 2.5, alpha=0.1, color='green', label='Benign zone (<2.5)')
ax3.fill_between(weights, 2.5, 4.0, alpha=0.1, color='orange', label='Indeterminate (2.5-4.0)')
ax3.fill_between(weights, 4.0, 20, alpha=0.1, color='red', label='Suspicious (>4.0)')
ax3.set_xlabel('Patient Weight (kg)', fontsize=12)
ax3.set_ylabel('SUV (at fixed tissue activity)', fontsize=12)
ax3.set_title('SUV Dependence on Patient Weight\n(Tissue Activity = 100 kBq/g, Injected = 370 MBq)', fontsize=13)
ax3.legend(fontsize=9)
ax3.grid(True, alpha=0.3)
ax3.set_ylim(0, 15)

print("\n=== SUV at different weights ===")
for w in [50, 70, 100]:
    suv_w = tissue_activity / (injected / (w * 1000))
    print(f"  Weight = {w} kg: SUV = {suv_w:.2f}")

# =============================================================
# 4. DECAY CORRECTION
# =============================================================
ax4 = axes[1, 1]
t_scan = np.linspace(30, 180, 100)  # Scan from 30-180 min
halflife_18F = 110  # min
injected = 370  # MBq

# Measured SUV (uncorrected for decay)
measured_SUV = 5.0 * np.ones_like(t_scan)

# Decay factor
decay_factors = 2 ** (-(t_scan - 60) / halflife_18F)  # relative to 60 min
corrected_SUV = measured_SUV / decay_factors

ax4.plot(t_scan, measured_SUV, 'b--', lw=2, label='Measured SUV (uncorrected)')
ax4.plot(t_scan, corrected_SUV, 'r-', lw=2, label='Decay-corrected SUV')
ax4.axhline(2.5, color='orange', ls='--', lw=1.5)
ax4.axvline(60, color='gray', ls=':', lw=1.5, label='Reference (60 min)')
ax4.set_xlabel('Scan time after injection (min)', fontsize=12)
ax4.set_ylabel('SUV', fontsize=12)
ax4.set_title('Decay Correction:\n¹⁸F-FDG SUV vs. Scan Time', fontsize=14)
ax4.legend(fontsize=10)
ax4.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W18_pet_suv.png', dpi=150)
plt.show()

# =============================================================
# PRINT SUMMARY TABLE
# =============================================================
print("\n=== PET SUV Summary Table ===")
print(f"{'Patient':<15} {'SUV_max':>8} {'SUV_mean':>9} {'Volume':>8} {'Location':<20} {'Classification'}")
print("-" * 80)
for p, d in tumors.items():
    suv = d['SUV_max']
    if suv < 2.5:
        cls = "Likely benign"
    elif suv < 4.0:
        cls = "Indeterminate"
    else:
        cls = "Suspicious (malignant)"
    print(f"{p:<15} {d['SUV_max']:>8.1f} {d['SUV_mean']:>9.1f} {d['volume_mL']:>8.0f} {d['location']:<20} {cls}")
```

---

## Exercise 3: Optical Imaging — OCT Depth Profile

```python
"""
W18_CodeLab_Ex3_OCT.py
=========================
Simulate Optical Coherence Tomography (OCT) A-scan.
Model low-coherence interferometry.
Compare axial resolution for different bandwidths.
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy.fft import fft, ifft, fftfreq

# =============================================================
# PARAMETERS
# =============================================================
c = 3e8           # Speed of light (m/s)
n_tissue = 1.44   # Refractive index of tissue
c_tissue = c / n_tissue  # Speed in tissue

# OCT parameters
central_lambda = 1300e-9   # 1300 nm (typical SS-OCT)
bandwidth_100 = 100e-9     # 100 nm bandwidth
bandwidth_50  = 50e-9      # 50 nm bandwidth
bandwidth_200 = 200e-9     # 200 nm bandwidth

# Depth parameters
z_max = 2e-3    # 2 mm depth
z_res_100 = 0.44 * (central_lambda**2) / (bandwidth_100 * n_tissue)
z_res_50  = 0.44 * (central_lambda**2) / (bandwidth_50 * n_tissue)
z_res_200 = 0.44 * (central_lambda**2) / (bandwidth_200 * n_tissue)

print(f"=== OCT Axial Resolution ===")
print(f"  λ₀ = {central_lambda*1e9:.0f} nm")
print(f"  Bandwidth  50 nm: δz = {z_res_50*1e6:.2f} μm")
print(f"  Bandwidth 100 nm: δz = {z_res_100*1e6:.2f} μm")
print(f"  Bandwidth 200 nm: δz = {z_res_200*1e6:.2f} μm")

# =============================================================
# LAYERS: Simulated retinal layers
# =============================================================
layers = [
    {'name': 'Cornea',         'z_start': 0e-3,   'z_end': 0.05e-3,  'reflectivity': 0.001},
    {'name': 'Aqueous humor',  'z_start': 0.05e-3, 'z_end': 0.30e-3,  'reflectivity': 0.0001},
    {'name': 'Lens (anterior)', 'z_start': 0.30e-3, 'z_end': 0.40e-3, 'reflectivity': 0.005},
    {'name': 'Vitreous',       'z_start': 0.40e-3, 'z_end': 2.00e-3,  'reflectivity': 0.00005},
    {'name': 'Retina',         'z_start': 2.00e-3, 'z_end': 2.20e-3,  'reflectivity': 0.010},
    {'name': 'Choroid',        'z_start': 2.20e-3, 'z_end': 2.50e-3,  'reflectivity': 0.008},
]

# =============================================================
# OCT A-SCAN SIMULATION
# =============================================================
def simulate_oct_ascàn(layers, z_max, central_lambda, bandwidth, n_tissue):
    """
    Simulate OCT A-scan using spectral interferometry.
    """
    N_z = 2048
    z = np.linspace(0, z_max, N_z)
    dz = z[1] - z[0]
    
    # Light source spectrum (Gaussian)
    lambda_range = np.linspace(central_lambda - 2*bandwidth, 
                               central_lambda + 2*bandwidth, N_z)
    k_range = 2*np.pi / lambda_range  # Wave numbers
    
    # Reference arm reflectivity
    R_ref = 1.0  # Normalized
    
    # Sample reflectivity profile
    R_sample = np.zeros(N_z)
    for layer in layers:
        mask = (z >= layer['z_start']) & (z < layer['z_end'])
        R_sample[mask] = layer['reflectivity']
    
    # Add some scattering noise
    R_sample += 0.00001 * np.random.randn(N_z)
    
    # Simulated interferogram (simplified)
    # I(k) ∝ R_ref + R_sample(z) + 2√(R_ref R_sample(z)) cos(2k·n·z)
    # For a single reflector at depth z₀: cos(2k·n·z₀) oscillates with k
    # FFT over k gives depth profile
    
    # Simplified: model each layer as a delta reflector
    interferogram = np.zeros(N_z)
    
    # For each k bin, compute contribution from all layers
    for i, k in enumerate(k_range):
        signal = R_ref
        for layer in layers:
            # Average reflectivity over layer
            r_eff = layer['reflectivity']
            # Phase from reference-sample path difference
            z_layer = (layer['z_start'] + layer['z_end']) / 2
            phase = 2 * k * n_tissue * z_layer
            signal += r_eff + 2 * np.sqrt(R_ref * r_eff) * np.cos(phase)
        interferogram[i] = signal
    
    # FFT to get A-scan (depth profile)
    # Use only the interferometric part (AC component)
    ac_signal = interferogram - np.mean(interferogram)
    ac_signal *= np.hanning(len(ac_signal))  # Window
    ascàn = np.abs(fft(ac_signal))
    
    # Depth axis from FFT
    k_range_sorted = np.sort(k_range)
    delta_k = np.mean(np.diff(k_range_sorted))
    z_reconstructed = 2 * np.pi / (2 * np.arange(len(ascàn)) * delta_k * n_tissue)
    
    # Take first half (positive depths)
    N_half = len(ascàn) // 2
    z_recon = z_reconstructed[:N_half]
    ascàn_plot = ascàn[:N_half]
    
    return z_recon, ascàn_plot

# =============================================================
# SIMPLIFIED OCT A-SCAN (Direct model)
# =============================================================
def simple_ascàn(layers, z_max, central_lambda, bandwidth, n_tissue):
    """
    Direct calculation: A-scan = autocorrelation of sample reflectivity
    with coherence function envelope.
    """
    N_z = 2048
    z = np.linspace(0, z_max, N_z)
    
    # Coherence function (Gaussian envelope)
    z_res = 0.44 * (central_lambda**2) / (bandwidth * n_tissue)
    sigma_z = z_res / (2 * np.sqrt(2 * np.log(2)))
    
    # Build A-scan
    ascàn = np.zeros(N_z)
    for layer in layers:
        z_layer = (layer['z_start'] + layer['z_end']) / 2
        R_layer = layer['reflectivity']
        # Gaussian envelope around each reflector
        ascàn += R_layer * np.exp(-((z - z_layer)**2) / (2 * sigma_z**2))
    
    # Add noise floor
    noise_floor = 0.00001 * np.max(ascàn)
    ascàn += noise_floor * np.random.randn(N_z)
    
    return z, ascàn

# =============================================================
# PLOT
# =============================================================
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# 1. Light source spectra
ax1 = axes[0, 0]
lambda_nm = np.linspace(1100, 1500, 500)
for bw, col, label in [(50, 'blue', 'Δλ = 50 nm'), 
                        (100, 'red', 'Δλ = 100 nm'),
                        (200, 'green', 'Δλ = 200 nm')]:
    sigma = bw / (2*np.sqrt(2*np.log(2)))
    I = np.exp(-((lambda_nm - 1300)**2) / (2 * sigma**2))
    ax1.plot(lambda_nm, I, color=col, lw=2.5, label=label)
ax1.set_xlabel('Wavelength (nm)', fontsize=12)
ax1.set_ylabel('Intensity (normalized)', fontsize=12)
ax1.set_title('OCT Light Source Spectra:\nNarrower bandwidth → Better axial resolution', fontsize=14)
ax1.legend(fontsize=10)
ax1.grid(True, alpha=0.3)

# 2. Axial resolution comparison
ax2 = axes[0, 1]
z_common = np.linspace(0, 2.5, 500) * 1e-3  # 0-2.5 mm
z_resolutions = [(50e-9, 'blue'), (100e-9, 'red'), (200e-9, 'green')]

for bw, col in z_resolutions:
    z_res = 0.44 * (central_lambda**2) / (bw * n_tissue) * 1e6  # μm
    sigma = z_res / (2*np.sqrt(2*np.log(2))
    # Point spread function (Gaussian)
    psf = np.exp(-(z_common * 1e6)**2 / (2 * sigma**2))
    ax2.plot(z_common * 1e3, psf, color=col, lw=2, label=f'Δλ={bw*1e9:.0f} nm → δz={z_res:.1f} μm')

ax2.set_xlabel('Depth (mm)', fontsize=12)
ax2.set_ylabel('PSF (normalized)', fontsize=12)
ax2.set_title('OCT Point Spread Function:\nBroader bandwidth → Better axial resolution', fontsize=14)
ax2.legend(fontsize=10)
ax2.grid(True, alpha=0.3)

# 3. A-scan through retinal model
ax3 = axes[1, 0]
z_ascàn, ascàn_signal = simple_ascàn(layers, z_max, central_lambda, bandwidth_100, n_tissue)
ax3.plot(z_ascàn * 1e3, ascàn_signal / np.max(ascàn_signal), 'b-', lw=1.0)

# Label layers
for layer in layers:
    ax3.axvline(layer['z_start'] * 1e3, color='gray', ls='--', lw=0.5)
    z_label = (layer['z_start'] + layer['z_end']) / 2
    ax3.text(z_label * 1e3, 0.9, layer['name'], fontsize=7, ha='center', rotation=90, va='top')

ax3.set_xlabel('Depth (mm)', fontsize=12)
ax3.set_ylabel('A-scan (normalized)', fontsize=12)
ax3.set_title(f'OCT A-scan through Retinal Model\nλ₀={central_lambda*1e-9:.0f} nm, Δλ={bandwidth_100*1e-9:.0f} nm', fontsize=14)
ax3.set_xlim(0, z_max * 1e3)
ax3.invert_xaxis()  # convention: near surface on right
ax3.grid(True, alpha=0.3)

# 4. Resolution comparison with tissue structures
ax4 = axes[1, 1]
structures = ['Cell nucleus', 'Mitochondria', 'Bacterial cell', 'Red blood cell', 
              'Capillary lumen', 'Retinal layer']
sizes = [5, 1, 2, 7, 50, 50]  # μm
colors_s = ['red', 'blue', 'purple', 'orange', 'green', 'gray']

ax4.barh(structures, sizes, color=colors_s, alpha=0.7, edgecolor='black')
ax4.axvline(z_res_100*1e6, color='red', ls='--', lw=2, label=f'OCT δz={z_res_100*1e6:.1f} μm')
ax4.set_xlabel('Size (μm)', fontsize=12)
ax4.set_title('OCT Resolution vs. Biological Structures\n(OCT resolves ≥ ~5 μm at 100 nm bandwidth)', fontsize=14)
ax4.legend(fontsize=10)
ax4.grid(True, alpha=0.3, axis='x')

print("\n=== OCT Resolution Analysis ===")
print(f"  Retinal layers: ~50-100 μm → OCT resolves easily")
print(f"  Cell nuclei: ~5 μm → OCT marginal")
print(f"  Mitochondria: ~1 μm → OCT cannot resolve individually")

plt.tight_layout()
plt.savefig('W18_oct_imaging.png', dpi=150)
plt.show()
```

---

## Lab Report Checklist

| Task | Description | File |
|------|-------------|------|
| ☐ Ex 1 | T1/T2 relaxation curves, spin echo, T1W vs T2W contrast | `W18_mri_bloch.png` |
| ☐ Ex 2 | PET SUV analysis, time-activity curves, decay correction | `W18_pet_suv.png` |
| ☐ Ex 3 | OCT A-scan through retinal model, resolution comparison | `W18_oct_imaging.png` |
| ☐ Written | Answer: Why does OCT use interferometry? What limits PET spatial resolution? |

---

## Extension Challenge

**Challenge 1**: Implement k-space MRI simulation. Create a simple 2D image, simulate k-space filling with Cartesian trajectories, and reconstruct using inverse FFT. Compare with and without central k-space truncation (resolution degradation).

**Challenge 2**: Simulate PET time-of-flight (TOF) reconstruction. Show how TOF information improves signal-to-noise ratio by a factor of √(D/Δx_tof) where D is the object diameter and Δx_tof is the TOF resolution (~5 cm for current systems).

**Challenge 3**: Build a simple fluorescence imaging simulator. Model excitation light penetration through tissue using the diffusion approximation, and calculate the optimal depth for NIR window II imaging (1000-1700 nm) vs. window I (650-900 nm).
