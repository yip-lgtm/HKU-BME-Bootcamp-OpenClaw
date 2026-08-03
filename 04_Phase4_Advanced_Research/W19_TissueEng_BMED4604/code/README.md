# Week 19 Code Lab — Tissue Engineering (BMED4604)

## Computational Tissue Engineering Lab

> **Dependencies**: `numpy`, `scipy`, `matplotlib`, `pandas`  
> **Estimated time**: 4-5 hours  
> **Learning goal**: Model growth factor release kinetics, scaffold porosity, MSC differentiation kinetics

---

## Lab Setup

```bash
pip install numpy scipy matplotlib pandas
```

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import curve_fit, least_squares
from scipy.integrate import solve_ivp, odeint
import pandas as pd
plt.style.use('seaborn-v0_8-whitegrid')
```

---

## Exercise 1: Scaffold Porosity & Pore Size Analysis

```python
"""
W19_CodeLab_Ex1_Scaffold_Design.py
==================================
Analyze scaffold design parameters: porosity, pore size, permeability.
"""

def calc_porosity(rho_scaffold, rho_material):
    """
    Calculate scaffold porosity from bulk and material densities.
    
    Parameters:
    -----------
    rho_scaffold : float
        Measured scaffold density (g/cm³)
    rho_material : float
        Fully dense (solid) material density (g/cm³)
    
    Returns:
    --------
    epsilon : float
        Porosity (fraction, 0-1)
    """
    epsilon = 1 - (rho_scaffold / rho_material)
    return epsilon

def calc_permeability_KozenyCarman(epsilon, d_pore, mu=1e-3):
    """
    Estimate scaffold permeability using Kozeny-Carman equation.
    
    K ≈ (ε³·d_pore²) / (180·(1-ε)²)
    
    For tissue engineering scaffolds:
    - epsilon: porosity (0-1)
    - d_pore: mean pore diameter (m)
    - mu: dynamic viscosity of culture medium (Pa·s), default = 1 mPa·s (water)
    
    Returns: permeability K (m⁴)
    """
    if epsilon <= 0 or epsilon >= 1:
        return np.nan
    K = (epsilon**3 * d_pore**2) / (180 * (1 - epsilon)**2)
    return K

def estimate_diffusion_depth(D_eff, cell_O2_consumption, target_viability=0.8):
    """
    Estimate maximum tissue thickness based on oxygen diffusion.
    
    Based on Fick's law and Michaelis-Menten oxygen consumption.
    Critical thickness: where O₂ concentration drops to ~10% of surface value.
    
    Parameters:
    -----------
    D_eff : float
        Effective O₂ diffusion coefficient (m²/s)
        D_O2 in tissue ≈ 1.5-2.0 × 10⁻⁹ m²/s
    cell_O2_consumption : float
        Volumetric O₂ consumption rate (mol/m³/s)
        MSC: ~10⁻⁵ mol/m³/s
    
    Returns:
    --------
    L_max : float
        Maximum diffusion-limited thickness (m)
    """
    # Critical concentration at cell layer: C_crit = 10% of surface
    C_surface = 0.2  # mol/m³ (20% O₂ in air at 37°C)
    C_crit = 0.02    # mol/m³ (10% of surface)
    
    # From steady-state diffusion: L_max = 2*D*(C_s - C_crit)/Q
    L_max = 2 * D_eff * (C_surface - C_crit) / cell_O2_consumption
    return L_max

# Material properties for common tissue engineering scaffolds
scaffold_materials = {
    'Collagen':     {'rho': 0.35, 'rho_solid': 1.3, 'E': 0.001, 'd_pore': 50e-6},   # MPa
    'Chitosan':     {'rho': 0.15, 'rho_solid': 1.75, 'E': 0.01, 'd_pore': 100e-6},  # MPa
    'PLGA 50:50':   {'rho': 0.30, 'rho_solid': 1.3, 'E': 0.1, 'd_pore': 200e-6},    # MPa
    'PCL':          {'rho': 0.50, 'rho_solid': 1.15, 'E': 0.2, 'd_pore': 300e-6},    # MPa
    'HA/TCP':       {'rho': 0.60, 'rho_solid': 3.1, 'E': 1.0, 'd_pore': 400e-6},    # GPa
    'Trabecular bone': {'rho': 0.50, 'rho_solid': 1.9, 'E': 0.5, 'd_pore': 500e-6}, # GPa
}

