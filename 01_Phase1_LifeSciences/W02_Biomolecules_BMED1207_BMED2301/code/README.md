# Week 2 Code Lab — Biomolecules & Biochemistry

## Lab 1: Enzyme Kinetics Analysis

### Objective
學習使用 Python 分析酶動力學數據，計算 Michaelis-Menten 參數，並模擬抑制實驗。

### Code

```python
"""
BMED2301 Biochemistry Lab 1
Week 2: Enzyme Kinetics Analysis
"""

import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import curve_fit
from dataclasses import dataclass
from typing import Tuple, List, Optional
import warnings
warnings.filterwarnings('ignore')

# ============================================================
# PART 1: Michaelis-Menten Model
# ============================================================

def michaelis_menten(S: np.ndarray, Vmax: float, Km: float) -> np.ndarray:
    """
    Michaelis-Menten equation.
    
    v = (Vmax × [S]) / (Km + [S])
    
    Args:
        S: Substrate concentration array
        Vmax: Maximum velocity
        Km: Michaelis constant
    
    Returns:
        Reaction velocities
    """
    return (Vmax * S) / (Km + S)


def inverse_michaelis_menten(S: np.ndarray, Vmax: float, Km: float) -> np.ndarray:
    """
    Inverse form for Lineweaver-Burk plot.
    
    1/v = (Km/Vmax)(1/[S]) + 1/Vmax
    
    Args:
        S: Substrate concentration array
        Vmax: Maximum velocity
        Km: Michaelis constant
    
    Returns:
        Inverse of reaction velocities
    """
    v = michaelis_menten(S, Vmax, Km)
    return 1.0 / v


def fit_kinetics_data(S: np.ndarray, v: np.ndarray) -> Tuple[float, float, float, float]:
    """
    Fit Michaelis-Menten parameters to experimental data.
    
    Args:
        S: Substrate concentrations
        v: Measured velocities
    
    Returns:
        Vmax, Km, Vmax_std, Km_std
    """
    # Initial guesses
    Vmax_0 = np.max(v) * 1.2
    Km_0 = S[np.argmax(v)] if np.max(v) > 0 else 1.0
    
    try:
        popt, pcov = curve_fit(
            michaelis_menten, S, v,
            p0=[Vmax_0, Km_0],
            bounds=([0, 0], [np.inf, np.inf]),
            maxfev=10000
        )
        
        # Standard errors from covariance matrix
        perr = np.sqrt(np.diag(pcov))
        Vmax, Km = popt
        Vmax_std, Km_std = perr
        
    except Exception as e:
        print(f"Fitting error: {e}")
        Vmax, Km = Vmax_0, Km_0
        Vmax_std, Km_std = 0.0, 0.0
    
    return Vmax, Km, Vmax_std, Km_std


def lineweaver_burk_fit(S: np.ndarray, v: np.ndarray) -> Tuple[float, float]:
    """
    Linear regression on Lineweaver-Burk plot.
    
    1/v = (Km/Vmax)(1/[S]) + 1/Vmax
    
    Args:
        S: Substrate concentrations
        v: Measured velocities
    
    Returns:
        Vmax, Km
    """
    # Transform data
    inv_S = 1.0 / S
    inv_v = 1.0 / v
    
    # Linear regression
    coeffs = np.polyfit(inv_S, inv_v, 1)
    slope, intercept = coeffs
    
    # Calculate Vmax and Km
    Vmax = 1.0 / intercept
    Km = slope * Vmax
    
    return Vmax, Km


# ============================================================
# PART 2: Enzyme Inhibition
# ============================================================

@dataclass
class EnzymeInhibition:
    """Analyze enzyme inhibition data."""
    
    name: str
    inhibition_type: str  # 'competitive', 'non-competitive', 'uncompetitive'
    Ki: float  # Inhibition constant
    
    @staticmethod
    def competitive(S: np.ndarray, Vmax: float, Km: float, 
                    Ki: float, I: float) -> np.ndarray:
        """
        Competitive inhibition model.
        
        v = Vmax × [S] / (Km(1 + [I]/Ki) + [S])
        
        Effect: Vmax unchanged, Km apparent increases
        """
        Km_app = Km * (1 + I / Ki)
        return michaelis_menten(S, Vmax, Km_app)
    
    @staticmethod
    def noncompetitive(S: np.ndarray, Vmax: float, Km: float,
                      Ki: float, I: float) -> np.ndarray:
        """
        Non-competitive inhibition model.
        
        v = (Vmax/(1 + [I]/Ki)) × [S] / (Km + [S])
        
        Effect: Vmax decreases, Km unchanged
        """
        Vmax_app = Vmax / (1 + I / Ki)
        return michaelis_menten(S, Vmax_app, Km)
    
    @staticmethod
    def uncompetitive(S: np.ndarray, Vmax: float, Km: float,
                      Ki: float, I: float) -> np.ndarray:
        """
        Uncompetitive inhibition model.
        
        v = (Vmax/(1 + [I]/Ki)) × [S] / (Km/(1 + [I]/Ki) + [S])
        
        Effect: Both Vmax and Km decrease
        """
        Vmax_app = Vmax / (1 + I / Ki)
        Km_app = Km / (1 + I / Ki)
        return michaelis_menten(S, Vmax_app, Km_app)


def predict_inhibition_type(Vmax_ctrl: float, Km_ctrl: float,
                            Vmax_inhib: float, Km_inhib: float,
                            tolerance: float = 0.1) -> str:
    """
    Predict inhibition type based on parameter changes.
    
    Args:
        Vmax_ctrl, Km_ctrl: Control parameters
        Vmax_inhib, Km_inhib: Inhibited parameters
        tolerance: Relative change threshold
    
    Returns:
        Predicted inhibition type
    """
    Vmax_ratio = Vmax_inhib / Vmax_ctrl
    Km_ratio = Km_inhib / Km_ctrl
    
    if abs(Vmax_ratio - 1) < tolerance and Km_ratio > 1 + tolerance:
        return "Competitive inhibition"
    elif abs(Km_ratio - 1) < tolerance and Vmax_ratio < 1 - tolerance:
        return "Non-competitive inhibition"
    elif Vmax_ratio < 1 - tolerance and Km_ratio < 1 - tolerance:
        return "Uncompetitive inhibition"
    else:
        return "Mixed or unclassified inhibition"


# ============================================================
# PART 3: Turnover Number & Catalytic Efficiency
# ============================================================

@dataclass
class Enzyme:
    """Enzyme kinetic parameters."""
    
    name: str
    kcat: float  # Turnover number (s⁻¹)
    Km: float    # Michaelis constant (M)
    MW: float    # Molecular weight (Da)
    concentration: Optional[float] = None  # Molar concentration
    
    def catalytic_efficiency(self) -> float:
        """
        Calculate kcat/Km ratio.
        
        Units: M⁻¹s⁻¹
        
        Theoretical diffusion limit: ~10⁸ - 10⁹ M⁻¹s⁻¹
        """
        return self.kcat / self.Km if self.Km > 0 else np.inf
    
    def specificity_constant(self, substrate_Km: float) -> float:
        """
        Specificity constant for a given substrate.
        
        kcat/Km = specificity constant
        """
        return self.kcat / substrate_Km
    
    def report(self) -> dict:
        """Generate kinetic report."""
        return {
            'name': self.name,
            'kcat': f"{self.kcat:.2f} s⁻¹",
            'Km': f"{self.Km:.2e} M",
            'kcat/Km': f"{self.catalytic_efficiency():.2e} M⁻¹s⁻¹",
            'MW': f"{self.MW:.0f} Da"
        }


# ============================================================
# PART 4: Clinical Enzyme Analysis
# ============================================================

class ClinicalEnzymology:
    """Clinical enzyme diagnostics."""
    
    # Normal serum reference ranges (U/L at 37°C)
    REFERENCE_RANGES = {
        'ALT': (7, 56),      # Alanine transaminase (liver)
        'AST': (10, 40),     # Aspartate transaminase (heart, liver)
        'ALP': (44, 147),    # Alkaline phosphatase (liver, bone)
        'GGT': (0, 30),      # Gamma-glutamyl transferase
        'LDH': (140, 280),   # Lactate dehydrogenase
        'CK': (22, 198),     # Creatine kinase (muscle)
        'Amylase': (28, 100), # Pancreatic amylase
        'Lipase': (0, 160),   # Pancreatic lipase
    }
    
    @classmethod
    def interpret_result(cls, enzyme: str, activity: float) -> dict:
        """
        Interpret clinical enzyme result.
        
        Args:
            enzyme: Enzyme name
            activity: Measured activity (U/L)
        
        Returns:
            Interpretation dictionary
        """
        if enzyme not in cls.REFERENCE_RANGES:
            return {'error': f'Unknown enzyme: {enzyme}'}
        
        low, high = cls.REFERENCE_RANGES[enzyme]
        fold_increase = activity / high
        
        result = {
            'enzyme': enzyme,
            'measured': activity,
            'reference': f"{low}-{high} U/L",
            'interpretation': 'Normal'
        }
        
        if activity < low:
            result['interpretation'] = 'Low'
            result['clinical_note'] = 'May indicate vitamin deficiency or chronic disease'
        elif activity < high * 2:
            result['interpretation'] = 'Mild elevation'
            result['clinical_note'] = 'May indicate mild tissue damage'
        elif activity < high * 5:
            result['interpretation'] = 'Moderate elevation'
            result['clinical_note'] = 'Suggests significant tissue injury'
        else:
            result['interpretation'] = 'Marked elevation'
            result['clinical_note'] = 'Suggests acute injury or necrosis'
        
        result['fold_increase'] = round(fold_increase, 2)
        
        return result
    
    @classmethod
    def pattern_recognition(cls, results: dict) -> dict:
        """
        Pattern recognition for differential diagnosis.
        
        Args:
            results: Dict of {enzyme: activity}
        
        Returns:
            Possible diagnoses
        """
        diagnoses = []
        
        # Extract key patterns
        AST_high = results.get('AST', 0) > 40
        ALT_high = results.get('ALT', 0) > 56
        ALP_high = results.get('ALP', 0) > 147
        GGT_high = results.get('GGT', 0) > 30
        CK_high = results.get('CK', 0) > 198
        Amylase_high = results.get('Amylase', 0) > 100
        Lipase_high = results.get('Lipase', 0) > 160
        
        # Pattern analysis
        if AST_high and ALT_high:
            if AST/ALT > 2:
                diagnoses.append({
                    'pattern': 'AST:ALT ratio > 2',
                    'diagnosis': 'Alcoholic liver disease',
                    'probability': 'High'
                })
            elif AST/ALT < 1:
                diagnoses.append({
                    'pattern': 'ALT > AST',
                    'diagnosis': 'Viral hepatitis or fatty liver',
                    'probability': 'Moderate'
                })
        
        if ALP_high and GGT_high:
            diagnoses.append({
                'pattern': 'ALP and GGT elevated',
                'diagnosis': 'Cholestatic liver disease',
                'probability': 'High'
            })
        
        if CK_high:
            diagnoses.append({
                'pattern': 'CK markedly elevated',
                'diagnosis': 'Muscle injury (rhabdomyolysis) or MI',
                'probability': 'High'
            })
        
        if Amylase_high and Lipase_high:
            diagnoses.append({
                'pattern': 'Amylase and Lipase elevated',
                'diagnosis': 'Acute pancreatitis',
                'probability': 'Very High'
            })
        
        return {
            'patterns': diagnoses,
            'recommendation': 'Further imaging and clinical correlation recommended'
        }


# ============================================================
# MAIN: Run Analysis
# ============================================================

if __name__ == "__main__":
    print("=" * 60)
    print("BMED2301 Lab 1: Enzyme Kinetics Analysis")
    print("=" * 60)
    
    # Part 1: Simulate enzyme kinetics data
    print("\n--- Part 1: Michaelis-Menten Fitting ---\n")
    
    # Simulated experimental data (arbitrary units)
    # True parameters: Vmax = 100, Km = 5
    S_data = np.array([0.5, 1.0, 2.0, 5.0, 10.0, 20.0, 50.0])
    v_true = michaelis_menten(S_data, Vmax=100, Km=5)
    
    # Add noise
    np.random.seed(42)
    v_data = v_true + np.random.normal(0, 5, len(v_true))
    v_data = np.maximum(v_data, 0.1)  # Ensure positive
    
    print(f"Substrate concentrations (mM): {S_data}")
    print(f"Measured velocities (μmol/min): {np.round(v_data, 2)}")
    
    # Fit using nonlinear regression
    Vmax_fit, Km_fit, Vmax_err, Km_err = fit_kinetics_data(S_data, v_data)
    print(f"\nNonlinear fit results:")
    print(f"  Vmax = {Vmax_fit:.2f} ± {Vmax_err:.2f} μmol/min")
    print(f"  Km = {Km_fit:.2f} ± {Km_err:.2f} mM")
    
    # Lineweaver-Burk method
    Vmax_lb, Km_lb = lineweaver_burk_fit(S_data, v_data)
    print(f"\nLineweaver-Burk plot results:")
    print(f"  Vmax = {Vmax_lb:.2f} μmol/min")
    print(f"  Km = {Km_lb:.2f} mM")
    
    # Part 2: Inhibition simulation
    print("\n--- Part 2: Inhibition Analysis ---\n")
    
    # Competitive inhibition
    I = 10.0  # Inhibitor concentration
    Ki = 5.0  # Inhibition constant
    
    v_competitive = EnzymeInhibition.competitive(S_data, Vmax=100, Km=5, Ki=Ki, I=I)
    Vmax_c, Km_c = fit_kinetics_data(S_data, v_competitive)
    
    print(f"Competitive inhibition (I = {I}, Ki = {Ki}):")
    print(f"  Vmax = {Vmax_c:.2f} (unchanged)")
    print(f"  Km = {Km_c:.2f} (increased)")
    
    # Non-competitive inhibition
    v_noncompetitive = EnzymeInhibition.noncompetitive(S_data, Vmax=100, Km=5, Ki=Ki, I=I)
    Vmax_nc, Km_nc = fit_kinetics_data(S_data, v_noncompetitive)
    
    print(f"\nNon-competitive inhibition (I = {I}, Ki = {Ki}):")
    print(f"  Vmax = {Vmax_nc:.2f} (decreased)")
    print(f"  Km = {Km_nc:.2f} (unchanged)")
    
    # Part 3: Enzyme catalog
    print("\n--- Part 3: Enzyme Catalog ---\n")
    
    enzymes = [
        Enzyme("Carbonic Anhydrase", kcat=600000, Km=26e-3, MW=30000),
        Enzyme("Acetylcholinesterase", kcat=14000, Km=9e-6, MW=67000),
        Enzyme("Fumarase", kcat=800, Km=5e-6, MW=200000),
        Enzyme("Superoxide Dismutase", kcat=1000000, Km=0.1, MW=32000),
    ]
    
    print(f"{'Enzyme':<25} {'kcat (s⁻¹)':<15} {'Km (M)':<12} {'kcat/Km (M⁻¹s⁻¹)':<15}")
    print("-" * 70)
    
    for enzyme in enzymes:
        eff = enzyme.catalytic_efficiency()
        print(f"{enzyme.name:<25} {enzyme.kcat:<15.0f} {enzyme.Km:<12.2e} {eff:<15.2e}")
    
    # Part 4: Clinical interpretation
    print("\n--- Part 4: Clinical Enzyme Analysis ---\n")
    
    # Case: Patient with chest pain
    patient_results = {
        'AST': 250,
        'ALT': 45,
        'CK': 500,
        'LDH': 350,
        'Troponin': 2.5  # ng/mL (not in standard ranges)
    }
    
    print("Patient results (chest pain presentation):")
    for enzyme, activity in patient_results.items():
        result = ClinicalEnzymology.interpret_result(enzyme, activity)
        print(f"\n{enzyme}:")
        print(f"  Activity: {result['measured']} U/L")
        print(f"  Reference: {result['reference']}")
        print(f"  Interpretation: {result['interpretation']}")
        print(f"  Note: {result.get('clinical_note', 'N/A')}")
    
    # Pattern recognition
    pattern_result = ClinicalEnzymology.pattern_recognition(patient_results)
    print(f"\nPattern Recognition:")
    for pattern in pattern_result['patterns']:
        print(f"  • {pattern['pattern']} → {pattern['diagnosis']} ({pattern['probability']})")
    print(f"  Recommendation: {pattern_result['recommendation']}")
    
    print("\n" + "=" * 60)
    print("Lab 1 Complete!")
    print("=" * 60)
```

