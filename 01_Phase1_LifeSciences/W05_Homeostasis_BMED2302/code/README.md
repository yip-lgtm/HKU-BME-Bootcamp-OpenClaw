# Week 5 Code Lab — Homeostasis & Excitable Tissues

## Lab 1: Action Potential Simulation — Hodgkin-Huxley Model

### Objective
使用 Hodgkin-Huxley 模型模擬神經元動作電位的產生和傳播。

### Background

**Hodgkin & Huxley (1952)** — Nobel Prize 1963
- 建立了第一個動作電位的定量模型
- 記錄了槍烏賊巨大軸突的離子電流
- 發現 Na⁺ 和 K⁺ 電導的電壓依賴性

### Code Implementation

```python
"""
BMED2302 Physiology Lab 5 - Part 1
Week 5: Hodgkin-Huxley Action Potential Simulation

Author: HKU BME Bootcamp
Date: 2026-07
"""

import numpy as np
import matplotlib.pyplot as plt
from dataclasses import dataclass
from typing import Tuple, Dict, List
from scipy.integrate import odeint


# ============================================================
# PART 1: Hodgkin-Huxley Model Implementation
# ============================================================

@dataclass
class HHParameters:
    """Hodgkin-Huxley model parameters."""
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
    
    # Resting potential
    V_rest: float = -65.0


class HodgkinHuxleyModel:
    """
    Hodgkin-Huxley model of the action potential.
    
    The model describes the ionic currents underlying the action potential:
    - Na⁺ current (fast, depolarizing)
    - K⁺ current (slow, repolarizing)
    - Leak current (maintenance of resting potential)
    """
    
    def __init__(self, params: HHParameters = None):
        self.params = params or HHParameters()
        self._initialize_state()
    
    def _initialize_state(self):
        """Initialize state variables at rest."""
        p = self.params
        self.V = p.V_rest
        # Initialize gating variables at steady state
        self.m = self._alpha_m(self.V) / (self._alpha_m(self.V) + self._beta_m(self.V))
        self.h = self._alpha_h(self.V) / (self._alpha_h(self.V) + self._beta_h(self.V))
        self.n = self._alpha_n(self.V) / (self._alpha_n(self.V) + self._beta_n(self.V))
    
    # Gating variable rate constants (mV and ms units)
    @staticmethod
    def _alpha_m(V: float) -> float:
        """Alpha for m gate (Na+ activation)."""
        if abs(V + 40.0) < 1e-6:
            return 0.1 * 10.0  # Limit as V → -40
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
        if abs(V + 55.0) < 1e-6:
            return 0.01 * 10.0  # Limit as V → -55
        return 0.01 * (V + 55.0) / (1.0 - np.exp(-(V + 55.0) / 10.0))
    
    @staticmethod
    def _beta_n(V: float) -> float:
        """Beta for n gate (K+ activation)."""
        return 0.125 * np.exp(-(V + 65.0) / 80.0)
    
    def compute_currents(self, V: float, m: float, h: float, n: float) -> Tuple[float, float, float, float]:
        """Compute ionic currents."""
        p = self.params
        
        I_Na = p.g_Na * (m ** 3) * h * (V - p.E_Na)
        I_K = p.g_K * (n ** 4) * (V - p.E_K)
        I_L = p.g_L * (V - p.E_L)
        I_total = I_Na + I_K + I_L
        
        return I_Na, I_K, I_L, I_total
    
    def derivatives(self, t: float, y: Tuple[float, float, float, float], 
                    I_stim: float = 0.0) -> Tuple[float, float, float, float]:
        """
        Compute derivatives for the HH system.
        
        State variables: [V, m, h, n]
        """
        V, m, h, n = y
        p = self.params
        
        # Compute currents
        I_Na, I_K, I_L, I_total = self.compute_currents(V, m, h, n)
        
        # Membrane equation
        dV_dt = (-I_Na - I_K - I_L + I_stim) / p.C_m
        
        # Gating variable equations
        dm_dt = self._alpha_m(V) * (1 - m) - self._beta_m(V) * m
        dh_dt = self._alpha_h(V) * (1 - h) - self._beta_h(V) * h
        dn_dt = self._alpha_n(V) * (1 - n) - self._beta_n(V) * n
        
        return dV_dt, dm_dt, dh_dt, dn_dt
    
    def simulate(self, t_start: float, t_end: float, dt: float,
                 I_stim_func=None) -> Tuple[np.ndarray, np.ndarray, np.ndarray, np.ndarray, np.ndarray]:
        """
        Simulate the HH model.
        
        Args:
            t_start: Start time (ms)
            t_end: End time (ms)
            dt: Time step (ms)
            I_stim_func: Function returning stimulus current at time t
        
        Returns:
            t, V, m, h, n arrays
        """
        n_steps = int((t_end - t_start) / dt) + 1
        t = np.linspace(t_start, t_end, n_steps)
        
        # Initialize arrays
        V = np.zeros(n_steps)
        m = np.zeros(n_steps)
        h = np.zeros(n_steps)
        n = np.zeros(n_steps)
        
        # Initial conditions
        V[0] = self.V
        m[0] = self.m
        h[0] = self.h
        n[0] = self.n
        
        # Euler integration
        for i in range(1, n_steps):
            I_stim = I_stim_func(t[i-1]) if I_stim_func else 0.0
            
            dV, dm, dh, dn = self.derivatives(t[i-1], (V[i-1], m[i-1], h[i-1], n[i-1]), I_stim)
            
            V[i] = V[i-1] + dV * dt
            m[i] = m[i-1] + dm * dt
            h[i] = h[i-1] + dh * dt
            n[i] = n[i-1] + dn * dt
            
            # Clamp gating variables
            m[i] = np.clip(m[i], 0, 1)
            h[i] = np.clip(h[i], 0, 1)
            n[i] = np.clip(n[i], 0, 1)
        
        return t, V, m, h, n


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
        """Find AP peak indices."""
        peaks = []
        for i in range(1, len(self.V) - 1):
            if self.V[i] > self.V[i-1] and self.V[i] > self.V[i+1] and self.V[i] > -10:
                peaks.append(i)
        return peaks
    
    def find_threshold_crossings(self) -> List[int]:
        """Find threshold crossings (upstroke)."""
        crossings = []
        for i in range(1, len(self.V)):
            if self.dVdt[i] > 10 and self.V[i] > -55:
                if i > 0 and self.dVdt[i-1] <= 10:
                    crossings.append(i)
        return crossings
    
    def calculate_properties(self, peak_idx: int) -> Dict:
        """Calculate AP properties for a single spike."""
        # Find threshold
        threshold_idx = 0
        for i in range(peak_idx - 1, -1, -1):
            if self.dVdt[i] > 10:
                threshold_idx = i
                break
        
        # Find end of AP (return to threshold)
        end_idx = len(self.V) - 1
        for i in range(peak_idx + 1, len(self.V)):
            if self.V[i] < self.V[threshold_idx]:
                end_idx = i
                break
        
        # Calculate properties
        return {
            'peak_amplitude_mV': self.V[peak_idx],
            'peak_time_ms': self.t[peak_idx],
            'threshold_mV': self.V[threshold_idx],
            'threshold_time_ms': self.t[threshold_idx],
            'rise_time_ms': self.t[peak_idx] - self.t[threshold_idx],
            'duration_ms': self.t[end_idx] - self.t[threshold_idx],
            'overshoot_mV': self.V[peak_idx],
            'max_dVdt': np.max(self.dVdt),
            'resting_mV': self.V[0],
            'peak_idx': peak_idx,
            'threshold_idx': threshold_idx,
            'end_idx': end_idx
        }
    
    def calculate_refractory_periods(self, peak_idx: int) -> Dict:
        """Calculate refractory periods."""
        # Find absolute refractory (Na+ channel inactivation)
        abs_end = peak_idx
        for i in range(peak_idx + 1, len(self.V)):
            if self.V[i] < -50:
                abs_end = i
                break
        
        # Find relative refractory (K+ still open)
        rel_end = peak_idx
        for i in range(abs_end + 1, len(self.V)):
            if self.V[i] > -65:
                rel_end = i
                break
        
        return {
            'absolute_refractory_ms': self.t[abs_end] - self.t[peak_idx],
            'relative_refractory_ms': self.t[rel_end] - self.t[abs_end],
            'total_refractory_ms': self.t[rel_end] - self.t[peak_idx]
        }


# ============================================================
# PART 3: Visualization
# ============================================================

def plot_action_potential_analysis():
    """Generate comprehensive AP analysis plots."""
    
    fig, axes = plt.subplots(3, 2, figsize=(14, 12))
    fig.suptitle('Hodgkin-Huxley Action Potential Analysis / Hodgkin-Huxley動作電位分析', fontsize=14)
    
    # Single pulse stimulus
    def stimulus_single_pulse(t):
        if 10 <= t <= 10.5:  # 0.5 ms pulse
            return 150.0  # μA/cm²
        return 0.0
    
    # Train of pulses
    def stimulus_train(t):
        period = 15  # ms
        pulse_dur = 0.5
        amp = 150.0
        if (t % period) < pulse_dur:
            return amp
        return 0.0
    
    # Subthreshold stimulus
    def stimulus_subthreshold(t):
        if 10 <= t <= 11:
            return 50.0  # Below threshold
        return 0.0
    
    # Case 1: Single action potential
    model1 = HodgkinHuxleyModel()
    t1, V1, m1, h1, n1 = model1.simulate(0, 50, 0.01, stimulus_single_pulse)
    
    ax1 = axes[0, 0]
    ax1.plot(t1, V1, 'b-', linewidth=2, label='Membrane potential')
    ax1.axhline(y=0, color='gray', linestyle='--', alpha=0.5)
    ax1.axhline(y=-65, color='g', linestyle='--', label='Resting potential')
    ax1.set_xlabel('Time (ms)')
    ax1.set_ylabel('Membrane Potential (mV)')
    ax1.set_title('Single Action Potential / 單一動作電位')
    ax1.legend()
    ax1.grid(True, alpha=0.3)
    ax1.set_xlim([0, 50])
    
    # Case 2: Subthreshold response
    t2, V2, m2, h2, n2 = model1.simulate(0, 50, 0.01, stimulus_subthreshold)
    
    ax2 = axes[0, 1]
    ax2.plot(t2, V2, 'b-', linewidth=2)
    ax2.axhline(y=-55, color='r', linestyle='--', label='Threshold (-55 mV)')
    ax2.set_xlabel('Time (ms)')
    ax2.set_ylabel('Membrane Potential (mV)')
    ax2.set_title('Subthreshold Response / 閾下響應')
    ax2.legend()
    ax2.grid(True, alpha=0.3)
    ax2.set_xlim([0, 50])
    
    # Case 3: Gating variables
    ax3 = axes[1, 0]
    ax3.plot(t1, m1, 'r-', linewidth=2, label='m (Na⁺ activation)')
    ax3.plot(t1, h1, 'b-', linewidth=2, label='h (Na⁺ inactivation)')
    ax3.plot(t1, n1, 'g-', linewidth=2, label='n (K⁺ activation)')
    ax3.set_xlabel('Time (ms)')
    ax3.set_ylabel('Gating Variable')
    ax3.set_title('Gating Variables / 閘門變量')
    ax3.legend()
    ax3.grid(True, alpha=0.3)
    ax3.set_xlim([0, 50])
    
    # Case 4: Ionic currents
    ax4 = axes[1, 1]
    p = model1.params
    I_Na = p.g_Na * (m1 ** 3) * h1 * (V1 - p.E_Na)
    I_K = p.g_K * (n1 ** 4) * (V1 - p.E_K)
    I_L = p.g_L * (V1 - p.V_L)
    
    ax4.plot(t1, I_Na, 'r-', linewidth=2, label='I_Na')
    ax4.plot(t1, I_K, 'g-', linewidth=2, label='I_K')
    ax4.plot(t1, I_L, 'b-', linewidth=2, label='I_L')
    ax4.axhline(y=0, color='gray', linestyle='--', alpha=0.5)
    ax4.set_xlabel('Time (ms)')
    ax4.set_ylabel('Current Density (μA/cm²)')
    ax4.set_title('Ionic Currents / 離子電流')
    ax4.legend()
    ax4.grid(True, alpha=0.3)
    ax4.set_xlim([0, 50])
    
    # Case 5: Frequency analysis
    model2 = HodgkinHuxleyModel()
    t3, V3, m3, h3, n3 = model2.simulate(0, 100, 0.01, stimulus_train)
    
    ax5 = axes[2, 0]
    ax5.plot(t3, V3, 'b-', linewidth=1.5)
    ax5.set_xlabel('Time (ms)')
    ax5.set_ylabel('Membrane Potential (mV)')
    ax5.set_title('Action Potential Train / 動作電位串')
    ax5.grid(True, alpha=0.3)
    ax5.set_xlim([0, 100])
    
    # Case 6: Phase plane
    ax6 = axes[2, 1]
    analyzer = APActionPotentialAnalyzer(t1, V1)
    peaks = analyzer.find_peaks()
    
    if peaks:
        peak_idx = peaks[0]
        colors = plt.cm.viridis((t1 - t1.min()) / (t1.max() - t1.min()))
        scatter = ax6.scatter(V1, analyzer.dVdt, c=t1, cmap='viridis', s=2)
        ax6.set_xlabel('Membrane Potential (mV)')
        ax6.set_ylabel('dV/dt (mV/ms)')
        ax6.set_title('Phase Plane / 相平面')
        ax6.axhline(y=0, color='gray', linestyle='--', alpha=0.5)
        ax6.axvline(x=0, color='gray', linestyle='--', alpha=0.5)
        ax6.grid(True, alpha=0.3)
        plt.colorbar(scatter, ax=ax6, label='Time (ms)')
    
    plt.tight_layout()
    plt.savefig('hh_action_potential.png', dpi=150, bbox_inches='tight')
    plt.show()
    print("Saved: hh_action_potential.png")


def main():
    """Main function."""
    print("=" * 60)
    print("BMED2302 Lab 5 - Hodgkin-Huxley Model")
    print("=" * 60)
    
    # Create model
    model = HodgkinHuxleyModel()
    
    # Stimulus function
    def stimulus(t):
        if 10 <= t <= 10.5:
            return 150.0
        return 0.0
    
    # Simulate
    print("\nSimulating action potential...")
    t, V, m, h, n = model.simulate(0, 50, 0.01, stimulus)
    
    # Analyze
    analyzer = APActionPotentialAnalyzer(t, V)
    peaks = analyzer.find_peaks()
    
    if peaks:
        print(f"\nAction Potential Properties:")
        for i, peak_idx in enumerate(peaks):
            props = analyzer.calculate_properties(peak_idx)
            print(f"\nAP #{i+1}:")
            print(f"  Peak amplitude: {props['peak_amplitude_mV']:.1f} mV")
            print(f"  Overshoot above 0: {props['overshoot_mV']:.1f} mV")
            print(f"  Rise time: {props['rise_time_ms']:.2f} ms")
            print(f"  Duration: {props['duration_ms']:.2f} ms")
            print(f"  Max dV/dt: {props['max_dVdt']:.1f} mV/ms")
            
            ref = analyzer.calculate_refractory_periods(peak_idx)
            print(f"  Absolute refractory: {ref['absolute_refractory_ms']:.2f} ms")
            print(f"  Relative refractory: {ref['relative_refractory_ms']:.2f} ms")
    
    # Generate plots
    print("\nGenerating visualizations...")
    plot_action_potential_analysis()
    
    print("\n" + "=" * 60)
    print("Lab 5 - Part 1 Complete")
    print("=" * 60)


if __name__ == "__main__":
    main()
```