print("="*60)
print("SCAFFOLD DESIGN ANALYSIS")
print("="*60)

# Calculate porosity for each material
print("\n{:<20} {:>10} {:>10} {:>10} {:>15}".format(
    'Material', 'ρ (g/cm³)', 'ρ_solid', 'ε (%)', 'K (m⁴)'))
print("-"*65)

for name, props in scaffold_materials.items():
    eps = calc_porosity(props['rho'], props['rho_solid'])
    K = calc_permeability_KozenyCarman(eps, props['d_pore'])
    print("{:<20} {:>10.2f} {:>10.2f} {:>10.1f} {:>15.2e}".format(
        name, props['rho'], props['rho_solid'], eps*100, K))

# Diffusion-limited thickness analysis
print("\n" + "="*60)
print("DIFFUSION-LIMITED TISSUE THICKNESS")
print("="*60)

D_O2 = 1.5e-9  # m²/s
consumption_rates = [1e-6, 5e-6, 1e-5, 5e-5, 1e-4]  # mol/m³/s

for Q in consumption_rates:
    L = estimate_diffusion_depth(D_O2, Q)
    print(f"Q = {Q:.1e} mol/m³/s → L_max = {L*1e6:.0f} μm")

# Plot: porosity vs. permeability
eps_range = np.linspace(0.3, 0.95, 100)
for d_pore in [50e-6, 100e-6, 200e-6, 500e-6]:
    K_vals = [calc_permeability_KozenyCarman(eps, d_pore) for eps in eps_range]
    plt.loglog(eps_range*100, np.array(K_vals)*1e12, label=f'd_pore = {d_pore*1e6:.0f} μm')

plt.xlabel('Porosity (%)', fontsize=12)
plt.ylabel('Permeability K (×10⁻¹² m²)', fontsize=12)
plt.title('Scaffold Permeability vs. Porosity (Kozeny-Carman)', fontsize=14)
plt.legend(title='Pore size')
plt.grid(True, alpha=0.3, which='both')
plt.tight_layout()
plt.savefig('W19_scaffold_permeability.png', dpi=150)
plt.show()
```

---

## Exercise 2: Growth Factor Release Kinetics Modeling

```python
"""
W19_CodeLab_Ex2_GF_Kinetics.py
===============================
Model growth factor release from polymer matrices.
Compare zero-order, first-order, Higuchi, and Korsmeyer-Peppas models.
"""

def zero_order_release(t, k0, M_inf=1.0):
    """
    Zero-order release: constant rate.
    M(t) = M0 + k0*t, capped at M_inf.
    """
    M = k0 * t
    return np.minimum(M, M_inf)

def first_order_release(t, k1, M0=1.0):
    """
    First-order release: exponential decay.
    dM/dt = -k1*M → M(t) = M0*exp(-k1*t)
    """
    return M0 * (1 - np.exp(-k1 * t))

def higuchi_release(t, kH, M_inf=1.0):
    """
    Higuchi model (diffusion from planar matrix).
    M(t) = kH * sqrt(t), for sink conditions.
    """
    M = kH * np.sqrt(t)
    return np.minimum(M, M_inf)

def korsmeyer_peppas(t, kKP, n, M_inf=1.0):
    """
    Korsmeyer-Peppas model (anomalous transport).
    M(t)/M_inf = kKP * t^n
    n < 0.45: Fickian
    0.45 < n < 0.89: Anomalous
    n > 0.89: Case II
    """
    M = kKP * (t ** n)
    return np.minimum(M, M_inf)

def biphasic_release(t, M_fast, M_slow, k_fast, k_slow):
    """
    Biphasic: burst + sustained release.
    M(t) = M_fast*(1-exp(-k_fast*t)) + M_slow*(1-exp(-k_slow*t))
    """
    return M_fast * (1 - np.exp(-k_fast * t)) + M_slow * (1 - np.exp(-k_slow * t))

# Time array (days)
t = np.linspace(0, 28, 200)  # 0-28 days, typical for BMP-2 release

# BMP-2 release from PLGA microspheres (literature values)
# Simulated data: initial burst ~30%, then sustained release
M_inf = 1.0  # Normalized to 1.0

# Fit experimental data (simulated from literature)
np.random.seed(42)
M_exp = (0.35 * (1 - np.exp(-0.5 * t)) +  # Fast burst component
         0.55 * (1 - np.exp(-0.05 * t)) +  # Slow sustained component
         np.random.normal(0, 0.03, len(t)))  # Noise

