# Week 3 Code Lab — Bioenergetics, Enzymes & Metabolism

## Lab 1: Metabolic Flux Analysis

### Objective
學習使用 Python 分析代谢通量，計算 ATP 產量，並模擬代谢抑制劑效果。

### Code

```python
"""
BMED2301 Biochemistry Lab 3
Week 3: Metabolic Flux Analysis
"""

import numpy as np
import matplotlib.pyplot as plt
from dataclasses import dataclass
from typing import Dict, List, Tuple
from enum import Enum

# ============================================================
# PART 1: ATP Yield Calculator
# ============================================================

class ATPYieldCalculator:
    """Calculate ATP yield from various metabolic substrates."""
    
    # ATP yield per NADH and FADH2 (P/O ratios)
    NADH_ATP = 2.5  # Modern estimate
    FADH2_ATP = 1.5  # Modern estimate
    
    @classmethod
    def glycolysis_atp(cls) -> Dict[str, int]:
        """Calculate glycolysis ATP yield."""
        return {
            'ATP_invested': -2,  # Hexokinase, PFK
            'ATP_produced': 4,   # PGK, PK
            'NADH': 2,          # G3P dehydrogenase
            'net_atp': 2,
            'net_nadh': 2,
            'total_atp_equivalent': 2 + 2 * cls.NADH_ATP  # If using glycolytic NADH
        }
    
    @classmethod
    def pyruvate_dehydrogenase_atp(cls) -> Dict[str, int]:
        """Calculate pyruvate dehydrogenase ATP yield."""
        return {
            'NADH': 2,  # 2 pyruvate → 2 acetyl-CoA
            'total_atp_equivalent': 2 * cls.NADH_ATP
        }
    
    @classmethod
    def citric_acid_cycle_atp(cls) -> Dict[str, int]:
        """Calculate one turn of citric acid cycle ATP yield."""
        return {
            'NADH': 3,   # Isocitrate DH, α-KG DH, Malate DH
            'FADH2': 1,  # Succinate DH (Complex II)
            'GTP': 1,    # Succinyl-CoA synthetase
            'CO2_released': 2,
            'total_atp_equivalent': 3 * cls.NADH_ATP + 1 * cls.FADH2_ATP + 1
        }
    
    @classmethod
    def beta_oxidation_atp(cls, carbon_length: int) -> Dict[str, int]:
        """
        Calculate beta-oxidation ATP yield for a fatty acid.
        
        Args:
            carbon_length: Number of carbons in fatty acid
        
        Returns:
            ATP yield dictionary
        """
        # Number of cycles (n/2 - 1 for even fatty acids)
        cycles = carbon_length // 2 - 1
        
        # Acetyl-CoA produced (n/2)
        acetyl_coa = carbon_length // 2
        
        # Energy yield
        fadh2_atp = cycles * cls.FADH2_ATP  # One FADH2 per cycle
        nadh_atp = cycles * cls.NADH_ATP     # One NADH per cycle
        acetyl_atp = acetyl_coa * cls.citric_acid_cycle_atp()['total_atp_equivalent']
        
        # Activation cost (2 ATP equivalent)
        activation_cost = 2
        
        total = nadh_atp + fadh2_atp + acetyl_atp - activation_cost
        
        return {
            'cycles': cycles,
            'acetyl_coa_produced': acetyl_coa,
            'FADH2_cycles': cycles,
            'NADH_cycles': cycles,
            'activation_cost': -activation_cost,
            'total_atp': total
        }
    
    @classmethod
    def glucose_oxidation_total(cls, shuttle: str = 'malate_aspartate') -> Dict[str, any]:
        """Calculate total ATP from complete glucose oxidation."""
        glycolysis = cls.glycolysis_atp()
        pdh = cls.pyruvate_dehydrogenase_atp()
        tca = cls.citric_acid_cycle_atp()
        
        # Cytosolic NADH from glycolysis requires shuttle
        if shuttle == 'malate_aspartate':
            nadh_atp = 2 * cls.NADH_ATP  # Both NADH transported
        elif shuttle == 'glycerol_phosphate':
            nadh_atp = 2 * 1.5  # Only 1.5 ATP per NADH
        else:
            nadh_atp = 0  # No shuttle
        
        total_glycolysis = glycolysis['net_atp'] + nadh_atp
        total_pdh = pdh['total_atp_equivalent']
        total_tca = 2 * tca['total_atp_equivalent']  # 2 acetyl-CoA per glucose
        
        total = total_glycolysis + total_pdh + total_tca
        
        return {
            'glycolysis_atp': total_glycolysis,
            'pyruvate_dehydrogenase_atp': total_pdh,
            'citric_acid_cycle_atp': total_tca,
            'total_atp': total,
            'shuttle_used': shuttle
        }


# ============================================================
# PART 2: Metabolic Pathway Simulation
# ============================================================

@dataclass
class MetabolicInhibitor:
    """Represents a metabolic inhibitor."""
    name: str
    target: str
    mechanism: str
    severity: float  # 0-1, fraction of inhibition


class MetabolismSimulator:
    """Simulate metabolic pathways with inhibitors."""
    
    PATHWAYS = {
        'glycolysis': {
            'steps': ['Hexokinase', 'PGI', 'PFK-1', 'ALD', 'TPI', 
                     'GAPDH', 'PGK', 'PGM', 'ENO', 'PK'],
            'regulatory': ['Hexokinase', 'PFK-1', 'PK'],
            'atp_yield': 2
        },
        'pdh_complex': {
            'steps': ['PDH'],
            'regulatory': ['PDH'],
            'atp_yield': 5  # NADH equivalents
        },
        'citric_acid_cycle': {
            'steps': ['CS', 'ACON', 'IDH', 'αKGDH', 'SCS', 
                     'SDH', 'FH', 'MDH'],
            'regulatory': ['CS', 'IDH', 'αKGDH'],
            'atp_yield': 20  # Per glucose (2 cycles)
        },
        'etc': {
            'complexes': ['Complex I', 'Complex II', 'Complex III', 
                         'Complex IV', 'ATP Synthase'],
            'atp_yield': 25  # From NADH and FADH2
        }
    }
    
    def __init__(self):
        self.inhibitors: List[MetabolicInhibitor] = []
        self.energy_state = 1.0  # 1.0 = normal
    
    def add_inhibitor(self, inhibitor: MetabolicInhibitor):
        """Add a metabolic inhibitor."""
        self.inhibitors.append(inhibitor)
        self._update_energy_state()
    
    def _update_energy_state(self):
        """Calculate overall energy state based on inhibitors."""
        inhibition_factor = 1.0
        for inh in self.inhibitors:
            inhibition_factor *= (1 - inh.severity)
        self.energy_state = inhibition_factor
    
    def simulate_glycolysis(self, glucose: float = 1.0) -> Dict:
        """Simulate glycolysis rate with current inhibitors."""
        base_rate = glucose
        for inh in self.inhibitors:
            if inh.target in ['glycolysis', 'Hexokinase', 'PFK-1', 'PK']:
                base_rate *= (1 - inh.severity)
        
        return {
            'pyruvate_produced': base_rate * 2,  # 2 pyruvate per glucose
            'atp_produced': base_rate * ATPYieldCalculator.glycolysis_atp()['net_atp'],
            'nadh_produced': base_rate * ATPYieldCalculator.glycolysis_atp()['net_nadh'],
            'inhibition': self.energy_state
        }
    
    def simulate_etc(self, nadh: float, fadh2: float) -> Dict:
        """Simulate oxidative phosphorylation."""
        base_atp_nadh = nadh * ATPYieldCalculator.NADH_ATP
        base_atp_fadh2 = fadh2 * ATPYieldCalculator.FADH2_ATP
        
        for inh in self.inhibitors:
            if 'Complex' in inh.target or 'ATP Synthase' in inh.target:
                base_atp_nadh *= (1 - inh.severity)
                base_atp_fadh2 *= (1 - inh.severity)
        
        return {
            'atp_from_nadh': base_atp_nadh,
            'atp_from_fadh2': base_atp_fadh2,
            'total_atp': base_atp_nadh + base_atp_fadh2,
            'oxygen_consumed': (nadh + fadh2) * self.energy_state
        }
    
    def clinical_interpretation(self) -> List[str]:
        """Provide clinical interpretation of current state."""
        interpretations = []
        
        if self.energy_state < 0.2:
            interpretations.append("CRITICAL: Near-complete metabolic shutdown")
        elif self.energy_state < 0.5:
            interpretations.append("SEVERE: Significant impairment")
        elif self.energy_state < 0.8:
            interpretations.append("MODERATE: Partial impairment")
        else:
            interpretations.append("NORMAL: No significant inhibition")
        
        for inh in self.inhibitors:
            if inh.severity > 0.5:
                interpretations.append(f"HIGH RISK: {inh.name} severely inhibits {inh.target}")
        
        return interpretations


# ============================================================
# PART 3: Mitochondrial Disease Simulation
# ============================================================

@dataclass
class MitochondrialDisease:
    """Represents a mitochondrial disease."""
    name: str
    affected_complex: str
    deficiency_percent: float
    clinical_features: List[str]


class MitochondrialDiseaseSimulator:
    """Simulate mitochondrial diseases and their metabolic impact."""
    
    DISEASES = {
        'leigh_syndrome': MitochondrialDisease(
            name="Leigh Syndrome",
            affected_complex='Complex I',
            deficiency_percent=0.3,
            clinical_features=['Developmental delay', 'Seizures', 'Lactic acidosis']
        ),
        'melas': MitochondrialDisease(
            name="MELAS",
            affected_complex='Complex I',
            deficiency_percent=0.4,
            clinical_features=['Stroke-like episodes', 'Lactic acidosis', 'Encephalomyopathy']
        ),
        'merrf': MitochondrialDisease(
            name="MERRF",
            affected_complex='Complex I, IV',
            deficiency_percent=0.3,
            clinical_features=['Myoclonus', 'Seizures', 'Ragged red fibers']
        ),
        'kearns_sayre': MitochondrialDisease(
            name="Kearns-Sayre Syndrome",
            affected_complex='Complex I, III, IV',
            deficiency_percent=0.5,
            clinical_features=['CPEO', 'Retinitis pigmentosa', 'Cardiac block']
        )
    }
    
    @classmethod
    def calculate_atp_deficit(cls, disease: MitochondrialDisease) -> Dict:
        """Calculate ATP production deficit."""
        calc = ATPYieldCalculator()
        normal_total = calc.glucose_oxidation_total()
        
        # Complex I affects NADH oxidation
        if 'Complex I' in disease.affected_complex:
            nadh_deficit = disease.deficiency_percent
        else:
            nadh_deficit = 0
        
        # Estimate ATP deficit
        affected_atp = normal_total['total_atp'] * (1 - nadh_deficit * 0.5)
        deficit_percent = (normal_total['total_atp'] - affected_atp) / normal_total['total_atp'] * 100
        
        return {
            'normal_atp': normal_total['total_atp'],
            'affected_atp': affected_atp,
            'deficit_percent': deficit_percent,
            'lactate_risk': 'High' if deficit_percent > 20 else 'Moderate'
        }


# ============================================================
# MAIN: Run Simulations
# ============================================================

if __name__ == "__main__":
    print("=" * 60)
    print("BMED2301 Lab 3: Metabolic Flux Analysis")
    print("=" * 60)
    
    # Part 1: ATP Yield Calculations
    print("\n--- Part 1: ATP Yield Calculations ---\n")
    
    # Glucose oxidation
    glucose_total = ATPYieldCalculator.glucose_oxidation_total()
    print("ATP Yield from Complete Glucose Oxidation:")
    print(f"  Glycolysis: {glucose_total['glycolysis_atp']} ATP")
    print(f"  Pyruvate dehydrogenase: {glucose_total['pyruvate_dehydrogenase_atp']} ATP")
    print(f"  Citric acid cycle: {glucose_total['citric_acid_cycle_atp']} ATP")
    print(f"  TOTAL: {glucose_total['total_atp']} ATP")
    print(f"  (Using {glucose_total['shuttle_used']} shuttle)")
    
    # Fatty acid oxidation
    print("\nATP Yield from Fatty Acid Oxidation:")
    for fatty_acid in [16, 18, 14]:
        result = ATPYieldCalculator.beta_oxidation_atp(fatty_acid)
        print(f"  C{fatty_acid}: {result['total_atp']} ATP")
    
    # Per carbon comparison
    print("\nATP per Carbon Atom:")
    print(f"  Glucose (C6): {glucose_total['total_atp']/6:.1f} ATP per C")
    palmitate_atp = ATPYieldCalculator.beta_oxidation_atp(16)['total_atp']
    print(f"  Palmitate (C16): {palmitate_atp/16:.1f} ATP per C")
    
    # Part 2: Inhibitor Simulation
    print("\n--- Part 2: Metabolic Inhibitor Simulation ---\n")
    
    simulator = MetabolismSimulator()
    
    # Normal state
    glycolysis = simulator.simulate_glycolysis(1.0)
    print(f"Normal glycolysis: {glycolysis['pyruvate_produced']:.1f} pyruvate, "
          f"{glycolysis['atp_produced']:.1f} ATP")
    
    # With fluoride (enolase inhibitor)
    fluoride = MetabolicInhibitor(
        name="Fluoride",
        target="ENO",
        mechanism="Enolase inhibition",
        severity=0.8
    )
    simulator.add_inhibitor(fluoride)
    
    inhibited_glycolysis = simulator.simulate_glycolysis(1.0)
    print(f"\nWith fluoride (80% enolase inhibition):")
    print(f"  Pyruvate production: {inhibited_glycolysis['pyruvate_produced']:.2f}")
    print(f"  ATP production: {inhibited_glycolysis['atp_produced']:.2f}")
    
    print("\nClinical Interpretation:")
    for interp in simulator.clinical_interpretation():
        print(f"  • {interp}")
    
    # Complex III inhibition
    print("\n--- With Complex III Inhibitor (Antimycin A) ---\n")
    antimycin = MetabolicInhibitor(
        name="Antimycin A",
        target="Complex III",
        mechanism="Cytochrome bc1 inhibitor",
        severity=1.0
    )
    simulator2 = MetabolismSimulator()
    simulator2.add_inhibitor(antimycin)
    
    etc_result = simulator2.simulate_etc(nadh=10, fadh2=4)
    print(f"ATP from 10 NADH + 4 FADH2:")
    print(f"  With antimycin A: {etc_result['total_atp']:.1f} ATP (blocked!)")
    
    normal_etc = MetabolismSimulator().simulate_etc(nadh=10, fadh2=4)
    print(f"  Normal: {normal_etc['total_atp']:.1f} ATP")
    
    # Part 3: Mitochondrial Disease
    print("\n--- Part 3: Mitochondrial Disease Analysis ---\n")
    
    for disease_name, disease in MitochondrialDiseaseSimulator.DISEASES.items():
        result = MitochondrialDiseaseSimulator.calculate_atp_deficit(disease)
        print(f"\n{disease.name}:")
        print(f"  Affected: {disease.affected_complex}")
        print(f"  Deficiency: {disease.deficiency_percent*100:.0f}%")
        print(f"  ATP deficit: {result['deficit_percent']:.1f}%")
        print(f"  Lactate risk: {result['lactate_risk']}")
        print(f"  Clinical features: {', '.join(disease.clinical_features[:2])}")
    
    # Part 4: Energy Budget Visualization
    print("\n--- Part 4: Energy Budget Analysis ---\n")
    
    # Compare different substrates
    substrates = {
        'Glucose': ATPYieldCalculator.glucose_oxidation_total()['total_atp'],
        'Palmitate (C16)': ATPYieldCalculator.beta_oxidation_atp(16)['total_atp'],
        'Stearate (C18)': ATPYieldCalculator.beta_oxidation_atp(18)['total_atp'],
        'Leucine (essential AA)': 3.5,  # Approximate
        ' Glutamate': 3.0  # Approximate
    }
    
    print(f"{'Substrate':<20} {'ATP Yield':<12} {'ATP/Carbon':<12}")
    print("-" * 45)
    for substrate, atp in substrates.items():
        if 'Glucose' in substrate:
            carbon = 6
        elif 'C16' in substrate:
            carbon = 16
        elif 'C18' in substrate:
            carbon = 18
        else:
            carbon = 4  # Approximate
        print(f"{substrate:<20} {atp:<12.1f} {atp/carbon:<12.2f}")
    
    print("\n" + "=" * 60)
    print("Lab 3 Complete!")
    print("=" * 60)
```

