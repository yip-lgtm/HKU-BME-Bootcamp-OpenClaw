# Week 13 Code Lab — Biomechanics (Stress, Strain, Viscoelasticity)
## Python Computational Biomechanics Lab

> **Dependencies**: `numpy`, `scipy`, `matplotlib`, `pandas`  
> **Estimated time**: 3-4 hours  
> **Learning goal**: Build stress-strain analysis tools, fit viscoelastic models, simulate bone loading

---

## Lab Setup

```bash
pip install numpy scipy matplotlib pandas
```

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import curve_fit, least_squares
from scipy.integrate import odeint, solve_ivp
import pandas as pd
plt.style.use('seaborn-v0_8-whitegrid')
```

---

## Exercise 1: Stress-Strain Analysis of Cortical Bone

```python
"""
W13_CodeLab_Ex1_StressStrain.py
================================
Cortical bone tensile test simulation and mechanical property extraction.

Bone properties (literature values):
  - E_longitudinal = 18.0 GPa
  - E_transverse = 10.0 GPa
  - ν_xy = 0.30
  - σ_yield = 120 MPa
  - σ_ult = 200 MPa
  - ε_failure = 0.015 (1.5%)
"""

# Material properties (in MPa for consistency)
E_L = 18000      # MPa = 18 GPa (longitudinal)
E_T = 10000      # MPa = 10 GPa (transverse)
nu = 0.30        # Poisson's ratio
sigma_yield = 120  # MPa
sigma_ult = 200    # MPa
eps_failure = 0.015  # dimensionless

# Specimen geometry
L0 = 50.0        # mm (gauge length)
d = 8.0          # mm (diameter)
A = np.pi * (d/2)**2  # mm²

print(f"Specimen: L0={L0} mm, d={d} mm, A={A:.2f} mm²")

# Generate stress-strain curve (piecewise linear with some nonlinearity)
eps = np.linspace(0, eps_failure * 1.1, 200)

# Nonlinear model: σ = E*ε + soft tissue micro-damage
# Use polynomial to add slight nonlinearity before yield
sigma = np.zeros_like(eps)
for i, e in enumerate(eps):
    if e < 0.004:  # Toe region (collagen uncrimping)
        sigma[i] = 3000 * e + 2000 * e**2  # MPa
    elif e < eps_failure:  # Linear region
        sigma[i] = E_L * e  # MPa
    else:
        sigma[i] = sigma_ult  # Post-failure plateau

# Plot
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
ax1.plot(eps * 100, sigma, 'b-', lw=2, label='Cortical bone σ-ε')
ax1.axhline(sigma_yield, color='orange', ls='--', lw=1.5, label=f'Yield σ={sigma_yield} MPa')
ax1.axhline(sigma_ult, color='red', ls='--', lw=1.5, label=f'Ultimate σ={sigma_ult} MPa')
ax1.axvline(eps_failure * 100, color='gray', ls=':', lw=1.5, label=f'Failure ε={eps_failure*100:.1f}%')
ax1.set_xlabel('Strain ε (%)', fontsize=12)
ax1.set_ylabel('Stress σ (MPa)', fontsize=12)
ax1.set_title('Cortical Bone Tensile Stress-Strain Curve', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3)
ax1.set_xlim(0, 2.0)
ax1.set_ylim(0, 250)

# Calculate key mechanical properties
idx_yield = np.argmin(np.abs(sigma - sigma_yield))
eps_yield = eps[idx_yield]
E_apparent = sigma_yield / eps_yield  # Apparent modulus from yield point

# Strain energy density (area under curve)
U = np.trapz(sigma, eps)  # J/m³ = MPa * dimensionless

print(f"\n=== Extracted Mechanical Properties ===")
print(f"Apparent Young's Modulus (0-to-yield): E = {E_apparent:.0f} MPa = {E_apparent/1000:.1f} GPa")
print(f"Yield Strain: ε_y = {eps_yield*100:.3f}%")
print(f"Strain Energy Density (toughness): U = {U:.2f} kJ/m³")

