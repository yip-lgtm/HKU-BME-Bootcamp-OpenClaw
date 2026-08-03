# Week 14 Code Lab — Kinematics, Gait, and Bone Fracture

> **Dependencies**: `numpy`, `scipy`, `matplotlib`, `pandas`, `seaborn`  
> **Estimated time**: 3-4 hours  
> **Learning goal**: Process motion capture data, compute joint angles, inverse dynamics analysis, GRF analysis

---

## Lab Setup

```bash
pip install numpy scipy matplotlib pandas seaborn
```

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.signal import butter, filtfilt, savgol_filter
from scipy.interpolate import interp1d
import pandas as pd
plt.style.use('seaborn-v0_8-whitegrid')
```

---

## Exercise 1: Generate and Analyze the Gait Cycle

```python
"""
W14_CodeLab_Ex1_GaitCycle.py
================================
Generate a synthetic gait cycle from GRF data and analyze temporal parameters.
"""

# Gait cycle timing (normal walking, v = 1.4 m/s)
# Total stride time: T_stride = 60/55 ≈ 1.09 s (55 steps/min = 27.5 strides/min)
T_stride = 1.09  # seconds per stride
dt = 0.001  # 1000 Hz sampling
t = np.arange(0, T_stride, dt)
n_samples = len(t)

# Gait cycle percentages
pct_stance = 0.60  # 60% stance
pct_swing = 0.40  # 40% swing
pct_loading = 0.12  # 0-12%
pct_midstance = 0.19  # 12-31%
pct_terminal_stance = 0.19  # 31-50%
pct_preswing = 0.12  # 50-62%
pct_init_swing = 0.13  # 62-75%
pct_mid_swing = 0.12  # 75-87%
pct_term_swing = 0.13  # 87-100%

BW = 700  # N (70 kg person × 10 m/s²)
m_body = 70  # kg

# Vertical GRF profile (normalized to BW)
# Piecewise cubic interpolation for smooth curve
pct_points = [0.00, 0.02, 0.06, 0.10, 0.12, 0.20, 0.31, 0.40, 0.50, 
              0.52, 0.60, 0.62, 0.70, 0.80, 0.90, 1.00]
grf_vertical_norm = [1.00, 1.10, 1.20, 1.10, 1.05, 0.95, 0.85, 0.90, 1.00,
                     1.20, 1.05, 0.10, 0.05, 0.05, 0.05, 0.02]

# Interpolate
grf_interp = interp1d(pct_points, grf_vertical_norm, kind='cubic', fill_value='extrapolate')
grf_z_norm = grf_interp(t / T_stride) * BW

# Anterior-posterior GRF
ap_brake = -0.12 * BW * np.sin(np.pi * t / T_stride * 2)  # braking
ap_prop = 0.18 * BW * np.sin(np.pi * t / T_stride * 2 + np.pi/2)  # propulsive
grf_ap = np.where(t/T_stride < 0.50, ap_brake, ap_prop)

# Plot
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

ax1 = axes[0, 0]
ax1.plot(t, grf_z_norm, 'b-', lw=2.5, label='Vertical GRF')
ax1.axhline(BW, color='gray', ls='--', lw=1, label=f'BW = {BW} N')
ax1.axvline(0.12*T_stride, color='green', ls=':', lw=1.5, label='Loading → Midstance')
ax1.axvline(0.50*T_stride, color='orange', ls=':', lw=1.5, label='Midstance → Preswing')
ax1.axvline(0.62*T_stride, color='red', ls=':', lw=1.5, label='Stance → Swing')
ax1.set_xlabel('Time (s)', fontsize=12)
ax1.set_ylabel('Vertical GRF (N)', fontsize=12)
ax1.set_title('Vertical Ground Reaction Force — Normal Walking', fontsize=14)
ax1.legend(fontsize=9)
ax1.grid(True, alpha=0.3)

ax2 = axes[0, 1]
ax2.plot(t, grf_ap, 'r-', lw=2.5)
ax2.axhline(0, color='gray', ls='--', lw=1)
ax2.fill_between(t, 0, grf_ap, where=grf_ap<0, alpha=0.3, color='red', label='Braking')
ax2.fill_between(t, 0, grf_ap, where=grf_ap>0, alpha=0.3, color='green', label='Propulsive')
ax2.set_xlabel('Time (s)', fontsize=12)
ax2.set_ylabel('A-P GRF (N)', fontsize=12)
ax2.set_title('Anterior-Posterior Ground Reaction Force', fontsize=14)
ax2.legend()
ax2.grid(True, alpha=0.3)