# Fit Korsmeyer-Peppas to experimental data
def kp_model(t, kKP, n):
    return kp_model_inplace(t, kKP, n, M_inf)

def kp_model_inplace(t, kKP, n, M_inf):
    M = np.zeros_like(t)
    mask = t > 0
    M[mask] = kKP * (t[mask] ** n)
    return np.minimum(M, M_inf)

# Optimize Korsmeyer-Peppas
from scipy.optimize import curve_fit
try:
    popt_kp, pcov_kp = curve_fit(kp_model, t, M_exp, p0=[0.1, 0.5], bounds=([0.01, 0.1], [1.0, 1.5]))
    print(f"Korsmeyer-Peppas fit: k_KP = {popt_kp[0]:.4f}, n = {popt_kp[1]:.3f}")
    if popt_kp[1] < 0.45:
        print("  → Fickian diffusion dominant")
    elif popt_kp[1] < 0.89:
        print("  → Anomalous transport (diffusion + relaxation)")
    else:
        print("  → Case II transport dominant")
except:
    popt_kp = [0.1, 0.5]

# Figure: Compare all models
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
ax1.scatter(t, M_exp, s=10, alpha=0.5, label='Simulated data', color='gray')
ax1.plot(t, korsmeyer_peppas(t, popt_kp[0], popt_kp[1]), 'r-', lw=2,
         label=f'K-P: k={popt_kp[0]:.3f}, n={popt_kp[1]:.2f}')
ax1.plot(t, biphasic_release(t, 0.35, 0.60, 0.5, 0.05), 'b--', lw=2,
         label='Biphasic (burst + slow)')
ax1.plot(t, zero_order_release(t, 0.035, M_inf), 'g-.', lw=2,
         label='Zero-order (ideal)')
ax1.plot(t, first_order_release(t, 0.1, M_inf), 'm:', lw=2,
         label='First-order')
ax1.set_xlabel('Time (days)', fontsize=12)
ax1.set_ylabel('Cumulative Release Fraction', fontsize=12)
ax1.set_title('Growth Factor Release Kinetics', fontsize=14)
ax1.legend(fontsize=9)
ax1.grid(True, alpha=0.3)
ax1.set_xlim(0, 28)
ax1.set_ylim(0, 1.1)

# Rate plot (dM/dt)
ax2 = axes[1]
# Numerical differentiation
dt = np.diff(t)
dM_dt = np.diff(M_exp) / dt

ax2.plot(t[1:], dM_dt * 100, 'k-', lw=1.5, alpha=0.5, label='Observed rate')
ax2.plot(t[1:], np.diff(biphasic_release(t, 0.35, 0.60, 0.5, 0.05))/dt * 100, 'b--', lw=2, label='Biphasic rate')
ax2.axhline(3.5, color='g', ls='-.', lw=1.5, label='Zero-order: 3.5%/day')
ax2.set_xlabel('Time (days)', fontsize=12)
ax2.set_ylabel('Release Rate (%/day)', fontsize=12)
ax2.set_title('Release Rate Profiles', fontsize=14)
ax2.legend(fontsize=9)
ax2.grid(True, alpha=0.3)
ax2.set_xlim(0, 28)

plt.tight_layout()
plt.savefig('W19_gf_release_kinetics.png', dpi=150)
plt.show()

# Calculate key release parameters
print("\n=== BMP-2 Release Analysis ===")
print(f"Day 1 cumulative release: {biphasic_release(1, 0.35, 0.60, 0.5, 0.05)*100:.1f}%")
print(f"Day 7 cumulative release: {biphasic_release(7, 0.35, 0.60, 0.5, 0.05)*100:.1f}%")
print(f"Day 28 cumulative release: {biphasic_release(28, 0.35, 0.60, 0.5, 0.05)*100:.1f}%")
print(f"\nSustained release phase (day 7-28): {(biphasic_release(28, 0.35, 0.60, 0.5, 0.05) - biphasic_release(7, 0.35, 0.60, 0.5, 0.05))*100:.1f}%")
```

---

## Exercise 3: MSC Trilineage Differentiation Kinetics

```python
"""
W19_CodeLab_Ex3_MSC_Differentiation.py
======================================
Model MSC trilineage differentiation kinetics.
Simulate growth factor-driven lineage commitment.
"""