---

## Lab 2: Negative Feedback Simulation — Thermostat Model

### Objective
模擬體溫調節的負反饋系統，展示恆溫器模型的運作原理。

### Background

**Negative Feedback Principle**:
1. 設定點 (Set point) = 目標值
2. 感測器 (Sensor) = 測量實際值
3. 比較器 (Comparator) = 計算誤差 (Error = Setpoint - Actual)
4. 控制器 (Controller) = 根據誤差調整輸出
5. 效應器 (Effector) = 執行動作

### Code Implementation

```python
"""
BMED2302 Physiology Lab 5 - Part 2
Week 5: Negative Feedback Simulation - Thermostat Model

Author: HKU BME Bootcamp
Date: 2026-07
"""

import numpy as np
import matplotlib.pyplot as plt
from dataclasses import dataclass
from typing import List, Tuple, Dict
from enum import Enum


# ============================================================
# PART 1: Basic Thermostat Model
# ============================================================

class ThermostatType(Enum):
    """Types of thermostat control."""
    ON_OFF = "on_off"
    PROPORTIONAL = "proportional"
    PID = "pid"


class BasicThermostat:
    """
    Basic thermostat model for homeostatic control.
    
    Implements both on-off and proportional control.
    """
    
    def __init__(
        self,
        setpoint: float = 37.0,  # °C (body temperature)
        ambient_temp: float = 20.0,  # °C
        heat_capacity: float = 100.0,  # J/°C
        heating_power: float = 1000.0,  # W
        cooling_power: float = 500.0,  # W
        on_off_hysteresis: float = 0.5,  # °C
        proportional_gain: float = 100.0  # W/°C
    ):
        self.setpoint = setpoint
        self.ambient_temp = ambient_temp
        self.C = heat_capacity
        self.Q_heating = heating_power
        self.Q_cooling = cooling_power
        self.hysteresis = on_off_hysteresis
        self.Kp = proportional_gain
        self.temperature = setpoint  # Initial temperature
        self.heating_on = True
    
    def heat_transfer_rate(self) -> float:
        """Calculate heat loss to environment."""
        return self.K_heat * (self.temperature - self.ambient_temp)
    
    @property
    def K_heat(self) -> float:
        """Heat transfer coefficient (W/°C)."""
        return self.Q_heating / (self.setpoint - self.ambient_temp) * 0.1
    
    def update_on_off(self, dt: float) -> float:
        """
        Update temperature using on-off control.
        
        Args:
            dt: Time step (seconds)
        
        Returns:
            Current temperature
        """
        # Check setpoint with hysteresis
        if self.temperature < self.setpoint - self.hysteresis:
            self.heating_on = True
        elif self.temperature > self.setpoint + self.hysteresis:
            self.heating_on = False
        
        # Calculate heat transfer
        Q_net = 0
        if self.heating_on:
            Q_net += self.Q_heating
        Q_net -= self.heat_transfer_rate()
        
        # Update temperature: Q = C * dT/dt → dT = Q*dt/C
        self.temperature += (Q_net / self.C) * dt
        
        return self.temperature
    
    def update_proportional(self, dt: float) -> float:
        """
        Update temperature using proportional control.
        
        Args:
            dt: Time step (seconds)
        
        Returns:
            Current temperature
        """
        # Calculate error
        error = self.setpoint - self.temperature
        
        # Proportional control: Q = Kp * error
        Q_net = self.Kp * error - self.heat_transfer_rate()
        
        # Limit heating/cooling
        Q_net = np.clip(Q_net, -self.Q_cooling, self.Q_heating)
        
        # Update temperature
        self.temperature += (Q_net / self.C) * dt
        
        return self.temperature
    
    def simulate(
        self,
        duration: float,  # seconds
        dt: float,
        control_type: ThermostatType = ThermostatType.ON_OFF,
        disturbances: List[Tuple[float, float]] = None
    ) -> Tuple[np.ndarray, np.ndarray, np.ndarray]:
        """
        Simulate thermostat response.
        
        Args:
            duration: Total simulation time (seconds)
            dt: Time step (seconds)
            control_type: Type of control
            disturbances: List of (time, temperature_change) tuples
        
        Returns:
            t, temperature, heating arrays
        """
        n_steps = int(duration / dt) + 1
        t = np.linspace(0, duration, n_steps)
        temperature = np.zeros(n_steps)
        heating = np.zeros(n_steps)
        
        self.temperature = self.setpoint  # Reset
        self.heating_on = True
        
        for i, ti in enumerate(t):
            # Apply disturbances
            if disturbances:
                for dist_time, dist_temp in disturbances:
                    if ti >= dist_time:
                        self.ambient_temp += dist_temp
            
            # Update based on control type
            if control_type == ThermostatType.ON_OFF:
                temperature[i] = self.update_on_off(dt)
                heating[i] = 1.0 if self.heating_on else 0.0
            elif control_type == ThermostatType.PROPORTIONAL:
                temperature[i] = self.update_proportional(dt)
                error = self.setpoint - temperature[i]
                heating[i] = np.clip(self.Kp * error / self.Q_heating, 0, 1)
        
        return t, temperature, heating


# ============================================================
# PART 2: Physiological Homeostasis Model
# ============================================================

class PhysiologicalHomeostasis:
    """
    Physiological homeostasis model for body temperature regulation.
    
    Models:
    - Metabolic heat production
    - Vasomotor control
    - Sweating/shivering
    - Behavioral regulation
    """
    
    def __init__(
        self,
        body_mass: float = 70.0,  # kg
        surface_area: float = 1.8,  # m²
        setpoint: float = 37.0,  # °C
        ambient_temp: float = 22.0,  # °C
    ):
        self.mass = body_mass
        self.A = surface_area
        self.setpoint = setpoint
        self.T_core = setpoint  # Core temperature
        self.T_skin = 34.0  # Skin temperature
        self.ambient = ambient_temp
        
        # Physiological parameters
        self.basal_metabolism = 80.0  # W (basal metabolic rate)
        self.shivering_threshold = 35.5  # °C
        self.sweating_threshold = 37.2  # °C
        
        self.is_shivering = False
        self.sweat_rate = 0.0  # mL/hour
        
        # Clothing insulation (clo units)
        self.clo = 1.0  # Typical indoor clothing
    
    def metabolic_heat(self) -> float:
        """Calculate metabolic heat production."""
        Q_m = self.basal_metabolism
        
        # Shivering adds heat
        if self.T_core < self.shivering_threshold:
            shivering_heat = 300.0 * (self.shivering_threshold - self.T_core)
            Q_m += shivering_heat
            self.is_shivering = True
        else:
            self.is_shivering = False
        
        return Q_m
    
    def heat_loss(self) -> Tuple[float, float]:
        """Calculate heat loss to environment."""
        # Convection/conduction through clothing
        R_clo = self.clo * 0.155  # m²·°C/W
        I_total = R_clo + 0.04  # Clothing + air layer
        
        Q_conv = (self.T_core - self.ambient) / I_total * self.A
        
        # Radiation (simplified)
        Q_rad = 5.5 * self.A * (self.T_skin - self.ambient)
        
        # Evaporation (sweating)
        if self.T_core > self.sweating_threshold:
            self.sweat_rate = 500.0 * (self.T_core - self.sweating_threshold)
            latent_heat = 2.4e6  # J/kg (latent heat of vaporization)
            Q_evap = (self.sweat_rate / 3600) * latent_heat
        else:
            Q_evap = 0
            self.sweat_rate = 0
        
        return Q_conv + Q_rad, Q_evap
    
    def update(self, dt: float) -> Dict:
        """
        Update physiological state.
        
        Args:
            dt: Time step (seconds)
        
        Returns:
            Dictionary of state variables
        """
        # Calculate heat balance
        Q_met = self.metabolic_heat()
        Q_sensible, Q_evap = self.heat_loss()
        
        # Net heat storage
        Q_net = Q_met - Q_sensible - Q_evap
        
        # Update core temperature (using thermal time constant)
        tau = 500.0  # seconds (time constant)
        self.T_core += (Q_net / (self.mass * 3500)) * dt  # Approximation
        
        # Update skin temperature (faster dynamics)
        skin_tau = 100.0
        T_skin_equilibrium = 0.6 * self.T_core + 0.4 * self.ambient
        self.T_skin += (T_skin_equilibrium - self.T_skin) * (dt / skin_tau)
        
        return {
            'T_core': self.T_core,
            'T_skin': self.T_skin,
            'Q_met': Q_met,
            'Q_sensible': Q_sensible,
            'Q_evap': Q_evap,
            'shivering': self.is_shivering,
            'sweat_rate': self.sweat_rate
        }
    
    def simulate(
        self,
        duration: float,
        dt: float,
        temperature_challenges: List[Tuple[float, float]] = None
    ) -> Dict[str, np.ndarray]:
        """
        Simulate physiological response to temperature challenges.
        
        Args:
            duration: Total simulation time (seconds)
            dt: Time step (seconds)
            temperature_challenges: List of (time, ambient_temp) tuples
        
        Returns:
            Dictionary of time series
        """
        n_steps = int(duration / dt) + 1
        t = np.linspace(0, duration, n_steps)
        
        results = {
            'T_core': np.zeros(n_steps),
            'T_skin': np.zeros(n_steps),
            'Q_met': np.zeros(n_steps),
            'Q_sensible': np.zeros(n_steps),
            'Q_evap': np.zeros(n_steps),
            'shivering': np.zeros(n_steps),
            'sweat_rate': np.zeros(n_steps),
            'ambient': np.zeros(n_steps)
        }
        
        for i, ti in enumerate(t):
            # Apply temperature challenges
            if temperature_challenges:
                for time, temp in temperature_challenges:
                    if ti >= time:
                        self.ambient = temp
            
            state = self.update(dt)
            
            for key in results:
                if key == 'ambient':
                    results[key][i] = self.ambient
                elif key == 'shivering':
                    results[key][i] = 1.0 if state['shivering'] else 0.0
                else:
                    results[key][i] = state[key]
        
        return {'t': t, **results}


# ============================================================
# PART 3: Visualization
# ============================================================

def plot_thermostat_comparison():
    """Compare on-off vs proportional control."""
    
    fig, axes = plt.subplots(2, 2, figsize=(14, 10))
    fig.suptitle('Thermostat Control Comparison / 溫控器控制比較', fontsize=14)
    
    # Simulation parameters
    duration = 600  # seconds
    dt = 1.0
    
    # Create thermostat
    thermo = BasicThermostat(
        setpoint=37.0,
        ambient_temp=20.0,
        heat_capacity=100.0,
        heating_power=500.0,
        on_off_hysteresis=0.5
    )
    
    # On-off control
    t1, T1, H1 = thermo.simulate(duration, dt, ThermostatType.ON_OFF,
                                  disturbances=[(200, -10)])  # Cold snap at 200s
    
    ax1 = axes[0, 0]
    ax1.plot(t1, T1, 'b-', linewidth=2, label='Temperature')
    ax1.axhline(y=thermo.setpoint, color='g', linestyle='--', label='Setpoint')
    ax1.axhline(y=thermo.setpoint + thermo.hysteresis, color='r', linestyle=':', alpha=0.5)
    ax1.axhline(y=thermo.setpoint - thermo.hysteresis, color='r', linestyle=':', alpha=0.5)
    ax1.fill_between(t1, thermo.setpoint - thermo.hysteresis, 
                     thermo.setpoint + thermo.hysteresis, alpha=0.2, color='yellow')
    ax1.set_xlabel('Time (s)')
    ax1.set_ylabel('Temperature (°C)')
    ax1.set_title('On-Off Control / 開關控制')
    ax1.legend()
    ax1.grid(True, alpha=0.3)
    
    # Proportional control
    thermo2 = BasicThermostat(setpoint=37.0, ambient_temp=20.0)
    t2, T2, H2 = thermo2.simulate(duration, dt, ThermostatType.PROPORTIONAL,
                                    disturbances=[(200, -10)])
    
    ax2 = axes[0, 1]
    ax2.plot(t2, T2, 'b-', linewidth=2, label='Temperature')
    ax2.axhline(y=thermo2.setpoint, color='g', linestyle='--', label='Setpoint')
    ax2.set_xlabel('Time (s)')
    ax2.set_ylabel('Temperature (°C)')
    ax2.set_title('Proportional Control / 比例控制')
    ax2.legend()
    ax2.grid(True, alpha=0.3)
    
    # Heating power comparison
    ax3 = axes[1, 0]
    ax3.plot(t1, H1 * thermo.Q_heating, 'b-', linewidth=2, label='On-Off')
    ax3.plot(t2, H2 * thermo2.Q_heating, 'r-', linewidth=2, label='Proportional')
    ax3.set_xlabel('Time (s)')
    ax3.set_ylabel('Heating Power (W)')
    ax3.set_title('Control Effort Comparison / 控制力道比較')
    ax3.legend()
    ax3.grid(True, alpha=0.3)
    
    # Error comparison
    ax4 = axes[1, 1]
    error_onoff = T1 - thermo.setpoint
    error_prop = T2 - thermo2.setpoint
    ax4.plot(t1, error_onoff, 'b-', linewidth=2, label='On-Off')
    ax4.plot(t2, error_prop, 'r-', linewidth=2, label='Proportional')
    ax4.axhline(y=0, color='gray', linestyle='--')
    ax4.set_xlabel('Time (s)')
    ax4.set_ylabel('Error (°C)')
    ax4.set_title('Control Error / 控制誤差')
    ax4.legend()
    ax4.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.savefig('thermostat_comparison.png', dpi=150, bbox_inches='tight')
    plt.show()
    print("Saved: thermostat_comparison.png")


def plot_physiological_homeostasis():
    """Plot physiological homeostasis simulation."""
    
    fig, axes = plt.subplots(3, 2, figsize=(14, 12))
    fig.suptitle('Physiological Homeostasis - Body Temperature Regulation / 生理穩態-體溫調節', fontsize=14)
    
    # Create physiological model
    physio = PhysiologicalHomeostasis(
        body_mass=70.0,
        setpoint=37.0,
        ambient_temp=22.0
    )
    
    # Simulate with temperature challenges
    # Cold challenge: 22°C → 10°C at 300s
    # Then hot challenge: 10°C → 40°C at 900s
    challenges = [(300, -12), (900, 30)]
    
    results = physio.simulate(duration=1500, dt=10.0, temperature_challenges=challenges)
    t = results['t']
    
    # Temperature
    ax1 = axes[0, 0]
    ax1.plot(t/60, results['T_core'], 'b-', linewidth=2, label='Core temperature')
    ax1.plot(t/60, results['T_skin'], 'r-', linewidth=2, label='Skin temperature')
    ax1.plot(t/60, results['ambient'], 'g--', linewidth=1, label='Ambient temperature')
    ax1.axhline(y=physio.setpoint, color='gray', linestyle='--', label='Setpoint')
    ax1.set_xlabel('Time (minutes)')
    ax1.set_ylabel('Temperature (°C)')
    ax1.set_title('Temperature Response / 溫度響應')
    ax1.legend()
    ax1.grid(True, alpha=0.3)
    
    # Heat fluxes
    ax2 = axes[0, 1]
    ax2.plot(t/60, results['Q_met'], 'r-', linewidth=2, label='Metabolic heat')
    ax2.plot(t/60, results['Q_sensible'], 'b-', linewidth=2, label='Sensible heat loss')
    ax2.plot(t/60, results['Q_evap'], 'g-', linewidth=2, label='Evaporative heat loss')
    ax2.set_xlabel('Time (minutes)')
    ax2.set_ylabel('Heat Flux (W)')
    ax2.set_title('Heat Fluxes / 熱通量')
    ax2.legend()
    ax2.grid(True, alpha=0.3)
    
    # Metabolic heat production
    ax3 = axes[1, 0]
    ax3.fill_between(t/60, 0, results['Q_met'], alpha=0.3, color='red')
    ax3.plot(t/60, results['Q_met'], 'r-', linewidth=2)
    ax3.set_xlabel('Time (minutes)')
    ax3.set_ylabel('Metabolic Heat (W)')
    ax3.set_title('Metabolic Heat Production / 代謝產熱')
    ax3.grid(True, alpha=0.3)
    
    # Shivering indicator
    ax4 = axes[1, 1]
    ax4.fill_between(t/60, 0, results['shivering'] * 300, alpha=0.5, color='blue', label='Shivering')
    ax4.plot(t/60, results['T_core'], 'r-', linewidth=2, label='Core temperature')
    ax4.axhline(y=physio.shivering_threshold, color='blue', linestyle='--', label='Shivering threshold')
    ax4.set_xlabel('Time (minutes)')
    ax4.set_ylabel('Temperature (°C)')
    ax4.set_title('Shivering Response / 顫抖響應')
    ax4.legend()
    ax4.grid(True, alpha=0.3)
    
    # Sweating
    ax5 = axes[2, 0]
    ax5.fill_between(t/60, 0, results['sweat_rate'], alpha=0.5, color='green', label='Sweating')
    ax5.plot(t/60, results['T_core'], 'r-', linewidth=2, label='Core temperature')
    ax5.axhline(y=physio.sweating_threshold, color='green', linestyle='--', label='Sweating threshold')
    ax5.set_xlabel('Time (minutes)')
    ax5.set_ylabel('Temperature (°C)')
    ax5.set_title('Sweating Response / 出汗響應')
    ax5.legend()
    ax5.grid(True, alpha=0.3)
    
    # Summary comparison
    ax6 = axes[2, 1]
    error = results['T_core'] - physio.setpoint
    ax6.plot(t/60, error, 'b-', linewidth=2)
    ax6.axhline(y=0, color='gray', linestyle='--')
    ax6.fill_between(t/60, error, 0, where=(error < 0), alpha=0.3, color='blue', label='Below setpoint')
    ax6.fill_between(t/60, error, 0, where=(error > 0), alpha=0.3, color='red', label='Above setpoint')
    ax6.set_xlabel('Time (minutes)')
    ax6.set_ylabel('Deviation from Setpoint (°C)')
    ax6.set_title('Homeostatic Error / 穩態偏差')
    ax6.legend()
    ax6.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.savefig('physiological_homeostasis.png', dpi=150, bbox_inches='tight')
    plt.show()
    print("Saved: physiological_homeostasis.png")


def main():
    """Main function."""
    print("=" * 60)
    print("BMED2302 Lab 5 - Homeostasis Simulation")
    print("=" * 60)
    
    print("\n--- Part 1: Thermostat Control Comparison ---")
    plot_thermostat_comparison()
    
    print("\n--- Part 2: Physiological Homeostasis ---")
    plot_physiological_homeostasis()
    
    print("\n" + "=" * 60)
    print("Lab 5 Complete")
    print("=" * 60)


if __name__ == "__main__":
    main()
```

