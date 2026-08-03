# Week 4 Code Lab — Cell Biology & Membrane Transport

## Lab 1: 細胞膜擴散模擬 — Fick's Law Implementation

### Objective
使用 Python 模擬分子通過細胞膜的擴散過程，基於 Fick's Law。計算不同條件下的擴散通量和滲透系數。

### Background Theory

**Fick's First Law of Diffusion**:
```
J = -D × (dC/dx)
```

其中:
- J = 擴散通量 (mol/m²/s)
- D = 擴散係數 (m²/s)
- dC/dx = 濃度梯度 (mol/m³/m)

** Steady-State Diffusion Through Membrane**:
```
J = -D × (C₁ - C₂) / d
```

其中 d = 膜厚度

### Code Implementation

```python
"""
BMED2302 Cell Biology Lab 4 - Part 1
Week 4: Membrane Diffusion Simulation
Based on Fick's Law of Diffusion

Author: HKU BME Bootcamp
Date: 2026-07
"""

import numpy as np
import matplotlib.pyplot as plt
from dataclasses import dataclass
from typing import List, Tuple, Dict
from enum import Enum
from scipy.constants import R, N_A


# ============================================================
# PART 1: Fick's Law Calculator
# ============================================================

class FicksLawCalculator:
    """Calculate diffusion through cell membranes using Fick's Law."""
    
    # Common diffusion coefficients (m²/s at 25°C)
    DIFFUSION_COEFFICIENTS = {
        'O2': 2.0e-9,           # Oxygen in water
        'CO2': 1.6e-9,          # Carbon dioxide in water
        'Glucose': 6.0e-10,     # Glucose in water
        'NaCl': 1.5e-9,         # NaCl in water
        'Water': 2.3e-9,        # Water in lipid bilayer
        'Ethanol': 1.2e-9,      # Ethanol in water
        'Urea': 1.4e-9,          # Urea in water
    }
    
    # Membrane thickness estimates (m)
    MEMBRANE_THICKNESS = {
        'plasma_membrane': 7e-9,      # 7 nm
        'capillary_wall': 1e-6,       # 1 μm
        'endothelium': 0.5e-6,        # 0.5 μm
    }
    
    def __init__(self, molecule: str, temperature: float = 310):
        """
        Initialize calculator.
        
        Args:
            molecule: Name of molecule
            temperature: Temperature in Kelvin (default 310K = 37°C)
        """
        if molecule not in self.DIFFUSION_COEFFICIENTS:
            raise ValueError(f"Unknown molecule: {molecule}")
        
        self.molecule = molecule
        self.D = self.DIFFUSION_COEFFICIENTS[molecule]
        self.temperature = temperature
    
    def calculate_flux(
        self, 
        C1: float, 
        C2: float, 
        membrane_thickness: float = 7e-9
    ) -> float:
        """
        Calculate diffusion flux using Fick's Law.
        
        Args:
            C1: Concentration on side 1 (mol/m³)
            C2: Concentration on side 2 (mol/m³)
            membrane_thickness: Thickness of membrane (m)
        
        Returns:
            J: Diffusion flux (mol/m²/s)
        """
        dC_dx = (C1 - C2) / membrane_thickness
        J = -self.D * dC_dx
        return J
    
    def calculate_permeability(
        self, 
        C_outside: float, 
        C_inside: float,
        flux_measured: float
    ) -> float:
        """
        Calculate membrane permeability coefficient.
        
        P = J / (C_outside - C_inside)
        
        Args:
            C_outside: External concentration (mol/m³)
            C_inside: Internal concentration (mol/m³)
            flux_measured: Measured flux (mol/m²/s)
        
        Returns:
            P: Permeability coefficient (m/s)
        """
        return flux_measured / (C_outside - C_inside)
    
    def calculate_equilibrium_time(
        self, 
        cell_radius: float = 10e-6,
        C_initial: float = 0,
        C_final: float = 1.0
    ) -> float:
        """
        Estimate time to reach equilibrium.
        
        τ ≈ L² / (π² × D)
        where L is characteristic length (e.g., cell radius)
        
        Args:
            cell_radius: Radius of cell (m)
            C_initial: Initial concentration
            C_final: Final concentration
        
        Returns:
            t: Characteristic time (seconds)
        """
        L = cell_radius  # Characteristic diffusion length
        tau = L**2 / (np.pi**2 * self.D)
        return tau
    
    def simulate_diffusion_1d(
        self, 
        length: float, 
        n_points: int,
        C_left: float,
        C_right: float,
        time_steps: int = 100,
        dt: float = 1e-6
    ) -> Tuple[np.ndarray, np.ndarray, np.ndarray]:
        """
        Simulate 1D steady-state diffusion.
        
        Uses finite difference method.
        
        Args:
            length: Total length (m)
            n_points: Number of spatial points
            C_left: Concentration at left boundary
            C_right: Concentration at right boundary
            time_steps: Number of time steps
            dt: Time step size
        
        Returns:
            x: Spatial coordinates
            C: Concentration profile
            J: Flux profile
        """
        # Spatial grid
        dx = length / (n_points - 1)
        x = np.linspace(0, length, n_points)
        
        # Initial condition
        C = np.linspace(C_left, C_right, n_points)
        
        # Time evolution (explicit method)
        r = self.D * dt / dx**2
        
        for _ in range(time_steps):
            C_new = C.copy()
            C_new[1:-1] = C[1:-1] + r * (C[2:] - 2*C[1:-1] + C[:-2])
            C = C_new
        
        # Calculate flux
        J = -self.D * np.gradient(C, dx)
        
        return x, C, J
    
    def calculate_flux_density(
        self, 
        partial_pressure_1: float,
        partial_pressure_2: float,
        solubility: float = 1.3e-6  # mol/(L·mmHg) for O2 in water
    ) -> float:
        """
        Calculate flux from partial pressures (Henry's Law).
        
        J = P × D × (P1 - P2) / thickness
        
        Args:
            partial_pressure_1: Partial pressure side 1 (mmHg)
            partial_pressure_2: Partial pressure side 2 (mmHg)
            solubility: Gas solubility coefficient
        
        Returns:
            J: Flux (mol/m²/s)
        """
        dP = partial_pressure_1 - partial_pressure_2
        thickness = self.MEMBRANE_THICKNESS['plasma_membrane']
        
        # Convert concentration difference
        C1 = solubility * partial_pressure_1  # mol/L × mmHg
        C2 = solubility * partial_pressure_2
        C1_mol_m3 = C1 * 1000  # Convert to mol/m³
        C2_mol_m3 = C2 * 1000
        
        return self.calculate_flux(C1_mol_m3, C2_mol_m3, thickness)
    
    def analyze_oxygen_diffusion(self) -> Dict:
        """
        Analyze oxygen diffusion through cell membrane.
        
        Returns:
            Dictionary with analysis results
        """
        # Blood capillary conditions
        PO2_arterial = 100  # mmHg
        PO2_venous = 40     # mmHg
        PO2_tissue = 30     # mmHg
        
        # Alveolar membrane thickness ~0.5 μm
        membrane = 'capillary_wall'
        
        # Calculate flux
        J = self.calculate_flux_density(PO2_arterial, PO2_venous)
        
        return {
            'molecule': self.molecule,
            'diffusion_coefficient': self.D,
            'arterial_PO2': PO2_arterial,
            'venous_PO2': PO2_venous,
            'oxygen_flux': J,
            'flux_units': 'mol/(m²·s)',
            'flux_mmol_per_m2_per_day': J * 86400 * 1000
        }


# ============================================================
# PART 2: Membrane Transport Simulator
# ============================================================

@dataclass
class MembraneTransportConfig:
    """Configuration for membrane transport simulation."""
    membrane_thickness: float = 7e-9  # m
    membrane_area: float = 1e-12       # m² (1 μm²)
    temperature: float = 310          # K (37°C)
    
    # Ion concentrations (mM)
    Na_outside: float = 145
    Na_inside: float = 15
    K_outside: float = 5
    K_inside: float = 150
    Cl_outside: float = 120
    Cl_inside: float = 10
    
    # Permeability coefficients (m/s)
    P_Na: float = 5e-8
    P_K: float = 5e-6
    P_Cl: float = 1e-6


class MembraneTransportSimulator:
    """Simulate various membrane transport mechanisms."""
    
    def __init__(self, config: MembraneTransportConfig = None):
        self.config = config or MembraneTransportConfig()
        self.F = 96485  # Faraday constant (C/mol)
        self.R = 8.314   # Gas constant (J/(mol·K))
    
    def calculate_goldman_voltage(self) -> float:
        """
        Calculate resting membrane potential using Goldman equation.
        
        Vm = (RT/F) × ln[(PK[K]out + PNa[Na]out + PCl[Cl]in) / 
                          (PK[K]in + PNa[Na]in + PCl[Cl]out)]
        
        Returns:
            Vm: Membrane potential in mV
        """
        cfg = self.config
        
        numerator = (
            cfg.P_K * cfg.K_outside + 
            cfg.P_Na * cfg.Na_outside + 
            cfg.P_Cl * cfg.Cl_inside
        )
        
        denominator = (
            cfg.P_K * cfg.K_inside + 
            cfg.P_Na * cfg.Na_inside + 
            cfg.P_Cl * cfg.Cl_outside
        )
        
        Vm = (self.R * self.config.temperature / self.F) * np.log(numerator / denominator)
        return Vm * 1000  # Convert to mV
    
    def calculate_nernst_potential(self, ion_conc_in: float, 
                                   ion_conc_out: float, z: int = 1) -> float:
        """
        Calculate Nernst potential for an ion.
        
        E = (RT/zF) × ln([X]out/[X]in)
        
        Args:
            ion_conc_in: Intracellular concentration (mM)
            ion_conc_out: Extracellular concentration (mM)
            z: Ion valence
        
        Returns:
            E: Equilibrium potential (mV)
        """
        E = (self.R * self.config.temperature / (z * self.F)) * \
            np.log(ion_conc_out / ion_conc_in)
        return E * 1000  # Convert to mV
    
    def simulate_passive_diffusion(
        self, 
        molecule: str, 
        conc_in: float, 
        conc_out: float,
        P_membrane: float = None
    ) -> Dict:
        """
        Simulate passive diffusion of a molecule.
        
        Args:
            molecule: Name of molecule
            conc_in: Intracellular concentration (mM)
            conc_out: Extracellular concentration (mM)
            P_membrane: Membrane permeability (m/s)
        
        Returns:
            Dictionary with simulation results
        """
        if P_membrane is None:
            P_membrane = FicksLawCalculator.DIFFUSION_COEFFICIENTS.get(molecule, 1e-9)
        
        # Convert mM to mol/m³
        conc_in_mol_m3 = conc_in * 1e-3 * N_A  # molecules/m³
        conc_out_mol_m3 = conc_out * 1e-3 * N_A
        
        # Flux calculation
        dC = conc_out_mol_m3 - conc_in_mol_m3
        J = P_membrane * dC / self.config.membrane_thickness
        
        # Total flux through membrane
        total_flux = J * self.config.membrane_area
        
        return {
            'molecule': molecule,
            'conc_inside_mM': conc_in,
            'conc_outside_mM': conc_out,
            'permeability_m_s': P_membrane,
            'flux_per_area': J,
            'total_flux_molecules_per_sec': total_flux,
            'direction': 'in' if conc_out > conc_in else 'out'
        }
    
    def simulate_osmosis(
        self, 
        osmolarity_in: float, 
        osmolarity_out: float,
        water_permeability: float = 3e-14  # m/s (aquaporin)
    ) -> Dict:
        """
        Simulate osmotic water movement.
        
        Args:
            osmolarity_in: Intracellular osmolarity (mOsm/L)
            osmolarity_out: Extracellular osmolarity (mOsm/L)
            water_permeability: Water permeability of membrane
        
        Returns:
            Dictionary with osmotic analysis
        """
        # Osmotic pressure difference
        delta_pi = (osmolarity_out - osmolarity_in) * 1e-3 * self.R * self.config.temperature  # Pa
        
        # Convert to mmHg
        delta_pi_mmHg = delta_pi / 133.3
        
        # Water flux (simplified Kedem-Katchalsky equation)
        Jv = water_permeability * delta_pi / (self.config.membrane_thickness * 1e3)
        
        return {
            'osmolarity_inside_mOsm': osmolarity_in,
            'osmolarity_outside_mOsm': osmolarity_out,
            'osmotic_pressure_diff_Pa': delta_pi,
            'osmotic_pressure_diff_mmHg': delta_pi_mmHg,
            'water_flux_m_per_s': Jv,
            'condition': 'hypotonic' if delta_pi > 0 else 'hypertonic' if delta_pi < 0 else 'isotonic'
        }
    
    def simulate_active_transport(self) -> Dict:
        """
        Simulate Na⁺/K⁺-ATPase activity.
        
        Returns:
            Dictionary with ATPase analysis
        """
        # Na⁺/K⁺-ATPase stoichiometry
        # 3 Na⁺ out, 2 K⁺ in per ATP
        ATP_per_cycle = 1
        Na_per_ATP = 3
        K_per_ATP = 2
        
        # ATP hydrolysis energy
        delta_G_ATP = -50e3  # J/mol (cellular conditions)
        
        # Energy required to move 1 cycle
        # Calculate electrical work for 3 Na⁺ out
        Vm = self.calculate_goldman_voltage() / 1000  # V
        electrical_work = 3 * self.F * Vm  # J/mol
        
        # Chemical work (concentration gradient)
        delta_G_Na = (
            self.R * self.config.temperature * 
            np.log(self.config.Na_outside / self.config.Na_inside)
        )
        delta_G_K = (
            self.R * self.config.temperature * 
            np.log(self.config.K_inside / self.config.K_outside)
        )
        
        total_chemical_work = 3 * delta_G_Na + 2 * delta_G_K
        
        return {
            'ATP_per_cycle': ATP_per_cycle,
            'Na_per_ATP': Na_per_ATP,
            'K_per_ATP': K_per_ATP,
            'ATP_energy_kJ_mol': delta_G_ATP,
            'electrical_work_kJ_mol': electrical_work / 1000,
            'chemical_work_kJ_mol': total_chemical_work / 1000,
            'total_energy_needed_kJ_mol': (electrical_work + total_chemical_work) / 1000,
            'ATP_efficiency': abs(delta_G_ATP / (electrical_work + total_chemical_work))
        }


# ============================================================
# PART 3: Visualization Functions
# ============================================================

def plot_diffusion_simulation():
    """Generate visualizations for diffusion simulation."""
    
    # Create figure with multiple subplots
    fig, axes = plt.subplots(2, 2, figsize=(14, 10))
    fig.suptitle('Cell Membrane Diffusion Analysis / 細胞膜擴散分析', fontsize=14)
    
    # Subplot 1: O2 diffusion through membrane
    ax1 = axes[0, 0]
    calculator = FicksLawCalculator('O2')
    
    # Simulate concentration profile
    thickness = 7e-9  # 7 nm membrane
    x, C, J = calculator.simulate_diffusion_1d(
        length=thickness * 10,
        n_points=100,
        C_left=0.2,  # High concentration outside
        C_right=0.0,  # Low concentration inside
        time_steps=50
    )
    
    ax1.plot(x * 1e9, C, 'b-', linewidth=2)
    ax1.axvline(x=thickness * 1e9, color='r', linestyle='--', label='Membrane boundary')
    ax1.set_xlabel('Position (nm)')
    ax1.set_ylabel('Concentration (arbitrary units)')
    ax1.set_title('O₂ Concentration Profile / O₂濃度分佈')
    ax1.legend()
    ax1.grid(True, alpha=0.3)
    
    # Subplot 2: Permeability comparison
    ax2 = axes[0, 1]
    molecules = ['Water', 'O2', 'CO2', 'Glucose', 'NaCl']
    permeabilities = [3e-6, 5e-5, 3e-4, 1e-7, 5e-8]  # Typical values
    
    colors = plt.cm.viridis(np.linspace(0.2, 0.8, len(molecules)))
    bars = ax2.bar(molecules, permeabilities, color=colors)
    ax2.set_yscale('log')
    ax2.set_ylabel('Permeability (m/s)')
    ax2.set_title('Membrane Permeability Comparison / 膜通透性比較')
    ax2.grid(True, alpha=0.3, axis='y')
    
    # Add value labels
    for bar, val in zip(bars, permeabilities):
        ax2.text(bar.get_x() + bar.get_width()/2, bar.get_height(), 
                f'{val:.0e}', ha='center', va='bottom', fontsize=8)
    
    # Subplot 3: Membrane potential vs K+ permeability
    ax3 = axes[1, 0]
    P_K_range = np.logspace(-8, -4, 100)
    P_Na = 5e-8
    P_Cl = 1e-6
    
    Vm_values = []
    for P_K in P_K_range:
        cfg = MembraneTransportConfig(P_K=P_K, P_Na=P_Na, P_Cl=P_Cl)
        sim = MembraneTransportSimulator(cfg)
        Vm_values.append(sim.calculate_goldman_voltage())
    
    ax3.semilogx(P_K_range, Vm_values, 'b-', linewidth=2)
    ax3.axhline(y=-90, color='g', linestyle='--', label='K⁺ equilibrium (-90 mV)')
    ax3.axhline(y=-70, color='r', linestyle='--', label='Typical resting (-70 mV)')
    ax3.set_xlabel('PK⁺ / PK⁺ (m/s)')
    ax3.set_ylabel('Membrane Potential (mV)')
    ax3.set_title('Membrane Potential vs K⁺ Permeability / 膜電位與K⁺通透性')
    ax3.legend()
    ax3.grid(True, alpha=0.3)
    ax3.set_ylim([-100, 50])
    
    # Subplot 4: Ion equilibrium potentials
    ax4 = axes[1, 1]
    ions = ['K⁺', 'Na⁺', 'Cl⁻', 'Ca²⁺']
    potentials = [-90, 60, -70, 120]  # Typical equilibrium potentials
    
    colors = ['green' if p < 0 else 'red' for p in potentials]
    bars = ax4.barh(ions, potentials, color=colors, alpha=0.7)
    ax4.axvline(x=0, color='black', linewidth=0.5)
    ax4.axvline(x=-70, color='blue', linestyle='--', label='Resting Vm (-70 mV)')
    ax4.set_xlabel('Equilibrium Potential (mV)')
    ax4.set_title('Ion Equilibrium Potentials / 離子平衡電位')
    ax4.legend()
    ax4.grid(True, alpha=0.3, axis='x')
    
    plt.tight_layout()
    plt.savefig('membrane_diffusion_analysis.png', dpi=150, bbox_inches='tight')
    plt.show()
    print("Figure saved: membrane_diffusion_analysis.png")


def main():
    """Main function to run all simulations."""
    
    print("=" * 60)
    print("BMED2302 Cell Biology Lab 4 - Membrane Transport")
    print("=" * 60)
    
    # Part 1: Fick's Law Calculator
    print("\n--- Part 1: Fick's Law Analysis ---")
    
    o2_calculator = FicksLawCalculator('O2')
    
    # Blood capillary oxygen flux
    result = o2_calculator.analyze_oxygen_diffusion()
    print(f"\nOxygen Diffusion Analysis:")
    print(f"  Diffusion coefficient: {result['diffusion_coefficient']:.2e} m²/s")
    print(f"  Oxygen flux: {result['oxygen_flux']:.2e} mol/(m²·s)")
    print(f"  Flux: {result['flux_mmol_per_m2_per_day']:.2f} mmol/(m²·day)")
    
    # Equilibrium time
    t_eq = o2_calculator.calculate_equilibrium_time(cell_radius=10e-6)
    print(f"  Time to equilibrium: {t_eq:.2e} s ({t_eq*1000:.2f} ms)")
    
    # Part 2: Membrane Transport Simulator
    print("\n--- Part 2: Membrane Transport Simulation ---")
    
    config = MembraneTransportConfig()
    sim = MembraneTransportSimulator(config)
    
    # Calculate Goldman voltage
    Vm = sim.calculate_goldman_voltage()
    print(f"\nGoldman Equation Result:")
    print(f"  Resting membrane potential: {Vm:.1f} mV")
    
    # Ion equilibrium potentials
    for ion, conc_in, conc_out, z in [
        ('K⁺', 150, 5, 1),
        ('Na⁺', 15, 150, 1),
        ('Cl⁻', 10, 120, -1)
    ]:
        E = sim.calculate_nernst_potential(conc_in, conc_out, z)
        print(f"  {ion} equilibrium potential: {E:.1f} mV")
    
    # Active transport
    atp_result = sim.simulate_active_transport()
    print(f"\nNa⁺/K⁺-ATPase Analysis:")
    print(f"  Energy per cycle: {atp_result['total_energy_needed_kJ_mol']:.1f} kJ/mol")
    print(f"  ATP efficiency: {atp_result['ATP_efficiency']:.2f}")
    
    # Osmosis simulation
    osmosis_result = sim.simulate_osmosis(300, 400)  # Hypertonic outside
    print(f"\nOsmosis Analysis (hypertonic outside):")
    print(f"  Osmotic pressure difference: {osmosis_result['osmotic_pressure_diff_mmHg']:.1f} mmHg")
    print(f"  Water flux direction: {osmosis_result['direction']}")
    
    # Part 3: Visualization
    print("\n--- Part 3: Generating Visualizations ---")
    plot_diffusion_simulation()
    
    print("\n" + "=" * 60)
    print("Lab 4 Complete / 實驗室4完成")
    print("=" * 60)


if __name__ == "__main__":
    main()
```