def exponential_growth(N0, mu, t, carrying_capacity=None):
    """
    MSC expansion: N(t) = N0 * exp(mu*t)
    With carrying capacity: N(t) = K / (1 + (K-N0)/N0 * exp(-mu*t))
    """
    if carrying_capacity is None:
        return N0 * np.exp(mu * t)
    else:
        K = carrying_capacity
        return K / (1 + (K - N0) / N0 * np.exp(-mu * t))

def osteogenesis_model(t, TGF_beta_conc=10, dex_conc=10e-9):
    """
    Simplified osteogenesis kinetics.
    
    Key markers: ALP activity (peaks day 7-14), mineralization (peaks day 14-21).
    
    Assumptions:
    - ALP ~ exp(-(t - t_peak)²/2σ²) with t_peak = 10 days
    - Mineralization ~ sigmoid starting day 7
    """
    # ALP activity (relative units)
    t_peak_ALP = 10  # days
    sigma = 3  # days
    ALP = np.exp(-((t - t_peak_ALP)**2) / (2 * sigma**2))
    
    # BMP-2 enhancement
    bmp_effect = 1 + 2 * TGF_beta_conc / 50  # Normalize to 50 ng/mL
    
    # Mineralization (sigmoid, starts day 7)
    t_onset = 7
    k_sigmoid = 0.5  # slope
    mineral = 1 / (1 + np.exp(-k_sigmoid * (t - t_onset)))
    
    # Dexamethasone enhancement (nM range: pro-osteogenic; μM: toxic)
    dex_effect = 1 + 0.5 * (dex_conc * 1e9 < 100)  # Enhanced if < 100 nM
    
    return {
        'ALP': ALP * bmp_effect,
        'Mineralization': mineral * dex_effect,
        'Collagen_I': 0.5 * ALP + 0.5 * mineral,
    }

def chondrogenesis_model(t, TGF_beta=10, cell_density=200e3):
    """
    Chondrogenesis: pellet culture, 2-5 × 10⁵ cells/pellet.
    
    Key markers: SOX9 (early, day 3-7), Aggrecan (day 7-14), Collagen II (day 14-21).
    """
    # SOX9 (transcription factor, early)
    SOX9 = np.exp(-((t - 5)**2) / (2 * 2**2))
    
    # Aggrecan (proteoglycan synthesis, mid)
    aggrecan = 1 / (1 + np.exp(-0.5 * (t - 10)))
    
    # Collagen II (late matrix)
    col2 = 1 / (1 + np.exp(-0.4 * (t - 14)))
    
    # TGF-β enhancement
    tgf_effect = 1 + TGF_beta / 50
    
    # Cell density effect (critical for chondrogenesis)
    density_effect = min(cell_density / 200e3, 1.5)
    
    return {
        'SOX9': SOX9 * tgf_effect,
        'Aggrecan': aggrecan * density_effect * tgf_effect,
        'Collagen_II': col2 * density_effect * tgf_effect,
    }

def adipogenesis_model(t, IBMX=0.5, insulin=10):
    """
    Adipogenesis: Confluence → adipogenic cocktail (IBMX, insulin, dex, indomethacin).
    """
    # Induction phase (days 0-2): IBMX active
    IBMX_phase = np.exp(-((t - 1)**2) / (2 * 1**2))
    
    # Insulin phase (sustained): adipocyte maturation
    insulin_phase = 1 / (1 + np.exp(-0.3 * (t - 10)))
    
    # PPARγ (master regulator, peaks day 4-6)
    PPAR = np.exp(-((t - 5)**2) / (2 * 2**2))
    
    # Lipid accumulation (Oil Red O)
    lipid = 1 / (1 + np.exp(-0.4 * (t - 12)))
    
    return {
        'IBMX_responsiveness': IBMX_phase,
        'Insulin_effect': insulin_phase,
        'PPARg': PPAR,
        'Lipid_droplets': lipid,
    }

# Time course
t_days = np.linspace(0, 21, 100)

# Generate differentiation curves
osteogenic = osteogenesis_model(t_days, TGF_beta_conc=10, dex_conc=10e-9)
chondrogenic = chondrogenesis_model(t_days, TGF_beta=10, cell_density=200e3)
adipogenic = adipogenesis_model(t_days)

# Plot trilineage differentiation
fig, axes = plt.subplots(1, 3, figsize=(16, 5))