# Gait phase annotations
ax3 = axes[1, 0]
phases = ['IC', 'LR', 'MSt', 'TSt', 'PSw', 'ISw', 'MSw', 'TSw']
phase_times = [0, 0.06, 0.22, 0.41, 0.56, 0.69, 0.81, 0.94]
colors = ['#4CAF50']*5 + ['#2196F3']*3
for i, (pt, ph, col) in enumerate(zip(phase_times, phases, colors)):
    ax3.axvspan(pt*T_stride if i==0 else (phase_times[i-1]+pt)*T_stride/2, 
                pt*T_stride, alpha=0.3, color=col)
    ax3.text(pt*T_stride, 0.5, ph, ha='center', fontsize=8, fontweight='bold')
ax3.plot(t, grf_z_norm/BW, 'b-', lw=2)
ax3.set_xlabel('Time (s)', fontsize=12)
ax3.set_ylabel('GRF / BW', fontsize=12)
ax3.set_title('Gait Cycle Phases', fontsize=14)
ax3.set_ylim(0, 1.5)

# Cadence and stride analysis
ax4 = axes[1, 1]
speeds = [0.8, 1.0, 1.2, 1.4, 1.6, 1.8, 2.0, 2.5, 3.0]  # m/s
cadences = [85, 95, 105, 110, 120, 130, 140, 160, 180]  # steps/min
stride_lengths = [v * 60 / c * 2 for v, c in zip(speeds, cadences)]
ax4.plot(speeds, cadences, 'bo-', lw=2, ms=6, label='Cadence')
ax4.set_xlabel('Walking Speed (m/s)', fontsize=12)
ax4.set_ylabel('Cadence (steps/min)', fontsize=12)
ax4.set_title('Cadence vs. Walking Speed', fontsize=14)
ax4.grid(True, alpha=0.3)

ax4_twin = ax4.twinx()
ax4_twin.plot(speeds, stride_lengths, 'rs-', lw=2, ms=6, label='Stride Length')
ax4_twin.set_ylabel('Stride Length (m)', fontsize=12)
ax4_twin.legend(loc='lower right')

plt.tight_layout()
plt.savefig('W14_gait_cycle_analysis.png', dpi=150)
plt.show()

print(f"\n=== Gait Parameters ===")
print(f"Stride time: {T_stride:.2f} s")
print(f"Stance time: {T_stride*pct_stance:.3f} s ({(pct_stance)*100:.0f}%)")
print(f"Swing time: {T_stride*pct_swing:.3f} s ({(pct_swing)*100:.0f}%)")
print(f"Double support time: {T_stride*0.20:.3f} s")
print(f"GRF first peak: {grf_z_norm.max():.1f} N = {grf_z_norm.max()/BW:.2f}×BW")
```

---

## Exercise 2: Joint Angle Computation from Marker Trajectories

```python
"""
W14_CodeLab_Ex2_JointAngles.py
================================
Simulate hip, knee, and ankle joint angles during the gait cycle.
Use rotation matrices and Euler angle decomposition.
"""

def rotation_matrix_2D(theta_deg):
    """2D rotation matrix for sagittal plane analysis"""
    theta = np.radians(theta_deg)
    c, s = np.cos(theta), np.sin(theta)
    return np.array([[c, -s], [s, c]])

def joint_angle_from_markers(p_prox, p_mid, p_dist, axis='sagittal'):
    """
    Compute joint angle from 3 markers (proximal, mid-joint, distal).
    Positive = extension for hip/knee, dorsiflexion for ankle.
    """
    # Segment 1: proximal to mid (reference segment)
    v1 = p_mid - p_prox
    # Segment 2: mid to distal
    v2 = p_dist - p_mid
    
    # Normalize
    v1_u = v1 / np.linalg.norm(v1)
    v2_u = v2 / np.linalg.norm(v2)
    
    # Angle between vectors
    cos_angle = np.clip(np.dot(v1_u, v2_u), -1.0, 1.0)
    angle = np.degrees(np.arccos(cos_angle))
    
    # Determine sign (positive = extension, negative = flexion)
    # Use cross product sign in 2D
    cross = v1_u[0]*v2_u[1] - v1_u[1]*v2_u[0]
    angle = angle if cross >= 0 else -angle
    
    return angle

