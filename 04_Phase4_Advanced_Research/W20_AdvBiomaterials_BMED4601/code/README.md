# Week 20 Code Lab — Advanced Biomaterials (BMED4601)

## Computational Biomaterials Lab

> **Dependencies**: `numpy`, `scipy`, `matplotlib`, `pandas`  
> **Estimated time**: 4-5 hours  
> **Learning goal**: Model NP drug delivery, PEGylation half-life, biomaterial degradation

---

## Exercise 1: Nanoparticle Drug Release Modeling

```python
"""
W20_CodeLab_Ex1_NP_DrugRelease.py
===================================
Simulate drug release from polymeric nanoparticles.
Compare burst, sustained, and controlled release profiles.
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import curve_fit

def fickian_release(t, D, L, C0=1.0):
    """
    Fickian diffusion from spherical nanoparticle.
    
    For a sphere: M(t)/M_inf = 1 - (6/π²) × Σ(1/n²) × exp(-D×n²×π²×t/L²)
    
    Simplified Higuchi for early times:
    M(t)/M_inf ≈ k × sqrt(t)
    """
    # For thin film approximation
    k = 2 * np.sqrt(D / np.pi) * C0
    M = k * np.sqrt(t)
    return np.minimum(M, 1.0)

def anomalous_release(t, k1, k2, n=0.7):
    """
    Korsmeyer-Peppas anomalous (non-Fickian) release.
    M(t)/M_inf = k1*t^n + k2*t
    """
    return k1 * (t ** n) + k2 * t

def polymer_erosion_release(t, k_erosion, k_diffusion):
    """
    Combined polymer erosion + diffusion release.
    Initial burst (diffusion) + sustained (erosion).
    """
    # Burst phase (diffusion)
    M_burst = 1 - np.exp(-k_diffusion * t)
    # Erosion phase (linear-ish)
    M_erosion = 1 - np.exp(-k_erosion * t)
    # Combined (weight fractions)
    return 0.3 * M_burst + 0.65 * M_erosion

# Literature parameters for PLGA nanoparticles
D_0 = 1e-12  # Initial diffusion coefficient (m²/s)
# Degradation slows diffusion
D_app = lambda t: D_0 * np.exp(-0.01 * t)  # Slows over time

# Time array (days)
t = np.linspace(0, 28, 200)

# Experimental data (simulated from literature)
np.random.seed(42)
M_exp = (0.25 * (1 - np.exp(-1.0 * t)) +   # Burst
         0.65 * (1 - np.exp(-0.08 * t)) +   # Slow erosion
         np.random.normal(0, 0.02, len(t)))  # Noise
M_exp = np.minimum(M_exp, 1.0)

# Figure: Compare release models
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
ax1.scatter(t, M_exp, s=15, alpha=0.5, color='gray', label='Simulated data')

# Fit Korsmeyer-Peppas
def kp_model(t, k, n):
    return np.minimum(k * t**n, 1.0)

try:
    popt, pcov = curve_fit(kp_model, t, M_exp, p0=[0.1, 0.5], bounds=([0.01, 0.1], [1.0, 1.5]))
    ax1.plot(t, kp_model(t, *popt), 'r-', lw=2.5, 
             label=f'K-P: k={popt[0]:.3f}, n={popt[1]:.2f}')
    print(f"K-P fit: k={popt[0]:.4f}, n={popt[1]:.3f}")
    print(f"  → {'Fickian' if popt[1] < 0.45 else 'Anomalous' if popt[1] < 0.89 else 'Case II'}")
except:
    pass

# Erosion model
M_erosion = polymer_erosion_release(t, 0.08, 1.0)
ax1.plot(t, M_erosion, 'b--', lw=2, label='Erosion model (burst + sustained)')

ax1.set_xlabel('Time (days)', fontsize=12)
ax1.set_ylabel('Cumulative Release Fraction', fontsize=12)
ax1.set_title('PLGA Nanoparticle: Drug Release Profiles', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3)
ax1.set_xlim(0, 28)
ax1.set_ylim(0, 1.1)

# Release rate analysis
ax2 = axes[1]
dM_dt = np.gradient(M_exp, t)
ax2.plot(t, dM_dt * 100, 'k-', lw=1.5, alpha=0.7, label='Observed rate')
ax2.plot(t, np.gradient(M_erosion, t) * 100, 'b--', lw=2, label='Erosion model rate')

# Highlight burst phase
ax2.fill_between(t[t < 3], 0, np.gradient(M_erosion, t)[t < 3] * 100, 
                  alpha=0.2, color='red', label='Burst phase')
ax2.set_xlabel('Time (days)', fontsize=12)
ax2.set_ylabel('Release Rate (%/day)', fontsize=12)
ax2.set_title('Release Rate Profile', fontsize=14)
ax2.legend()
ax2.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W20_np_release.png', dpi=150)
plt.show()

# Calculate key release parameters
print("\n=== Release Kinetics Summary ===")
print(f"Day 1 release: {M_erosion[10]*100:.1f}% (burst)")
print(f"Day 7 release: {M_erosion[50]*100:.1f}%")
print(f"Day 14 release: {M_erosion[100]*100:.1f}%")
print(f"Day 28 release: {M_erosion[-1]*100:.1f}%")
```