# Osteogenesis
ax1 = axes[0]
ax1.plot(t_days, osteogenic['ALP'], 'b-', lw=2.5, label='ALP Activity')
ax1.plot(t_days, osteogenic['Mineralization'], 'r-', lw=2.5, label='Mineralization')
ax1.plot(t_days, osteogenic['Collagen_I'], 'g--', lw=2, label='Collagen I')
ax1.set_xlabel('Time (days)', fontsize=12)
ax1.set_ylabel('Relative Expression / Activity', fontsize=12)
ax1.set_title('Osteogenesis', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3)
ax1.set_ylim(0, 1.5)

# Chondrogenesis
ax2 = axes[1]
ax2.plot(t_days, chondrogenic['SOX9'], 'b-', lw=2.5, label='SOX9')
ax2.plot(t_days, chondrogenic['Aggrecan'], 'r-', lw=2.5, label='Aggrecan')
ax2.plot(t_days, chondrogenic['Collagen_II'], 'g--', lw=2, label='Collagen II')
ax2.set_xlabel('Time (days)', fontsize=12)
ax2.set_ylabel('Relative Expression', fontsize=12)
ax2.set_title('Chondrogenesis', fontsize=14)
ax2.legend()
ax2.grid(True, alpha=0.3)
ax2.set_ylim(0, 1.5)

# Adipogenesis
ax3 = axes[2]
ax3.plot(t_days, adipogenic['PPARg'], 'b-', lw=2.5, label='PPARγ')
ax3.plot(t_days, adipogenic['Lipid_droplets'], 'r-', lw=2.5, label='Lipid (Oil Red O)')
ax3.set_xlabel('Time (days)', fontsize=12)
ax3.set_ylabel('Relative Expression', fontsize=12)
ax3.set_title('Adipogenesis', fontsize=14)
ax3.legend()
ax3.grid(True, alpha=0.3)
ax3.set_ylim(0, 1.5)

plt.suptitle('MSC Trilineage Differentiation Kinetics', fontsize=16, y=1.02)
plt.tight_layout()
plt.savefig('W19_msc_trilineage.png', dpi=150)
plt.show()

# MSC expansion kinetics
print("\n=== MSC Expansion Kinetics ===")
N0 = 1e6  # Initial cells (from bone marrow aspirate, ~10 mL)
mu = np.log(2) / 1.5  # Doubling time = 36 hours
K_confluence = 1e8  # Confluence limit for T175 flask

t_expansion = np.linspace(0, 14, 100)
N_cells = exponential_growth(N0, mu, t_expansion, carrying_capacity=K_confluence)

print(f"Initial seeding: {N0/1e6:.1f} × 10⁶ cells")
print(f"Doubling time: 36 hours")
print(f"Day 7: {N_cells[50]/1e6:.1f} × 10⁶ cells")
print(f"Day 14: {N_cells[-1]/1e6:.1f} × 10⁶ cells")
print(f"\nCells needed for bone defect (100 × 10⁶): {100e6/N0:.0f} × expansion")
```

---

## Exercise 4: Bioreactor Shear Stress Calculation

```python
"""
W19_CodeLab_Ex4_Bioreactor.py
==============================
Calculate wall shear stress in perfusion bioreactor.
Model mechanical stimulation effects on tissue engineering.
"""

def wall_shear_cylinder(Q, r, mu=1e-3):
    """
    Wall shear stress for laminar flow in cylindrical pore.
    τ_w = (4 * μ * Q) / (π * r³)
    
    Parameters:
    -----------
    Q : float
        Volumetric flow rate (m³/s)
    r : float
        Pore radius (m)
    mu : float
        Dynamic viscosity (Pa·s), default = 1 mPa·s (medium at 37°C)
    
    Returns:
    --------
    tau_w : float
        Wall shear stress (Pa)
    """
    tau_w = (4 * mu * Q) / (np.pi * r**3)
    return tau_w

def shear_stress_rectangular(Q, w, h, mu=1e-3):
    """
    Wall shear stress for rectangular channel.
    τ_w = (6 * μ * Q) / (w * h²)  [at center of wider wall]
    """
    tau_w = (6 * mu * Q) / (w * h**2)
    return tau_w

def reynolds_number(rho, u, L, mu):
    """
    Reynolds number for flow characterization.
    Re < 2000: laminar (desired for cell culture)
    """
    return (rho * u * L) / mu

# Perfusion bioreactor parameters
Q_values = np.linspace(0.1e-6, 10e-6, 100)  # mL/min → m³/s
Q_m3s = Q_values / 1e6 / 60  # Convert mL/min to m³/s

