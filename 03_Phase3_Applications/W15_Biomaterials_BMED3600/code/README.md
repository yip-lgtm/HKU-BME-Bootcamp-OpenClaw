# Week 15 Code Lab — Biomaterials (Structure and Properties)
## Python Computational Biomaterials Lab

> **Dependencies**: `numpy`, `scipy`, `matplotlib`, `pandas`, `seaborn`  
> **Estimated time**: 3-4 hours  
> **Learning goal**: Analyze material properties, model polymer degradation kinetics, corrosion analysis

---

## Exercise 1: Biomaterial Property Comparison

```python
"""
W15_CodeLab_Ex1_MaterialProperties.py
=======================================
Compare mechanical, corrosion, and surface properties of common biomaterials.
"""

import numpy as np
import matplotlib.pyplot as plt
import pandas as pd

# Biomaterial property database
materials = {
    'SS316L':   {'E': 190e9, 'sigma_y': 215e6, 'rho': 8.0e3, 'corrosion': 0.3, 'H': 200e6},
    'Ti-6Al-4V': {'E': 110e9, 'sigma_y': 880e6, 'rho': 4.43e3, 'corrosion': 0.02, 'H': 350e6},
    'CoCrMo':   {'E': 210e9, 'sigma_y': 450e6, 'rho': 8.3e3, 'corrosion': 0.05, 'H': 700e6},
    'Al2O3':    {'E': 380e9, 'sigma_y': 500e6, 'rho': 4.0e3, 'corrosion': 0.001, 'H': 18e9},
    'ZrO2':     {'E': 200e9, 'sigma_y': 800e6, 'rho': 6.0e3, 'corrosion': 0.001, 'H': 12e9},
    'UHMWPE':   {'E': 0.5e9, 'sigma_y': 25e6, 'rho': 0.94e3, 'corrosion': 0, 'H': 50e6},
    'PMMA':     {'E': 2.5e9, 'sigma_y': 35e6, 'rho': 1.2e3, 'corrosion': 0, 'H': 200e6},
    'PLA':      {'E': 3.0e9, 'sigma_y': 60e6, 'rho': 1.3e3, 'corrosion': 0, 'H': 100e6},
}

# Cortical bone for reference
bone = {'E': 18e9, 'sigma_y': 120e6, 'rho': 1.9e3, 'H': 0.5e9}

# Create comparison DataFrame
data = []
for name, props in materials.items():
    data.append({
        'Material': name,
        'E (GPa)': props['E']/1e9,
        'sigma_y (MPa)': props['sigma_y']/1e6,
        'rho (g/cm³)': props['rho']/1e3,
        'Hardness (MPa)': props['H']/1e6,
        'Corrosion (mm/yr)': props['corrosion'],
        'E_mismatch': props['E']/bone['E']
    })

df = pd.DataFrame(data)
print(df.to_string(index=False))

# Plot: E vs sigma_y for all materials
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
colors = ['steelblue', 'orange', 'green', 'red', 'purple', 'brown', 'pink', 'gray']
for i, (name, row) in enumerate(df.iterrows()):
    ax1.scatter(row['E (GPa)'], row['sigma_y (MPa)'], 
                s=200, c=colors[i], label=name, edgecolors='black', linewidth=1)
ax1.scatter(bone['E']/1e9, bone['sigma_y']/1e6, s=300, c='gold', 
            marker='*', label='Cortical Bone', edgecolors='black', linewidth=1)
ax1.set_xscale('log')
ax1.set_yscale('log')
ax1.set_xlabel('Young\'s Modulus E (GPa)', fontsize=12)
ax1.set_ylabel('Yield Strength σ_y (MPa)', fontsize=12)
ax1.set_title('Biomaterial Property Space: E vs. σ_y', fontsize=14)
ax1.legend(fontsize=8, loc='lower right')
ax1.grid(True, alpha=0.3, which='both')

# Plot: Corrosion rate comparison
ax2 = axes[1]
names = [d['Material'] for d in data]
corrosion = [d['Corrosion (mm/yr)'] for d in data]
colors_bar = colors[:len(names)]
bars = ax2.bar(names, corrosion, color=colors_bar, edgecolor='black', alpha=0.8)
ax2.set_ylabel('Corrosion Rate (mm/yr)', fontsize=12)
ax2.set_title('Corrosion Resistance of Biomaterials', fontsize=14)
ax2.set_yscale('log')
ax2.axhline(0.1, color='red', ls='--', lw=1.5, label='Acceptable limit (0.1 mm/yr)')
ax2.set_xticklabels(names, rotation=45, ha='right', fontsize=9)
ax2.legend()

plt.tight_layout()
plt.savefig('W15_biomaterial_properties.png', dpi=150)
plt.show()
```