# 2D plot: comparison with other materials
ax2 = axes[1]
materials = {
    'Cortical Bone': (18000, 200),
    'Ti-6Al-4V': (110000, 880),
    'PMMA': (2500, 70),
    'PE': (200, 30),
    'Cartilage': (20, 5),
}
for name, (E, sy) in materials.items():
    eps_range = np.linspace(0, sy/E * 1.05, 50)
    sigma_range = E * eps_range
    ax2.plot(eps_range * 100, sigma_range, lw=2, label=f'{name} (E={E/1000:.0f} GPa)')

ax2.set_xlabel('Strain ε (%)', fontsize=12)
ax2.set_ylabel('Stress σ (MPa)', fontsize=12)
ax2.set_title('Mechanical Property Comparison (Linear Region)', fontsize=14)
ax2.legend(fontsize=9)
ax2.grid(True, alpha=0.3)
ax2.set_xlim(0, 10)

plt.tight_layout()
plt.savefig('W13_stress_strain_comparison.png', dpi=150)
plt.show()

# Save stress-strain data
df = pd.DataFrame({'strain': eps, 'stress_MPa': sigma})
df.to_csv('W13_bone_stress_strain.csv', index=False)
print("Saved: W13_bone_stress_strain.csv")
```

---

## Exercise 2: Viscoelastic Model Simulation (Maxwell & Kelvin-Voigt)

```python
"""
W13_CodeLab_Ex2_Viscoelastic.py
================================
Simulate creep and stress relaxation for Maxwell and Kelvin-Voigt models.
Compare with Fung's quasi-linear viscoelastic (QLV) theory.
"""

def maxwell_creep(t, sigma0, E, eta):
    """Maxwell model: creep under constant stress"""
    eps = sigma0 / E + sigma0 / eta * t
    return eps

def kelvin_voigt_creep(t, sigma0, E, eta):
    """Kelvin-Voigt model: creep under constant stress"""
    tau = eta / E
    eps = sigma0 / E * (1 - np.exp(-t / tau))
    return eps

def maxwell_relaxation(t, eps0, E, eta):
    """Maxwell model: stress relaxation under constant strain"""
    sigma = E * eps0 * np.exp(-E / eta * t)
    return sigma

def kelvin_voigt_relaxation(t, eps0, E, eta):
    """Kelvin-Voigt model: stress relaxation under constant strain"""
    # Analytical solution: σ(t) = E*ε0 + E*ε0*(η/(E*t))*[1 - exp(-E*t/η)]
    # Actually: σ(t) = E*ε0 + η*ε0*δ(t) in idealized form
    # For step strain: immediate σ = E*eps0, then increases (not relaxation)
    tau = eta / E
    sigma = E * eps0 * (1 - np.exp(-t / tau))
    return sigma

def sls_response(t, sigma0, E1, E2, eta):
    """Standard Linear Solid (SLS) under step stress"""
    # Creep compliance: J(t) = 1/E1 + (1/E2)(1 - exp(-t*E2/eta))
    tau = eta / E2
    eps = sigma0 / E1 + (sigma0 / E2) * (1 - np.exp(-t / tau))
    return eps

def sls_relaxation(t, eps0, E1, E2, eta):
    """Standard Linear Solid relaxation under step strain"""
    tau = eta / (E1 + E2)
    sigma_eq = E1 * eps0  # Equilibrium stress
    sigma_instant = (E1 + E2) * eps0  # Instantaneous stress
    sigma = sigma_eq + (sigma_instant - sigma_eq) * np.exp(-t / tau)
    return sigma

def fung_qlv_creep(t, sigma0, A, tau1, n):
    """Fung's quasi-linear viscoelastic creep approximation"""
    # Relaxation function: G(t) = [1 + (t/tau1)]^(-n)
    G = (1 + t / tau1) ** (-n)
    # For step stress: ε(t) = σ0 * J(t) where J = J_e * G(t)
    # Using linear elastic baseline
    J_e = 1.0 / E_L  # Use bone E for normalization
    eps = sigma0 * J_e * G
    return eps

