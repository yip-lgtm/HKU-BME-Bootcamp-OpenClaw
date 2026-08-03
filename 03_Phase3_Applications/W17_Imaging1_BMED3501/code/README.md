# Week 17 Code Lab — Medical Imaging I (X-ray, CT, Ultrasound)
## Python Computational Medical Imaging Lab

> **Dependencies**: `numpy`, `scipy`, `matplotlib`, `skimage` (optional)  
> **Estimated time**: 3-4 hours  
> **Learning goal**: Simulate X-ray attenuation, CT reconstruction, ultrasound A-mode echo

---

## Exercise 1: X-ray Attenuation (Lambert-Beer Law)

```python
"""
W17_CodeLab_Ex1_XrayAttenuation.py
=====================================
Simulate X-ray attenuation through tissue layers using Lambert-Beer law.
Calculate Hounsfield units for different tissues.
Demonstrate beam hardening effect.
"""

import numpy as np
import matplotlib.pyplot as plt

# =============================================================
# PARAMETERS: Linear attenuation coefficients (at 60 keV)
# =============================================================
mu_db = {
    'Air':       0.0003,   # cm^-1
    'Lung':      0.18,      # cm^-1 (approximately)
    'Fat':       0.20,      # cm^-1
    'Water':     0.20,      # cm^-1 (reference)
    'Muscle':    0.22,      # cm^-1
    'Blood':     0.23,      # cm^-1
    'Bone':      0.65,      # cm^-1
    'Iodine':    2.50,      # cm^-1 (contrast agent, 60 keV)
    'Barium':    3.20,      # cm^-1 (contrast agent)
}

def lambert_beer(I0, mu, x):
    """Lambert-Beer law: I = I0 * exp(-mu * x)"""
    return I0 * np.exp(-mu * x)

def half_value_layer(mu):
    """Calculate HVL: thickness for 50% attenuation"""
    return np.log(2) / mu

def hounsfield_units(mu_tissue, mu_water):
    """Calculate HU"""
    return 1000.0 * (mu_tissue - mu_water) / mu_water

# =============================================================
# 1. ATTENUATION CURVES
# =============================================================
I0 = 100.0  # Normalized to 100%
x_range = np.linspace(0, 20, 200)  # 0-20 cm depth

fig, axes = plt.subplots(2, 2, figsize=(14, 10))

ax1 = axes[0, 0]
tissues = ['Water', 'Muscle', 'Fat', 'Lung', 'Bone']
colors = ['blue', 'red', 'orange', 'green', 'black']
for tissue, col in zip(tissues, colors):
    mu = mu_db[tissue]
    I = lambert_beer(I0, mu, x_range)
    ax1.plot(x_range, I, color=col, lw=2.5, label=f'{tissue} (μ={mu:.2f} cm⁻¹)')

ax1.set_xlabel('Depth x (cm)', fontsize=12)
ax1.set_ylabel('X-ray Intensity I (normalized)', fontsize=12)
ax1.set_title('Lambert-Beer Attenuation:\nX-ray Intensity vs. Depth', fontsize=14)
ax1.legend(fontsize=9)
ax1.grid(True, alpha=0.3)
ax1.set_xlim(0, 20)
ax1.set_ylim(0, 105)

# =============================================================
# 2. Hounsfield Units
# =============================================================
ax2 = axes[0, 1]
tissue_list = list(mu_db.keys())
hu_values = [hounsfield_units(mu_db[t], mu_db['Water']) for t in tissue_list]
colors_hu = ['steelblue' if hu > -200 else 'orange' if hu < -200 else 'gray' for hu in hu_values]
bars = ax2.barh(tissue_list, hu_values, color=colors_hu, edgecolor='black', alpha=0.8)
ax2.axvline(0, color='black', ls='-', lw=1)
ax2.axvline(1000, color='red', ls='--', lw=1.5, label='Bone HU = +1000')
ax2.axvline(-1000, color='blue', ls='--', lw=1.5, label='Air HU = -1000')
ax2.set_xlabel('Hounsfield Units (HU)', fontsize=12)
ax2.set_title('Hounsfield Units:\nAttenuation Relative to Water', fontsize=14)
ax2.legend(fontsize=9)
ax2.grid(True, alpha=0.3, axis='x')

# Annotate key values
for tissue, hu in zip(tissue_list, hu_values):
    if abs(hu) > 50:
        ax2.text(hu + 50, tissue, f'{hu:.0f}', va='center', fontsize=9)

# =============================================================
# 3. HVL Calculation
# =============================================================
ax3 = axes[1, 0]
print("=== Half-Value Layers ===")
hvl_values = {}
for tissue, mu in mu_db.items():
    hvl = half_value_layer(mu)
    hvl_values[tissue] = hvl
    print(f"  {tissue:12s}: HVL = {hvl:.2f} cm")

hvl_tissues = ['Air', 'Lung', 'Water', 'Muscle', 'Bone']
hvl_cm = [hvl_values[t] for t in hvl_tissues]
colors_hvl = ['blue', 'lightgreen', 'cyan', 'red', 'black']
ax3.bar(hvl_tissues, hvl_cm, color=colors_hvl, edgecolor='black', alpha=0.8)
ax3.set_ylabel('Half-Value Layer (cm)', fontsize=12)
ax3.set_title('Half-Value Layer:\nThickness for 50% Attenuation', fontsize=14)
ax3.grid(True, alpha=0.3, axis='y')
for i, hvl in enumerate(hvl_cm):
    ax3.text(i, hvl + 0.2, f'{hvl:.2f}', ha='center', fontsize=10)

# =============================================================
# 4. Multi-layer attenuation
# =============================================================
ax4 = axes[1, 1]
# Simulate: X-ray through 3 cm fat + 5 cm muscle + 2 cm bone
layers = [
    ('Fat', 3.0, mu_db['Fat']),
    ('Muscle', 5.0, mu_db['Muscle']),
    ('Bone', 2.0, mu_db['Bone']),
]

x_cumulative = 0.0
I = I0
for name, thickness, mu in layers:
    x_layer = np.linspace(x_cumulative, x_cumulative + thickness, 50)
    I_layer = lambert_beer(I0, mu, x_layer - x_cumulative)
    ax4.fill_between(x_layer, I_layer, alpha=0.3, color='blue')
    ax4.plot(x_layer, I_layer, 'b-', lw=1.5)
    x_cumulative += thickness
    I = lambert_beer(I0, mu, thickness)
    ax4.axvline(x_cumulative, color='gray', ls='--', lw=1)

# Annotate layers
ax4.axvspan(0, 3, alpha=0.2, color='orange', label='Fat (3 cm)')
ax4.axvspan(3, 8, alpha=0.2, color='red', label='Muscle (5 cm)')
ax4.axvspan(8, 10, alpha=0.2, color='black', label='Bone (2 cm)')
ax4.set_xlabel('Total depth (cm)', fontsize=12)
ax4.set_ylabel('Intensity I', fontsize=12)
ax4.set_title('Multi-layer Attenuation:\nFat → Muscle → Bone', fontsize=14)
ax4.legend(fontsize=9)
ax4.grid(True, alpha=0.3)

# Final intensity
print(f"\n  Final I after 10 cm: {I:.2f}% of I₀")

plt.tight_layout()
plt.savefig('W17_xray_attenuation.png', dpi=150)
plt.show()

# =============================================================
# 5. Beam Hardening Approximation
# =============================================================
print("\n=== Beam Hardening Effect ===")
# As X-ray passes through, effective energy increases (low-energy photons absorbed faster)
energies = [40, 60, 80, 100, 120]  # keV
mu_soft = [0.35, 0.22, 0.18, 0.16, 0.15]  # cm^-1 (approximate)
mu_bone  = [1.2, 0.65, 0.45, 0.38, 0.35]   # cm^-1

for e, ms, mb in zip(energies, mu_soft, mu_bone):
    I_soft = lambert_beer(100, ms, 10)
    I_bone = lambert_beer(100, mb, 5)
    print(f"  {e} keV: Soft tissue I = {I_soft:.1f}% at 10cm, Bone I = {I_bone:.1f}% at 5cm")
```