---

## Exercise 2: Polymer Degradation Kinetics

```python
"""
W15_CodeLab_Ex2_Degradation.py
================================
Model polymer degradation kinetics for PLA/PLGA.
Compare bulk erosion vs. surface erosion models.
"""

def bulk_erosion_mn(t, Mn0, kd):
    """First-order degradation of number-average molecular weight"""
    return Mn0 * np.exp(-kd * t)

def bulk_erosion_mass(t, M0, kd):
    """Mass loss model (autocatalytic second stage)"""
    # Two-stage: initial slow degradation → autocatalytic rapid loss
    t_lag = 2  # months lag phase
    t_active = t - t_lag
    t_active = np.maximum(t_active, 0)
    return M0 * np.exp(-kd * np.maximum(t_active, 0))

# PLGA degradation data (literature)
# 50:50 PLGA: kd ~ 0.8 month^-1 (fast)
# 75:25 PLGA: kd ~ 0.25 month^-1 (medium)
# 85:15 PLGA: kd ~ 0.15 month^-1 (slow)
# 100:0 PLA: kd ~ 0.05 month^-1 (very slow)

kd_values = {'50:50 PLGA': 0.8, '75:25 PLGA': 0.25, '85:15 PLGA': 0.15, 'PLA': 0.05}
Mn0 = 50000  # Initial Mn = 50 kDa
t = np.linspace(0, 18, 200)  # 0-18 months

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
for name, kd in kd_values.items():
    Mn_t = bulk_erosion_mn(t, Mn0, kd)
    ax1.plot(t, Mn_t/1000, lw=2.5, label=f'{name} (kd={kd} mo⁻¹)')
    
ax1.axhline(10000, color='red', ls='--', lw=1.5, label='Mechanical failure threshold (10 kDa)')
ax1.set_xlabel('Time (months)', fontsize=12)
ax1.set_ylabel('Molecular Weight Mn (kDa)', fontsize=12)
ax1.set_title('Polymer Degradation: Molecular Weight Loss', fontsize=14)
ax1.legend(fontsize=10)
ax1.grid(True, alpha=0.3)
ax1.set_ylim(0, 55)
ax1.set_xlim(0, 18)

# Surface erosion model (zero-order)
ax2 = axes[1]
degradation_rate = {'Polyanhydride': 0.05, 'Poly(orthoester)': 0.10, 'PLA': 0.01}
M0 = 100  # 100% mass remaining
for name, r in degradation_rate.items():
    mass = np.maximum(M0 - r * t, 0)
    ax2.plot(t, mass, lw=2.5, label=f'{name} (r={r} %/month)')

ax2.set_xlabel('Time (months)', fontsize=12)
ax2.set_ylabel('Mass Remaining (%)', fontsize=12)
ax2.set_title('Surface Erosion: Zero-Order Mass Loss', fontsize=14)
ax2.legend(fontsize=10)
ax2.grid(True, alpha=0.3)
ax2.set_ylim(0, 110)

plt.tight_layout()
plt.savefig('W15_polymer_degradation.png', dpi=150)
plt.show()

# Print degradation half-life
print("Degradation half-life (time for Mn to reach 25 kDa):")
for name, kd in kd_values.items():
    t_half = np.log(Mn0/25000) / kd
    print(f"  {name}: t_50% = {t_half:.2f} months")
```

---

## Exercise 3: Surface Energy and Contact Angle Analysis