# Parameters (realistic values for tendon/bone)
E_tendon = 800   # MPa (tendon elastic modulus)
E_bone_soft = 5000  # MPa (approx for ligament)
E1_sls = 1000   # MPa (SLS spring 1)
E2_sls = 500    # MPa (SLS spring 2)
eta_sls = 5000  # MPa·s (SLS dashpot)
tau = eta_sls / E2_sls  # Relaxation time = 10 s

# Time array
t = np.linspace(0, 100, 500)  # 0 to 100 seconds

# Creep under constant stress σ0 = 10 MPa
sigma0 = 10.0  # MPa
eps0 = 0.005    # Initial strain

fig, axes = plt.subplots(1, 3, figsize=(18, 5))

# --- CREEP COMPARISON ---
ax1 = axes[0]
eps_maxwell = maxwell_creep(t, sigma0, E_bone_soft, 50000)
eps_kelvin = kelvin_voigt_creep(t, sigma0, E_bone_soft, 50000)
eps_sls = sls_response(t, sigma0, E1_sls, E2_sls, eta_sls)
eps_qlv = fung_qlv_creep(t, sigma0, 1.0, 10, 0.5)

ax1.plot(t, eps_maxwell * 100, 'r-', lw=2, label=f'Maxwell (τ=∞, linear creep)')
ax1.plot(t, eps_kelvin * 100, 'b-', lw=2, label=f'Kelvin-Voigt (τ={tau:.0f}s)')
ax1.plot(t, eps_sls * 100, 'g-', lw=2, label=f'Standard Linear Solid')
ax1.plot(t, eps_qlv * 100, 'm--', lw=2, label=f"Fung QLV (n=0.5)")
ax1.set_xlabel('Time t (s)', fontsize=12)
ax1.set_ylabel('Strain ε (%)', fontsize=12)
ax1.set_title('Creep: Strain vs Time under σ₀ = 10 MPa', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3)
ax1.set_ylim(0, 2.0)

# --- STRESS RELAXATION ---
ax2 = axes[1]
sigma_mr = maxwell_relaxation(t, eps0, E_bone_soft, 50000)
sigma_sr = sls_relaxation(t, eps0, E1_sls, E2_sls, eta_sls)
ax2.plot(t, sigma_mr, 'r-', lw=2, label='Maxwell (exponential decay)')
ax2.plot(t, sigma_sr, 'g-', lw=2, label='Standard Linear Solid')
ax2.axhline(E1_sls * eps0, color='gray', ls=':', lw=1.5, label=f'Equilibrium σ = {E1_sls*eps0:.1f} MPa')
ax2.axhline((E1_sls+E2_sls)*eps0, color='orange', ls=':', lw=1.5, label=f'Instant σ = {(E1_sls+E2_sls)*eps0:.1f} MPa')
ax2.set_xlabel('Time t (s)', fontsize=12)
ax2.set_ylabel('Stress σ (MPa)', fontsize=12)
ax2.set_title('Stress Relaxation: σ vs Time under ε₀ = 0.5%', fontsize=14)
ax2.legend()
ax2.grid(True, alpha=0.3)

# --- PARAMETRIC STUDY: Effect of relaxation time ---
ax3 = axes[2]
for tau_val in [1, 5, 10, 20, 50]:
    E_test = 1000
    eta_test = E_test * tau_val
    eps_kv = kelvin_voigt_creep(t, sigma0, E_test, eta_test)
    ax3.plot(t, eps_kv * 100, lw=1.5, label=f'τ = {tau_val} s (η = {eta_test} MPa·s)')