# Simulate marker trajectories for one stride (simplified sinusoidal)
T_stride = 1.09  # s
N = 200
t_gait = np.linspace(0, 1, N)  # 0 to 100% of gait cycle

# Hip angle (sagittal): flexion during swing, extension during stance
# Range: -30° (flexion) to +20° (extension) during walking
hip_angle = -15 * np.sin(2*np.pi * t_gait + np.pi/2) + 5  # degrees
hip_angle = np.where(t_gait < 0.6, -10 + 25*t_gait/0.6, 15 - 10*(t_gait-0.6)/0.4)  # simplified

# Knee angle: flexed during swing, extended during stance
# Range: 0-70°
knee_angle = np.where(t_gait < 0.6, -5*t_gait/0.6, 40*np.sin(np.pi*(t_gait-0.6)/0.4))

# Ankle angle: plantarflexed at heel strike, dorsiflexed during stance
# Range: -10° (plantar) to +15° (dorsi)
ankle_angle = -5 + 20*t_gait  # simplified linear
ankle_angle = np.where(ankle_angle > 15, 15, ankle_angle)
ankle_angle = np.where(t_gait > 0.62, -10*np.ones_like(t_gait), ankle_angle)

fig, axes = plt.subplots(3, 1, figsize=(14, 9))

joints = [('Hip', hip_angle, '#1f77b4'), 
          ('Knee', knee_angle, '#ff7f0e'),
          ('Ankle', ankle_angle, '#2ca02c')]

for ax, (name, angles, color) in zip(axes, joints):
    ax.plot(t_gait*100, angles, color=color, lw=2.5, label=f'{name} angle')
    ax.fill_between(t_gait*100, angles, alpha=0.2, color=color)
    ax.axhline(0, color='gray', ls='--', lw=1)
    ax.axvline(60, color='red', ls=':', lw=1.5, label='Stance/Swing transition')
    ax.set_ylabel(f'{name} Angle (°)', fontsize=12)
    ax.set_title(f'{name} Joint Angle — Sagittal Plane', fontsize=14)
    ax.grid(True, alpha=0.3)
    ax.legend(fontsize=10)
    ax.set_xlim(0, 100)
    
    # Mark key gait events
    for pct, event in [(0, 'HS'), (12, 'FF'), (31, 'HO'), (50, 'TO'), (62, 'HS')]:
        ax.axvline(pct, color='gray', ls=':', lw=0.5, alpha=0.5)

axes[-1].set_xlabel('Gait Cycle (%)', fontsize=12)
plt.tight_layout()
plt.savefig('W14_joint_angles.png', dpi=150)
plt.show()

# Print ranges
for name, angles, _ in joints:
    print(f"{name}: min = {angles.min():.1f}°, max = {angles.max():.1f}°")
```

---

## Exercise 3: Inverse Dynamics — Joint Torque Calculation

```python
"""
W14_CodeLab_Ex3_InverseDynamics.py
===================================
Compute hip, knee, and ankle moments during walking using inverse dynamics.
Segment parameters from Winter (2009).
"""

# Body mass
m_total = 70  # kg
BW = m_total * 9.81  # N

# Segment inertial parameters (Winter 2009, 50th percentile male)
segments = {
    'foot': {'mass_frac': 0.0137, 'COM_frac': 0.50, 'r_gyr_frac': 0.475},
    'shank': {'mass_frac': 0.0433, 'COM_frac': 0.433, 'r_gyr_frac': 0.526},
    'thigh': {'mass_frac': 0.103, 'COM_frac': 0.433, 'r_gyr_frac': 0.540},
}

segment_lengths = {
    'foot': 0.25,   # m
    'shank': 0.42,  # m  
    'thigh': 0.43,  # m
}

# Segment masses
for seg in segments:
    segments[seg]['mass'] = m_total * segments[seg]['mass_frac']

# Use gait data from Ex 1 (GRF_z_norm)
# Approximate GRF as vertical only for simplicity
# Ankle: at heel, GRF ~heel marker position
# Knee: at knee marker
# Hip: at hip joint center

# Simplified: assume GRF is evenly distributed
F_GRF = grf_z_norm  # N, vertical

# Angular acceleration of segments (approximated from joint angle data)
# Approximate α = d²θ/dt²
# For simplicity, use sinusoidal approximation
N = len(t)
alpha_foot = np.zeros(N)
alpha_shank = np.zeros(N)
alpha_thigh = np.zeros(N)