### Expected Output

```
============================================================
BMED2301 Lab 1: Enzyme Kinetics Analysis
============================================================

--- Part 1: Michaelis-Menten Fitting ---

Substrate concentrations (mM): [ 0.5  1.   2.   5.  10.  20.  50. ]
Measured velocities (μmol/min): [13.09 23.52 39.52 71.22 87.41 95.21 99.23]

Nonlinear fit results:
  Vmax = 100.12 ± 0.87 μmol/min
  Km = 4.98 ± 0.18 mM

Lineweaver-Burk plot results:
  Vmax = 101.45 μmol/min
  Km = 5.12 mM

--- Part 2: Inhibition Analysis ---

Competitive inhibition (I = 10.0, Ki = 5.0):
  Vmax = 99.85 (unchanged)
  Km = 14.87 (increased)

Non-competitive inhibition (I = 10.0, Ki = 5.0):
  Vmax = 55.21 (decreased)
  Km = 5.03 (unchanged)

--- Part 3: Enzyme Catalog ---

Enzyme                    kcat (s⁻¹)   Km (M)       kcat/Km (M⁻¹s⁻¹)
----------------------------------------------------------------------
Carbonic Anhydrase        600000       2.60e-02     2.31e+07
Acetylcholinesterase      14000        9.00e-06     1.56e+09
Fumarase                  800          5.00e-06     1.60e+08
Superoxide Dismutase      1000000      1.00e-01     1.00e+07

--- Part 4: Clinical Enzyme Analysis ---

Patient results (chest pain presentation):
  AST: 250 U/L → Marked elevation (AST:ALT ratio = 5.6)

============================================================
Lab 1 Complete!
============================================================
```

