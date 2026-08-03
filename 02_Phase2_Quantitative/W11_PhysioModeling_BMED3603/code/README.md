# Week 11 Code Lab — Physiological Modeling with ODEs (BMED3603)

> **Topics Covered**: First-order ODEs, second-order systems, compartment models, Hodgkin-Huxley, scipy.integrate.odeint
> **Prerequisites**: numpy, matplotlib, scipy
> **Estimated Time**: 2-3 hours

---

## Lab 1: First-Order ODEs — Pharmacokinetics

### Objective
Model one-compartment pharmacokinetics using first-order ODEs, solve using scipy.integrate.odeint, compute model parameters from data, and analyze the effect of dosing regimens.

### Code

```python
"""
Week 11 Lab 1: Pharmacokinetic Modeling with First-Order ODEs
============================================================
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy.integrate import odeint, solve_ivp
from scipy.optimize import curve_fit, minimize
import warnings
warnings.filterwarnings('ignore')

print("=" * 65)
print("LAB 1: Pharmacokinetic Modeling with First-Order ODEs")
print("=" * 65)

# ---------------------------------------------------------
# SECTION 1: One-Compartment Model — Single Dose
# ---------------------------------------------------------
print("\n[1] One-Compartment PK Model — Single Dose IV Bolus")

def pk_one_compartment(y, t, V_d, k_e):
    """
    One-compartment PK model with IV bolus dose.
    
    dC/dt = -k_e * C
    
    Parameters:
        y: [C] — drug concentration (mg/L)
        t: time (hours)
        V_d: apparent volume of distribution (L)
        k_e: elimination rate constant (hr^-1)
    
    Returns:
        dC/dt: rate of change of concentration
    """
    C = y[0]
    dCdt = -k_e * C
    return [dCdt]

# Model parameters
V_d = 50.0      # L (volume of distribution)
k_e = 0.1       # hr^-1 (elimination rate constant)
half_life = np.log(2) / k_e  # hours

print(f"  Model parameters:")
print(f"    V_d = {V_d} L")
print(f"    k_e = {k_e} hr^-1")
print(f"    Half-life: t_1/2 = {half_life:.2f} hours")
print(f"    Time constant: τ = {1/k_e:.2f} hours")

# Initial condition: dose/V_d
dose = 500  # mg (IV bolus)
C0 = dose / V_d  # mg/L

# Time span: 0 to 48 hours
t_span = np.linspace(0, 48, 500)

# Solve ODE
solution = odeint(pk_one_compartment, [C0], t_span, args=(V_d, k_e))
C_t = solution[:, 0]

print(f"\n  Simulation results:")
print(f"    Initial concentration: C(0) = {C0:.2f} mg/L")
print(f"    Concentration at 24h: C(24) = {C0*np.exp(-k_e*24):.4f} mg/L")
print(f"    Concentration at 48h: C(48) = {C0*np.exp(-k_e*48):.6f} mg/L")
print(f"    Fraction remaining at 24h: {np.exp(-k_e*24)*100:.2f}%")

# Analytical solution for comparison
C_analytical = C0 * np.exp(-k_e * t_span)
max_error = np.max(np.abs(C_t - C_analytical))
print(f"    Max error (ODE vs analytical): {max_error:.2e}")

# Plot
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

axes[0].plot(t_span, C_t, 'b-', linewidth=2, label='Numerical ODE')
axes[0].plot(t_span, C_analytical, 'r--', linewidth=1.5, alpha=0.7, label='Analytical')
axes[0].axhline(y=C0 * 0.5, color='gray', linestyle=':', alpha=0.7)
axes[0].axvline(x=half_life, color='gray', linestyle=':', alpha=0.7)
axes[0].annotate(f't_1/2 = {half_life:.1f}h', xy=(half_life, C0*0.5), 
                xytext=(half_life+3, C0*0.6), fontsize=10)
axes[0].set_xlabel('Time (hours)', fontsize=11)
axes[0].set_ylabel('Concentration (mg/L)', fontsize=11)
axes[0].set_title('One-Compartment PK: Single IV Bolus Dose', fontsize=12)
axes[0].legend()
axes[0].grid(True, alpha=0.3)

# Semi-log plot
axes[1].semilogy(t_span, C_t, 'b-', linewidth=2)
axes[1].axhline(y=C0 * 0.5, color='gray', linestyle=':', alpha=0.7, label='50% of C0')
axes[1].axvline(x=half_life, color='gray', linestyle=':', alpha=0.7)
axes[1].set_xlabel('Time (hours)', fontsize=11)
axes[1].set_ylabel('Concentration (mg/L) — log scale', fontsize=11)
axes[1].set_title('Semi-Log Plot (Exponential = Linear)', fontsize=12)
axes[1].legend()
axes[1].grid(True, alpha=0.3, which='both')

plt.tight_layout()
plt.savefig('lab1_section1_single_dose.png', dpi=150)
plt.show()
print("  [Saved] lab1_section1_single_dose.png")

# ---------------------------------------------------------
# SECTION 2: Multiple Dosing — Steady State
# ---------------------------------------------------------
print("\n[2] Multiple Dosing — Accumulation and Steady State")

def pk_multiple_doses(t, dose, V_d, k_e, tau):
    """
    Multiple dose regimen: dose every tau hours.
    
    Concentration after n doses at time t (within dosing interval):
    C(t) = (D/V_d) * e^{-k_e*t} * (1 + e^{-k_e*tau} + ... + e^{-k_e*(n-1)*tau})
    
    At steady state (n → ∞):
    C_ss(t) = (D/V_d) * e^{-k_e*t} / (1 - e^{-k_e*tau})
    """
    n_doses = int(t / tau) + 1
    C_total = 0
    for n in range(n_doses):
        t_dose = t - n * tau
        if t_dose >= 0:
            C_total += (dose / V_d) * np.exp(-k_e * t_dose)
    return C_total

# Dosing regimen: 100 mg every 12 hours
dose_interval = 100  # mg
tau = 12  # hours between doses

# Simulate for 7 days
t_multiple = np.linspace(0, 168, 2000)
C_multiple = np.array([pk_multiple_doses(ti, dose_interval, V_d, k_e, tau) 
                        for ti in t_multiple])

# Compute steady-state concentration
C_ss_max = (dose_interval / V_d) / (1 - np.exp(-k_e * tau))
C_ss_min = C_ss_max * np.exp(-k_e * tau)
accumulation_factor = 1 / (1 - np.exp(-k_e * tau))

print(f"  Dosing regimen: {dose_interval} mg every {tau} hours")
print(f"  C_max at steady state: {C_ss_max:.4f} mg/L")
print(f"  C_min at steady state: {C_ss_min:.4f} mg/L")
print(f"  Accumulation factor: {accumulation_factor:.2f}")
print(f"  Time to steady state (~95%): {3*half_life:.1f} hours ≈ {int(3*half_life/tau)} dosing intervals")

# Plot
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

axes[0].plot(t_multiple, C_multiple, 'b-', linewidth=1)
axes[0].axhline(y=C_ss_max, color='r', linestyle='--', label=f'SS C_max = {C_ss_max:.2f}')
axes[0].axhline(y=C_ss_min, color='orange', linestyle='--', label=f'SS C_min = {C_ss_min:.2f}')
axes[0].set_xlabel('Time (hours)', fontsize=11)
axes[0].set_ylabel('Concentration (mg/L)', fontsize=11)
axes[0].set_title('Multiple Dose Regimen: Accumulation', fontsize=12)
axes[0].legend()
axes[0].grid(True, alpha=0.3)

# Zoom: show last 3 dosing intervals
mask = t_multiple > 144
axes[1].plot(t_multiple[mask], C_multiple[mask], 'b-', linewidth=2)
axes[1].axhline(y=C_ss_max, color='r', linestyle='--')
axes[1].axhline(y=C_ss_min, color='orange', linestyle='--')
axes[1].axhline(y=C_ss_max * 0.95, color='gray', linestyle=':', alpha=0.7)
axes[1].axvlines([156, 168], 0, C_ss_max, color='green', linestyle=':', alpha=0.5)
axes[1].set_xlabel('Time (hours)', fontsize=11)
axes[1].set_ylabel('Concentration (mg/L)', fontsize=11)
axes[1].set_title('Last 24 Hours: Steady State Reached', fontsize=12)
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('lab1_section2_multiple_dosing.png', dpi=150)
plt.show()
print("  [Saved] lab1_section2_multiple_dosing.png")

# ---------------------------------------------------------
# SECTION 3: Parameter Estimation from Data
# ---------------------------------------------------------
print("\n[3] Parameter Estimation — Fitting PK Model to Data")

# Simulated concentration data (with noise)
np.random.seed(999)
t_data = np.array([0.5, 1, 2, 4, 8, 12, 24])
C_observed = C0 * np.exp(-k_e * t_data) + 0.15 * np.random.randn(len(t_data))
C_observed = np.maximum(C_observed, 0.1)  # no negative concentrations

def pk_model(t, V_d, k_e):
    """PK model for fitting"""
    return (dose / V_d) * np.exp(-k_e * t)

# Fit using curve_fit
popt, pcov = curve_fit(pk_model, t_data, C_observed, 
                         p0=[40, 0.2],  # initial guesses
                         bounds=([1, 0.01], [200, 2.0]))  # bounds

V_d_fit, k_e_fit = popt
V_d_err, k_e_err = np.sqrt(np.diag(pcov))
half_life_fit = np.log(2) / k_e_fit

print(f"  True parameters: V_d = {V_d} L, k_e = {k_e} hr^-1, t_1/2 = {half_life:.2f} hr")
print(f"  Fitted parameters: V_d = {V_d_fit:.2f} ± {V_d_err:.2f} L")
print(f"                      k_e = {k_e_fit:.4f} ± {k_e_err:.4f} hr^-1")
print(f"                      t_1/2 = {half_life_fit:.2f} hr")
print(f"  Relative error: V_d: {abs(V_d_fit-V_d)/V_d*100:.1f}%, k_e: {abs(k_e_fit-k_e)/k_e*100:.1f}%")

# R-squared
C_pred = pk_model(t_data, V_d_fit, k_e_fit)
ss_res = np.sum((C_observed - C_pred)**2)
ss_tot = np.sum((C_observed - np.mean(C_observed))**2)
r_squared = 1 - ss_res/ss_tot
print(f"  R² = {r_squared:.4f}")

# Plot fit
t_smooth = np.linspace(0, 24, 200)
C_fit = pk_model(t_smooth, V_d_fit, k_e_fit)

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

axes[0].scatter(t_data, C_observed, s=100, color='red', zorder=5, label='Observed data')
axes[0].plot(t_smooth, C_fit, 'b-', linewidth=2, label=f'Fitted model\nV_d={V_d_fit:.1f}L, t_1/2={half_life_fit:.1f}h')
axes[0].plot(t_smooth, pk_model(t_smooth, V_d, k_e), 'g--', linewidth=1.5, alpha=0.7, label='True model')
axes[0].set_xlabel('Time (hours)', fontsize=11)
axes[0].set_ylabel('Concentration (mg/L)', fontsize=11)
axes[0].set_title('Parameter Estimation: Fitting PK Model to Data', fontsize=12)
axes[0].legend()
axes[0].grid(True, alpha=0.3)

# Residuals
residuals = C_observed - C_pred
axes[1].scatter(t_data, residuals, s=100, color='purple')
axes[1].axhline(y=0, color='k', linestyle='--', linewidth=1)
axes[1].set_xlabel('Time (hours)', fontsize=11)
axes[1].set_ylabel('Residual (mg/L)', fontsize=11)
axes[1].set_title('Residuals: Observed − Predicted', fontsize=12)
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('lab1_section3_parameter_estimation.png', dpi=150)
plt.show()
print("  [Saved] lab1_section3_parameter_estimation.png")

print("\n✅ Lab 1 Complete!")
```