# Knee angular acceleration (from knee angle trajectory)
dtheta_knee = np.gradient(knee_angle, t_gait)
alpha_knee_approx = np.gradient(dtheta_knee, t_gait)
alpha_shank = alpha_knee_approx * 10  # scaled for visualization

# Ankle moment (simplified): M = F × moment arm
# Moment arm from heel to ankle ≈ 0.05 m
ankle_moment = F_GRF * 0.05  # N·m
ankle_moment = np.where(t/T_stride < 0.62, ankle_moment, 0)  # Only during stance

# Knee moment: sum of ankle moment + shank inertia
# Simplified: M_knee ≈ 0.5 × ankle_moment (due to shank weight)
knee_moment = 0.5 * ankle_moment + 50 * alpha_shank  # N·m
knee_moment = np.where(t/T_stride < 0.62, knee_moment, 0)

# Hip moment: sum of knee moment + thigh inertia
hip_moment = 1.2 * knee_moment  # N·m
hip_moment = np.where(t/T_stride < 0.62, hip_moment, 0)

fig, axes = plt.subplots(3, 1, figsize=(14, 9))

ax1 = axes[0]
ax1.plot(t, ankle_moment, 'b-', lw=2.5, label='Ankle Dorsiflexor Moment')
ax1.axhline(0, color='gray', ls='--', lw=1)
ax1.fill_between(t, ankle_moment, alpha=0.3, color='blue')
ax1.set_ylabel('Ankle Moment (N·m)', fontsize=12)
ax1.set_title('Joint Moments During Walking (Inverse Dynamics)', fontsize=14)
ax1.grid(True, alpha=0.3)
ax1.axvline(0.62*T_stride, color='red', ls=':', lw=1.5, label='Toe-off')
ax1.legend(fontsize=10)

ax2 = axes[1]
ax2.plot(t, knee_moment, 'orange', lw=2.5, label='Knee Extensor Moment')
ax2.axhline(0, color='gray', ls='--', lw=1)
ax2.fill_between(t, knee_moment, alpha=0.3, color='orange')
ax2.set_ylabel('Knee Moment (N·m)', fontsize=12)
ax2.grid(True, alpha=0.3)
ax2.axvline(0.62*T_stride, color='red', ls=':', lw=1.5)

ax3 = axes[2]
ax3.plot(t, hip_moment, 'green', lw=2.5, label='Hip Extensor Moment')
ax3.axhline(0, color='gray', ls='--', lw=1)
ax3.fill_between(t, hip_moment, alpha=0.3, color='green')
ax3.set_ylabel('Hip Moment (N·m)', fontsize=12)
ax3.set_xlabel('Time (s)', fontsize=12)
ax3.grid(True, alpha=0.3)
ax3.axvline(0.62*T_stride, color='red', ls=':', lw=1.5)

plt.tight_layout()
plt.savefig('W14_joint_moments.png', dpi=150)
plt.show()

print(f"\n=== Peak Joint Moments ===")
print(f"Ankle: max moment = {ankle_moment.max():.1f} N·m")
print(f"Knee: max moment = {knee_moment.max():.1f} N·m")
print(f"Hip: max moment = {hip_moment.max():.1f} N·m")
```

---

## Exercise 4: Fracture Mechanics — Bone Failure Analysis

```python
"""
W14_CodeLab_Ex4_BoneFracture.py
================================
Calculate fracture risk under different loading modes.
Compare three-point bending, torsion, and axial loading.
"""

# Femoral shaft properties (mid-diaphysis)
d = 0.030  # m (30 mm diameter)
d_inner = 0.015  # m (hollow medullary canal)
L = 0.40  # m (400 mm length between supports)
E = 18e9  # Pa (cortical bone)
sigma_yield = 120e6  # Pa
sigma_ult = 200e6  # Pa
K_IC = 4e6  # Pa·√m

# Cross-sectional properties
A = np.pi/4 * (d**2 - d_inner**2)  # m²
I = np.pi/64 * (d**4 - d_inner**4)  # m⁴
Z = I / (d/2)  # section modulus, m³
J = np.pi/32 * (d**4 - d_inner**4)  # polar moment of inertia, m⁴

print(f"Cross-section: A = {A*1e6:.2f} mm², I = {I*1e12:.1f} mm⁴, Z = {Z*1e9:.3f} × 10⁻⁹ m³")

# Three-point bending: F applied at mid-span
# M_max = FL/4
F_3pt = sigma_yield * Z * 4 / L  # Force to yield
F_3pt_ult = sigma_ult * Z * 4 / L  # Force at ultimate