---

## Exercise 2: CT Reconstruction — Filtered Back Projection

```python
"""
W17_CodeLab_Ex2_CTReconstruction.py
====================================
Simulate CT acquisition and reconstruction using Filtered Back Projection.
Generate sinograms and reconstruct a simple phantom.
Demonstrate the effect of ramp filter.
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy.fft import fft, ifft, fftfreq, fftshift

# =============================================================
# PARAMETERS
# =============================================================
N = 128        # Image size N×N
theta_n = 180  # Number of angles (half-scan, 180° for parallel beam)
detector_bins = 128

# =============================================================
# PHANTOM: Simple 2D object
# =============================================================
def create_phantom(N):
    """Create a simple Shepp-Logan-style phantom"""
    x = np.linspace(-1, 1, N)
    y = np.linspace(-1, 1, N)
    X, Y = np.meshgrid(x, y)
    
    phantom = np.zeros((N, N))
    
    # Large ellipse (background soft tissue HU ≈ +30)
    phantom += 0.03 * (X**2 + Y**2 < 0.8**2).astype(float)
    
    # Inner circle (dark, fat HU ≈ -80)
    phantom += (-0.11) * (X**2 + (Y+0.2)**2 < 0.3**2).astype(float)
    
    # Small circle (bright, bone HU ≈ +800)
    phantom += 0.77 * (X**2 + (Y-0.4)**2 < 0.12**2).astype(float)
    
    # Another ellipse (muscle HU ≈ +40)
    phantom += 0.04 * (((X-0.3)*2)**2 + Y**2 < 0.15**2).astype(float)
    
    # Add Gaussian noise
    phantom += 0.01 * np.random.randn(N, N)
    
    return phantom

phantom = create_phantom(N)

# =============================================================
# RADON TRANSFORM (Forward Projection)
# =============================================================
def radon_transform(image, n_angles=180):
    """Simple Radon transform using linear interpolation"""
    N = image.shape[0]
    sinogram = np.zeros((n_angles, N))
    angles = np.linspace(0, np.pi, n_angles, endpoint=False)
    
    for i, theta in enumerate(angles):
        # Rotate image by -theta and sum rows
        rotated = np.roll(image, N//2, axis=0)
        rotated = np.roll(rotated, N//2, axis=1)
        sinogram[i] = rotated.sum(axis=1)
    
    return sinogram

def simple_radon(image, n_angles=180):
    """Parallel-beam Radon transform"""
    N = image.shape[0]
    sinogram = np.zeros((n_angles, N))
    angles = np.linspace(0, np.pi, n_angles, endpoint=False)
    
    # For each angle, project onto detector
    cx, cy = N//2, N//2
    for i, theta in enumerate(angles):
        for j in range(N):
            # Detector position (from -N/2 to N/2)
            r = (j - N//2)
            # Sum along line: x*cosθ + y*sinθ = r
            # Use simple backprojection-style summation
            x = np.arange(N) - cx
            y = np.arange(N) - cy
            xx, yy = np.meshgrid(x, y)
            line = (xx * np.cos(theta) + yy * np.sin(theta)) / (N//2) * (N//2)
            # Project onto detector bin j
            for m in range(N):
                for n in range(N):
                    if abs(line[m, n] - r) < 1.0:
                        sinogram[i, j] += image[m, n]
    
    return sinogram

# Faster version using rotation and projection
def radon_fast(image, n_angles=180):
    N = image.shape[0]
    sinogram = np.zeros((n_angles, N))
    angles = np.linspace(0, np.pi, n_angles, endpoint=False)
    
    for i, theta in enumerate(angles):
        # Rotate image by theta
        cx, cy = N//2, N//2
        x = np.arange(N) - cx
        y = np.arange(N) - cy
        xx, yy = np.meshgrid(x, y)
        x_rot = xx * np.cos(theta) + yy * np.sin(theta)
        y_rot = -xx * np.sin(theta) + yy * np.cos(theta)
        
        # Sum over y_rot (integrate along perpendicular direction)
        for j in range(N):
            mask = (np.abs(y_rot) < 0.5) & (np.abs(x_rot - (j - N//2)) < 1.5)
            sinogram[i, j] = image[mask].sum()
    
    return sinogram

print("Computing Radon transform...")
sinogram = radon_fast(phantom, n_angles=180)

# =============================================================
# RAMP FILTER
# =============================================================
def ramp_filter(n, dx=1.0):
    """Create ramp filter in frequency domain"""
    f = fftfreq(n, dx)
    # Ram-Lak (ideal ramp)
    H = np.abs(f)
    # Apply Hamming window to reduce noise
    window = 0.54 + 0.46 * np.cos(2*np.pi * f / (2/n))
    H *= window
    return fftshift(H)

def filtered_back_projection(sinogram):
    """Filtered back projection reconstruction"""
    N_det = sinogram.shape[1]
    n_angles = sinogram.shape[0]
    N_recon = N_det
    
    # Create ramp filter
    H = ramp_filter(N_det)
    
    recon = np.zeros((N_recon, N_recon))
    angles = np.linspace(0, np.pi, n_angles, endpoint=False)
    
    for i, theta in enumerate(angles):
        # Fourier transform of projection
        proj = sinogram[i]
        P = fft(proj)
        
        # Apply ramp filter
        P_filtered = P * H
        
        # Inverse FT
        proj_filtered = np.real(ifft(P_filtered))
        
        # Back project
        for j in range(N_recon):
            recon[j] += proj_filtered
    
    # Normalize
    recon *= np.pi / n_angles
    return recon

print("Computing Filtered Back Projection...")
reconstruction = filtered_back_projection(sinogram)

# =============================================================
# PLOT RESULTS
# =============================================================
fig, axes = plt.subplots(2, 3, figsize=(15, 10))

ax1 = axes[0, 0]
im1 = ax1.imshow(phantom, cmap='gray', vmin=-0.2, vmax=1.0)
ax1.set_title('Original Phantom\n(Shepp-Logan style)', fontsize=12)
plt.colorbar(im1, ax=ax1, shrink=0.7)

ax2 = axes[0, 1]
ax2.imshow(sinogram, cmap='gray', aspect='auto')
ax2.set_xlabel('Detector position', fontsize=12)
ax2.set_ylabel('Projection angle (°)', fontsize=12)
ax2.set_title('Sinogram\n(Radon Transform)', fontsize=12)

ax3 = axes[0, 2]
# Show a few projections
for angle_idx in [0, 45, 90, 135]:
    ax3.plot(sinogram[angle_idx], lw=1.5, label=f'{angle_idx}°')
ax3.set_xlabel('Detector bin', fontsize=12)
ax3.set_ylabel('Projection value', fontsize=12)
ax3.set_title('Sample Projections\nat Different Angles', fontsize=12)
ax3.legend(fontsize=8)
ax3.grid(True, alpha=0.3)

ax4 = axes[1, 0]
im4 = ax4.imshow(reconstruction, cmap='gray', vmin=-0.2, vmax=1.0)
ax4.set_title('Reconstructed Image\n(FBP)', fontsize=12)
plt.colorbar(im4, ax=ax4, shrink=0.7)

ax5 = axes[1, 1]
# Profile through center
row = N_recon // 2
ax5.plot(phantom[row, :], 'b-', lw=1.5, label='Original')
ax5.plot(reconstruction[row, :], 'r--', lw=1.5, label='FBP Reconstructed')
ax5.set_xlabel('Pixel', fontsize=12)
ax5.set_ylabel('Value (normalized)', fontsize=12)
ax5.set_title('Centerline Profile:\nOriginal vs. Reconstructed', fontsize=12)
ax5.legend(fontsize=10)
ax5.grid(True, alpha=0.3)

ax6 = axes[1, 2]
# Error map
error = np.abs(phantom - reconstruction)
im6 = ax6.imshow(error, cmap='hot', vmin=0, vmax=0.1)
ax6.set_title('Reconstruction Error\n|Absolute|', fontsize=12)
plt.colorbar(im6, ax=ax6, shrink=0.7)

plt.tight_layout()
plt.savefig('W17_ct_reconstruction.png', dpi=150)
plt.show()

print(f"Max reconstruction error: {np.max(error):.4f}")
print(f"Mean reconstruction error: {np.mean(error):.4f}")
```