---

## Exercise 2: PEGylation Half-Life Calculator

```python
"""
W20_CodeLab_Ex2_PEGylation.py
===============================
Calculate PEG chain properties and stealth effect.
Model circulation half-life as function of PEG size and NP core.
"""

def peg_corona_thickness(MW):
    """
    Estimate PEG corona thickness based on molecular weight.
    
    Empirical correlation (De.yml et al., 2008):
    h ≈ 0.5 × (MW)^0.5 nm
    
    For linear PEG at grafting density.
    """
    return 0.5 * np.sqrt(MW)

def peg_density_for_brush(DS):
    """
    Estimate PEG grafting density from degree of substitution.
    
    DS = mol PEG / mol surface functional groups
    σ = DS × Γ₀ (where Γ₀ = surface group density)
    """
    gamma_0 = 5e-6  # mol/m² (typical surface group density)
    sigma = DS * gamma_0  # mol/m²
    return sigma

def steric_repulsion_energy(h, chi_N):
    """
    Estimate steric repulsion energy from PEG layer.
    
    E ≈ kT × (h/l) × χN × (h/s)⁻¹
    where l = monomer length, s = grafting spacing, χN = Flory parameter
    """
    kT = 4.1e-21  # J at 37°C (298 K)
    l = 0.35e-9   # PEG monomer length (nm)
    return kT * chi_N / l

def circulation_half_life(MW_NP, MW_PEG, is_pegylated):
    """
    Estimate circulation half-life based on NP properties.
    
    Empirical model from literature:
    - Unmodified: t½ ≈ 2-5 min (rapid RES clearance)
    - PEGylated: t½ ≈ 6-72 hours (size-dependent)
    """
    if not is_pegylated:
        return 5  # minutes
    
    # PEG protection factor
    # Larger PEG = longer circulation
    protection_factor = 1 + 0.1 * MW_PEG
    
    # Core size effect (larger = longer)
    size_factor = 1 + 0.005 * MW_NP
    
    t_half = 10 * protection_factor * size_factor  # minutes
    return t_half / 60  # Convert to hours

# Calculate for different NP systems
peg_mw = [1000, 2000, 5000, 10000, 20000]  # Da
core_mw = [10000, 50000, 100000, 500000]    # Da

print("="*70)
print("PEG CHAIN PROPERTIES")
print("="*70)
print(f"\n{'PEG MW':<10} {'Corona h (nm)':<15} {'t½_unPEG (min)':<18} {'t½_PEG (hr)':<15}")
print("-"*60)

for mw in peg_mw:
    h = peg_corona_thickness(mw)
    t_un = circulation_half_life(100000, mw, False)
    t_peg = circulation_half_life(100000, mw, True)
    print(f"{mw:<10} {h:<15.2f} {t_un:<18.1f} {t_peg:<15.1f}")

# Figure: PEG density effects
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Panel 1: Corona thickness vs. MW
ax1 = axes[0]
MW_range = np.linspace(500, 30000, 100)
h_range = [peg_corona_thickness(mw) for mw in MW_range]

ax1.plot(MW_range, h_range, 'b-', lw=2.5)
ax1.axhline(5, color='red', ls='--', lw=2, label='Critical thickness for stealth')
ax1.fill_between(MW_range, 0, 5, alpha=0.1, color='green', label='Stealth window')
ax1.set_xlabel('PEG Molecular Weight (Da)', fontsize=12)
ax1.set_ylabel('Corona Thickness h (nm)', fontsize=12)
ax1.set_title('PEG Corona Thickness vs. Molecular Weight', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3)

# Panel 2: Circulation half-life
ax2 = axes[1]
MW_NP_range = np.linspace(10000, 500000, 100)

for mw_peg, color in [(2000, 'b'), (5000, 'g'), (10000, 'orange'), (20000, 'r')]:
    t_half = [circulation_half_life(mw_np, mw_peg, True) for mw_np in MW_NP_range]
    ax2.plot(MW_NP_range/1000, t_half, color=color, lw=2, 
             label=f'PEG-{mw_peg} Da')

ax2.axhline(24, color='gray', ls=':', lw=2, label='24 hr threshold')
ax2.set_xlabel('NP Core Molecular Weight (kDa)', fontsize=12)
ax2.set_ylabel('Circulation Half-life (hours)', fontsize=12)
ax2.set_title('Effect of PEG Size on Circulation Time', fontsize=14)
ax2.legend(title='PEGylation')
ax2.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W20_pecylation.png', dpi=150)
plt.show()

print("\n=== Design Guidelines ===")
print("For tumor targeting (EPR): t½ > 6-12 hours needed")
print("For active targeting: t½ > 24 hours preferred")
print(f"Optimal PEG MW: 5,000-10,000 Da for 100 nm NPs")
```