ax3.set_xlabel('Time t (s)', fontsize=12)
ax3.set_ylabel('Strain ε (%)', fontsize=12)
ax3.set_title('Kelvin-Voigt: Effect of τ on Creep', fontsize=14)
ax3.legend(fontsize=9)
ax3.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W13_viscoelastic_models.png', dpi=150)
plt.show()

# Print key results
print(f"\n=== Viscoelastic Model Comparison at t = τ ===")
print(f"Maxwell Creep (t=τ): ε = σ₀/E + σ₀/η·τ = σ₀/E + σ₀/E = 2σ₀/E = {2*sigma0/E_bone_soft*100:.3f}%")
print(f"Kelvin-Voigt Creep (t=τ): ε = (σ₀/E)(1-e⁻¹) = {(sigma0/E_bone_soft)*(1-np.exp(-1))*100:.3f}%")
print(f"SLS Creep (t=τ): ε = σ₀/E₁ + (σ₀/E₂)(1-e⁻¹) = {sigma0/E1_sls + (sigma0/E2_sls)*(1-np.exp(-1))*100:.3f}%")
```

---

## Exercise 3: Trabecular Bone Density-Stiffness Relationship

```python
"""
W13_CodeLab_Ex3_Trabecular_E.py
================================
Model trabecular bone apparent modulus vs. apparent density.
Gibson-Ashby model: E = E₀ · ρ*^n
Compare with experimental data.
"""

# Apparent density range (typical vertebral trabecular bone)
rho_app = np.linspace(0.1, 0.8, 100)  # g/cm³
rho_solid = 1.9  # g/cm³ (fully dense bone mineral)

# Gibson-Ashby model (open-cell foam)
E0 = 3390  # MPa (solid phase modulus parameter)
n = 2.0    # Power law exponent

# Compute apparent modulus
E_app = E0 * (rho_app / rho_solid) ** n

# With anisotropy correction (longitudinal vs. transverse)
E_long = E_app * 1.3  # ~30% higher in loading direction
E_trans = E_app * 0.7

# Experimental data points (literature)
rho_exp = np.array([0.15, 0.25, 0.35, 0.45, 0.55, 0.65])
E_exp = np.array([20, 100, 250, 500, 800, 1200])  # MPa

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
ax1.loglog(rho_app, E_app, 'b-', lw=2.5, label=f'Gibson-Ashby: E = {E0}·(ρ/ρs)²')
ax1.loglog(rho_app, E_long, 'g--', lw=2, label='Longitudinal (×1.3)')
ax1.loglog(rho_app, E_trans, 'r--', lw=2, label='Transverse (×0.7)')
ax1.loglog(rho_exp, E_exp, 'ko', ms=10, label='Experimental data (literature)')
ax1.set_xlabel('Apparent Density ρ (g/cm³)', fontsize=12)
ax1.set_ylabel('Apparent Modulus E (MPa)', fontsize=12)
ax1.set_title('Trabecular Bone: Density vs. Stiffness', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3, which='both')

# Fit power law to experimental data
from scipy.optimize import curve_fit
def power_law(rho, E0, n):
    return E0 * rho**n

popt, pcov = curve_fit(power_law, rho_exp, E_exp, p0=[3390, 2.0], maxfev=5000)
print(f"Fitted Gibson-Ashby: E₀ = {popt[0]:.1f} MPa, n = {popt[1]:.2f}")

# Stress at typical vertebral load
sigma_vert = 1.0  # MPa (1 MPa = ~10 atm ≈ vertebral body stress under normal activity)
# Safety factor: σ / σ_allowable
sigma_allow = E_app * 0.01  # 1% strain limit
safety_factor = sigma_allow / sigma_vert