```python
"""
W15_CodeLab_Ex3_ContactAngle.py
================================
Calculate surface energy components and predict cell adhesion.
"""

import numpy as np
import matplotlib.pyplot as plt

# Owens-Wendt-Rabel-Kaelble (OWRK) method
# γ_d = dispersive component, γ_p = polar component
# γ = γ_d + γ_p

# Literature surface energy components (mN/m)
surface_data = {
    'PTFE':    {'gd': 21.9, 'gp': 0.1},
    'PE':      {'gd': 33.0, 'gp': 0.0},
    'PS':      {'gd': 34.5, 'gp': 6.5},
    'PMMA':    {'gd': 30.1, 'gp': 11.6},
    'TiO2':    {'gd': 27.0, 'gp': 22.0},
    'HA':      {'gd': 20.0, 'gp': 40.0},
    'Cell membrane': {'gd': 25.0, 'gp': 28.0},  # approximate
}

# Water and culture medium surface tensions (mN/m)
water = {'gd': 21.8, 'gp': 51.0}
dmso = {'gd': 18.4, 'gp': 6.4}

def owens_wendt_theta(gamma_s, gamma_l):
    """Calculate contact angle from surface energies"""
    gd_s, gp_s = gamma_s['gd'], gamma_s['gp']
    gd_l, gp_l = gamma_l['gd'], gamma_l['gp']
    # γ_l(1 + cosθ) = 2(√(γ_s^d·γ_l^d) + √(γ_s^p·γ_l^p))
    rhs = 2 * (np.sqrt(gd_s * gd_l) + np.sqrt(gp_s * gp_l))
    gamma_l_total = gd_l + gp_l
    cos_theta = rhs / gamma_l_total - 1
    cos_theta = np.clip(cos_theta, -1, 1)
    theta = np.degrees(np.arccos(cos_theta))
    return theta

# Calculate contact angles
test_liquids = {'Water': water, 'DMSO': dmso}

results = []
for surf_name, gamma_s in surface_data.items():
    for liq_name, gamma_l in test_liquids.items():
        theta = owens_wendt_theta(gamma_s, gamma_l)
        results.append({'Surface': surf_name, 'Liquid': liq_name, 'θ (°)': theta})

import pandas as pd
df = pd.DataFrame(results).pivot(index='Surface', columns='Liquid', values='θ (°)')
print(df.round(1))

# Plot contact angle map
fig, ax = plt.subplots(figsize=(10, 6))
surfaces = list(surface_data.keys())
water_angles = [owens_wendt_theta(surface_data[s], water) for s in surfaces]
dmso_angles = [owens_wendt_theta(surface_data[s], dmso) for s in surfaces]

x = np.arange(len(surfaces))
width = 0.35
ax.bar(x - width/2, water_angles, width, label='Water (γ=72.8 mN/m)', color='royalblue', alpha=0.8)
ax.bar(x + width/2, dmso_angles, width, label='DMSO (γ=24.8 mN/m)', color='coral', alpha=0.8)
ax.axhline(90, color='green', ls='--', lw=1.5, label='θ = 90° (hydrophobic/hydrophilic threshold)')
ax.set_ylabel('Contact Angle θ (°)', fontsize=12)
ax.set_title('Contact Angles of Biomaterial Surfaces', fontsize=14)
ax.set_xticks(x)
ax.set_xticklabels(surfaces, rotation=45, ha='right', fontsize=9)
ax.legend(fontsize=10)
ax.grid(True, alpha=0.3, axis='y')
plt.tight_layout()
plt.savefig('W15_contact_angles.png', dpi=150)
plt.show()
```

---

## Exercise 4: Wear Analysis for Joint Replacements