---

## Exercise 3: PLGA Degradation Kinetics

```python
"""
W20_CodeLab_Ex3_PLGA_Degradation.py
====================================
Model PLGA hydrolytic degradation.
Compare LA:GA ratios and molecular weight evolution.
"""

def plga_mw_degradation(MW0, t, k_hydrolysis, LA_fraction=0.5):
    """
    PLGA molecular weight loss via hydrolytic degradation.
    
    dMW/dt = -k × MW
    MW(t) = MW0 × exp(-k × t)
    
    k depends on:
    - LA:GA ratio (higher LA = slower degradation)
    - pH (acidic autocatalysis)
    - Temperature (Arrhenius)
    """
    # Hydrolysis rate constant (day^-1)
    # LA-rich: k ≈ 0.01-0.02 day^-1
    # GA-rich: k ≈ 0.05-0.1 day^-1
    k = k_hydrolysis * (1 + (1 - LA_fraction) * 2)  # Higher GA = faster
    
    MW_t = MW0 * np.exp(-k * t)
    return MW_t

def mass_loss_from_mw(MW_t, MW0):
    """
    Calculate cumulative mass loss from MW data.
    
    Assuming random chain scission:
    Chains broken = (MW0 - MW_t) / M_repeat_unit
    
    Mass loss fraction = chains broken × M_repeat_unit / MW0
    """
    M_repeat = 72 * 0.5 + 58 * 0.5  # LA=72, GA=58 average ≈ 65 g/mol
    n_chains_initial = MW0 / M_repeat
    n_chains_current = MW_t / M_repeat
    
    # Each broken chain releases 1 monomer unit
    monomers_released = n_chains_initial - n_chains_current
    
    # Mass loss (simplified)
    mass_loss = monomers_released * M_repeat / (MW0 * 10)  # Normalized
    return np.minimum(mass_loss, 1.0)

# PLGA compositions
compositions = {
    'PLGA 50:50': {'LA': 0.5, 'k': 0.02, 't_deg': '2-4 months'},
    'PLGA 65:35': {'LA': 0.65, 'k': 0.015, 't_deg': '4-6 months'},
    'PLGA 75:25': {'LA': 0.75, 'k': 0.01, 't_deg': '6-9 months'},
    'PLGA 85:15': {'LA': 0.85, 'k': 0.005, 't_deg': '9-15 months'},
    'PLA 100:0':  {'LA': 1.0,  'k': 0.003, 't_deg': '12-24 months'},
}

# Time (days) up to 365
t = np.linspace(0, 365, 200)
MW0 = 50000  # Initial MW = 50 kDa

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
colors = ['b', 'g', 'orange', 'r', 'purple']

for (name, props), color in zip(compositions.items(), colors):
    MW = plga_mw_degradation(MW0, t, props['k'], props['LA'])
    ax1.plot(t/30, MW/1000, color=color, lw=2.5, label=f'{name} (k={props["k"]:.3f}/day)')

ax1.set_xlabel('Time (months)', fontsize=12)
ax1.set_ylabel('Molecular Weight (kDa)', fontsize=12)
ax1.set_title('PLGA Molecular Weight Degradation', fontsize=14)
ax1.legend(fontsize=9)
ax1.grid(True, alpha=0.3)
ax1.set_xlim(0, 12)
ax1.axhline(10000, color='gray', ls=':', lw=1.5, label='Critical MW (10 kDa)')
ax1.axhline(5000, color='red', ls=':', lw=1.5, label='Gel point (~5 kDa)')

# Mass loss
ax2 = axes[1]
for (name, props), color in zip(compositions.items(), colors):
    MW = plga_mw_degradation(MW0, t, props['k'], props['LA'])
    mass_loss = mass_loss_from_mw(MW, MW0)
    ax2.plot(t/30, mass_loss*100, color=color, lw=2.5, label=name)

ax2.set_xlabel('Time (months)', fontsize=12)
ax2.set_ylabel('Mass Loss (%)', fontsize=12)
ax2.set_title('PLGA Cumulative Mass Loss', fontsize=14)
ax2.legend(fontsize=9)
ax2.grid(True, alpha=0.3)
ax2.set_xlim(0, 12)
ax2.axhline(50, color='gray', ls=':', lw=1.5)

plt.tight_layout()
plt.savefig('W20_plga_degradation.png', dpi=150)
plt.show()

# pH effect (autocatalysis)
print("\n=== PLGA Degradation Summary ===")
print(f"Initial MW: {MW0/1000:.0f} kDa")
for name, props in compositions.items():
    MW = plga_mw_degradation(MW0, 180, props['k'], props['LA'])  # 6 months
    print(f"{name:<15}: MW at 6 mo = {MW/1000:.1f} kDa, t_deg ≈ {props['t_deg']}")
```