ax2 = axes[1]
ax2.semilogy(rho_app, safety_factor, 'b-', lw=2.5)
ax2.axhline(1.0, color='red', ls='--', lw=1.5, label='Safety factor = 1.0 (yield)')
ax2.set_xlabel('Apparent Density ρ (g/cm³)', fontsize=12)
ax2.set_ylabel('Safety Factor (at 1% strain)', fontsize=12)
ax2.set_title('Trabecular Bone Safety Factor vs. Density', fontsize=14)
ax2.legend()
ax2.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W13_trabecular_density.png', dpi=150)
plt.show()
```

---

## Exercise 4: Fracture Mechanics — Stress Intensity Factor

```python
"""
W13_CodeLab_Ex4_Fracture.py
============================
Calculate stress intensity factor K and assess fracture risk in bone.
Paris law for fatigue crack growth.
"""

# Material properties
K_IC = 4.0    # MPa·√m (cortical bone fracture toughness, range 2-6)
sigma_app = 80  # MPa (physiological stress in femur during walking)

# Crack geometry: surface crack in femur
a_values = np.linspace(0.1, 10, 100)  # mm crack depth
a_m = a_values / 1000  # convert to meters

# Geometry factor Y for surface crack (semi-elliptical)
# Y ≈ 0.65 for aspect ratio a/c = 1 (semi-circular)
Y = 0.65

# Stress intensity factor K = Y·σ·√(π·a)
K = Y * sigma_app * np.sqrt(np.pi * a_m)

# Fracture condition: K > K_IC
fracture_idx = np.where(K > K_IC)[0]
if len(fracture_idx) > 0:
    a_critical = a_values[fracture_idx[0]]
    print(f"Critical crack size for fracture: a_c = {a_critical:.2f} mm")
    print(f"  (at σ = {sigma_app} MPa, Y = {Y}, K_IC = {K_IC} MPa√m)")
else:
    print(f"No fracture predicted in range (K_max = {K[-1]:.2f} MPa√m < K_IC = {K_IC})")

# Paris law parameters for cortical bone
C_paris = 1e-8   # mm/cycle (varies widely in literature)
m_paris = 3.5    # Exponent

# Calculate crack growth per cycle
dK_da = np.gradient(K, a_m * 1000)  # dK/da in MPa
delta_K = 0.5 * K  # Assume ΔK = 0.5 K_max (R = 0 minimum)
da_dN = C_paris * (delta_K * 1000) ** m_paris  # mm/cycle (K in MPa·mm^0.5)

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
ax1.semilogy(a_values, K, 'b-', lw=2.5, label=f'K = {Y}·σ·√(πa)')
ax1.axhline(K_IC, color='red', ls='--', lw=2, label=f'K_IC = {K_IC} MPa√m')
if len(fracture_idx) > 0:
    ax1.axvline(a_critical, color='orange', ls=':', lw=2, label=f'a_c = {a_critical:.2f} mm')
ax1.fill_between(a_values[a_values > a_critical], 0, K_IC * 1.5, 
                   alpha=0.2, color='red', label='Fracture zone')
ax1.set_xlabel('Crack Depth a (mm)', fontsize=12)
ax1.set_ylabel('Stress Intensity K (MPa·√m)', fontsize=12)
ax1.set_title('Bone Fracture Risk: K vs. Crack Size', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3)
ax1.set_ylim(0.1, 20)

# Crack growth rate vs ΔK
ax2 = axes[1]
dk_range = np.logspace(-1, 1.5, 100)  # MPa·mm^0.5 (ΔK range)
da_dN_range = C_paris * dk_range ** m_paris
ax2.loglog(dk_range, da_dN_range, 'b-', lw=2.5, label=f'Paris: C={C_paris}, m={m_paris}')
ax2.axvspan(0.1, 1.0, alpha=0.1, color='green', label='Threshold ΔK_th')
ax2.axhline(0.001, color='gray', ls=':', lw=1, label='Near-threshold')
ax2.set_xlabel('Stress Intensity Range ΔK (MPa·√mm)', fontsize=12)
ax2.set_ylabel('Crack Growth Rate da/dN (mm/cycle)', fontsize=12)
ax2.set_title('Paris Law: Fatigue Crack Growth in Bone', fontsize=14)
ax2.legend()
ax2.grid(True, alpha=0.3, which='both')