---

## Exercise 3: Ultrasound A-mode Echo Simulation

```python
"""
W17_CodeLab_Ex3_UltrasoundEcho.py
===================================
Simulate A-mode ultrasound echo from multi-layer tissue.
Apply TGC (time-gain compensation).
Calculate Doppler shift for blood flow.
"""

import numpy as np
import matplotlib.pyplot as plt

# =============================================================
# PARAMETERS
# =============================================================
c = 1540.0        # Speed of sound (m/s) in soft tissue
f_t = 5e6         # Transducer frequency (Hz)
n_cycles = 4       # Cycles per pulse
dt = 1e-8          # Time step (s)

# Depth of interest
z_max = 0.08      # 8 cm depth
z = np.arange(0, z_max, c*dt/2)  # Depth array
t = 2*z/c          # Time array

# =============================================================
# TISSUE LAYERS (A-mode: depth profile)
# =============================================================
layers = [
    {'z': 0.002,  'name': 'Skin',       'Z': 1.63, 'alpha': 0.5},   # 2 mm
    {'z': 0.015,  'name': 'Subcutaneous fat', 'Z': 1.34, 'alpha': 1.0},  # 13 mm
    {'z': 0.040,  'name': 'Muscle',      'Z': 1.71, 'alpha': 0.7},   # 25 mm
    {'z': 0.060,  'name': 'Bone (superficial)', 'Z': 6.80, 'alpha': 20.0}, # 20 mm
    {'z': 0.080,  'name': 'Bone (deep)',  'Z': 6.80, 'alpha': 50.0},  # 20 mm
]

# =============================================================
# REFLECTION COEFFICIENTS
# =============================================================
def reflection_coeff(Z1, Z2):
    """Pressure reflection coefficient"""
    return ((Z2 - Z1) / (Z2 + Z1))**2

# Calculate reflections at each interface
Z_background = 1.63  # Starting tissue (soft tissue)
interfaces = []
for layer in layers:
    Z_layer = layer['Z']
    R = reflection_coeff(Z_background, Z_layer)
    dB = 10 * np.log10(R) if R > 0 else -100
    interfaces.append({
        'name': layer['name'],
        'z': layer['z'],
        'Z': Z_layer,
        'R': R,
        'R_dB': dB,
        'alpha': layer['alpha'],
    })
    Z_background = Z_layer

print("=== Ultrasound Interfaces ===")
for itf in interfaces:
    print(f"  {itf['name']:25s}: z={itf['z']*1000:.0f} mm, Z={itf['Z']:.2f}, R={itf['R']:.4f} ({itf['R_dB']:.1f} dB)")

# =============================================================
# TRANSMIT PULSE
# =============================================================
def tx_pulse(t_arr, f=f_t, n=n_cycles):
    """Generate transmit pulse (sinusoid with Gaussian envelope)"""
    T = n / f  # Pulse duration
    pulse = np.sin(2*np.pi*f*t_arr) * np.exp(-(t_arr - T/2)**2 / (2*(T/10)**2))
    pulse[t_arr > T] = 0
    return pulse

t_pulse = np.linspace(0, 2e-5, 2000)
tx = tx_pulse(t_pulse)

# =============================================================
# A-MODE ECHO SIMULATION
# =============================================================
t_total = 2 * z_max / c  # Total time for full depth
N_samples = int(t_total / dt)
t_arr = np.arange(N_samples) * dt

echo = np.zeros(N_samples)

# Time-gain compensation (linear)
TGC_rate = 0.5  # dB/cm·MHz
f_MHz = f_t / 1e6
tgc = 10**(TGC_rate * f_MHz * c * t_arr / 20)  # Compensate for attenuation

for itf in interfaces:
    # Time to reach this interface
    t_interface = 2 * itf['z'] / c
    
    # Reflection amplitude (accounting for attenuation to that depth)
    attenuation_dB = itf['alpha'] * f_MHz * itf['z'] * 2  # Round-trip
    amplitude = np.sqrt(itf['R']) * 10**(-attenuation_dB/20)
    
    # Sample index
    idx = int(t_interface / dt)
    
    if idx < N_samples:
        # Create echo pulse at this time
        echo_width = int((n_cycles/f_t) / dt)
        for k in range(len(tx)):
            if idx + k < N_samples:
                echo[idx + k] += amplitude * tx[k]

# Apply TGC
echo_tgc = echo * tgc

# =============================================================
# PLOT: A-mode display
# =============================================================
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

ax1 = axes[0, 0]
ax1.plot(t_arr * 1e6, tx * 0.5, 'b-', lw=1.5, label='Transmit pulse')
ax1.set_xlabel('Time (μs)', fontsize=12)
ax1.set_ylabel('Amplitude', fontsize=12)
ax1.set_title('Transmit Pulse\n5 MHz, 4 cycles', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3)

ax2 = axes[0, 1]
ax2.plot(t_arr * 1e6, echo, 'b-', lw=0.8, label='Raw echo')
ax2.plot(t_arr * 1e6, echo_tgc, 'r-', lw=0.8, alpha=0.8, label='With TGC')
ax2.set_xlabel('Time (μs)', fontsize=12)
ax2.set_ylabel('Amplitude', fontsize=12)
ax2.set_title('A-mode Echo Signal:\nRaw vs. TGC-compensated', fontsize=14)
ax2.legend()
ax2.grid(True, alpha=0.3)

ax3 = axes[1, 0]
# Convert to depth
z_arr = c * t_arr / 2
ax3.plot(echo_tgc, z_arr * 100, 'b-', lw=1.0)
ax3.set_xlabel('Amplitude', fontsize=12)
ax3.set_ylabel('Depth (cm)', fontsize=12)
ax3.set_title('A-mode Display (TGC):\nDepth vs. Amplitude', fontsize=14)
ax3.set_xlim(-0.3, 0.3)
ax3.invert_yaxis()
ax3.grid(True, alpha=0.3)
# Label interfaces
for itf in interfaces:
    ax3.axhline(itf['z']*100, color='red', ls='--', lw=0.5)
    ax3.text(0.32, itf['z']*100, f"{itf['name']} (R={itf['R']:.3f})", fontsize=7, va='center')

# =============================================================
# DOPPLER SHIFT CALCULATION
# =============================================================
ax4 = axes[1, 1]
frequencies = [2e6, 5e6, 10e6]  # 2, 5, 10 MHz
angles = np.linspace(0, 90, 100)  # 0-90 degrees
velocities = np.linspace(0.01, 2.0, 200)  # 0-2 m/s

# Calculate Doppler shift vs. angle
v = 0.5  # m/s
colors_d = ['blue', 'red', 'green']
for f_tx, col in zip(frequencies, colors_d):
    delta_f = 2 * f_tx * v * np.cos(np.radians(angles)) / c
    ax4.plot(angles, delta_f / 1000, color=col, lw=2.5, label=f'f_t = {f_tx/1e6:.0f} MHz')

ax4.axhline(5, color='gray', ls='--', lw=1.5, label='PRF/2 Nyquist (5 kHz)')
ax4.set_xlabel('Doppler Angle θ (degrees)', fontsize=12)
ax4.set_ylabel('Doppler Shift Δf (kHz)', fontsize=12)
ax4.set_title('Doppler Shift vs. Angle\n(v = 0.5 m/s)', fontsize=14)
ax4.legend(fontsize=10)
ax4.grid(True, alpha=0.3)
ax4.set_xlim(0, 90)

print("\n=== Doppler Shift Examples ===")
for f_tx in [2e6, 5e6, 10e6]:
    for v_val, theta in [(0.5, 0), (1.0, 30), (0.5, 60)]:
        df = 2 * f_tx * v_val * np.cos(np.radians(theta)) / c
        print(f"  f_t={f_tx/1e6:.0f} MHz, v={v_val:.1f} m/s, θ={theta}°: Δf = {df/1000:.2f} kHz")

plt.tight_layout()
plt.savefig('W17_ultrasound_echo.png', dpi=150)
plt.show()
```

---

## Lab Report Checklist

| Task | Description | File |
|------|-------------|------|
| ☐ Ex 1 | X-ray attenuation curves for 5 tissues | `W17_xray_attenuation.png` |
| ☐ Ex 2 | Sinogram, FBP reconstruction, error analysis | `W17_ct_reconstruction.png` |
| ☐ Ex 3 | A-mode echo with TGC, Doppler shift plot | `W17_ultrasound_echo.png` |
| ☐ Written | Answer: Why ramp filter in FBP? What causes aliasing? |

---

## Extension Challenge

**Challenge 1**: Implement iterative SART (Simultaneous Algebraic Reconstruction Technique) for CT and compare noise vs. FBP at different numbers of projections (10, 45, 90, 180). Plot CNR vs. number of projections.

**Challenge 2**: Simulate beam hardening artifact by using a polyenergetic X-ray spectrum (sum of 10 energy bins from 30-120 keV) and computing effective μ. Show that beam hardening causes the center of a uniform phantom to appear brighter than edges.

**Challenge 3**: Implement color Doppler ultrasound simulation. Create a simulated spectral Doppler display showing laminar flow (narrow band) vs. stenosis (broadband, aliasing).