---

## Exercise 4: Surface Modification — RGD Density Optimization

```python
"""
W20_CodeLab_Ex4_SurfaceMod.py
==============================
Model RGD peptide density effects on cell adhesion.
Compare integrin binding kinetics at different spacing.
"""

def integrin_binding_energy(DRGD, R_crit=10e-9):
    """
    Estimate integrin binding probability based on RGD spacing.
    
    Based on Massia & Hubbell (1990) work:
    - Optimal spacing: < 20 nm
    - Below threshold: > 70 nm (no adhesion)
    
    Binding probability P ∝ exp(-ΔE/kT)
    where ΔE increases with RGD spacing.
    """
    R = DRGD * 1e-9  # Convert nm to m
    R0 = R_crit  # Critical spacing = 10 nm
    
    if R <= R0:
        return 1.0
    else:
        # Exponential decay beyond critical spacing
        k_decay = 0.05  # nm^-1 (from literature)
        return np.exp(-k_decay * (R - R0) * 1e9)

def cell_spreading_area(P_binding, A_max=2000):
    """
    Estimate cell spreading area from binding probability.
    
    P_binding = 1: full spreading, A ≈ 2000 μm²
    P_binding = 0: round, A ≈ 200 μm²
    """
    A_min = 200  # Round cell area
    return A_min + (A_max - A_min) * P_binding

# RGD spacing study
spacing_nm = np.linspace(5, 100, 100)

P_binding = [integrin_binding_energy(s) for s in spacing_nm]
A_spread = [cell_spreading_area(p) for p in P_binding]

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
ax1.plot(spacing_nm, P_binding, 'b-', lw=2.5)
ax1.axhline(0.5, color='red', ls='--', lw=2, label='50% binding threshold')
ax1.axvline(10, color='green', ls=':', lw=2, label='Critical spacing (10 nm)')
ax1.axvline(20, color='orange', ls=':', lw=2, label='Optimal threshold (20 nm)')
ax1.axvline(70, color='red', ls=':', lw=2, label='Non-adhesive (70 nm)')
ax1.fill_between(spacing_nm, 0, P_binding, alpha=0.2, color='green')
ax1.set_xlabel('RGD Spacing (nm)', fontsize=12)
ax1.set_ylabel('Integrin Binding Probability', fontsize=12)
ax1.set_title('RGD Density Effect on Cell Adhesion', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3)
ax1.set_xlim(0, 100)
ax1.set_ylim(0, 1.1)

ax2 = axes[1]
ax2.plot(spacing_nm, A_spread, 'r-', lw=2.5)
ax2.axhline(200, color='gray', ls=':', lw=1.5, label='Round cell (200 μm²)')
ax2.axhline(2000, color='green', ls=':', lw=1.5, label='Fully spread (2000 μm²)')
ax2.set_xlabel('RGD Spacing (nm)', fontsize=12)
ax2.set_ylabel('Cell Spreading Area (μm²)', fontsize=12)
ax2.set_title('Cell Morphology vs. RGD Density', fontsize=14)
ax2.legend()
ax2.grid(True, alpha=0.3)
ax2.set_xlim(0, 100)

plt.tight_layout()
plt.savefig('W20_rgd_density.png', dpi=150)
plt.show()

# Design calculation
print("\n=== RGD Surface Density Design ===")
print(f"Target: Full cell spreading (A > 1500 μm²)")
print(f"Required binding probability: P > {cell_spreading_area.inverse(1500):.2f}")
print(f"Maximum RGD spacing: < 15 nm")

# Design surface with given PEG chain length
MW_PEG = 5000  # Da
h_PEG = 0.5 * np.sqrt(MW_PEG)  # nm
print(f"\nWith PEG-{MW_PEG} corona (h = {h_PEG:.1f} nm):")
print(f"  → RGD must extend beyond PEG layer")
print(f"  → Effective spacing = physical spacing + PEG thickness")
```