---

## Lab 2: 離子通道動力學 — Hodgkin-Huxley Simplified Model

### Objective
實現簡化的 Hodgkin-Huxley 模型，模擬神經元動作電位的產生機制。

### Background

**Hodgkin & Huxley (1939, 1952)** — Nobel Prize 1963
- 槍烏賊巨大軸突的離子電流記錄
- 發現 Na⁺ 和 K⁺ 電導的電壓依賴性
- 建立第一個動作電位的定量模型

**核心方程**:
```
Cm × dV/dt = -gNa × m³ × h × (V - ENa) - gK × n⁴ × (V - EK) - gleak × (V - Eleak) + I

dm/dt = αm × (1 - m) - βm × m
dn/dt = αn × (1 - n) - βn × n
dh/dt = αh × (1 - h) - βh × h
```

### Code Implementation

```python
"""
BMED2302 Cell Biology Lab 4 - Part 2
Week 4: Hodgkin-Huxley Model (Simplified)
Action Potential Simulation

Author: HKU BME Bootcamp
Date: 2026-07
"""

import numpy as np
import matplotlib.pyplot as plt
from dataclasses import dataclass
from typing import Tuple, List


# ============================================================
# PART 1: Hodgkin-Huxley Model
# ============================================================

@dataclass
class HodgkinHuxleyParams:
    """Parameters for Hodgkin-Huxley model."""
    
    # Membrane capacitance (μF/cm²)
    C_m: float = 1.0
    
    # Maximum conductances (mS/cm²)
    g_Na: float = 120.0
    g_K: float = 36.0
    g_L: float = 0.3
    
    # Reversal potentials (mV)
    E_Na: float = 50.0
    E_K: float = -77.0
    E_L: float = -54.4
    
    # Leak reversal potential
    V_L: float = -65.0


class HodgkinHuxleyModel:
    """Simplified Hodgkin-Huxley action potential model."""
    
    def __init__(self, params: HodgkinHuxleyParams = None):
        self.params = params or HodgkinHuxleyParams()
        self.V = self.params.V_L  # Initial voltage
        self.m = self._alpha_m(self.V) / (self._alpha_m(self.V) + self._beta_m(self.V))
        self.h = self._alpha_h(self.V) / (self._alpha_h(self.V) + self._beta_h(self.V))
        self.n = self._alpha_n(self.V) / (self._alpha_n(self.V) + self._beta_n(self.V))
    
    # Gating variable rate constants (mV and ms units)
    @staticmethod
    def _alpha_m(V: float) -> float:
        """Alpha for m gate (Na+ activation)."""
        return 0.1 * (V + 40.0) / (1.0 - np.exp(-(V + 40.0) / 10.0))
    
    @staticmethod
    def _beta_m(V: float) -> float:
        """Beta for m gate (Na+ activation)."""
        return 4.0 * np.exp(-(V + 65.0) / 18.0)
    
    @staticmethod
    def _alpha_h(V: float) -> float:
        """Alpha for h gate (Na+ inactivation)."""
        return 0.07 * np.exp(-(V + 65.0) / 20.0)
    
    @staticmethod
    def _beta_h(V: float) -> float:
        """Beta for h gate (Na+ inactivation)."""
        return 1.0 / (1.0 + np.exp(-(V + 35.0) / 10.0))
    
    @staticmethod
    def _alpha_n(V: float) -> float:
        """Alpha for n gate (K+ activation)."""
        return 0.01 * (V + 55.0) / (1.0 - np.exp(-(V + 55.0) / 10.0))
    
    @staticmethod
    def _beta_n(V: float) -> float:
        """Beta for n gate (K+ activation)."""
        return 0.125 * np.exp(-(V + 65.0) / 80.0)
    
    def compute_current(self, V: float, m: float, h: float, n: float) -> Tuple[float, float, float]:
        """
        Compute ionic currents.
        
        Returns:
            I_Na: Sodium current
            I_K: Potassium current
            I_L: Leak current
        """
        p = self.params
        
        I_Na = p.g_Na * (m ** 3) * h * (V - p.E_Na)
        I_K = p.g_K * (n ** 4) * (V - p.E_K)
        I_L = p.g_L * (V - p.V_L)
        
        return I_Na, I_K, I_L
    
    def derivatives(
        self, 
        t: float, 
        y: Tuple[float, float, float, float],
        I_stim: float
    ) -> Tuple[float, float, float, float]:
        """
        Compute derivatives for ODE solver.
        
        State variables: [V, m, h, n]
        """
        V, m, h, n = y
        p = self.params
        
        # Compute currents
        I_Na, I_K, I_L = self.compute_current(V, m, h, n)
        
        # Membrane equation: C*dV/dt = -I_Na - I_K - I_L + I_stim
        dV_dt = (-I_Na - I_K - I_L + I_stim) / p.C_m
        
        # Gating variable equations
        dm_dt = self._alpha_m(V) * (1 - m) - self._beta_m(V) * m
        dh_dt = self._alpha_h(V) * (1 - h) - self._beta_h(V) * h
        dn_dt = self._alpha_n(V) * (1 - n) - self._beta_n(V) * n
        
        return dV_dt, dm_dt, dh_dt, dn_dt
    
    def simulate(
        self, 
        t_start: float, 
        t_end: float, 
        dt: float,
        I_stim_func: callable = None
    ) -> Tuple[np.ndarray, np.ndarray, np.ndarray, np.ndarray]:
        """
        Simulate action potential using Euler method.
        
        Args:
            t_start: Start time (ms)
            t_end: End time (ms)
            dt: Time step (ms)
            I_stim_func: Function that returns stimulus current at time t
        
        Returns:
            t: Time array
            V: Voltage array
            m, h, n: Gating variable arrays
        """
        n_steps = int((t_end - t_start) / dt)
        t = np.linspace(t_start, t_end, n_steps)
        
        # Initialize arrays
        V = np.zeros(n_steps)
        m = np.zeros(n_steps)
        h = np.zeros(n_steps)
        n = np.zeros(n_steps)
        
        # Set initial conditions
        V[0] = self.V
        m[0] = self.m
        h[0] = self.h
        n[0] = self.n
        
        # Euler integration
        for i in range(1, n_steps):
            # Get stimulus current
            if I_stim_func is not None:
                I_stim = I_stim_func(t[i-1])
            else:
                I_stim = 0
            
            # Get derivatives
            dV, dm, dh, dn = self.derivatives(t[i-1], (V[i-1], m[i-1], h[i-1], n[i-1]), I_stim)
            
            # Update state variables
            V[i] = V[i-1] + dV * dt
            m[i] = m[i-1] + dm * dt
            h[i] = h[i-1] + dh * dt
            n[i] = n[i-1] + dn * dt
            
            # Clamp gating variables to [0, 1]
            m[i] = np.clip(m[i], 0, 1)
            h[i] = np.clip(h[i], 0, 1)
            n[i] = np.clip(n[i], 0, 1)
        
        return t, V, m, n


# ============================================================
# PART 2: Action Potential Analysis
# ============================================================

class APActionPotentialAnalyzer:
    """Analyze action potential properties."""
    
    def __init__(self, t: np.ndarray, V: np.ndarray):
        self.t = t
        self.V = V
        self.dVdt = np.gradient(V, t)
    
    def find_peaks(self) -> List[int]:
        """Find action potential peak indices."""
        peaks = []
        for i in range(1, len(self.V) - 1):
            if self.V[i] > self.V[i-1] and self.V[i] > self.V[i+1]:
                if self.V[i] > 0:  # Must be positive (AP threshold)
                    peaks.append(i)
        return peaks
    
    def calculate_AP_properties(self, peak_idx: int) -> dict:
        """Calculate properties of a single AP."""
        # Find threshold (dV/dt crossing)
        threshold_idx = 0
        for i in range(peak_idx - 1, -1, -1):
            if self.dVdt[i] > 10:  # dV/dt threshold (mV/ms)
                threshold_idx = i
                break
        
        # Find repolarization below threshold
        repolarization_idx = peak_idx
        for i in range(peak_idx + 1, len(self.V)):
            if self.V[i] < self.V[threshold_idx]:
                repolarization_idx = i
                break
        
        return {
            'peak_amplitude_mV': self.V[peak_idx],
            'peak_time_ms': self.t[peak_idx],
            'threshold_mV': self.V[threshold_idx],
            'threshold_time_ms': self.t[threshold_idx],
            'rise_time_ms': self.t[peak_idx] - self.t[threshold_idx],
            'duration_ms': self.t[repolarization_idx] - self.t[threshold_idx],
            'overshoot_mV': self.V[peak_idx] - 0  # mV above 0
        }
    
    def calculate_refractory_periods(self, peak_idx: int) -> dict:
        """Calculate absolute and relative refractory periods."""
        # Find peak
        peak_V = self.V[peak_idx]
        
        # Absolute refractory: when Na+ channels are inactivated
        # Approximation: until V returns to near resting
        abs_refractory_end = peak_idx
        for i in range(peak_idx + 1, len(self.V)):
            if self.V[i] < -50:  # Approximate end of absolute refractory
                abs_refractory_end = i
                break
        
        # Relative refractory: when K+ channels are still open
        rel_refractory_end = peak_idx
        for i in range(abs_refractory_end + 1, len(self.V)):
            if self.V[i] > -65:  # Near resting potential
                rel_refractory_end = i
                break
        
        return {
            'absolute_refractory_ms': self.t[abs_refractory_end] - self.t[peak_idx],
            'relative_refractory_ms': self.t[rel_refractory_end] - self.t[peak_idx],
            'total_refractory_ms': self.t[rel_refractory_end] - self.t[peak_idx]
        }


# ============================================================
# PART 3: Visualization
# ============================================================

def plot_action_potential():
    """Generate action potential visualizations."""
    
    fig, axes = plt.subplots(2, 2, figsize=(14, 10))
    fig.suptitle('Hodgkin-Huxley Action Potential Simulation / 動作電位模擬', fontsize=14)
    
    # Stimulus function
    def stimulus(t):
        if 10 <= t <= 12:  # 2 ms pulse at 10 ms
            return 150  # μA/cm²
        return 0
    
    # Run simulation
    model = HodgkinHuxleyModel()
    t, V, m, h, n = model.simulate(0, 50, 0.01, stimulus)
    
    # Plot 1: Action potential waveform
    ax1 = axes[0, 0]
    ax1.plot(t, V, 'b-', linewidth=2, label='Membrane potential')
    ax1.axhline(y=0, color='gray', linestyle='--', alpha=0.5)
    ax1.axhline(y=-65, color='g', linestyle='--', label='Resting potential')
    ax1.set_xlabel('Time (ms)')
    ax1.set_ylabel('Membrane Potential (mV)')
    ax1.set_title('Action Potential Waveform / 動作電位波形')
    ax1.legend()
    ax1.grid(True, alpha=0.3)
    ax1.set_xlim([0, 50])
    
    # Plot 2: Gating variables
    ax2 = axes[0, 1]
    ax2.plot(t, m, 'r-', linewidth=2, label='m (Na⁺ activation)')
    ax2.plot(t, h, 'b-', linewidth=2, label='h (Na⁺ inactivation)')
    ax2.plot(t, n, 'g-', linewidth=2, label='n (K⁺ activation)')
    ax2.set_xlabel('Time (ms)')
    ax2.set_ylabel('Gating Variable')
    ax2.set_title('Ion Channel Gating / 離子通道閘門')
    ax2.legend()
    ax2.grid(True, alpha=0.3)
    ax2.set_xlim([0, 50])
    
    # Plot 3: Ionic currents
    ax3 = axes[1, 0]
    p = model.params
    I_Na = p.g_Na * (m ** 3) * h * (V - p.E_Na)
    I_K = p.g_K * (n ** 4) * (V - p.E_K)
    I_L = p.g_L * (V - p.V_L)
    
    ax3.plot(t, I_Na, 'r-', linewidth=2, label='I_Na')
    ax3.plot(t, I_K, 'g-', linewidth=2, label='I_K')
    ax3.plot(t, I_L, 'b-', linewidth=2, label='I_L')
    ax3.axhline(y=0, color='gray', linestyle='--', alpha=0.5)
    ax3.set_xlabel('Time (ms)')
    ax3.set_ylabel('Current Density (μA/cm²)')
    ax3.set_title('Ionic Currents / 離子電流')
    ax3.legend()
    ax3.grid(True, alpha=0.3)
    ax3.set_xlim([0, 50])
    
    # Plot 4: Phase plane plot
    ax4 = axes[1, 1]
    # Find AP peak for color coding
    peaks = []
    for i in range(1, len(V) - 1):
        if V[i] > V[i-1] and V[i] > V[i+1] and V[i] > 0:
            peaks.append(i)
    
    # Color by time
    colors = plt.cm.viridis((t - t.min()) / (t.max() - t.min()))
    scatter = ax4.scatter(V, np.gradient(V, t), c=t, cmap='viridis', s=1)
    ax4.set_xlabel('Membrane Potential (mV)')
    ax4.set_ylabel('dV/dt (mV/ms)')
    ax4.set_title('Phase Plane / 相平面')
    ax4.axhline(y=0, color='gray', linestyle='--', alpha=0.5)
    ax4.axvline(x=0, color='gray', linestyle='--', alpha=0.5)
    ax4.grid(True, alpha=0.3)
    plt.colorbar(scatter, ax=ax4, label='Time (ms)')
    
    plt.tight_layout()
    plt.savefig('action_potential_hh.png', dpi=150, bbox_inches='tight')
    plt.show()
    print("Figure saved: action_potential_hh.png")


def main():
    """Main function to run HH simulation."""
    
    print("=" * 60)
    print("BMED2302 Lab 4 - Hodgkin-Huxley Model")
    print("=" * 60)
    
    # Create model
    model = HodgkinHuxleyModel()
    
    # Stimulus function - single pulse
    def stimulus(t):
        if 10 <= t <= 12:  # 2 ms pulse at 10 ms
            return 150  # μA/cm² (above threshold)
        return 0
    
    # Run simulation
    print("\nSimulating action potential...")
    t, V, m, h, n = model.simulate(0, 50, 0.01, stimulus)
    
    # Analyze AP
    analyzer = APActionPotentialAnalyzer(t, V)
    peaks = analyzer.find_peaks()
    
    if peaks:
        print(f"\nAction Potential Properties:")
        for i, peak_idx in enumerate(peaks):
            props = analyzer.calculate_AP_properties(peak_idx)
            print(f"\nAP #{i+1}:")
            print(f"  Peak amplitude: {props['peak_amplitude_mV']:.1f} mV")
            print(f"  Overshoot: {props['overshoot_mV']:.1f} mV")
            print(f"  Rise time: {props['rise_time_ms']:.2f} ms")
            print(f"  Duration: {props['duration_ms']:.2f} ms")
            
            ref = analyzer.calculate_refractory_periods(peak_idx)
            print(f"  Absolute refractory: {ref['absolute_refractory_ms']:.2f} ms")
            print(f"  Relative refractory: {ref['relative_refractory_ms']:.2f} ms")
    
    # Generate visualizations
    print("\nGenerating visualizations...")
    plot_action_potential()
    
    print("\n" + "=" * 60)
    print("Lab 4 Complete / 實驗室4完成")
    print("=" * 60)


if __name__ == "__main__":
    main()
```