print(f"\n=== Three-Point Bending ===")
print(f"Yield load: F_y = {F_3pt/1000:.2f} kN")
print(f"Ultimate load: F_ult = {F_3pt_ult/1000:.2f} kN")

# Torsion: T applied
# τ_max = TR/J
R = d/2
T_ult = tau_ult * J / R  # τ_ult ≈ 65 MPa for cortical bone
tau_ult = 65e6  # Pa (ultimate shear strength)
T_yield = sigma_yield * J / (R * np.sqrt(3))  # Von Mises equivalent

print(f"\n=== Torsion ===")
print(f"Yield torque: T_y = {T_yield:.1f} N·m")
print(f"Ultimate torque: T_ult = {T_ult:.1f} N·m")

# Axial compression
F_axial = sigma_yield * A
print(f"\n=== Axial Compression ===")
print(f"Yield load: F_axial = {F_axial/1000:.1f} kN")

# Plot failure envelopes
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Interaction diagram: axial + bending
P_range = np.linspace(0, F_axial, 100)  # axial force
M_range = sigma_yield * Z * (1 - P_range/F_axial)  # allowable moment
P_normalized = P_range / F_axial
M_normalized = M_range / (sigma_yield * Z)

ax1 = axes[0]
ax1.plot(M_normalized, P_normalized, 'b-', lw=2.5, label='Interaction Curve (yield)')
ax1.fill_between(M_normalized, P_normalized, alpha=0.2, color='blue')
ax1.plot([0, 1], [1, 0], 'r--', lw=1.5, label='Linear interaction')
ax1.set_xlabel('M / M_yield (normalized)', fontsize=12)
ax1.set_ylabel('P / P_yield (normalized)', fontsize=12)
ax1.set_title('Axial-Bending Interaction Diagram for Femoral Shaft', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3)
ax1.set_xlim(0, 1.2)
ax1.set_ylim(0, 1.2)
ax1.set_aspect('equal')

# Failure mode comparison
ax2 = axes[1]
loading_modes = ['3-Point\nBending', 'Torsion', 'Axial\nCompression']
loads_yield = [F_3pt/1000, T_yield/(d/2)*1000, F_axial/1000]
loads_ult = [F_3pt_ult/1000, T_ult/(d/2)*1000, sigma_ult*A/1000]

x = np.arange(len(loading_modes))
width = 0.35
bars1 = ax2.bar(x - width/2, loads_yield, width, label='Yield', color='orange', alpha=0.8)
bars2 = ax2.bar(x + width/2, loads_ult, width, label='Ultimate', color='red', alpha=0.8)
ax2.set_ylabel('Load (kN or kN-equivalent)', fontsize=12)
ax2.set_title('Femoral Shaft Failure Loads by Loading Mode', fontsize=14)
ax2.set_xticks(x)
ax2.set_xticklabels(loading_modes)
ax2.legend()
ax2.grid(True, alpha=0.3, axis='y')

plt.tight_layout()
plt.savefig('W14_bone_fracture_modes.png', dpi=150)
plt.show()
```

---

## Exercise 5: COM Trajectory and Energy Analysis

```python
"""
W14_CodeLab_Ex5_COM_Energy.py
===============================
Compute center of mass (COM) trajectory and mechanical energy during walking.
"""

# COM vertical trajectory (simplified sinusoidal approximation)
# Amplitude: ~3 cm vertical, ~2 cm lateral
T_stride = 1.09  # s
t_energy = np.linspace(0, T_stride, 200)
m = 70  # kg

h0 = 1.0  # COM height at midstance, m
A_vert = 0.03  # 3 cm amplitude
A_lat = 0.02   # 2 cm lateral

# COM height oscillates: peaks at midstance (single support)
h_COM = h0 + A_vert * np.sin(2 * np.pi * t_energy / T_stride)

# COM lateral: peaks at single support of each leg
x_lat = A_lat * np.sin(2 * np.pi * t_energy / T_stride * 2 + np.pi/2)

# COM velocity
v_z = np.gradient(h_COM, t_energy[1] - t_energy[0])
v_x = np.gradient(x_lat, t_energy[1] - t_energy[0])
v_total = np.sqrt(v_x**2 + v_z**2)

# Mechanical energy
KE = 0.5 * m * v_total**2
PE = m * 9.81 * h_COM
E_total = KE + PE

# Energy at reference (midstance)
h_ref = h_COM.mean()
PE_ref = m * 9.81 * h_ref
E_ref = KE + PE_ref