---

## Exercise 5: Biomaterial Property Database

```python
"""
W20_CodeLab_Ex5_PropertyDB.py
==============================
Comprehensive database of biomaterial properties.
"""

# Biomaterial property database
biomaterials = {
    # Metals
    'Ti-6Al-4V': {
        'E': 110e9,  # Pa
        'sigma_yield': 880e6,  # Pa
        'rho': 4430,  # kg/m³
        'corrosion': 'excellent',
        'use': 'Orthopedic implants, dental implants'
    },
    'CoCrMo': {
        'E': 210e9,
        'sigma_yield': 450e6,
        'rho': 8300,
        'corrosion': 'good',
        'use': 'Joint prostheses'
    },
    'SS316L': {
        'E': 190e9,
        'sigma_yield': 215e6,
        'rho': 8000,
        'corrosion': 'good',
        'use': 'Temporary implants, stents'
    },
    'Nitinol': {
        'E': 30e9,
        'sigma_yield': 100e6,
        'rho': 6450,
        'corrosion': 'good',
        'use': 'Vascular stents, orthodontics'
    },
    
    # Polymers
    'PLGA 50:50': {
        'E': 1e9,
        'sigma_yield': 50e6,
        'rho': 1300,
        'degradation': '2-4 months',
        'use': 'Drug delivery, sutures'
    },
    'PCL': {
        'E': 200e6,
        'sigma_yield': 20e6,
        'rho': 1150,
        'degradation': '12-24 months',
        'use': 'Scaffolds, drug delivery'
    },
    'PEG': {
        'E': 1e6,
        'sigma_yield': 5e6,
        'rho': 1200,
        'degradation': 'Non-degradable (renal clearance < 30 kDa)',
        'use': 'Stealth coating, hydrogels'
    },
    'PMMA': {
        'E': 2e9,
        'sigma_yield': 70e6,
        'rho': 1180,
        'degradation': 'Non-degradable',
        'use': 'Bone cement'
    },
    
    # Ceramics
    'Hydroxyapatite': {
        'E': 80e9,
        'sigma_yield': 100e6,
        'rho': 3100,
        'degradation': 'Slow (years)',
        'use': 'Bone grafts, coatings'
    },
    'Beta-TCP': {
        'E': 30e9,
        'sigma_yield': 50e6,
        'rho': 3040,
        'degradation': '6-12 months',
        'use': 'Bone scaffolds'
    },
    'Bioglass': {
        'E': 35e9,
        'sigma_yield': 40e6,
        'rho': 2700,
        'degradation': 'Variable ( bioactive)',
        'use': 'Bone regeneration'
    },
}

# Print comparison table
print("="*90)
print("BIOMATERIAL PROPERTY DATABASE")
print("="*90)
print(f"\n{'Material':<15} {'E (GPa)':<12} {'σ_y (MPa)':<12} {'ρ (kg/m³)':<12} {'Degradation'}")
print("-"*90)

for name, props in biomaterials.items():
    E_GPa = props['E'] / 1e9
    sigma_MPa = props['sigma_yield'] / 1e6
    deg = props.get('degradation', 'N/A')
    print(f"{name:<15} {E_GPa:<12.1f} {sigma_MPa:<12.0f} {props['rho']:<12.0f} {deg}")

# Stress shielding analysis
print("\n" + "="*90)
print("STRESS SHIELDING ANALYSIS")
print("="*90)
E_bone = 18e9  # Cortical bone
print(f"\nCortical bone E = {E_bone/1e9:.0f} GPa")
print(f"\n{'Implant Material':<15} {'E (GPa)':<12} {'E_implant/E_bone':<18} {'Risk'}")
print("-"*70)

risk_levels = [(5, 'Low'), (2, 'Moderate'), (1, 'High')]
for name, props in biomaterials.items():
    if 'E' in props:
        ratio = props['E'] / E_bone
        risk = next((r for t, r in risk_levels if ratio > t), 'Very High')
        print(f"{name:<15} {props['E']/1e9:<12.1f} {ratio:<18.2f} {risk}")
```