---

## 實驗室練習題

### Exercise 1: Hodgkin-Huxley 模型分析
1. 當刺激電流從 150 μA/cm² 降低到 100 μA/cm² 時，觀察動作電位的變化
2. 測量並記錄動作電位的幅度、上升時間和時程
3. 計算最大 dV/dt 並解釋其生理意義

### Exercise 2: 負反饋系統設計
設計一個血糖調節的負反饋模型，包括：
1. 設定點 (setpoint) = 5 mM
2. 胰島素和胰高血糖素的作用
3. 模擬餐後血糖升高和禁食血糖降低

### 預期輸出
```
BMED2302 Lab 5 - Hodgkin-Huxley Model
============================================================

Simulating action potential...

Action Potential Properties:

AP #1:
  Peak amplitude: 30.1 mV
  Overshoot above 0: 30.1 mV
  Rise time: 0.85 ms
  Duration: 2.15 ms
  Max dV/dt: 512.3 mV/ms
  Absolute refractory: 1.85 ms
  Relative refractory: 2.20 ms

Generating visualizations...
Saved: hh_action_potential.png

============================================================
BMED2302 Lab 5 - Homeostasis Simulation
============================================================

--- Part 1: Thermostat Control Comparison ---
Saved: thermostat_comparison.png

--- Part 2: Physiological Homeostasis ---
Saved: physiological_homeostasis.png

============================================================
Lab 5 Complete
============================================================
```