plt.tight_layout()
plt.savefig('W13_fracture_mechanics.png', dpi=150)
plt.show()
```

---

## Exercise 5: Soft Tissue Nonlinear Stress-Strain (Tendon Model)

```python
"""
W13_CodeLab_Ex5_SoftTissue.py
===============================
Fit Gent and exponential models to tendon stress-strain data.
Compare with linear elastic model.
"""

from scipy.optimize import curve_fit

# Literature tendon stress-strain data (simulated from Fung 1993)
eps_tendon = np.array([0.000, 0.005, 0.010, 0.015, 0.020, 0.030, 0.040, 
                       0.050, 0.060, 0.070, 0.080, 0.090, 0.100, 0.110, 0.115])
sigma_tendon = np.array([0.0, 1.0, 3.0, 7.0, 14.0, 28.0, 42.0, 55.0, 
                          68.0, 80.0, 92.0, 103.0, 112.0, 120.0, 125.0])  # MPa

# Models
def linear_model(eps, E):
    return E * eps

def gent_model(eps, E0, eps_L):
    """Gent (1999) model: σ = E0*ε / (1 - ε/εL)"""
    mask = eps < eps_L
    sigma = np.zeros_like(eps)
    sigma[mask] = E0 * eps[mask] / (1 - eps[mask] / eps_L)
    sigma[~mask] = np.nan
    return sigma

def exponential_model(eps, A, B):
    """Fung-type exponential: σ = A(e^(Bε) - 1)"""
    return A * (np.exp(B * eps) - 1)

# Fit models
popt_linear, _ = curve_fit(linear_model, eps_tendon[:6], sigma_tendon[:6])
popt_gent, _ = curve_fit(gent_model, eps_tendon, sigma_tendon, 
                          p0=[800, 0.15], bounds=([100, 0.05], [5000, 0.30]))
popt_exp, _ = curve_fit(exponential_model, eps_tendon, sigma_tendon,
                        p0=[10, 20], bounds=([0.1, 5], [500, 100]))

print(f"Linear fit (toe region): E = {popt_linear[0]:.1f} MPa")
print(f"Gent model fit: E0 = {popt_gent[0]:.1f} MPa, εL = {popt_gent[1]*100:.2f}%")
print(f"Exponential fit: A = {popt_exp[0]:.2f} MPa, B = {popt_exp[1]:.1f}")

# Generate predictions
eps_fit = np.linspace(0, 0.12, 200)
sigma_linear = linear_model(eps_fit, *popt_linear)
sigma_gent = gent_model(eps_fit, *popt_gent)
sigma_exp = exponential_model(eps_fit, *popt_exp)

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
ax1.plot(eps_tendon * 100, sigma_tendon, 'ko', ms=8, label='Experimental data', zorder=5)
ax1.plot(eps_fit * 100, sigma_gent, 'b-', lw=2.5, label=f'Gent: E₀={popt_gent[0]:.0f} MPa, εL={popt_gent[1]*100:.1f}%')
ax1.plot(eps_fit * 100, sigma_exp, 'r-', lw=2.5, label=f'Exponential: A={popt_exp[0]:.1f}, B={popt_exp[1]:.1f}')
ax1.axvline(2.0, color='green', ls=':', lw=2, label='Toe region end (~2%)')
ax1.axvline(4.0, color='orange', ls=':', lw=2, label='Yield region (~4%)')
ax1.set_xlabel('Strain ε (%)', fontsize=12)
ax1.set_ylabel('Stress σ (MPa)', fontsize=12)
ax1.set_title('Tendon: Stress-Strain Curve Fitting', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3)