---

## Bonus: Immunomodulatory Biomaterial Design

```python
"""
W20_CodeLab_Bonus_Immunomodulation.py
======================================
Design immunomodulatory biomaterial for M2 macrophage polarization.
"""

def macrophage_phenotype(E_hydrogel):
    """
    Predict macrophage phenotype based on hydrogel stiffness.
    
    Based on Blakney et al. (2012):
    - E < 1 kPa: M2 (pro-healing)
    - E > 25 kPa: M1 (pro-inflammatory)
    - 1-25 kPa: mixed
    """
    if E_hydrogel < 1000:
        return 'M2', 'Pro-healing', 'IL-10, TGF-β'
    elif E_hydrogel < 25000:
        return 'Mixed', 'Variable', 'Intermediate cytokines'
    else:
        return 'M1', 'Pro-inflammatory', 'TNF-α, IL-1β'

def il4_release_design(duration_days, target_dose=10):
    """
    Design IL-4 release system for M2 polarization.
    
    IL-4 dose: 10 ng/mL for macrophage polarization
    """
    target_concentration = target_dose  # ng/mL
    total_dose = 1000  # ng per implant (for local delivery)
    release_rate = total_dose / duration_days  # ng/day
    return release_rate

# Stiffness sweep
E_range = np.logspace(2, 6, 100)  # 100 Pa to 1 MPa

phenotypes = [macrophage_phenotype(E) for E in E_range]
phenotype_colors = {'M2': 'green', 'M1': 'red', 'Mixed': 'orange'}

fig, ax = plt.subplots(figsize=(10, 5))
colors = [phenotype_colors[p[0]] for p in phenotypes]
ax.scatter(E_range/1000, [0]*len(E_range), c=colors, s=50, alpha=0.5)

ax.axvline(1, color='green', ls='--', lw=2, label='M2 threshold (1 kPa)')
ax.axvline(25, color='red', ls='--', lw=2, label='M1 threshold (25 kPa)')
ax.fill_betweenx([-0.5, 0.5], 0, 1, alpha=0.1, color='green')
ax.fill_betweenx([-0.5, 0.5], 25, 1000, alpha=0.1, color='red')

ax.set_xscale('log')
ax.set_xlabel('Hydrogel Stiffness E (kPa)', fontsize=12)
ax.set_yticks([])
ax.set_title('Macrophage Phenotype vs. Biomaterial Stiffness', fontsize=14)
ax.legend()
ax.set_xlim(0.1, 1000)

plt.tight_layout()
plt.savefig('W20_macrophage_polarization.png', dpi=150)
plt.show()

print("\n=== Immunomodulatory Design Summary ===")
print(f"Target: M2 polarization for tissue regeneration")
print(f"Required stiffness: E < 1 kPa")
print(f"IL-4 release rate: {il4_release_design(7):.1f} ng/day for 7-day delivery")
print(f"Alternative: Soft alginate (E ~ 0.1-1 kPa) with TGF-β1")
```

---

## Lab Report Checklist

| Task | Description | File |
|------|-------------|------|
| ☐ Ex 1 | Nanoparticle release kinetics | `W20_np_release.png` |
| ☐ Ex 2 | PEGylation half-life calculation | `W20_pecylation.png` |
| ☐ Ex 3 | PLGA degradation kinetics | `W20_plga_degradation.png` |
| ☐ Ex 4 | RGD density optimization | `W20_rgd_density.png` |
| ☐ Ex 5 | Biomaterial property database | Console output |
| ☐ Bonus | Macrophage polarization design | `W20_macrophage_polarization.png` |

---

## Extension Challenge

**Challenge**: Design a multi-functional biomaterial coating that simultaneously provides:
1. Antibacterial activity (Ag nanoparticles)
2. Drug elution (dexamethasone, anti-inflammatory)
3. RGD-mediated cell adhesion
4. PEG stealth properties

Calculate the required coating thickness, drug loading, and release profile for each component. Discuss potential interactions and conflicts between the functional elements.