# Plot
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

ax1 = axes[0, 0]
ax1.plot(t_energy, h_COM * 100, 'b-', lw=2.5, label='COM vertical')
ax1.axhline(h0 * 100, color='gray', ls='--', lw=1, label=f'Mean: {h0*100:.0f} cm')
ax1.set_xlabel('Time (s)', fontsize=12)
ax1.set_ylabel('COM Height (cm)', fontsize=12)
ax1.set_title('COM Vertical Trajectory During Walking', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3)

ax2 = axes[0, 1]
ax2.plot(t_energy, KE, 'r-', lw=2.5, label='Kinetic Energy')
ax2.plot(t_energy, PE - PE.min(), 'b-', lw=2.5, label='Potential Energy (offset)')
ax2.plot(t_energy, E_total - E_total.min(), 'g-', lw=2.5, label='Total Energy')
ax2.set_xlabel('Time (s)', fontsize=12)
ax2.set_ylabel('Energy (J)', fontsize=12)
ax2.set_title('Mechanical Energy of COM During Walking', fontsize=14)
ax2.legend()
ax2.grid(True, alpha=0.3)

ax3 = axes[1, 0]
ax3.plot(t_energy, v_total, 'purple', lw=2.5)
ax3.axhline(v_total.mean(), color='gray', ls='--', lw=1, label=f'Mean: {v_total.mean():.2f} m/s')
ax3.set_xlabel('Time (s)', fontsize=12)
ax3.set_ylabel('COM Speed (m/s)', fontsize=12)
ax3.set_title('COM Velocity During Walking', fontsize=14)
ax3.legend()
ax3.grid(True, alpha=0.3)

# Energy recovery calculation
delta_E_stored = (E_total.max() - E_total.min())
delta_KE_total = 2 * np.trapz(KE, t_energy)
delta_PE_total = 2 * np.trapz(PE - PE.min(), t_energy)
R_recovery = delta_E_stored / (delta_KE_total + delta_PE_total) * 100 if (delta_KE_total + delta_PE_total) > 0 else 0

ax4 = axes[1, 1]
ax4.plot(t_energy, KE, 'r-', lw=2, label='KE')
ax4.plot(t_energy, PE - PE.min(), 'b-', lw=2, label='PE (offset)')
ax4.fill_between(t_energy, KE, PE - PE.min(), where=(KE > PE - PE.min()), 
                  alpha=0.3, color='green', label='Energy exchange')
ax4.fill_between(t_energy, KE, PE - PE.min(), where=(KE < PE - PE.min()), 
                  alpha=0.3, color='orange')
ax4.set_xlabel('Time (s)', fontsize=12)
ax4.set_ylabel('Energy (J)', fontsize=12)
ax4.set_title(f'Energy Exchange (R ≈ {R_recovery:.0f}%)', fontsize=14)
ax4.legend()
ax4.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W14_COM_energy.png', dpi=150)
plt.show()

print(f"\n=== COM Energy Analysis ===")
print(f"COM height range: {h_COM.min()*100:.2f} to {h_COM.max()*100:.2f} cm")
print(f"Peak COM speed: {v_total.max():.3f} m/s")
print(f"Max KE: {KE.max():.2f} J")
print(f"Max PE variation: {(PE.max()-PE.min()):.2f} J")
print(f"Energy recovery: R ≈ {R_recovery:.1f}%")
print(f"Metabolic cost: ~{0.05*70*1.4:.1f} W = ~{0.05*70*1.4*4.184:.0f} cal/min")
```

---

## Lab Report Checklist

| Task | Description | File |
|------|-------------|------|
| ☐ Ex 1 | Gait cycle GRF profiles | `W14_gait_cycle_analysis.png` |
| ☐ Ex 2 | Hip/knee/ankle joint angles | `W14_joint_angles.png` |
| ☐ Ex 3 | Joint moment time series | `W14_joint_moments.png` |
| ☐ Ex 4 | Bone fracture failure modes | `W14_bone_fracture_modes.png` |
| ☐ Ex 5 | COM trajectory and energy | `W14_COM_energy.png` |

---

## Extension Challenge

**Challenge**: Load real gait data from a public dataset (e.g., ODP database or SimTK) and reproduce the joint angle and moment analysis. Calculate symmetry index SI = 2|X_left - X_right|/(|X_left| + |X_right|) × 100% for peak moments. Normal gait SI < 10%.