# Residual analysis
ax2 = axes[1]
sigma_gent_full = gent_model(eps_tendon, *popt_gent)
residuals = sigma_tendon - sigma_gent_full
ax2.bar(range(len(residuals)), residuals, color='steelblue', alpha=0.7)
ax2.axhline(0, color='red', ls='-', lw=1)
ax2.set_xlabel('Data Point Index', fontsize=12)
ax2.set_ylabel('Residual (MPa)', fontsize=12)
ax2.set_title('Gent Model Residual Analysis', fontsize=14)
ax2.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W13_tendon_model_fit.png', dpi=150)
plt.show()

# Print RMSE
rmse = np.sqrt(np.nanmean(residuals**2))
print(f"Gent Model RMSE: {rmse:.2f} MPa")
```

---

## Bonus: 3D Mohr's Circle & Von Mises Calculation

```python
"""
W13_CodeLab_Bonus_Mohr.py
===========================
Compute principal stresses and von Mises stress for a femoral implant-bone interface.
"""

def mohr_circles_3D(sx, sy, sz, txy, tyz, tzx):
    """
    Compute 3D Mohr's circles (maximum, intermediate, minimum circles).
    Returns (σ1, σ2, σ3) principal stresses.
    """
    # Stress tensor
    S = np.array([[sx, txy, tzx],
                  [txy, sy, tyz],
                  [tzx, tyz, sz]])
    
    # Eigenvalues = principal stresses
    eigenvalues, eigenvectors = np.linalg.eig(S)
    principal = np.sort(eigenvalues)[::-1]  # σ1 >= σ2 >= σ3
    
    return principal

def von_mises(σ1, σ2, σ3):
    """Von Mises stress from principal stresses"""
    return np.sqrt(0.5 * ((σ1 - σ2)**2 + (σ2 - σ3)**2 + (σ3 - σ1)**2))

# Femoral hip prosthesis interface scenario
# Stress state at femoral neck (combination of axial + bending + torsion)
sigma_x = -50   # MPa (compressive, from body weight)
sigma_y = -30   # MPa (medial-lateral)
sigma_z = -20   # MPa (anterior-posterior)
tau_xy = 10     # MPa (shear)
tau_yz = 5      # MPa
tau_zx = 8      # MPa

principals = mohr_circles_3D(sigma_x, sigma_y, sigma_z, tau_xy, tau_yz, tau_zx)
σ1, σ2, σ3 = principals

vm_stress = von_mises(σ1, σ2, σ3)

print(f"Principal stresses:")
print(f"  σ1 = {σ1:.1f} MPa")
print(f"  σ2 = {σ2:.1f} MPa") 
print(f"  σ3 = {σ3:.1f} MPa")
print(f"\nVon Mises stress: σ_vm = {vm_stress:.1f} MPa")
print(f"Cortical bone yield: σ_y = 120 MPa")
print(f"Safety factor: {120/vm_stress:.2f}")
```

---

## Lab Report Checklist

| Task | Description | File |
|------|-------------|------|
| ☐ Ex 1 | Stress-strain plot with material comparison | `W13_stress_strain_comparison.png` |
| ☐ Ex 2 | Viscoelastic creep/relaxation comparison | `W13_viscoelastic_models.png` |
| ☐ Ex 3 | Trabecular density-stiffness model | `W13_trabecular_density.png` |
| ☐ Ex 4 | Fracture mechanics and Paris law | `W13_fracture_mechanics.png` |
| ☐ Ex 5 | Soft tissue nonlinear fit (Gent model) | `W13_tendon_model_fit.png` |
| ☐ Bonus | Mohr's circles and von Mises | Script output |
| ☐ Data | All CSV data files | `W13_*.csv` |

---

## Extension Challenge (Capstone-level)

**Challenge**: Write a finite difference solver for 1D poroelastic consolidation in cartilage.

$$\frac{\partial u}{\partial t} = c_v \frac{\partial^2 u}{\partial x^2}$$

Parameters: biphasic cartilage layer thickness h = 2 mm, consolidation coefficient c_v = 0.01 mm²/s, permeability k = 10⁻¹⁵ m⁴/N·s.

Hint: Use implicit (backward Euler) scheme for stability.