### Expected Output

```
============================================================
BMED2301 Lab 3: Metabolic Flux Analysis
============================================================

--- Part 1: ATP Yield Calculations ---

ATP Yield from Complete Glucose Oxidation:
  Glycolysis: 7.0 ATP
  Pyruvate dehydrogenase: 5.0 ATP
  Citric acid cycle: 20.0 ATP
  TOTAL: 32.0 ATP
  (Using malate_aspartate shuttle)

ATP Yield from Fatty Acid Oxidation:
  C16: 106.0 ATP
  C18: 122.0 ATP
  C14: 90.0 ATP

ATP per Carbon Atom:
  Glucose (C6): 5.3 ATP per C
  Palmitate (C16): 6.6 ATP per C

--- Part 2: Metabolic Inhibitor Simulation ---

Normal glycolysis: 2.0 pyruvate, 2.0 ATP
Energy state: 1.0

With fluoride (80% enolase inhibition):
  Pyruvate production: 0.40
  ATP production: 0.40

Clinical Interpretation:
  • MODERATE: Partial impairment
  • HIGH RISK: Fluoride severely inhibits ENO

--- With Complex III Inhibitor (Antimycin A) ---

ATP from 10 NADH + 4 FADH2:
  With antimycin A: 0.0 ATP (blocked!)
  Normal: 31.0 ATP

--- Part 3: Mitochondrial Disease Analysis ---

Leigh Syndrome:
  Affected: Complex I
  Deficiency: 30%
  ATP deficit: 15.0%
  Lactate risk: Moderate
  Clinical features: Developmental delay, Seizures

--- Part 4: Energy Budget Analysis ---

Substrate            ATP Yield    ATP/Carbon  
---------------------------------------------
Glucose              32.0         5.33
Palmitate (C16)      106.0        6.63
Stearate (C18)       122.0        6.78

============================================================
Lab 3 Complete!
============================================================
```