```python
"""
W15_CodeLab_Ex4_WearAnalysis.py
================================
Compare wear rates for different bearing couples.
Model volumetric wear and particle generation.
"""

import numpy as np
import matplotlib.pyplot as plt

# Bearing couple properties
couples = {
    'Metal-UHMWPE':    {'k': 2.0e-6, 'H': 50e6},   # wear coefficient × 10^-6 mm³/N·m
    'HX-UHMWPE':       {'k': 0.5e-6, 'H': 50e6},   # highly crosslinked
    'Ceramic-Ceramic':  {'k': 0.05e-6, 'H': 18e9},
    'Metal-Metal':      {'k': 0.2e-6, 'H': 700e6},
}

# Simulation parameters
n_cycles = 10e6  # 10 million walking cycles (≈ 10 years)
step_length = 0.7  # m per step
F_normal = 3 * 700 * 9.81  # 3× BW = ~21 kN peak load

results = {}
for name, props in couples.items():
    # Volumetric wear rate: V = k × F × L
    V_per_step = props['k'] * F_normal * step_length  # mm³ per step
    V_total = V_per_step * n_cycles  # mm³
    results[name] = {
        'V_per_step_mm3': V_per_step,
        'V_total_mm3': V_total,
        'V_total_cm3': V_total / 1000,
    }
    # Number of particles (assuming 0.5% of wear volume becomes particles)
    n_particles = V_total * 1e12 / (0.5e-12)  # assuming 0.5 μm³ average particle volume
    results[name]['n_particles'] = n_particles

# Print results
print("=== Wear Analysis for Joint Bearing Couples ===")
print(f"Cycles: {n_cycles/1e6:.0f} million | Load: {F_normal/1000:.1f} kN")
print()
for name, r in results.items():
    print(f"{name}:")
    print(f"  Volumetric wear: {r['V_total_mm3']:.1f} mm³ = {r['V_total_cm3']:.4f} cm³")
    print(f"  Particles generated: {r['n_particles']:.2e}")

# Plot
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
names = list(results.keys())
volumes = [results[n]['V_total_mm3'] for n in names]
colors = ['orange', 'steelblue', 'green', 'gray']
bars = ax1.bar(names, volumes, color=colors, edgecolor='black', alpha=0.8)
ax1.set_ylabel('Total Volumetric Wear (mm³)', fontsize=12)
ax1.set_title(f'Wear Volume After {n_cycles/1e6:.0f} Million Cycles', fontsize=14)
ax1.set_xticklabels(names, rotation=15, ha='right')
ax1.grid(True, alpha=0.3, axis='y')

# Annotate with values
for bar, vol in zip(bars, volumes):
    ax1.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 5, 
             f'{vol:.1f}', ha='center', fontsize=10)

# Particle size distribution (simulated)
ax2 = axes[1]
sizes = np.linspace(0.1, 10, 100)  # μm
for name, props in couples.items():
    # Particle size distribution (log-normal)
    mu, sigma = 0.5, 1.0  # log-normal parameters
    pdf = (1/(sizes * sigma * np.sqrt(2*np.pi))) * np.exp(-(np.log(sizes) - mu)**2/(2*sigma**2))
    ax2.plot(sizes, pdf, lw=2.5, label=name)

ax2.set_xlabel('Particle Size (μm)', fontsize=12)
ax2.set_ylabel('Probability Density', fontsize=12)
ax2.set_title('Simulated Wear Debris Size Distribution', fontsize=14)
ax2.set_xscale('log')
ax2.legend(fontsize=10)
ax2.grid(True, alpha=0.3, which='both')

plt.tight_layout()
plt.savefig('W15_wear_analysis.png', dpi=150)
plt.show()
```

---

## Lab Report Checklist

| Task | Description | File |
|------|-------------|------|
| ☐ Ex 1 | Material property space E vs σ_y plot | `W15_biomaterial_properties.png` |
| ☐ Ex 2 | Polymer degradation curves | `W15_polymer_degradation.png` |
| ☐ Ex 3 | Contact angle analysis | `W15_contact_angles.png` |
| ☐ Ex 4 | Wear analysis comparison | `W15_wear_analysis.png` |

---

## Extension Challenge

**Challenge**: Build a material selection chart (Ashby chart) for biomaterials used in load-bearing applications. Plot specific stiffness (E/ρ) vs. specific strength (σ_y/ρ). Annotate with material class labels. Add a Pareto frontier showing optimal material classes for specific stiffness > 5 MPa/(kg/m³) AND specific strength > 100 kPa/(kg/m³).