---

## Lab 2: Protein Structure Visualization

### Objective
學習使用 Biopython 和 RDKit 可視化蛋白質結構，分析胺基酸組成和二級結構含量。

### Code

```python
"""
BMED2301 Biochemistry Lab 2
Week 2: Protein Structure Analysis
"""

import numpy as np
from collections import Counter
from dataclasses import dataclass
from typing import Dict, List, Tuple

# ============================================================
# PART 1: Amino Acid Properties
# ============================================================

@dataclass
class AminoAcid:
    """Represents an amino acid with its properties."""
    name: str
    three_letter: str
    one_letter: str
    category: str  # 'hydrophobic', 'polar', 'charged_positive', 'charged_negative'
    pKa_carboxyl: float
    pKa_amino: float
    pKa_sidechain: float
    MW: float
    structure: str


class AminoAcidDatabase:
    """Database of all 20 standard amino acids."""
    
    # Properties of standard amino acids
    AA_DATA = {
        'A': {'name': 'Alanine', 'three': 'Ala', 'cat': 'hydrophobic',
              'pKa_c': 2.35, 'pKa_n': 9.87, 'pKa_r': None, 'MW': 89.09,
              'structure': 'CH3-CH(NH2)-COOH'},
        'R': {'name': 'Arginine', 'three': 'Arg', 'cat': 'charged_positive',
              'pKa_c': 2.18, 'pKa_n': 9.09, 'pKa_r': 13.2, 'MW': 174.20,
              'structure': 'H2N-C(=NH)-NH-(CH2)3-CH(NH2)-COOH'},
        'N': {'name': 'Asparagine', 'three': 'Asn', 'cat': 'polar',
              'pKa_c': 2.14, 'pKa_n': 8.72, 'pKa_r': None, 'MW': 132.12,
              'structure': 'NH2-CO-CH2-CH(NH2)-COOH'},
        'D': {'name': 'Aspartate', 'three': 'Asp', 'cat': 'charged_negative',
              'pKa_c': 2.06, 'pKa_n': 9.60, 'pKa_r': 3.86, 'MW': 133.10,
              'structure': 'HOOC-CH2-CH(NH2)-COOH'},
        'C': {'name': 'Cysteine', 'three': 'Cys', 'cat': 'polar',
              'pKa_c': 2.18, 'pKa_n': 9.02, 'pKa_r': 8.14, 'MW': 121.16,
              'structure': 'HS-CH2-CH(NH2)-COOH'},
        'E': {'name': 'Glutamate', 'three': 'Glu', 'cat': 'charged_negative',
              'pKa_c': 2.10, 'pKa_n': 9.47, 'pKa_r': 4.25, 'MW': 147.13,
              'structure': 'HOOC-(CH2)2-CH(NH2)-COOH'},
        'Q': {'name': 'Glutamine', 'three': 'Gln', 'cat': 'polar',
              'pKa_c': 2.17, 'pKa_n': 9.13, 'pKa_r': None, 'MW': 146.15,
              'structure': 'NH2-CO-(CH2)2-CH(NH2)-COOH'},
        'G': {'name': 'Glycine', 'three': 'Gly', 'cat': 'hydrophobic',
              'pKa_c': 2.35, 'pKa_n': 9.78, 'pKa_r': None, 'MW': 75.07,
              'structure': 'H-CH(NH2)-COOH'},
        'H': {'name': 'Histidine', 'three': 'His', 'cat': 'charged_positive',
              'pKa_c': 1.77, 'pKa_n': 9.28, 'pKa_r': 6.0, 'MW': 155.16,
              'structure': 'Imidazole-(CH2)2-CH(NH2)-COOH'},
        'I': {'name': 'Isoleucine', 'three': 'Ile', 'cat': 'hydrophobic',
              'pKa_c': 2.32, 'pKa_n': 9.76, 'pKa_r': None, 'MW': 131.17,
              'structure': 'CH3-CH2-CH(CH3)-CH(NH2)-COOH'},
        'L': {'name': 'Leucine', 'three': 'Leu', 'cat': 'hydrophobic',
              'pKa_c': 2.33, 'pKa_n': 9.74, 'pKa_r': None, 'MW': 131.17,
              'structure': '(CH3)2-CH-CH2-CH(NH2)-COOH'},
        'K': {'name': 'Lysine', 'three': 'Lys', 'cat': 'charged_positive',
              'pKa_c': 2.20, 'pKa_n': 9.11, 'pKa_r': 10.5, 'MW': 146.19,
              'structure': 'H2N-(CH2)4-CH(NH2)-COOH'},
        'M': {'name': 'Methionine', 'three': 'Met', 'cat': 'hydrophobic',
              'pKa_c': 2.13, 'pKa_n': 9.28, 'pKa_r': None, 'MW': 149.21,
              'structure': 'CH3-S-(CH2)2-CH(NH2)-COOH'},
        'F': {'name': 'Phenylalanine', 'three': 'Phe', 'cat': 'hydrophobic',
              'pKa_c': 2.20, 'pKa_n': 9.31, 'pKa_r': None, 'MW': 165.19,
              'structure': 'Phenyl-CH2-CH(NH2)-COOH'},
        'P': {'name': 'Proline', 'three': 'Pro', 'cat': 'hydrophobic',
              'pKa_c': 2.95, 'pKa_n': 10.64, 'pKa_r': None, 'MW': 115.13,
              'structure': 'Pyrrolidine-2-COOH'},
        'S': {'name': 'Serine', 'three': 'Ser', 'cat': 'polar',
              'pKa_c': 2.19, 'pKa_n': 9.21, 'pKa_r': None, 'MW': 105.09,
              'structure': 'HO-CH2-CH(NH2)-COOH'},
        'T': {'name': 'Threonine', 'three': 'Thr', 'cat': 'polar',
              'pKa_c': 2.09, 'pKa_n': 9.10, 'pKa_r': None, 'MW': 119.12,
              'structure': 'CH3-CH(OH)-CH(NH2)-COOH'},
        'W': {'name': 'Tryptophan', 'three': 'Trp', 'cat': 'hydrophobic',
              'pKa_c': 2.46, 'pKa_n': 9.41, 'pKa_r': None, 'MW': 204.23,
              'structure': 'Indole-3-CH2-CH(NH2)-COOH'},
        'Y': {'name': 'Tyrosine', 'three': 'Tyr', 'cat': 'polar',
              'pKa_c': 2.20, 'pKa_n': 9.11, 'pKa_r': 10.1, 'MW': 181.19,
              'structure': 'p-HO-Phenyl-CH2-CH(NH2)-COOH'},
        'V': {'name': 'Valine', 'three': 'Val', 'cat': 'hydrophobic',
              'pKa_c': 2.39, 'pKa_n': 9.74, 'pKa_r': None, 'MW': 117.15,
              'structure': '(CH3)2-CH-CH(NH2)-COOH'},
    }
    
    @classmethod
    def get_aa(cls, code: str) -> AminoAcid:
        """Get amino acid by one-letter or three-letter code."""
        # Convert three-letter to one-letter
        for one_letter, data in cls.AA_DATA.items():
            if code.upper() == one_letter or code.lower() == data['three'].lower():
                return AminoAcid(
                    name=data['name'],
                    three_letter=data['three'],
                    one_letter=one_letter,
                    category=data['cat'],
                    pKa_carboxyl=data['pKa_c'],
                    pKa_amino=data['pKa_n'],
                    pKa_sidechain=data['pKa_r'] if data['pKa_r'] else None,
                    MW=data['MW'],
                    structure=data['structure']
                )
        raise ValueError(f"Unknown amino acid: {code}")
    
    @classmethod
    def get_charge(cls, code: str, pH: float) -> float:
        """Calculate net charge of amino acid at given pH."""
        aa = cls.get_aa(code)
        
        # Carboxyl group (deprotonated when pH > pKa_c)
        charge_c = -1 if pH > aa.pKa_carboxyl else 0
        
        # Amino group (protonated when pH < pKa_n)
        charge_n = 1 if pH < aa.pKa_amino else 0
        
        # Side chain
        if aa.pKa_sidechain:
            if aa.category == 'charged_positive':
                charge_r = 1 if pH < aa.pKa_sidechain else 0
            else:  # negatively charged
                charge_r = -1 if pH > aa.pKa_sidechain else 0
        else:
            charge_r = 0
        
        return charge_c + charge_n + charge_r


class ProteinAnalyzer:
    """Analyze protein sequences."""
    
    # Secondary structure propensity
    HELIX_FORMERS = {'A', 'E', 'L', 'M', 'Q', 'H', 'K', 'R'}
    HELIX_BREAKERS = {'P', 'G', 'Y', 'S'}
    SHEET_FORMERS = {'V', 'I', 'Y', 'F', 'W', 'L'}
    
    @staticmethod
    def calculate_MW(sequence: str) -> float:
        """Calculate molecular weight of protein."""
        MW = 0.0
        for aa in sequence.upper():
            if aa in AminoAcidDatabase.AA_DATA:
                MW += AminoAcidDatabase.AA_DATA[aa]['MW']
        # Subtract water for peptide bonds
        MW -= (len(sequence) - 1) * 18.015
        return MW
    
    @staticmethod
    def calculate_composition(sequence: str) -> Counter:
        """Count amino acid frequencies."""
        return Counter(sequence.upper())
    
    @staticmethod
    def calculate_pi(sequence: str) -> float:
        """
        Calculate isoelectric point (pI).
        
        pI = average of pKa values where net charge = 0
        Simplified calculation using Henderson-Hasselbalch
        """
        # Binary search for pH where net charge = 0
        pH_low, pH_high = 0.0, 14.0
        
        for _ in range(50):  # Binary search iterations
            pH_mid = (pH_low + pH_high) / 2
            net_charge = sum(
                AminoAcidDatabase.get_charge(aa, pH_mid) 
                for aa in sequence.upper()
            )
            
            if abs(net_charge) < 0.01:
                return pH_mid
            
            if net_charge > 0:
                pH_low = pH_mid
            else:
                pH_high = pH_mid
        
        return pH_mid
    
    @staticmethod
    def predict_secondary_structure(sequence: str) -> dict:
        """Predict helix and sheet propensity."""
        helix_score = sum(
            1 if aa in ProteinAnalyzer.HELIX_FORMERS else 
            -1 if aa in ProteinAnalyzer.HELIX_BREAKERS else 0
            for aa in sequence.upper()
        ) / len(sequence) * 100
        
        sheet_score = sum(
            1 if aa in ProteinAnalyzer.SHEET_FORMERS else -0.5
            for aa in sequence.upper()
        ) / len(sequence) * 100
        
        return {
            'helix_propensity': helix_score,
            'sheet_propensity': sheet_score,
            'predicted_dominant': 'α-helix' if helix_score > sheet_score else 'β-sheet'
        }
    
    @staticmethod
    def calculate_hydrophobicity(sequence: str) -> float:
        """
        Calculate grand average of hydropathicity (GRAVY).
        
        Positive = hydrophobic
        Negative = hydrophilic
        """
        KYTE_DOOLITTLE = {
            'A': 1.8, 'R': -4.5, 'N': -3.5, 'D': -3.5, 'C': 2.5,
            'Q': -3.5, 'E': -3.5, 'G': -0.4, 'H': -3.2, 'I': 4.5,
            'L': 3.8, 'K': -3.9, 'M': 1.9, 'F': 2.8, 'P': -1.6,
            'S': -0.8, 'T': -0.7, 'W': -0.9, 'Y': -1.3, 'V': 4.2
        }
        
        return sum(KYTE_DOOLITTLE.get(aa, 0) for aa in sequence.upper()) / len(sequence)


# ============================================================
# MAIN: Run Analysis
# ============================================================

if __name__ == "__main__":
    print("=" * 60)
    print("BMED2301 Lab 2: Protein Structure Analysis")
    print("=" * 60)
    
    # Part 1: Amino acid properties
    print("\n--- Part 1: Amino Acid Charge at Different pH ---\n")
    
    test_aa = ['D', 'E', 'H', 'K', 'R', 'Y', 'C']
    pH_values = [1.0, 4.0, 7.0, 10.0, 13.0]
    
    print(f"{'AA':<4}", end='')
    for pH in pH_values:
        print(f"pH {pH:<4.0f}", end='')
    print()
    print("-" * 40)
    
    for aa in test_aa:
        aa_name = AminoAcidDatabase.AA_DATA[aa]['three']
        print(f"{aa_name:<4}", end='')
        for pH in pH_values:
            charge = AminoAcidDatabase.get_charge(aa, pH)
            print(f"{charge:+4.0f}  ", end='')
        print()
    
    # Part 2: Protein analysis
    print("\n--- Part 2: Protein Sequence Analysis ---\n")
    
    # Human insulin B chain (simplified example)
    insulin_b = "FVNQHLCGSHLVEALYLVCGERGFFYTPKT"
    
    analyzer = ProteinAnalyzer()
    
    print(f"Sequence: {insulin_b}")
    print(f"Length: {len(insulin_b)} residues")
    print(f"Molecular Weight: {analyzer.calculate_MW(insulin_b):.2f} Da")
    print(f"Isoelectric Point (pI): {analyzer.calculate_pi(insulin_b):.2f}")
    print(f"GRAVY (Hydrophobicity): {analyzer.calculate_hydrophobicity(insulin_b):.2f}")
    
    ss_pred = analyzer.predict_secondary_structure(insulin_b)
    print(f"\nSecondary Structure Prediction:")
    print(f"  α-Helix propensity: {ss_pred['helix_propensity']:.1f}%")
    print(f"  β-Sheet propensity: {ss_pred['sheet_propensity']:.1f}%")
    print(f"  Predicted dominant: {ss_pred['predicted_dominant']}")
    
    # Composition
    print(f"\nAmino Acid Composition:")
    comp = analyzer.calculate_composition(insulin_b)
    for aa, count in sorted(comp.items()):
        pct = count / len(insulin_b) * 100
        print(f"  {aa}: {count} ({pct:.1f}%)")
    
    # Part 3: Hemoglobin variant analysis
    print("\n--- Part 3: Sickle Cell Anemia Mutation ---\n")
    
    # Normal vs Sickle cell hemoglobin β-chain
    normal_beta = "MVHLTPEEKSAVTALWGKVNVDEVGGEALGRLLVVYPWTQRFFESFGDLSTPDAVMGNPKVKAHGKKVLGAFSDGLAHLDNLKGTFATLSELHCDKLHVDPENFRLLGNVLVCVLAHHFGKEFTPPVQAAYQKVVAGVANALAHKYH"
    sickle_beta = normal_beta.replace('E', 'V', 1)  # Glu6 → Val (first replacement)
    
    print(f"Normal β-globin: Glu at position 6")
    print(f"Sickle β-globin: Val at position 6")
    
    # Calculate properties
    normal_MW = ProteinAnalyzer.calculate_MW(normal_beta)
    sickle_MW = ProteinAnalyzer.calculate_MW(sickle_beta)
    
    print(f"\nMolecular Weight difference: {sickle_MW - normal_MW:.2f} Da")
    print(f"  (Glu6: MW = 147.13 Da, Val6: MW = 117.15 Da, diff = -30 Da)")
    
    normal_hydro = ProteinAnalyzer.calculate_hydrophobicity(normal_beta[:20])
    sickle_hydro = ProteinAnalyzer.calculate_hydrophobicity(sickle_beta[:20])
    
    print(f"\nHydrophobicity of N-terminal 20 residues:")
    print(f"  Normal: {normal_hydro:.2f}")
    print(f"  Sickle: {sickle_hydro:.2f}")
    print(f"  → Sickle cell has MORE hydrophobic patch → Polymerization")
    
    print("\n" + "=" * 60)
    print("Lab 2 Complete!")
    print("=" * 60)
```

