# Week 16 Code Lab — Biotransport (Diffusion and Fluid Mechanics)
## Python Computational Biotransport Lab

> **Dependencies**: `numpy`, `scipy`, `matplotlib`, `pandas`  
> **Estimated time**: 3-4 hours  
> **Learning goal**: Simulate 1D diffusion (Fick's law), transient diffusion with finite differences, Starling forces in microcirculation, Krogh cylinder oxygenation

---

## Exercise 1: 1-D Steady-State Diffusion (Fick's Law)

```python
"""
W16_CodeLab_Ex1_SteadyStateDiffusion.py
=========================================
Model 1-D steady-state diffusion through a tissue slab.
Solve Fick's first law: dC/dx = constant, J = -D*dC/dx.
Compare diffusion times for O₂, glucose, and albumin.
"""

import numpy as np
import matplotlib.pyplot as plt

# =============================================================
# PARAMETERS
# =============================================================
# Diffusivities (cm²/s) in tissue at 37°C
D_O2     = 2.0e-5    # Oxygen: small, fast
D_glucose = 6.7e-7   # Glucose: medium
D_albumin = 6.0e-8  # Albumin: large protein, slow

# Geometry
L = 0.05     # Tissue thickness: 0.5 mm = 500 μm = 0.05 cm
C_surface = 1.0   # Surface concentration (normalized)
C_deep   = 0.0    # Deep tissue concentration

# =============================================================
# STEADY-STATE SOLUTION: linear concentration profile
# =============================================================
x = np.linspace(0, L, 200)

# C(x) = C_deep + (C_surface - C_deep) * (1 - x/L)
C = C_deep + (C_surface - C_deep) * (1 - x/L)

# Flux J = -D * dC/dx = -D * (C_surface - C_deep) / L
flux = {
    'O2':     -D_O2     * (C_surface - C_deep) / L,
    'Glucose': -D_glucose * (C_surface - C_deep) / L,
    'Albumin': -D_albumin * (C_surface - C_deep) / L,
}

print("=== Steady-State Diffusion Flux ===")
print(f"Tissue thickness: {L*10:.1f} mm")
print()
for name, J in flux.items():
    print(f"  {name}: J = {J:.2e} mol/cm²/s")

# =============================================================
# CHARACTERISTIC DIFFUSION TIME: τ = L² / D
# =============================================================
tau = {
    'O2':     L**2 / D_O2,
    'Glucose': L**2 / D_glucose,
    'Albumin': L**2 / D_albumin,
}

print("\n=== Characteristic Diffusion Times ===")
for name, t in tau.items():
    if t < 60:
        print(f"  {name}: τ = {t:.2f} seconds")
    elif t < 3600:
        print(f"  {name}: τ = {t/60:.2f} minutes")
    else:
        print(f"  {name}: τ = {t/3600:.2f} hours")

# =============================================================
# PLOT: Concentration profiles
# =============================================================
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
ax1.plot(x*10, C, 'b-', lw=2.5, label='C(x)')
ax1.set_xlabel('Distance x (mm)', fontsize=12)
ax1.set_ylabel('Concentration (normalized)', fontsize=12)
ax1.set_title('Steady-State Diffusion:\nLinear Concentration Profile', fontsize=14)
ax1.grid(True, alpha=0.3)
ax1.legend()

# =============================================================
# Exercise: Vary thickness and compare flux
# =============================================================
thicknesses = np.linspace(0.001, 0.1, 100)  # 10 μm to 1 mm
ax2 = axes[1]
for name, D in [('O₂', D_O2), ('Glucose', D_glucose), ('Albumin', D_albumin)]:
    J_vals = D * (C_surface - C_deep) / thicknesses
    ax2.plot(thicknesses*10, J_vals*1e9, lw=2.5, label=f'{name} (D={D:.1e} cm²/s)')

ax2.set_xlabel('Tissue Thickness (mm)', fontsize=12)
ax2.set_ylabel('Flux J (×10⁻⁹ mol/cm²/s)', fontsize=12)
ax2.set_title('Flux vs. Thickness:\nO₂ 10× Faster Than Albumin', fontsize=14)
ax2.set_xscale('log')
ax2.set_yscale('log')
ax2.legend()
ax2.grid(True, alpha=0.3, which='both')

plt.tight_layout()
plt.savefig('W16_steady_diffusion.png', dpi=150)
plt.show()
```

**Expected outputs**:
- Linear concentration profile (constant gradient)
- Flux inversely proportional to thickness
- Albumin flux ~100× smaller than O₂ flux

---

## Exercise 2: Transient Diffusion — Explicit Finite Difference

```python
"""
W16_CodeLab_Ex2_TransientDiffusion.py
======================================
Model 1-D transient diffusion using explicit finite difference method.
Solve: ∂C/∂t = D · ∂²C/∂x²
Boundary conditions: C(0,t)=C₁, C(L,t)=C₂, C(x,0)=C₀
Demonstrate explicit stability: Δt ≤ (Δx)² / (2D)
"""

import numpy as np
import matplotlib.pyplot as plt

# =============================================================
# PARAMETERS
# =============================================================
D = 2.0e-5   # O₂ diffusivity in tissue (cm²/s)
L = 0.05     # 0.5 mm tissue slab
N = 50        # spatial grid points
dx = L / N    # spatial step

# Stability criterion for explicit scheme: dt ≤ dx²/(2D)
dt_explicit = 0.4 * dx**2 / D   # Stable explicit
dt_implicit = 0.1               # Implicit is unconditionally stable

T_total = 10.0   # Total simulation time (seconds)
C_left  = 1.0    # C at x=0 (surface)
C_right = 0.0    # C at x=L (deep)
C_init  = 0.0    # Initial condition

# =============================================================
# EXPLICIT FINITE DIFFERENCE
# =============================================================
r = D * dt_explicit / dx**2   # Fourier number
print(f"Explicit: r = {r:.4f} (must be ≤ 0.5 for stability)")

x_grid = np.linspace(0, L, N+1)
C = np.full(N+1, C_init)     # Initial condition
C[0]  = C_left
C[-1] = C_right

# Time stepping
t = 0.0
t_out = [0.0, 0.5, 1.0, 2.0, 5.0, 10.0]  # Output times
colors = ['red', 'orange', 'yellow', 'green', 'blue', 'purple']

fig, axes = plt.subplots(1, 2, figsize=(14, 5))
ax1 = axes[0]

# Store solutions at output times
solutions = []

while t < T_total:
    C_old = C.copy()
    for i in range(1, N):
        C[i] = C_old[i] + r * (C_old[i+1] - 2*C_old[i] + C_old[i-1])
    t += dt_explicit
    
    # Record output times
    for i, t_check in enumerate(t_out):
        if abs(t - t_check) < dt_explicit/2:
            solutions.append((t_check, C.copy()))
            break

# Plot results
for t_out_val, C_out, col in zip(*zip(*solutions) if solutions else ([],[],[]), colors):
    ax1.plot(x_grid*10, C_out, color=col, lw=2, label=f't={t_out_val}s')

ax1.set_xlabel('Distance x (mm)', fontsize=12)
ax1.set_ylabel('Concentration (normalized)', fontsize=12)
ax1.set_title('Transient Diffusion (Explicit FD):\nC(x,t) Over Time', fontsize=14)
ax1.legend()
ax1.grid(True, alpha=0.3)

# =============================================================
# ANALYTICAL SOLUTION: Series expansion
# =============================================================
def analytical_solution(x, t, D, L, C1=1.0, C2=0.0):
    """Analytical solution for 1D diffusion with Dirichlet BCs"""
    C = np.zeros_like(x)
    for n in range(1, 100):
        lambda_n = n * np.pi / L
        coeff = (2/L) * (C1 * (-1)**(n+1) + C2) / lambda_n
        C += coeff * np.sin(lambda_n * x) * np.exp(-D * lambda_n**2 * t)
    return C

# Compare numerical vs analytical
ax2 = axes[1]
t_compare = 2.0
C_num = solutions[3][1] if len(solutions) > 3 else C
C_anal = analytical_solution(x_grid, t_compare, D, L)
ax2.plot(x_grid*10, C_num, 'bo-', markersize=4, label='Explicit FD', lw=1.5)
ax2.plot(x_grid*10, C_anal, 'r--', lw=2, label='Analytical')
ax2.set_xlabel('Distance x (mm)', fontsize=12)
ax2.set_ylabel('Concentration (normalized)', fontsize=12)
ax2.set_title(f'Explicit FD vs. Analytical\nt = {t_compare}s', fontsize=14)
ax2.legend()
ax2.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W16_transient_diffusion.png', dpi=150)
plt.show()

# =============================================================
# STUDY: Diffusion distance vs. time
# =============================================================
print("\n=== Diffusion Penetration Depth ===")
print(f"Penetration depth δ ≈ √(4Dt)")
penetration_times = [0.1, 0.5, 1.0, 5.0, 10.0]
for t_p in penetration_times:
    delta = np.sqrt(4 * D * t_p) * 10  # in mm
    print(f"  t = {t_p}s → δ ≈ {delta:.4f} mm")
```

---

## Exercise 3: Starling Forces and Microcirculation

```python
"""
W16_CodeLab_Ex3_StarlingForces.py
==================================
Calculate net transcapillary fluid exchange using Starling's equation.
Model edema formation and compare different physiological conditions.
"""

import numpy as np
import matplotlib.pyplot as plt

# =============================================================
# STIRLING FORCES — Baseline (systemic capillary)
# =============================================================
def starling_filtration(P_c, P_i, pi_c, pi_i, L_p, sigma, S_cm2):
    """
    Calculate net fluid flux using Starling's equation.
    
    Parameters:
    - P_c: capillary hydrostatic pressure (mmHg)
    - P_i: interstitial hydrostatic pressure (mmHg)  
    - pi_c: capillary oncotic pressure (mmHg)
    - pi_i: interstitial oncotic pressure (mmHg)
    - L_p: hydraulic conductivity (mL/min/mmHg/cm²)
    - sigma: reflection coefficient
    - S_cm2: capillary surface area (cm²)
    """
    delta_P = P_c - P_i          # Hydrostatic pressure difference
    delta_pi = pi_c - pi_i        # Oncotic pressure difference
    delta_P_eff = delta_P - sigma * delta_pi
    J_v = L_p * S_cm2 * delta_P_eff  # mL/min
    return J_v, delta_P_eff

# Baseline parameters (Michel's 2000 values)
P_c_arteriolar = 35.0   # mmHg (arteriolar end)
P_c_venular    = 15.0   # mmHg (venular end)
P_i            = -2.0   # mmHg (interstitial)
pi_c           = 25.0   # mmHg (plasma albumin ~25 mmHg)
pi_i           = 2.0    # mmHg (interstitial)
L_p            = 0.01   # mL/min/mmHg/cm²
sigma_albumin  = 0.95   # reflection coefficient for albumin
S_capillary    = 1.0    # cm² per capillary (normalized)
n_capillaries  = 1e6    # Number of capillaries per 100g tissue

# Calculate arteriolar vs venular filtration
J_v_art, net_art = starling_filtration(P_c_arteriolar, P_i, pi_c, pi_i, L_p, sigma_albumin, S_capillary)
J_v_ven,  net_ven = starling_filtration(P_c_venular,    P_i, pi_c, pi_i, L_p, sigma_albumin, S_capillary)

print("=== Starling Forces: Arteriolar vs Venular End ===")
print(f"Arteriolar end: P_c={P_c_arteriolar} mmHg, ΔP_eff={net_art:.2f} mmHg → J_v={J_v_art:.4f} mL/min/cm²")
print(f"Venular end:    P_c={P_c_venular} mmHg, ΔP_eff={net_ven:.2f} mmHg → J_v={J_v_ven:.4f} mL/min/cm²")
print(f"Net per capillary: {(J_v_art + J_v_ven)/2:.4f} mL/min/cm²")
print(f"Per 100g tissue: {(J_v_art + J_v_ven)/2 * n_capillaries * S_capillary * 0.5:.2f} mL/min/100g")

# =============================================================
# EDEMA SIMULATION: Effect of increased P_c (heart failure)
# =============================================================
P_c_range = np.linspace(10, 60, 100)  # 10 to 60 mmHg
J_v_range = np.zeros_like(P_c_range)

for i, P_c in enumerate(P_c_range):
    _, net_P = starling_filtration(P_c, P_i, pi_c, pi_i, L_p, sigma_albumin, S_capillary)
    J_v_range[i] = L_p * S_capillary * net_P

fig, axes = plt.subplots(2, 2, figsize=(14, 10))

ax1 = axes[0, 0]
ax1.plot(P_c_range, J_v_range * 1e3, 'b-', lw=2.5, label='J_v (mL/min/cm² → μL/min/cm²)')
ax1.axhline(0, color='gray', ls='--', lw=1)
ax1.axvline(35, color='red', ls='--', lw=1.5, label='Normal P_c arteriolar (35 mmHg)')
ax1.axvline(15, color='green', ls='--', lw=1.5, label='Normal P_c venular (15 mmHg)')
ax1.fill_between(P_c_range, J_v_range*1e3, 0, where=(J_v_range > 0), alpha=0.3, label='Filtration')
ax1.fill_between(P_c_range, J_v_range*1e3, 0, where=(J_v_range < 0), alpha=0.3, color='orange', label='Reabsorption')
ax1.set_xlabel('Capillary Pressure P_c (mmHg)', fontsize=12)
ax1.set_ylabel('J_v (μL/min/cm²)', fontsize=12)
ax1.set_title('Starling Filtration vs. Capillary Pressure', fontsize=14)
ax1.legend(fontsize=9)
ax1.grid(True, alpha=0.3)

# =============================================================
# Compare edema scenarios
# =============================================================
scenarios = {
    'Normal':               {'P_c_art': 35, 'P_c_ven': 15, 'pi_c': 25},
    'Heart Failure':        {'P_c_art': 55, 'P_c_ven': 35, 'pi_c': 25},
    'Hypoalbuminemia':      {'P_c_art': 35, 'P_c_ven': 15, 'pi_c': 12},
    'Inflammation (↑L_p)':  {'P_c_art': 35, 'P_c_ven': 15, 'pi_c': 25},
    'Sepsis':               {'P_c_art': 55, 'P_c_ven': 35, 'pi_c': 12},
}

ax2 = axes[0, 1]
colors = ['blue', 'red', 'green', 'orange', 'purple']
x_pos = np.arange(len(scenarios))
net_fluxes = []

for idx, (name, params) in enumerate(scenarios.items()):
    J_art, _ = starling_filtration(params['P_c_art'], P_i, params['pi_c'], pi_i, L_p, sigma_albumin, S_capillary)
    J_ven, _ = starling_filtration(params['P_c_ven'], P_i, params['pi_c'], pi_i, L_p, sigma_albumin, S_capillary)
    net = (J_art + J_ven) / 2
    net_fluxes.append(net * 1e3)
    
ax2.bar(x_pos, net_fluxes, color=colors, alpha=0.8, edgecolor='black')
ax2.set_xticks(x_pos)
ax2.set_xticklabels(list(scenarios.keys()), rotation=30, ha='right', fontsize=9)
ax2.set_ylabel('Net Filtration J_v (μL/min/cm²)', fontsize=12)
ax2.set_title('Edema Scenarios:\nNet Capillary Filtration', fontsize=14)
ax2.axhline(0, color='gray', ls='--', lw=1)
ax2.grid(True, alpha=0.3, axis='y')

# =============================================================
# Lymphatic return compensation
# =============================================================
lymphatic_rate = 0.05  # mL/min/cm² (lymphatic baseline)
ax3 = axes[1, 0]

net_with_lymph = np.array(net_fluxes) - lymphatic_rate * 1e3
ax3.bar(x_pos, net_with_lymph, color=colors, alpha=0.8, edgecolor='black')
ax3.set_xticks(x_pos)
ax3.set_xticklabels(list(scenarios.keys()), rotation=30, ha='right', fontsize=9)
ax3.set_ylabel('Net Accumulation (μL/min/cm²)', fontsize=12)
ax3.set_title('Edema Accumulation:\nFiltration - Lymphatic Return', fontsize=14)
ax3.axhline(0, color='gray', ls='--', lw=1)
ax3.grid(True, alpha=0.3, axis='y')

print("\n=== Edema Risk Assessment ===")
for name, flux in zip(scenarios.keys(), net_with_lymph):
    status = "⚠️ HIGH RISK" if flux > 20 else "Moderate" if flux > 5 else "Normal"
    print(f"  {name}: {flux:.2f} μL/min/cm² → {status}")

# =============================================================
# P_c along capillary length
# =============================================================
ax4 = axes[1, 1]
x_cap = np.linspace(0, 1, 100)  # Normalized capillary length
P_c_profile = 35 - 20 * x_cap   # Linear drop from 35 to 15 mmHg
ax4.plot(x_cap, P_c_profile, 'b-', lw=2.5, label='Normal')
ax4.plot(x_cap, 55 - 20 * x_cap, 'r--', lw=2, label='Heart Failure')
ax4.axhline(40, color='green', ls=':', lw=1.5, label='Edema threshold (~40 mmHg)')
ax4.set_xlabel('Normalized Capillary Length', fontsize=12)
ax4.set_ylabel('Capillary Pressure P_c (mmHg)', fontsize=12)
ax4.set_title('Capillary Pressure Profile:\nArteriolar → Venular', fontsize=14)
ax4.legend()
ax4.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W16_starling_forces.png', dpi=150)
plt.show()
```

---

## Exercise 4: Krogh Cylinder — Tissue Oxygenation Model

```python
"""
W16_CodeLab_Ex4_KroghOxygen.py
================================
Model tissue oxygenation using the Krogh cylinder model.
Calculate P_O₂ profiles from capillary to tissue.
Compare resting vs. exercise conditions.
"""

import numpy as np
import matplotlib.pyplot as plt

# =============================================================
# PARAMETERS
# =============================================================
D_O2     = 1.5e-5   # cm²/s, O₂ diffusivity in tissue
alpha_O2 = 3.0e-2   # mL O₂/100g/mmHg, O₂ solubility
rho_t    = 1.0      # g/mL, tissue density

# Capillary parameters
R_capillary = 3e-4   # cm = 3 μm
R_tissue    = 25e-4   # cm = 25 μm (Krogh cylinder radius)
L_cap       = 0.05   # cm = 500 μm capillary length

# Metabolic rates
M_rest      = 3.5    # mL O₂/100g/min (resting)
M_exercise   = 70.0   # mL O₂/100g/min (max exercise)
M_tumor     = 2.0    # mL O₂/100g/min (tumor, hypoxic)

# =============================================================
# KROGH MODEL: P_O₂ profile
# =============================================================
def krogh_PO2(r, R_t, P_cap, M, D=D_O2, alpha=alpha_O2):
    """
    Calculate tissue P_O₂ at radius r from capillary center.
    Krogh solution: P(r) = P_cap - M*r²/(4*D*alpha)
    """
    return P_cap - (M * r**2) / (4 * D * alpha)

# =============================================================
# Critical tissue radius (where P_O₂ → 0)
# =============================================================
def critical_radius(P_cap, M, D=D_O2, alpha=alpha_O2):
    return 2 * np.sqrt(D * alpha * P_cap / M)

# =============================================================
# SCENARIO COMPARISON
# =============================================================
scenarios = [
    ('Resting muscle',     40,  M_rest),
    ('Light exercise',     60,  M_rest * 3),
    ('Max exercise',       80,  M_exercise),
    ('Hypoxic tumor',     20,  M_tumor),
]

r_grid = np.linspace(0, R_tissue * 1.2, 200)

print("=== Krogh Model: Critical Radii ===")
for name, P_cap, M in scenarios:
    r_crit = critical_radius(P_cap, M)
    viable = "✓ Viable" if r_crit > R_tissue else "✗ HYPOXIA"
    print(f"  {name}: P_cap={P_cap} mmHg, Ṁ={M:.1f} mL/100g/min → r_crit={r_crit*1e4:.1f} μm → {viable}")

# =============================================================
# PLOT: P_O₂ profiles
# =============================================================
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

ax1 = axes[0]
colors = ['blue', 'green', 'red', 'orange']
for (name, P_cap, M), col in zip(scenarios, colors):
    P_O2 = krogh_PO2(r_grid, R_tissue, P_cap, M)
    ax1.plot(r_grid * 1e4, P_O2, color=col, lw=2.5, label=f'{name}')

ax1.axhline(10, color='gray', ls='--', lw=1.5, label='Hypoxia threshold (10 mmHg)')
ax1.axvline(R_tissue * 1e4, color='black', ls=':', lw=1.5, label=f'Tissue boundary (R_t={R_tissue*1e4:.0f} μm)')
ax1.set_xlabel('Distance from Capillary (μm)', fontsize=12)
ax1.set_ylabel('P_O₂ (mmHg)', fontsize=12)
ax1.set_title('Krogh Model: P_O₂ Profile\nFrom Capillary to Tissue', fontsize=14)
ax1.legend(fontsize=9)
ax1.grid(True, alpha=0.3)
ax1.set_xlim(0, R_tissue * 1.2 * 1e4)

# =============================================================
# OXYGEN CONSUMPTION vs. CAPILLARY P_O₂
# =============================================================
P_cap_range = np.linspace(5, 100, 100)
ax2 = axes[1]

for (name, P_base, M), col in zip(scenarios[:3], colors[:3]):
    r_crit_vals = critical_radius(P_cap_range, M)
    ax2.plot(P_cap_range, r_crit_vals * 1e4, color=col, lw=2.5, label=f'{name}')

ax2.axhline(R_tissue * 1e4, color='black', ls=':', lw=1.5, label=f'R_t = {R_tissue*1e4:.0f} μm')
ax2.set_xlabel('Capillary P_O₂ (mmHg)', fontsize=12)
ax2.set_ylabel('Critical Radius r_crit (μm)', fontsize=12)
ax2.set_title('Critical Radius vs. Capillary P_O₂\n(Krogh Model)', fontsize=14)
ax2.legend(fontsize=10)
ax2.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('W16_krogh_oxygen.png', dpi=150)
plt.show()

# =============================================================
# Exercise: Half-time of O₂ diffusion
# =============================================================
print("\n=== O₂ Diffusion Time Scales ===")
for dist in [1, 5, 10, 25, 50]:  # μm
    tau = (dist * 1e-4)**2 / D_O2
    print(f"  Distance {dist:2d} μm: τ = {tau:.3f} s")
```

---

## Lab Report Checklist

| Task | Description | File |
|------|-------------|------|
| ☐ Ex 1 | Steady-state diffusion profiles for O₂, glucose, albumin | `W16_steady_diffusion.png` |
| ☐ Ex 2 | Transient diffusion: concentration evolution over time | `W16_transient_diffusion.png` |
| ☐ Ex 3 | Starling forces: edema scenarios bar chart | `W16_starling_forces.png` |
| ☐ Ex 4 | Krogh model: P_O₂ profiles, critical radius analysis | `W16_krogh_oxygen.png` |
| ☐ Written | Answer questions on explicit FD stability, τ = L²/D |

---

## Extension Challenge

**Challenge 1**: Implement the implicit (Crank-Nicolson) finite difference scheme for transient diffusion. Compare stability and accuracy vs. explicit scheme for large time steps (Δt >> dx²/2D).

**Challenge 2**: Build a 2-capillary Krogh model where two capillaries are separated by distance D_sep. Calculate the overlap of oxygen fields and identify the "worst-case" tissue P_O₂ point in the midpoint between capillaries. Apply to tumor with intercapillary distance of 200 μm.

**Challenge 3**: Implement the Kedem-Katchalsky equations numerically for albumin transport across a capillary wall, varying σ from 0 to 1 and plotting J_s vs. J_v (the solvent drag curve).