### Expected Output
- Single dose PK curve with half-life annotation
- Multiple dosing accumulation and steady-state analysis
- Parameter estimation with fitted curve and residuals

---

## Lab 2: Second-Order Systems — Cardiovascular Modeling

### Objective
Model the cardiovascular system as a second-order system, analyze underdamped, critically damped, and overdamped responses, and relate damping ratio to clinical conditions.

### Code

```python
"""
Week 11 Lab 2: Second-Order Systems — Cardiovascular Dynamics
============================================================
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy.integrate import odeint

print("=" * 65)
print("LAB 2: Second-Order Systems — Cardiovascular Dynamics")
print("=" * 65)

# ---------------------------------------------------------
# SECTION 1: Second-Order ODE — Cardiovascular Response
# ---------------------------------------------------------
print("\n[1] Second-Order System — Cardiovascular Response to Step Input")

def second_order_system(y, t, wn, zeta):
    """
    Second-order system in standard form:
    d²y/dt² + 2ζω_n(dy/dt) + ω_n²y = ω_n²x(t)
    
    State variables: y[0] = x, y[1] = dx/dt
    """
    x = y[0]
    dxdt = y[1]
    
    # For unit step input: RHS = ω_n²
    d2xdt2 = wn**2 * 1.0 - 2 * zeta * wn * dxdt - wn**2 * x
    
    return [dxdt, d2xdt2]

# Cardiovascular system parameters
# ω_n: natural frequency (rad/s)
# ζ: damping ratio

# Normal cardiovascular: slightly underdamped
wn_normal = 2.0    # Natural frequency
zeta_normal = 0.4  # Damping ratio

# Hypertensive (stiffer vessels): higher natural frequency
wn_hyper = 3.5
zeta_hyper = 0.5

# Heart failure (lower cardiac output): lower natural frequency
wn_hf = 1.2
zeta_hf = 0.3

# Time span
t_span = np.linspace(0, 8, 500)

# Solve for different conditions
y_normal = odeint(second_order_system, [0, 0], t_span, args=(wn_normal, zeta_normal))
y_hyper = odeint(second_order_system, [0, 0], t_span, args=(wn_hyper, zeta_hyper))
y_hf = odeint(second_order_system, [0, 0], t_span, args=(wn_hf, zeta_hf))

# Extract positions (responses)
x_normal = y_normal[:, 0]
x_hyper = y_hyper[:, 0]
x_hf = y_hf[:, 0]

# Compute characteristics
def second_order_characteristics(wn, zeta):
    """Compute key characteristics of second-order system"""
    sigma = zeta * wn  # decay rate
    wd = wn * np.sqrt(max(0, 1 - zeta**2))  # damped frequency
    
    if zeta < 1:
        # Underdamped
        overshoot = np.exp(-zeta * np.pi / np.sqrt(1 - zeta**2)) * 100
        peak_time = np.pi / wd if wd > 0 else float('inf')
    elif zeta == 1:
        overshoot = 0
        peak_time = 1 / wn  # critical
    else:
        overshoot = 0
        peak_time = None
    
    settling_time = 4 / sigma  # 2% criterion
    rise_time = (np.pi - np.arctan(wd/sigma)) / wd if zeta < 1 else None
    
    return {
        'overshoot': overshoot,
        'peak_time': peak_time,
        'settling_time': settling_time,
        'rise_time': rise_time,
        'sigma': sigma,
        'wd': wd
    }

chars_normal = second_order_characteristics(wn_normal, zeta_normal)
chars_hyper = second_order_characteristics(wn_hyper, zeta_hyper)
chars_hf = second_order_characteristics(wn_hf, zeta_hf)

print(f"  Normal: overshoot = {chars_normal['overshoot']:.1f}%, settling = {chars_normal['settling_time']:.2f}s")
print(f"  Hypertensive: overshoot = {chars_hyper['overshoot']:.1f}%, settling = {chars_hyper['settling_time']:.2f}s")
print(f"  Heart failure: overshoot = {chars_hf['overshoot']:.1f}%, settling = {chars_hf['settling_time']:.2f}s")

fig, axes = plt.subplots(2, 2, figsize=(14, 10))

axes[0, 0].plot(t_span, x_normal, 'b-', linewidth=2, label=f'Normal (ζ={zeta_normal}, ω_n={wn_normal})')
axes[0, 0].plot(t_span, x_hyper, 'r-', linewidth=2, label=f'Hypertensive (ζ={zeta_hyper}, ω_n={wn_hyper})')
axes[0, 0].plot(t_span, x_hf, 'g-', linewidth=2, label=f'Heart Failure (ζ={zeta_hf}, ω_n={wn_hf})')
axes[0, 0].axhline(y=1, color='k', linestyle='--', alpha=0.5, label='Steady state')
axes[0, 0].axhline(y=1.05, color='gray', linestyle=':', alpha=0.5)  # 5% overshoot
axes[0, 0].axhline(y=0.98, color='gray', linestyle=':', alpha=0.5)  # 2% band
axes[0, 0].set_xlabel('Time (seconds)', fontsize=11)
axes[0, 0].set_ylabel('Response (normalized)', fontsize=11)
axes[0, 0].set_title('Step Response: Normal vs Pathological', fontsize=12)
axes[0, 0].legend()
axes[0, 0].grid(True, alpha=0.3)

# ---------------------------------------------------------
# SECTION 2: Effect of Damping Ratio
# ---------------------------------------------------------
print("\n[2] Effect of Damping Ratio on System Response")

zetas = [0.1, 0.3, 0.5, 0.7, 1.0, 2.0]
wn = 2.0  # fixed natural frequency

responses = {}
for zeta_val in zetas:
    y = odeint(second_order_system, [0, 0], t_span, args=(wn, zeta_val))
    responses[zeta_val] = y[:, 0]

axes[0, 1].plot(t_span, responses[0.1], 'r-', linewidth=1.5, label='ζ=0.1 (underdamped)')
axes[0, 1].plot(t_span, responses[0.3], 'orange', linewidth=1.5, label='ζ=0.3')
axes[0, 1].plot(t_span, responses[0.5], 'g-', linewidth=1.5, label='ζ=0.5')
axes[0, 1].plot(t_span, responses[0.7], 'cyan', linewidth=1.5, label='ζ=0.7')
axes[0, 1].plot(t_span, responses[1.0], 'b-', linewidth=1.5, label='ζ=1.0 (critically damped)')
axes[0, 1].plot(t_span, responses[2.0], 'purple', linewidth=1.5, label='ζ=2.0 (overdamped)')
axes[0, 1].axhline(y=1, color='k', linestyle='--', alpha=0.5)
axes[0, 1].set_xlabel('Time (seconds)', fontsize=11)
axes[0, 1].set_ylabel('Response', fontsize=11)
axes[0, 1].set_title(f'Effect of Damping Ratio (ω_n = {wn} rad/s)', fontsize=12)
axes[0, 1].legend(fontsize=9)
axes[0, 1].grid(True, alpha=0.3)

# ---------------------------------------------------------
# SECTION 3: Pole-Zero Analysis
# ---------------------------------------------------------
print("\n[3] Pole Locations in Complex Plane")

# Poles of second-order system: s = -ζω_n ± jω_n√(1-ζ²)
zetas_pz = [0.1, 0.3, 0.5, 0.7, 1.0]
wn_pz = 2.0

for zeta_val in zetas_pz:
    sigma = zeta_val * wn_pz
    if zeta_val < 1:
        wd = wn_pz * np.sqrt(1 - zeta_val**2)
        p1 = -sigma + 1j * wd
        p2 = -sigma - 1j * wd
        print(f"  ζ={zeta_val}: poles = {-sigma:.3f} ± j{wd:.3f} (underdamped)")
    elif zeta_val == 1:
        p = -wn_pz
        print(f"  ζ={zeta_val}: pole = {p:.3f} (critically damped)")
    else:
        import cmath
        sqrt_term = cmath.sqrt(zeta_val**2 - 1)
        p1 = -sigma + wn_pz * sqrt_term
        p2 = -sigma - wn_pz * sqrt_term
        print(f"  ζ={zeta_val}: poles = {p1:.3f}, {p2:.3f} (overdamped)")

# Plot pole locations
unit_circle = plt.Circle((0, 0), 1, fill=False, color='gray', linewidth=1, linestyle='--')
axes[1, 0].add_patch(unit_circle)
axes[1, 0].set_xlim(-5, 1)
axes[1, 0].set_ylim(-3, 3)
axes[1, 0].set_aspect('equal')

# Shade regions
axes[1, 0].fill_between([-5, 0], -5, 5, alpha=0.05, color='green')
axes[1, 0].fill_between([0, 1], -5, 5, alpha=0.05, color='red')

colors_pz = ['red', 'orange', 'green', 'cyan', 'blue']
for i, zeta_val in enumerate(zetas_pz):
    sigma = zeta_val * wn_pz
    if zeta_val < 1:
        wd = wn_pz * np.sqrt(1 - zeta_val**2)
        axes[1, 0].scatter([-sigma], [wd], s=100, c=colors_pz[i], marker='x', linewidths=2)
        axes[1, 0].scatter([-sigma], [-wd], s=100, c=colors_pz[i], marker='x', linewidths=2)
        axes[1, 0].plot([-sigma, -sigma], [wd, -wd], colors_pz[i], linewidth=1)
    elif zeta_val == 1:
        axes[1, 0].scatter([-wn_pz], [0], s=100, c=colors_pz[i], marker='x', linewidths=2)
    else:
        sqrt_term = np.sqrt(zeta_val**2 - 1)
        p1_real = -sigma + wn_pz * sqrt_term
        axes[1, 0].scatter([p1_real], [0], s=100, c=colors_pz[i], marker='x', linewidths=2)
        axes[1, 0].scatter([-sigma], [0], s=100, c=colors_pz[i], marker='x', linewidths=2)

axes[1, 0].axhline(y=0, color='k', linewidth=0.5)
axes[1, 0].axvline(x=0, color='k', linewidth=0.5)
axes[1, 0].set_xlabel('Real (σ)', fontsize=11)
axes[1, 0].set_ylabel('Imaginary (jω)', fontsize=11)
axes[1, 0].set_title('Pole Locations vs Damping Ratio', fontsize=12)
axes[1, 0].text(-4.5, 2.5, 'STABLE\n(LHP)', fontsize=9, color='green', alpha=0.7)
axes[1, 0].text(0.2, 2.5, 'UNSTABLE\n(RHP)', fontsize=9, color='red', alpha=0.7)
axes[1, 0].grid(True, alpha=0.3)

# Legend for poles
for i, zeta_val in enumerate(zetas_pz):
    axes[1, 0].scatter([], [], s=80, c=colors_pz[i], marker='x', linewidths=2, label=f'ζ={zeta_val}')

# ---------------------------------------------------------
# SECTION 4: Bode Plot — Frequency Response
# ---------------------------------------------------------
print("\n[4] Frequency Response — Bode Plot")

omega_range = np.logspace(-1, 2, 500)
w_range = omega_range  # rad/s

fig_bode, axes_bode = plt.subplots(2, 1, figsize=(12, 8))

for name, zeta_val, wn_val in [('Normal', zeta_normal, wn_normal),
                                 ('Hypertensive', zeta_hyper, wn_hyper),
                                 ('Heart Failure', zeta_hf, wn_hf)]:
    H = (wn_val**2) / (-w_range**2 + 2j*zeta_val*wn_val*w_range + wn_val**2)
    mag = 20 * np.log10(np.abs(H) + 1e-10)
    phase = np.unwrap(np.angle(H)) * 180 / np.pi
    
    axes_bode[0].semilogx(omega_range, mag, linewidth=2, label=name)
    axes_bode[1].semilogx(omega_range, phase, linewidth=2, label=name)

axes_bode[0].axhline(y=-3, color='gray', linestyle=':', alpha=0.5)
axes_bode[0].axvline(x=wn_normal, color='blue', linestyle=':', alpha=0.3)
axes_bode[0].set_ylabel('|H(jω)| (dB)', fontsize=11)
axes_bode[0].set_title('Bode Plot: Cardiovascular System Frequency Response', fontsize=12)
axes_bode[0].legend()
axes_bode[0].grid(True, which='both', alpha=0.3)
axes_bode[0].set_xlim(0.1, 100)
axes_bode[0].set_ylim(-40, 10)

axes_bode[1].set_xlabel('ω (rad/s)', fontsize=11)
axes_bode[1].set_ylabel('∠H(jω) (degrees)', fontsize=11)
axes_bode[1].legend()
axes_bode[1].grid(True, which='both', alpha=0.3)
axes_bode[1].set_xlim(0.1, 100)
axes_bode[1].set_ylim(-200, 20)

plt.tight_layout()
plt.savefig('lab2_section4_bode.png', dpi=150)
plt.show()
print("  [Saved] lab2_section4_bode.png")

# Main second-order figure
axes[1, 1].remove()
plt.tight_layout()
fig.set_size_inches(14, 12)
plt.savefig('lab2_section2_damping.png', dpi=150)
plt.show()
print("  [Saved] lab2_section2_damping.png")

print("\n✅ Lab 2 Complete!")
```