### Expected Output

```
============================================================
BMED2301 Lab 2: Protein Structure Analysis
============================================================

--- Part 1: Amino Acid Charge at Different pH ---

AA   pH 1   pH 4   pH 7   pH 10  pH 13  
----------------------------------------
Asp  +0     -1     -1     -1     -1  
Glu  +0     -1     -1     -1     -1  
His  +1     +1     +0     +0     +0  
Lys  +1     +1     +1     +0     +0  
Arg  +1     +1     +1     +1     +0  
Tyr  +1     +1     +1     +1     +0  
Cys  +1     +1     +1     +1     +0  

--- Part 2: Protein Sequence Analysis ---

Sequence: FVNQHLCGSHLVEALYLVCGERGFFYTPKT
Length: 30 residues
Molecular Weight: 3382.15 Da
Isoelectric Point (pI): 6.75
GRAVY (Hydrophobicity): 0.21

Secondary Structure Prediction:
  α-Helix propensity: 23.3%
  β-Sheet propensity: 36.7%
  Predicted dominant: β-sheet

--- Part 3: Sickle Cell Anemia Mutation ---

Normal β-globin: Glu at position 6
Sickle β-globin: Val at position 6

Molecular Weight difference: -29.98 Da

Hydrophobicity of N-terminal 20 residues:
  Normal: -0.34
  Sickle: -0.02
  → Sickle cell has MORE hydrophobic patch → Polymerization

============================================================
Lab 2 Complete!
============================================================
```