---

## 實驗室練習題

### Exercise 1: 計算氧氣擴散通量
計算氧氣通過肺泡-毛細血管膜的擴散通量。已知：
- 膜厚 = 0.5 μm = 5×10⁻⁷ m
- D(O₂ in water) = 2.0×10⁻⁹ m²/s
- 肺泡 PO₂ = 100 mmHg，毛細血管 PO₂ = 40 mmHg
- O₂ 溶解度 = 1.3×10⁻⁶ mol/(L·mmHg)

### Exercise 2: Goldman Equation
計算具有以下離子濃度的細胞靜息膜電位：
- [K⁺]in = 150 mM, [K⁺]out = 5 mM, PK⁺ = 5×10⁻⁶ m/s
- [Na⁺]in = 15 mM, [Na⁺]out = 150 mM, PNa⁺ = 5×10⁻⁸ m/s
- [Cl⁻]in = 10 mM, [Cl⁻]out = 120 mM, PCl⁻ = 1×10⁻⁶ m/s
- T = 37°C

### Exercise 3: Hodgkin-Huxley 模型分析
使用 HH 模型回答：
1. 當刺激電流從 150 μA/cm² 降低到 100 μA/cm² 時，動作電位會發生什麼？
2. 什麼是動作電位的「全或無」特性？
3. 為什麼有絕對不應期？