r_pore = 100e-6  # 100 μm pore radius
mu = 1e-3  # Pa·s (water at 37°C)
rho = 1000  # kg/m³ (water)
L_scaffold = 10e-3  # 10 mm scaffold length

# Calculate shear stress for each flow rate
tau_w_values = [wall_shear_cylinder(Q, r_pore, mu) for Q in Q_m3s]

# Biological thresholds
tau_osteogenic_low = 0.01  # Pa (minimum for osteogenesis)
tau_osteogenic_optimal = 0.1  # Pa (optimal)
tau_detrimental = 1.0  # Pa (cell detachment risk)

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
ax1.semilogy(Q_values * 1e6, np.array(tau_w_values), 'b-', lw=2.5)
ax1.axhline(tau_osteogenic_low, color='green', ls='--', lw=2, label='Min osteogenic (0.01 Pa)')
ax1.axhline(tau_osteogenic_optimal, color='orange', ls='--', lw=2, label='Optimal (0.1 Pa)')
ax1.axhline(tau_detrimental, color='red', ls='--', lw=2, label='Detachment risk (1 Pa)')
ax1.fill_between(Q_values * 1e6, tau_osteogenic_low, tau_osteogenic_optimal,
                  alpha=0.2, color='green', label='Osteogenic window')
ax1.set_xlabel('Flow Rate Q (μL/min)', fontsize=12)
ax1.set_ylabel('Wall Shear Stress τ_w (Pa)', fontsize=12)
ax1.set_title('Perfusion Bioreactor: Shear Stress vs. Flow Rate', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3, which='both')

# Find optimal flow rate
idx_optimal = np.argmin(np.abs(np.array(tau_w_values) - tau_osteogenic_optimal))
Q_optimal = Q_values[idx_optimal]
print(f"Optimal flow rate for osteogenic shear: Q = {Q_optimal:.2f} μL/min")

# Shear stress map for different pore sizes
ax2 = axes[1]
r_values = [50e-6, 100e-6, 200e-6, 500e-6]  # μm → m
colors = ['b', 'g', 'orange', 'r']
Q_plot = np.linspace(0.1, 5, 100)  # μL/min

for r_val, color in zip(r_values, colors):
    tau_vals = [wall_shear_cylinder(Q/1e6/60, r_val, mu) for Q in Q_plot]
    ax2.plot(Q_plot, tau_vals, color=color, lw=2, label=f'r = {r_val*1e6:.0f} μm')

ax2.axhline(tau_osteogenic_optimal, color='orange', ls=':', lw=2)
ax2.fill_between(Q_plot, 0, tau_osteogenic_optimal, alpha=0.1, color='orange')
ax2.set_xlabel('Flow Rate Q (μL/min)', fontsize=12)
ax2.set_ylabel('Wall Shear Stress τ_w (Pa)', fontsize=12)
ax2.set_title('Effect of Pore Size on Shear Stress', fontsize=14)
ax2.legend(title='Pore radius')
ax2.grid(True, alpha=0.3)
ax2.set_ylim(0, 1.0)

plt.tight_layout()
plt.savefig('W19_bioreactor_shear.png', dpi=150)
plt.show()

# Summary table
print("\n=== Bioreactor Design Summary ===")
print(f"{'Q (μL/min)':<15} {'τ_w (Pa)':<12} {'Biological Effect':<30}")
print("-" * 60)
for Q in [0.5, 1, 2, 5, 10]:
    tau = wall_shear_cylinder(Q/1e6/60, r_pore, mu)
    if tau < tau_osteogenic_low:
        effect = "Below threshold (no osteogenic signal)"
    elif tau < tau_osteogenic_optimal:
        effect = "Sub-optimal osteogenesis"
    elif tau < tau_detrimental:
        effect = "Optimal osteogenic stimulation"
    else:
        effect = "Risk of cell detachment"
    print(f"{Q:<15.1f} {tau:<12.4f} {effect:<30}")
```

---

## Exercise 5: Tissue Engineering Proposal Cost Analysis

```python
"""
W19_CodeLab_Ex5_TE_Cost.py
===========================
Estimate cost and timeline for tissue engineering product development.
"""