### Expected Output
- Step response curves for normal vs hypertensive vs heart failure cardiovascular systems
- Pole-zero diagram showing effect of damping ratio
- Bode plots showing frequency response

---

## Lab 3: Hodgkin-Huxley — Neuron Action Potential

### Objective
Implement a simplified Hodgkin-Huxley model for action potential generation, analyze the ion channel dynamics (m, h, n gates), and simulate different stimulation protocols.

### Code

```python
"""
Week 11 Lab 3: Hodgkin-Huxley Neuron Model — Action Potential
===============================================================
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy.integrate import odeint

print("=" * 65)
print("LAB 3: Hodgkin-Huxley Neuron Model")
print("=" * 65)

# ---------------------------------------------------------
# SECTION 1: Hodgkin-Huxley Model Implementation
# ---------------------------------------------------------
print("\n[1] Hodgkin-Huxley Model — Action Potential")

def hodgkin_huxley(y, t, I_stim, g_Na, g_K, g_L, E_Na, E_K, E_L):
    """
    Hodgkin-Huxley model for squid giant axon.
    
    State variables:
        V: membrane potential (mV)
        m: Na+ channel activation (fast)
        h: Na+ channel inactivation (slow)
        n: K+ channel activation
    
    Equations:
        C dV/dt = I_stim(t) - g_Na*m³*h*(V-E_Na) - g_K*n⁴*(V-E_K) - g_L*(V-E_L)
        dm/dt = α_m(V)*(1-m) - β_m(V)*m
        dh/dt = α_h(V)*(1-h) - β_h(V)*h
        dn/dt = α_n(V)*(1-n) - β_n(V)*n
    
    Parameters (typical squid axon at 6.3°C):
        C = 1 μF/cm²
        g_Na = 120 mS/cm²
        g_K = 36 mS/cm²  
        g_L = 0.3 mS/cm²
        E_Na = 50 mV
        E_K = -77 mV
        E_L = -54.4 mV
    """
    V, m, h, n = y
    
    # Channel kinetics (rate constants)
    alpha_m = 0.1 * (25 - V) / (np.exp((25 - V) / 10) - 1)
    beta_m = 4 * np.exp(-V / 18)
    
    alpha_h = 0.07 * np.exp(-V / 20)
    beta_h = 1 / (1 + np.exp((30 - V) / 10))
    
    alpha_n = 0.01 * (10 - V) / (np.exp((10 - V) / 10) - 1)
    beta_n = 0.125 * np.exp(-V / 80)
    
    # Ion currents
    I_Na = g_Na * (m ** 3) * h * (V - E_Na)
    I_K = g_K * (n ** 4) * (V - E_K)
    I_L = g_L * (V - E_L)
    
    # Membrane equation
    dVdt = (I_stim(t, V) - I_Na - I_K - I_L) / C
    dmdt = alpha_m * (1 - m) - beta_m * m
    dhdt = alpha_h * (1 - h) - beta_h * h
    dndt = alpha_n * (1 - n) - beta_n * n
    
    return [dVdt, dmdt, dhdt, dndt]

# Model parameters
C = 1.0          # membrane capacitance (μF/cm²)
g_Na = 120.0     # Na+ conductance (mS/cm²)
g_K = 36.0       # K+ conductance (mS/cm²)
g_L = 0.3        # leak conductance (mS/cm²)
E_Na = 50.0      # Na+ reversal potential (mV)
E_K = -77.0      # K+ reversal potential (mV)
E_L = -54.4      # leak reversal potential (mV)

# Stimulation protocol
def I_stim(t, V, amplitude=10, duration=1.0, delay=10):
    """Current injection: step current starting at t=delay"""
    if delay <= t <= delay + duration:
        return amplitude
    return 0.0

# Time span
t_span = np.linspace(0, 50, 2000)

# Initial conditions: rest
V_rest = -65  # mV
# Compute steady-state values at rest
def steady_state(V):
    alpha_m = 0.1 * (25 - V) / (np.exp((25 - V) / 10) - 1)
    beta_m = 4 * np.exp(-V / 18)
    alpha_h = 0.07 * np.exp(-V / 20)
    beta_h = 1 / (1 + np.exp((30 - V) / 10))
    alpha_n = 0.01 * (10 - V) / (np.exp((10 - V) / 10) - 1)
    beta_n = 0.125 * np.exp(-V / 80)
    m_ss = alpha_m / (alpha_m + beta_m)
    h_ss = alpha_h / (alpha_h + beta_h)
    n_ss = alpha_n / (alpha_n + beta_n)
    return m_ss, h_ss, n_ss

m0, h0, n0 = steady_state(V_rest)
y0 = [V_rest, m0, h0, n0]

# Create stimulation function with closure
class Stimulator:
    def __init__(self, amplitude=10, duration=1.0, delay=10):
        self.amplitude = amplitude
        self.duration = duration
        self.delay = delay
    
    def __call__(self, t, V):
        if self.delay <= t <= self.delay + self.duration:
            return self.amplitude
        return 0.0

stim = Stimulator(amplitude=15, duration=1.0, delay=10)

# Solve ODE
solution = odeint(hodgkin_huxley, y0, t_span, 
                  args=(stim, g_Na, g_K, g_L, E_Na, E_K, E_L))
V_t = solution[:, 0]
m_t = solution[:, 1]
h_t = solution[:, 2]
n_t = solution[:, 3]

# Compute currents
I_Na_t = g_Na * (m_t ** 3) * h_t * (V_t - E_Na)
I_K_t = g_K * (n_t ** 4) * (V_t - E_K)
I_L_t = g_L * (V_t - E_L)

print(f"  Resting potential: {V_rest} mV")
print(f"  Spike peak: {np.max(V_t):.1f} mV")
print(f"  Spike amplitude: {np.max(V_t) - V_rest:.1f} mV")

# Find spike timing
spike_threshold = -20  # mV
spike_mask = V_t > spike_threshold
spike_start_idx = np.argmax(spike_mask)
if spike_mask.any():
    spike_peak_idx = np.argmax(V_t[spike_start_idx:]) + spike_start_idx
    spike_end_idx = np.argmin(V_t[spike_peak_idx:spike_peak_idx+200]) + spike_peak_idx
    spike_width = t_span[spike_end_idx] - t_span[spike_start_idx]
    print(f"  Spike width: {spike_width:.1f} ms")
    print(f"  Spike threshold: {spike_threshold} mV")

# Plot
fig, axes = plt.subplots(3, 1, figsize=(14, 12), sharex=True)

# Membrane potential
axes[0].plot(t_span, V_t, 'b-', linewidth=1.5)
axes[0].axhline(y=spike_threshold, color='r', linestyle='--', alpha=0.5, label=f'Threshold ({spike_threshold} mV)')
axes[0].axhline(y=V_rest, color='gray', linestyle=':', alpha=0.5, label=f'Rest ({V_rest} mV)')
axes[0].axhline(y=np.max(V_t), color='green', linestyle=':', alpha=0.5)
# Mark stimulus
stim_current = np.array([stim(ti, V_rest) for ti in t_span])
axes[0].fill_between(t_span, V_rest-5, V_rest-5+stim_current*0.3, alpha=0.2, color='yellow', label='Stimulus')
axes[0].set_ylabel('Membrane Potential (mV)', fontsize=11)
axes[0].set_title('Hodgkin-Huxley Action Potential', fontsize=14)
axes[0].legend(loc='upper right')
axes[0].grid(True, alpha=0.3)

# Ion channel gating variables
axes[1].plot(t_span, m_t, 'r-', linewidth=1.5, label='m (Na+ activation)')
axes[1].plot(t_span, h_t, 'b-', linewidth=1.5, label='h (Na+ inactivation)')
axes[1].plot(t_span, n_t, 'g-', linewidth=1.5, label='n (K+ activation)')
axes[1].set_ylabel('Gating Variable', fontsize=11)
axes[1].set_title('Ion Channel Gating Variables', fontsize=12)
axes[1].legend()
axes[1].grid(True, alpha=0.3)

# Ion currents
axes[2].plot(t_span, I_Na_t, 'r-', linewidth=1.5, label='I_Na')
axes[2].plot(t_span, I_K_t, 'g-', linewidth=1.5, label='I_K')
axes[2].plot(t_span, I_L_t, 'k-', linewidth=1, label='I_L', alpha=0.5)
axes[2].set_xlabel('Time (ms)', fontsize=11)
axes[2].set_ylabel('Current (μA/cm²)', fontsize=11)
axes[2].set_title('Ion Currents During Action Potential', fontsize=12)
axes[2].legend()
axes[2].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('lab3_section1_hh_action_potential.png', dpi=150)
plt.show()
print("  [Saved] lab3_section1_hh_action_potential.png")

# ---------------------------------------------------------
# SECTION 2: Threshold and Refractory Period
# ---------------------------------------------------------
print("\n[2] Action Potential Threshold — Current Amplitude Study")

def simulate_neuron(amplitude, duration=1.0, delay=10):
    """Simulate HH neuron with given stimulus amplitude"""
    stim_test = Stimulator(amplitude=amplitude, duration=duration, delay=delay)
    sol = odeint(hodgkin_huxley, y0, t_span, 
                  args=(stim_test, g_Na, g_K, g_L, E_Na, E_K, E_L))
    return sol[:, 0]

amplitudes = np.arange(2, 25, 1)
responses = {}

for amp in amplitudes:
    V_response = simulate_neuron(amp)
    responses[amp] = V_response
    max_V = np.max(V_response)
    fired = max_V > spike_threshold
    print(f"  Amplitude = {amp:2.0f} μA/cm²: {'✓ FIRED' if fired else '✗ no spike'} (peak = {max_V:.1f} mV)")

# Plot voltage traces
fig, axes = plt.subplots(2, 1, figsize=(14, 10))

# Show subset of traces
for amp in [5, 8, 10, 12, 15, 20]:
    if amp in responses:
        axes[0].plot(t_span, responses[amp], linewidth=1.5, 
                       label=f'I = {amp} μA/cm²', alpha=0.8)

axes[0].axhline(y=spike_threshold, color='r', linestyle='--', alpha=0.5)
axes[0].set_xlabel('Time (ms)')
axes[0].set_ylabel('Membrane Potential (mV)')
axes[0].set_title('Action Potential Threshold: All-or-None Response')
axes[0].legend()
axes[0].grid(True, alpha=0.3)

# Firing frequency vs current (FI curve)
# Note: this is for a single pulse, not sustained current
firing_status = [np.max(V) > spike_threshold for V in responses.values()]
amplitudes_list = list(responses.keys())

axes[1].step(amplitudes_list, firing_status, 'b-', linewidth=2)
axes[1].set_xlabel('Stimulus Amplitude (μA/cm²)')
axes[1].set_ylabel('Fired? (0/1)')
axes[1].set_title('Firing Threshold: All-or-None Property')
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('lab3_section2_threshold.png', dpi=150)
plt.show()
print("  [Saved] lab3_section2_threshold.png")

# ---------------------------------------------------------
# SECTION 3: Simplified Compartmental Model
# ---------------------------------------------------------
print("\n[3] Compartmental Neuron Model — Cable Equation")

def compartment_model(y, t, I_inj, n_compartments=3, tau=10, R=1.0):
    """
    Simplified multi-compartment cable model.
    
    Each compartment follows: τ dV_i/dt = V_{i-1} - 2V_i + V_{i+1} + R*I_i(t)
    
    State: V_0, V_1, V_2 (for n=3 compartments)
    """
    V = y
    
    dVdt = np.zeros(n_compartments)
    
    # Inject current into middle compartment
    I_stim_arr = np.zeros(n_compartments)
    I_stim_arr[n_compartments // 2] = I_inj(t)
    
    for i in range(n_compartments):
        # Neighbor contributions
        left = V[i-1] if i > 0 else V[i]
        right = V[i+1] if i < n_compartments - 1 else V[i]
        
        # Cable equation
        dVdt[i] = (left - 2*V[i] + right + R * I_stim_arr[i]) / tau
    
    return dVdt

# Parameters
n_comp = 5
tau_cable = 5.0  # ms
R_cable = 1.0

def I_inject(t):
    """Inject current into middle compartment at t=5-6ms"""
    if 5 <= t <= 6:
        return 5.0
    return 0.0

# Initial conditions
V0 = np.zeros(n_comp)  # all at rest

t_comp = np.linspace(0, 50, 500)
sol_comp = odeint(compartment_model, V0, t_comp, 
                   args=(I_inject, n_comp, tau_cable, R_cable))

fig, axes = plt.subplots(2, 1, figsize=(14, 8), sharex=True)

# Space-time plot (waterfall)
for i in range(n_comp):
    offset = i * 5
    axes[0].plot(t_comp, sol_comp[:, i] + offset, linewidth=1.5, 
                   label=f'Compartment {i}')
    axes[0].fill_between(t_comp, offset, sol_comp[:, i] + offset, alpha=0.2)

axes[0].set_ylabel('Membrane Potential (mV, offset for clarity)', fontsize=11)
axes[0].set_title('Compartmental Model: Action Potential Propagation', fontsize=12)
axes[0].legend(loc='upper right')
axes[0].grid(True, alpha=0.3)

# Heatmap
im = axes[1].imshow(sol_comp.T, aspect='auto', origin='lower',
                      extent=[t_comp[0], t_comp[-1], 0, n_comp],
                      cmap='RdBu_r', vmin=-5, vmax=20)
axes[1].set_xlabel('Time (ms)', fontsize=11)
axes[1].set_ylabel('Compartment', fontsize=11)
axes[1].set_title('Space-Time Map: Voltage Distribution Along Cable', fontsize=12)
plt.colorbar(im, ax=axes[1], label='V (mV)')

plt.tight_layout()
plt.savefig('lab3_section3_compartment.png', dpi=150)
plt.show()
print("  [Saved] lab3_section3_compartment.png")

print("\n✅ Lab 3 Complete!")
print("\n" + "=" * 65)
print("LAB SUMMARY: Week 11 — Physiological Modeling")
print("=" * 65)
print("✅ One-compartment PK: single dose, multiple dosing, steady state")
print("✅ Parameter estimation: curve_fit to PK data")
print("✅ Second-order systems: underdamped, critically damped, overdamped")
print("✅ Cardiovascular model: normal vs hypertensive vs heart failure")
print("✅ Pole locations: stability analysis in s-plane")
print("✅ Hodgkin-Huxley: action potential, ion channel gating")
print("✅ Threshold phenomenon: all-or-none firing")
print("✅ Compartmental model: cable equation, propagation")
```

### Expected Output
- Action potential waveform with labeled phases (depolarization, repolarization, refractory)
- Ion channel gating variables (m, h, n) showing temporal dynamics
- Ion currents (Na+, K+) during action potential
- Threshold study: subthreshold vs suprathreshold responses
- Compartmental propagation

---

## References

- Hodgkin & Huxley (1952). *A Quantitative Description of Membrane Current*. J. Physiol.
- Keener & Sneyd (2009). *Mathematical Physiology*. Springer.
- Winter (2018). *Biomedical Signal Processing*. Academic Press.

**Maintainer**: BME Bootcamp Agent | **Week 11** | **BMED3603: Physiological Modeling**
