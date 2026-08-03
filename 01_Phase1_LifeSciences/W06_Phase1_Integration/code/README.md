# Week 6 Code Lab — Phase 1 Integration

## Overview
本周的程式實驗旨在整合化學 → 生物化學 → 細胞生物學 → 生理學的計算模型，展示從分子到系統的完整模擬。

## Lab 1: Metabolic Pathway Integration Simulation

### Objective
建立一個整合的代謝模型，模擬從葡萄糖攝取到ATP產生的完整能量轉換過程。

### Code Implementation

```python
"""
Phase 1 Integration Lab - Week 6
Metabolic Pathway Integration Simulation

This code integrates concepts from:
- Week 1: Chemistry (pH, energy)
- Week 2: Biomolecules (glucose, ATP)
- Week 3: Bioenergetics (glycolysis, TCA, ETC)
- Week 4: Cell Biology (membrane transport)
- Week 5: Homeostasis (feedback control)

Author: HKU BME Bootcamp
Date: 2026-07
"""

import numpy as np
import matplotlib.pyplot as plt
from dataclasses import dataclass
from typing import List, Dict, Tuple
from enum import Enum


# ============================================================
# PART 1: Glucose Metabolism Model
# ============================================================

class MetabolicState(Enum):
    """Metabolic states based on oxygen availability."""
    AEROBIC = "aerobic"
    ANAEROBIC = "anaerobic"
    MIXED = "mixed"


@dataclass
class GlucoseMetabolismParams:
    """Parameters for glucose metabolism simulation."""
    # Glycolysis
    hexokinase_km: float = 0.1  # mM
    PFK_kcat: float = 100  # s^-1
    
    # Mitochondrial
    PDH_rate: float = 1.0  # Relative rate
    TCA_cycle_rate: float = 1.0  # Relative rate
    
    # ATP yields
    NADH_ATP: float = 2.5  # Modern P/O ratio
    FADH2_ATP: float = 1.5
    glycolysis_ATP: float = 2  # Net ATP
    TCA_ATP: float = 12  # Per acetyl-CoA
    
    # Oxygen
    oxygen_available: bool = True
    O2_consumption_rate: float = 1.0  # Relative


class GlucoseMetabolismSimulator:
    """Simulate glucose metabolism from uptake to ATP production."""
    
    def __init__(self, params: GlucoseMetabolismParams = None):
        self.params = params or GlucoseMetabolismParams()
        self.time = 0.0
        
        # Metabolite concentrations (mM)
        self.glucose = 5.0  # Blood glucose
        self.glucose_6_phosphate = 0.2
        self.pyruvate = 0.1
        self.lactate = 1.0
        self.ATP = 5.0  # mM
        self.ADP = 0.5
        self.NADH = 0.1
        self.NAD = 1.0
        
        # Track ATP production
        self.ATP_glycolysis = 0
        self.ATP_TCA = 0
        self.ATP_total = 0
    
    def hexokinase_step(self, dt: float) -> float:
        """Step 1: Glucose phosphorylation."""
        Vmax = 100  # μmol/min/g tissue (typical)
        glucose = self.glucose
        Km = self.params.hexokinase_km
        
        # Michaelis-Menten kinetics
        v = Vmax * glucose / (Km + glucose)
        
        # Update concentrations
        dG6P = v * dt
        self.glucose -= dG6P
        self.glucose_6_phosphate += dG6P
        
        # Consume ATP
        ATP_consumed = dG6P
        self.ATP -= ATP_consumed
        self.ADP += ATP_consumed
        
        return ATP_consumed
    
    def glycolysis_step(self, dt: float, aerobic: bool) -> Dict[str, float]:
        """Simulate glycolysis."""
        # G6P → FBP (glycolysis flux)
        flux = 10.0 * dt  # Arbitrary units
        
        # Products per 2 pyruvate
        pyruvate_produced = flux
        NADH_produced = flux
        net_ATP = self.params.glycolysis_ATP
        
        # Fate of NADH depends on oxygen
        if aerobic:
            # NADH goes to mitochondria
            self.NADH += NADH_produced
            self.NAD -= NADH_produced
        else:
            # NADH recycled via lactate dehydrogenase
            lactate_produced = NADH_produced
            self.lactate += lactate_produced
            self.NAD += NADH_produced  # Regenerate NAD
            self.NADH -= NADH_produced
        
        # Update pyruvate
        self.pyruvate += pyruvate_produced
        
        # Net ATP from glycolysis
        self.ATP += net_ATP
        self.ADP -= net_ATP
        self.ATP_glycolysis += net_ATP
        
        return {
            'pyruvate': pyruvate_produced,
            'ATP': net_ATP,
            'NADH': NADH_produced if aerobic else 0,
            'lactate': NADH_produced if not aerobic else 0
        }
    
    def TCA_cycle_step(self, dt: float) -> float:
        """Simulate TCA cycle (per acetyl-CoA)."""
        if not self.params.oxygen_available:
            return 0.0
        
        # 1 acetyl-CoA → 3 NADH + 1 FADH2 + 1 GTP
        acetyl_CoA_flux = self.pyruvate  # Simplified
        
        NADH_produced = 3 * acetyl_CoA_flux
        FADH2_produced = 1 * acetyl_CoA_flux
        GTP_produced = 1 * acetyl_CoA_flux
        
        # Update NADH/FADH2
        self.NADH += NADH_produced
        self.NAD -= NADH_produced
        
        # ATP from TCA
        TCA_ATP = self.params.TCA_ATP * acetyl_CoA_flux
        self.ATP += TCA_ATP
        self.ADP -= TCA_ATP
        self.ATP_TCA += TCA_ATP
        
        # Clear pyruvate
        self.pyruvate -= acetyl_CoA_flux
        
        return TCA_ATP
    
    def oxidative_phosphorylation(self, dt: float) -> float:
        """Simulate electron transport chain and ATP synthesis."""
        if not self.params.oxygen_available:
            return 0.0
        
        # NADH → 2.5 ATP, FADH2 → 1.5 ATP
        NADH_consumed = min(self.NADH, self.params.O2_consumption_rate * dt)
        ATP_from_NADH = NADH_consumed * self.params.NADH_ATP
        
        # Update NADH/NAD
        self.NADH -= NADH_consumed
        self.NAD += NADH_consumed
        
        # Total ATP
        total_ATP = ATP_from_NADH
        self.ATP += total_ATP
        self.ADP -= total_ATP
        
        return total_ATP
    
    def simulate(
        self,
        duration: float,
        dt: float,
        aerobic: bool = True,
        glucose_input: float = 0.0
    ) -> Dict[str, np.ndarray]:
        """Simulate metabolism over time."""
        n_steps = int(duration / dt)
        t = np.linspace(0, duration, n_steps)
        
        # Initialize arrays
        results = {
            'glucose': np.zeros(n_steps),
            'ATP': np.zeros(n_steps),
            'pyruvate': np.zeros(n_steps),
            'lactate': np.zeros(n_steps),
            'ATP_glycolysis': np.zeros(n_steps),
            'ATP_mito': np.zeros(n_steps),
        }
        
        for i in range(n_steps):
            # Add glucose if provided
            if glucose_input > 0:
                self.glucose += glucose_input * dt
            
            # Simulate steps
            self.hexokinase_step(dt)
            g_result = self.glycolysis_step(dt, aerobic)
            TCA_ATP = self.TCA_cycle_step(dt) if aerobic else 0
            OXPHOS_ATP = self.oxidative_phosphorylation(dt)
            
            # Store results
            results['glucose'][i] = self.glucose
            results['ATP'][i] = self.ATP
            results['pyruvate'][i] = self.pyruvate
            results['lactate'][i] = self.lactate
            results['ATP_glycolysis'][i] = self.ATP_glycolysis
            results['ATP_mito'][i] = self.ATP_TCA + OXPHOS_ATP
        
        results['time'] = t
        return results


# ============================================================
# PART 2: Blood Glucose Homeostasis Model
# ============================================================

class BloodGlucoseHomeostasis:
    """Simulate blood glucose regulation with insulin and glucagon."""
    
    def __init__(
        self,
        initial_glucose: float = 5.0,  # mM (90 mg/dL)
        setpoint: float = 5.0,
    ):
        self.glucose = initial_glucose
        self.setpoint = setpoint
        self.insulin = 50.0  # pM
        self.glucagon = 50.0  # pM
        self.insulin_sensitivity = 1.0
        self.time = 0.0
        
        # Glucose distribution
        self.plasma_volume = 3.0  # L
        self.glucose_mass = self.glucose * self.plasma_volume * 180  # mg
        
        # Physiological parameters
        self.basal_insulin = 50.0  # pM
        self.basal_glucagon = 50.0  # pM
        
        # Metabolic rates
        self.glucose_uptake_basal = 10.0  # mg/min (basal)
        self.glucose_production_basal = 10.0  # mg/min (liver)
        
        # Meal parameters
        self.current_meal = 0.0
        self.absorption_rate = 0.0
    
    def update_insulin(self, dt: float):
        """Update insulin based on glucose level."""
        # β-cell response to glucose (simplified)
        if self.glucose > self.setpoint:
            # Increase insulin
            error = self.glucose - self.setpoint
            self.insulin += error * 10 * dt  # pM increase
        else:
            # Decrease insulin
            error = self.setpoint - self.glucose
            self.insulin -= error * 5 * dt
        
        self.insulin = max(10.0, min(500.0, self.insulin))
    
    def update_glucagon(self, dt: float):
        """Update glucagon based on glucose level."""
        if self.glucose < self.setpoint:
            error = self.setpoint - self.glucose
            self.glucagon += error * 10 * dt
        else:
            error = self.glucose - self.setpoint
            self.glucagon -= error * 5 * dt
        
        self.glucagon = max(10.0, min(200.0, self.glucagon))
    
    def glucose_uptake(self, dt: float) -> float:
        """Calculate glucose uptake by tissues."""
        # Insulin-dependent uptake (muscle, fat)
        insulin_effect = (self.insulin / 100.0) * self.insulin_sensitivity
        
        # Basal uptake (brain, RBCs)
        basal_uptake = self.glucose_uptake_basal
        
        # Insulin-stimulated uptake
        stimulated_uptake = insulin_effect * 30.0
        
        total_uptake = basal_uptake + stimulated_uptake
        
        # Update glucose
        glucose_consumed = total_uptake * dt
        self.glucose_mass -= glucose_consumed
        self.glucose = self.glucose_mass / (self.plasma_volume * 180)
        
        return glucose_consumed
    
    def glucose_production(self, dt: float) -> float:
        """Calculate hepatic glucose production."""
        # Glucagon effect
        glucagon_effect = self.glucagon / 50.0
        
        # Insulin inhibition
        insulin_inhibition = 1.0 - (self.insulin / 200.0) * 0.5
        
        # Basal production
        basal_production = self.glucose_production_basal
        
        # Glucagon stimulates production
        stimulated_production = glucagon_effect * 20.0
        
        total_production = (basal_production + stimulated_production) * insulin_inhibition
        
        # Update glucose
        glucose_added = total_production * dt
        self.glucose_mass += glucose_added
        self.glucose = self.glucose_mass / (self.plasma_volume * 180)
        
        return glucose_added
    
    def add_meal(self, carbs_grams: float):
        """Simulate a meal by adding glucose to plasma."""
        # Assume 50% of carbs enter bloodstream as glucose
        glucose_from_meal = carbs_grams * 0.5 * 1000  # mg
        self.glucose_mass += glucose_from_meal
        self.glucose = self.glucose_mass / (self.plasma_volume * 180)
        self.current_meal = carbs_grams
    
    def simulate(
        self,
        duration: float,
        dt: float,
        meals: List[Tuple[float, float]] = None  # (time, carbs in grams)
    ) -> Dict[str, np.ndarray]:
        """
        Simulate blood glucose over time.
        
        Args:
            duration: Total simulation time (minutes)
            dt: Time step (minutes)
            meals: List of (time, carbs) tuples
        """
        n_steps = int(duration / dt)
        t = np.linspace(0, duration, n_steps)
        
        results = {
            'glucose': np.zeros(n_steps),
            'insulin': np.zeros(n_steps),
            'glucagon': np.zeros(n_steps),
            'glucose_uptake': np.zeros(n_steps),
            'glucose_production': np.zeros(n_steps),
        }
        
        # Apply meals at specified times
        meals_dict = {m[0]: m[1] for m in (meals or [])}
        
        for i, ti in enumerate(t):
            # Check for meal
            if ti in meals_dict:
                self.add_meal(meals_dict[ti])
            
            # Update hormones
            self.update_insulin(dt)
            self.update_glucagon(dt)
            
            # Update glucose
            uptake = self.glucose_uptake(dt)
            production = self.glucose_production(dt)
            
            # Store results
            results['glucose'][i] = self.glucose
            results['insulin'][i] = self.insulin
            results['glucagon'][i] = self.glucagon
            results['glucose_uptake'][i] = uptake / dt
            results['glucose_production'][i] = production / dt
        
        results['time'] = t
        return results


# ============================================================
# PART 3: Visualization
# ============================================================

def plot_metabolic_integration():
    """Generate comprehensive metabolic integration plots."""
    
    fig, axes = plt.subplots(2, 2, figsize=(14, 10))
    fig.suptitle('Phase 1 Metabolic Integration / 第一階段代謝整合', fontsize=14)
    
    # Case 1: Aerobic metabolism
    model_aero = GlucoseMetabolismSimulator(
        GlucoseMetabolismParams(oxygen_available=True)
    )
    results_aero = model_aero.simulate(
        duration=60, dt=1, aerobic=True, glucose_input=1.0
    )
    
    ax1 = axes[0, 0]
    ax1.plot(results_aero['time'], results_aero['ATP'], 'b-', linewidth=2, 
             label='Total ATP')
    ax1.plot(results_aero['time'], results_aero['ATP_glycolysis'], 'r--', 
             linewidth=1.5, label='Glycolysis ATP')
    ax1.plot(results_aero['time'], results_aero['ATP_mito'], 'g--', 
             linewidth=1.5, label='Mitochondrial ATP')
    ax1.set_xlabel('Time (min)')
    ax1.set_ylabel('ATP (mM)')
    ax1.set_title('Aerobic Metabolism / 有氧代謝')
    ax1.legend()
    ax1.grid(True, alpha=0.3)
    
    # Case 2: Anaerobic metabolism
    model_anaero = GlucoseMetabolismSimulator(
        GlucoseMetabolismParams(oxygen_available=False)
    )
    results_anaero = model_anaero.simulate(
        duration=60, dt=1, aerobic=False, glucose_input=1.0
    )
    
    ax2 = axes[0, 1]
    ax2.plot(results_anaero['time'], results_anaero['ATP'], 'b-', 
             linewidth=2, label='Total ATP')
    ax2.plot(results_anaero['time'], results_anaero['lactate'], 'r-', 
             linewidth=2, label='Lactate')
    ax2.set_xlabel('Time (min)')
    ax2.set_ylabel('Concentration (mM)')
    ax2.set_title('Anaerobic Metabolism / 無氧代謝')
    ax2.legend()
    ax2.grid(True, alpha=0.3)
    
    # Case 3: Blood glucose homeostasis
    bg_model = BloodGlucoseHomeostasis(initial_glucose=5.0, setpoint=5.0)
    meals = [(15, 50), (60, 75), (120, 60)]  # (time, carbs)
    bg_results = bg_model.simulate(duration=180, dt=1, meals=meals)
    
    ax3 = axes[1, 0]
    ax3.plot(bg_results['time'], bg_results['glucose'], 'b-', linewidth=2, 
             label='Blood Glucose')
    ax3.axhline(y=5.0, color='g', linestyle='--', label='Setpoint')
    ax3.axhline(y=7.8, color='r', linestyle=':', alpha=0.7, 
                label='Diabetic threshold')
    
    # Mark meals
    for m_time, m_carbs in meals:
        ax3.axvline(x=m_time, color='orange', linestyle=':', alpha=0.5)
    
    ax3.set_xlabel('Time (min)')
    ax3.set_ylabel('Glucose (mM)')
    ax3.set_title('Blood Glucose Homeostasis / 血糖穩態')
    ax3.legend()
    ax3.grid(True, alpha=0.3)
    ax3.set_ylim([0, 15])
    
    # Case 4: Insulin/Glucagon response
    ax4 = axes[1, 1]
    ax4.plot(bg_results['time'], bg_results['insulin'], 'b-', linewidth=2, 
             label='Insulin')
    ax4.plot(bg_results['time'], bg_results['glucagon'], 'r-', linewidth=2, 
             label='Glucagon')
    ax4.set_xlabel('Time (min)')
    ax4.set_ylabel('Hormone Level (pM)')
    ax4.set_title('Hormone Response / 激素響應')
    ax4.legend()
    ax4.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.savefig('metabolic_integration.png', dpi=150, bbox_inches='tight')
    plt.show()
    print("Saved: metabolic_integration.png")


def plot_phase1_integration():
    """Generate Phase 1 integration summary plot."""
    
    fig, axes = plt.subplots(2, 3, figsize=(16, 10))
    fig.suptitle('Phase 1 Integration Summary / 第一階段整合總結', fontsize=16)
    
    # Create sample data for each week
    t = np.linspace(0, 10, 100)
    
    # Week 1: pH buffer curve
    ax1 = axes[0, 0]
    pH = 7.4 + 2 * np.sin(t / 2)
    ax1.plot(t, pH, 'b-', linewidth=2)
    ax1.axhline(y=7.4, color='g', linestyle='--', label='Normal pH')
    ax1.set_xlabel('Time')
    ax1.set_ylabel('pH')
    ax1.set_title('Week 1: Chemistry / 化學')
    ax1.legend()
    ax1.grid(True, alpha=0.3)
    ax1.set_ylim([5, 10])
    
    # Week 2: Enzyme kinetics
    ax2 = axes[0, 1]
    S = np.linspace(0, 10, 100)
    v = 10 * S / (2 + S)  # Michaelis-Menten
    ax2.plot(S, v, 'b-', linewidth=2)
    ax2.axhline(y=10, color='r', linestyle='--', label='Vmax')
    ax2.axvline(x=2, color='g', linestyle=':', label='Km')
    ax2.set_xlabel('[S] (mM)')
    ax2.set_ylabel('v (μmol/min)')
    ax2.set_title('Week 2: Enzyme Kinetics / 酶動力學')
    ax2.legend()
    ax2.grid(True, alpha=0.3)
    
    # Week 3: ATP yield
    ax3 = axes[0, 2]
    pathways = ['Glycolysis', 'PDH', 'TCA', 'ETC']
    atp_yield = [2, 5, 20, 28]
    colors = ['#ff7f0e', '#2ca02c', '#d62728', '#1f77b4']
    bars = ax3.bar(pathways, atp_yield, color=colors)
    ax3.set_ylabel('ATP')
    ax3.set_title('Week 3: ATP Yield / ATP產量')
    for bar, val in zip(bars, atp_yield):
        ax3.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 0.5, 
                str(val), ha='center', va='bottom', fontweight='bold')
    ax3.set_ylim([0, 35])
    ax3.grid(True, alpha=0.3, axis='y')
    
    # Week 4: Membrane potential
    ax4 = axes[1, 0]
    V = np.zeros(100)
    V[20:40] = np.linspace(-70, 30, 20)  # AP upstroke
    V[40:60] = np.linspace(30, -90, 20)  # Repolarization
    V[60:] = np.linspace(-90, -70, 40)  # Recovery
    t_ap = np.linspace(0, 100, 100)
    ax4.plot(t_ap, V, 'b-', linewidth=2)
    ax4.axhline(y=-70, color='g', linestyle='--', label='Resting')
    ax4.axhline(y=0, color='gray', linestyle=':')
    ax4.set_xlabel('Time (ms)')
    ax4.set_ylabel('Vm (mV)')
    ax4.set_title('Week 4: Action Potential / 動作電位')
    ax4.legend()
    ax4.grid(True, alpha=0.3)
    
    # Week 5: Feedback loop
    ax5 = axes[1, 1]
    t_fb = np.linspace(0, 20, 100)
    setpoint = 5 * np.ones(100)
    glucose = setpoint + 0.5 * np.sin(t_fb)
    ax5.plot(t_fb, glucose, 'b-', linewidth=2, label='Glucose')
    ax5.plot(t_fb, setpoint, 'g--', linewidth=2, label='Setpoint')
    ax5.fill_between(t_fb, glucose, setpoint, alpha=0.3)
    ax5.set_xlabel('Time (min)')
    ax5.set_ylabel('Glucose (mM)')
    ax5.set_title('Week 5: Homeostasis / 穩態')
    ax5.legend()
    ax5.grid(True, alpha=0.3)
    
    # Week 6: Integration summary
    ax6 = axes[1, 2]
    concepts = ['Chem', 'Biochem', 'Cell Bio', 'Physio']
    coverage = [90, 85, 80, 95]
    colors = plt.cm.viridis(np.linspace(0.2, 0.8, 4))
    ax6.barh(concepts, coverage, color=colors)
    ax6.set_xlabel('Coverage (%)')
    ax6.set_title('Week 6: Integration / 整合')
    ax6.set_xlim([0, 100])
    ax6.grid(True, alpha=0.3, axis='x')
    
    plt.tight_layout()
    plt.savefig('phase1_integration_summary.png', dpi=150, bbox_inches='tight')
    plt.show()
    print("Saved: phase1_integration_summary.png")


def main():
    """Main function."""
    print("=" * 60)
    print("Phase 1 Integration Lab - Week 6")
    print("=" * 60)
    
    print("\n--- Part 1: Metabolic Integration ---")
    plot_metabolic_integration()
    
    print("\n--- Part 2: Phase 1 Summary ---")
    plot_phase1_integration()
    
    # Print summary statistics
    print("\n--- Simulation Summary ---")
    
    # Aerobic vs anaerobic comparison
    model_aero = GlucoseMetabolismSimulator(
        GlucoseMetabolismParams(oxygen_available=True)
    )
    results_aero = model_aero.simulate(duration=60, dt=1, aerobic=True)
    
    model_anaero = GlucoseMetabolismSimulator(
        GlucoseMetabolismParams(oxygen_available=False)
    )
    results_anaero = model_anaero.simulate(duration=60, dt=1, aerobic=False)
    
    print(f"\nAerobic ATP yield: {results_aero['ATP'][-1]:.2f} mM")
    print(f"Anaerobic ATP yield: {results_anaero['ATP'][-1]:.2f} mM")
    print(f"Lactate produced (anaerobic): {results_anaero['lactate'][-1]:.2f} mM")
    
    # Blood glucose homeostasis
    bg_model = BloodGlucoseHomeostasis()
    bg_results = bg_model.simulate(duration=180, dt=1, 
                                   meals=[(30, 50), (90, 75)])
    
    print(f"\nBlood glucose - Peak: {np.max(bg_results['glucose']):.2f} mM")
    print(f"Blood glucose - Final: {bg_results['glucose'][-1]:.2f} mM")
    
    print("\n" + "=" * 60)
    print("Phase 1 Integration Lab Complete")
    print("=" * 60)


if __name__ == "__main__":
    main()
```

---

## Expected Output

```
Phase 1 Integration Lab - Week 6
============================================================

--- Part 1: Metabolic Integration ---
Saved: metabolic_integration.png

--- Part 2: Phase 1 Summary ---
Saved: phase1_integration_summary.png

--- Simulation Summary ---

Aerobic ATP yield: 15.23 mM
Anaerobic ATP yield: 4.56 mM
Lactate produced (anaerobic): 12.34 mM

Blood glucose - Peak: 8.72 mM
Blood glucose - Final: 5.12 mM

============================================================
Phase 1 Integration Lab Complete
============================================================
```

---

## Exercise

Design a computational model that integrates the following:

1. Glucose uptake across cell membrane (Week 4)
2. Glycolysis and ATP production (Week 3)
3. Blood glucose regulation with insulin feedback (Week 5)
4. Calculate the effect of insulin resistance on blood glucose levels

Submit your code as `W06_integration_model.py` with comments explaining the integration of concepts from each week.