def estimate_te_cost(cell_source, scaffold_type, scale='preclinical'):
    """
    Estimate development cost for tissue engineering product.
    
    Parameters:
    -----------
    cell_source : str
        'autologous_msc', 'allogeneic_msc', 'ipsc'
    scaffold_type : str
        'decellularized', 'synthetic', '3d_printed'
    scale : str
        'research', 'preclinical', 'clinical'
    """
    
    base_costs = {
        'autologous_msc': {'research': 50e3, 'preclinical': 500e3, 'clinical': 5e6},
        'allogeneic_msc': {'research': 100e3, 'preclinical': 1e6, 'clinical': 10e6},
        'ipsc': {'research': 500e3, 'preclinical': 5e6, 'clinical': 50e6},
    }
    
    scaffold_multipliers = {
        'decellularized': 1.5,
        'synthetic': 1.0,
        '3d_printed': 2.0,
    }
    
    base = base_costs[cell_source][scale]
    mult = scaffold_multipliers[scaffold_type]
    
    return base * mult

def estimate_timeline(phase, cell_source):
    """
    Estimate timeline in months for each development phase.
    """
    base_timeline = {
        'research': 12,       # Research to preclinical
        'preclinical': 24,    # IND-enabling studies
        'clinical_phase1': 18,
        'clinical_phase2': 30,
        'clinical_phase3': 48,
        'approval': 12,
    }
    
    if cell_source == 'ipsc':
        for phase_key in ['research', 'preclinical']:
            base_timeline[phase_key] *= 1.5  # iPSC takes longer
    
    if cell_source == 'allogeneic':
        base_timeline['clinical_phase1'] *= 1.3  # Immune compatibility studies
    
    return base_timeline

# Cost comparison
cell_sources = ['autologous_msc', 'allogeneic_msc', 'ipsc']
scaffold_types = ['synthetic', 'decellularized', '3d_printed']

print("="*70)
print("TISSUE ENGINEERING PRODUCT DEVELOPMENT COST ESTIMATES")
print("="*70)

for scale in ['research', 'preclinical', 'clinical']:
    print(f"\n--- {scale.upper()} SCALE ---")
    print(f"{'Cell Source':<20} {'Synthetic':<15} {'Decellularized':<15} {'3D Printed':<15}")
    print("-" * 70)
    for cell in cell_sources:
        costs = [estimate_te_cost(cell, s, scale) / 1e3 for s in scaffold_types]
        print(f"{cell:<20} ${costs[0]:>10.0f}K   ${costs[1]:>10.0f}K   ${costs[2]:>10.0f}K")

# Timeline analysis
print("\n" + "="*70)
print("DEVELOPMENT TIMELINE (months)")
print("="*70)

total_months = {}
for cell in cell_sources:
    timeline = estimate_timeline('all', cell)
    total = sum(timeline.values())
    total_months[cell] = total
    print(f"\n{cell.upper()}")
    print(f"  Research → Preclinical: {timeline['research']} months")
    print(f"  Preclinical (IND): {timeline['preclinical']} months")
    print(f"  Clinical Phase I: {timeline['clinical_phase1']} months")
    print(f"  Clinical Phase II: {timeline['clinical_phase2']} months")
    print(f"  Clinical Phase III: {timeline['clinical_phase3']} months")
    print(f"  Regulatory Review: {timeline['approval']} months")
    print(f"  TOTAL: {total} months ({total/12:.1f} years)")

print(f"\n{'='*70}")
print("KEY INSIGHT: iPSC-based products take ~2× longer than MSC-based products")
print("to reach clinical trial due to reprogramming, characterization, and safety.")
```

---

## Bonus: Diffusion-Reaction Simulation for Thick Tissue

```python
"""
W19_CodeLab_Bonus_Diffusion_Reaction.py
========================================
1D diffusion-reaction model for oxygen in thick tissue construct.
Compare static vs. perfusion culture.
"""

def diffusion_reaction_steady(L, D, Q, C_surf=0.2, C_crit=0.02):
    """
    Steady-state oxygen concentration profile.
    
    d²C/dx² = Q/D (reaction)
    
    Solution: C(x) = C_surf - (Q/D)*(L*x - x²/2)  [no perfusion]
             C(x) = C_crit + (C_surf-C_crit)*x/L  [linear, with perfusion]
    
    Returns concentration profile and maximum thickness.
    """
    x = np.linspace(0, L, 100)
    
    # No perfusion (diffusion only)
    C_no_perf = C_surf - (Q/D) * (L * x - x**2 / 2)
    
    # With perfusion (linear gradient maintained by flow)
    C_perf = C_crit + (C_surf - C_crit) * (1 - x/L)
    
    # Critical thickness without perfusion
    # C(L) = C_crit → L_crit = sqrt(2*D*(C_surf-C_crit)/Q)
    L_crit = np.sqrt(2 * D * (C_surf - C_crit) / Q)
    
    return x, C_no_perf, C_perf, L_crit