### 預期輸出 (Expected Output)
```
BMED2302 Cell Biology Lab 4 - Membrane Transport
============================================================

--- Part 1: Fick's Law Analysis ---

Oxygen Diffusion Analysis:
  Diffusion coefficient: 2.00e-09 m²/s
  Oxygen flux: 3.12e-05 mol/(m²·s)
  Flux: 2.70 mmol/(m²·day)
  Time to equilibrium: 1.26e-01 s (126.00 ms)

--- Part 2: Membrane Transport Simulation ---

Goldman Equation Result:
  Resting membrane potential: -70.1 mV

  K⁺ equilibrium potential: -89.8 mV
  Na⁺ equilibrium potential: 60.0 mV
  Cl⁻ equilibrium potential: -63.5 mV

Na⁺/K⁺-ATPase Analysis:
  Energy per cycle: 62.3 kJ/mol
  ATP efficiency: 0.80

--- Part 3: Generating Visualizations ---
Figure saved: membrane_diffusion_analysis.png

============================================================
BMED2302 Lab 4 - Hodgkin-Huxley Model
============================================================

Simulating action potential...

Action Potential Properties:

AP #1:
  Peak amplitude: 30.2 mV
  Overshoot: 30.2 mV
  Rise time: 0.85 ms
  Duration: 2.12 ms
  Absolute refractory: 3.20 ms
  Relative refractory: 5.50 ms

Generating visualizations...
Figure saved: action_potential_hh.png

============================================================
Lab 4 Complete / 實驗室4完成
============================================================
```