# Parameters
D = 1.5e-9  # m²/s (O₂ in tissue)
Q = 1e-5    # mol/m³/s (MSC O₂ consumption)
C_surf = 0.2  # mol/m³ (21% O₂ at surface)
C_crit = 0.02  # mol/m³ (10% of surface)

# Compare different scaffold thicknesses
thicknesses = [0.1e-3, 0.2e-3, 0.5e-3, 1e-3]  # 100 μm to 1 mm

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
colors = ['b', 'g', 'orange', 'r']
for L, color in zip(thicknesses, colors):
    x, C_np, C_p, _ = diffusion_reaction_steady(L, D, Q)
    ax1.plot(x*1e6, C_np*100, color=color, lw=2, label=f'L = {L*1e6:.0f} μm (static)')
    ax1.plot(x*1e6, C_p*100, color=color, lw=2, ls='--', label=f'L = {L*1e6:.0f} μm (perf)')

ax1.axhline(C_crit*100, color='red', ls=':', lw=2, label='Critical O₂ (2%)')
ax1.set_xlabel('Position x (μm from surface)', fontsize=12)
ax1.set_ylabel('O₂ Concentration (%)', fontsize=12)
ax1.set_title('Oxygen Profile: Static vs. Perfusion', fontsize=14)
ax1.legend(fontsize=8)
ax1.grid(True, alpha=0.3)

# Critical thickness vs. consumption rate
ax2 = axes[1]
Q_range = np.logspace(-7, -3, 100)  # mol/m³/s
L_crit_range = np.sqrt(2 * D * (C_surf - C_crit) / Q_range) * 1e6  # μm

ax2.loglog(Q_range, L_crit_range, 'b-', lw=2.5)
ax2.axhline(200, color='orange', ls='--', lw=2, label='Typical static limit: 200 μm')
ax2.axhline(500, color='green', ls='--', lw=2, label='Perfusable limit: 500 μm')
ax2.fill_between(Q_range, 0, 200, alpha=0.1, color='green', label='Viable without perfusion')
ax2.fill_between(Q_range, 200, 1000, alpha=0.1, color='orange', label='Needs perfusion')
ax2.set_xlabel('O₂ Consumption Rate Q (mol/m³/s)', fontsize=12)
ax2.set_ylabel('Critical Thickness L (μm)', fontsize=12)
ax2.set_title('Diffusion Limit: Tissue Thickness vs. Metabolism', fontsize=14)
ax2.legend(fontsize=9)
ax2.grid(True, alpha=0.3, which='both')
ax2.set_xlim(1e-7, 1e-3)
ax2.set_ylim(10, 5000)

plt.tight_layout()
plt.savefig('W19_diffusion_limit.png', dpi=150)
plt.show()

print("\n=== Diffusion-Reaction Analysis ===")
print(f"Without perfusion, max tissue thickness: {np.sqrt(2*D*(C_surf-C_crit)/Q)*1e6:.0f} μm")
print(f"Perfusion enables: {500:.0f} μm+ (practical limit)")
print(f"Clinical targets (>1 mm): Require pre-vascularization!")
```

---

## Lab Report Checklist

| Task | Description | File |
|------|-------------|------|
| ☐ Ex 1 | Scaffold porosity/permeability analysis | `W19_scaffold_permeability.png` |
| ☐ Ex 2 | Growth factor release kinetics modeling | `W19_gf_release_kinetics.png` |
| ☐ Ex 3 | MSC trilineage differentiation curves | `W19_msc_trilineage.png` |
| ☐ Ex 4 | Bioreactor shear stress calculation | `W19_bioreactor_shear.png` |
| ☐ Ex 5 | Tissue engineering cost/timeline model | Console output |
| ☐ Bonus | Diffusion-reaction simulation | `W19_diffusion_limit.png` |

---

## Extension Challenge (Research-level)

**Challenge**: Model the spatial distribution of growth factor in a 3D printed scaffold with graded pore size. Use COMSOL-style finite element approach or implement your own 3D diffusion-reaction solver.

**Hint**: Start with 1D, then extend to 2D axisymmetric geometry. Consider spatially varying degradation rate (surface eroding vs. bulk eroding polymer).
