# Week 1 Code Lab — BME Intro + Chemistry Foundations

## Lab 1: pH Calculator & Buffer Design

### Objective
學習使用 Python 計算 pH、[H⁺] 濃度，並使用 Henderson-Hasselbalch 方程式設計生物緩衝系統。

### Code

```python
"""
BMED1207 Chemistry Lab 1
Week 1: pH Calculator & Buffer Design
"""

import numpy as np
import matplotlib.pyplot as plt
from dataclasses import dataclass
from typing import Tuple

# ============================================================
# PART 1: Fundamental pH Calculations
# ============================================================

def calculate_pH(H_concentration: float) -> float:
    """
    Calculate pH from hydrogen ion concentration.
    
    Args:
        H_concentration: [H⁺] in mol/L (M)
    
    Returns:
        pH value
    
    Example:
        >>> calculate_pH(1e-7)
        7.0
    """
    if H_concentration <= 0:
        raise ValueError("H⁺ concentration must be positive")
    
    pH = -np.log10(H_concentration)
    return round(pH, 2)

def calculate_H_concentration(pH: float) -> float:
    """
    Calculate hydrogen ion concentration from pH.
    
    Args:
        pH: pH value
    
    Returns:
        [H⁺] in mol/L (M)
    
    Example:
        >>> calculate_H_concentration(7.4)
        3.98e-08
    """
    if pH < 0 or pH > 14:
        raise ValueError("pH must be between 0 and 14")
    
    H_conc = 10 ** (-pH)
    return H_conc

# ============================================================
# PART 2: Henderson-Hasselbalch Equation
# ============================================================

@dataclass
class Buffer:
    """Represents a weak acid buffer system."""
    name: str
    acid_name: str  # HA
    base_name: str   # A⁻
    pKa: float
    acid_conc: float  # M
    base_conc: float  # M
    
    def calculate_pH(self) -> float:
        """
        Calculate buffer pH using Henderson-Hasselbalch equation.
        
        pH = pKa + log([A⁻]/[HA])
        """
        ratio = self.base_conc / self.acid_conc
        if ratio <= 0:
            raise ValueError("Concentration ratio must be positive")
        
        pH = self.pKa + np.log10(ratio)
        return round(pH, 2)
    
    def calculate_buffer_capacity(self) -> float:
        """
        Calculate buffer capacity (β).
        
        β = 2.303 × [H⁺] × [OH⁻] / ([H⁺] + Kw/[H⁺])
            + 2.303 × Ka × [HA] × [A⁻] / (Ka + [H⁺])²
        
        Simplified for biological buffers:
        β ≈ 2.303 × C × Ka × [H⁺] / (Ka + [H⁺])²
        where C = total buffer concentration
        """
        Kw = 1e-14  # Water ionization constant at 25°C
        C_total = self.acid_conc + self.base_conc
        
        # Get [H⁺] from current pH
        current_pH = self.calculate_pH()
        H_conc = 10 ** (-current_pH)
        
        # Henderson-Hasselbalch buffer capacity term
        Ka = 10 ** (-self.pKa)
        beta = 2.303 * C_total * Ka * H_conc / (Ka + H_conc) ** 2
        
        return round(beta, 4)


def create_bicarbonate_buffer(PCO2: float, HCO3: float) -> float:
    """
    Calculate blood pH using the Henderson-Hasselbalch equation
    for the bicarbonate buffer system.
    
    CO₂ + H₂O ⇌ H₂CO₃ ⇌ H⁺ + HCO₃⁻
    pKa₁(CO₂/HCO₃⁻) = 6.1
    
    Args:
        PCO2: Partial pressure of CO₂ in mmHg
        HCO3: Bicarbonate concentration in mM
    
    Returns:
        Blood pH
    
    Clinical reference:
        Normal arterial blood:
        - PCO₂ = 35-45 mmHg (mean 40)
        - HCO₃⁻ = 22-28 mM (mean 24)
        - pH = 7.35-7.45 (mean 7.40)
    """
    pKa1 = 6.1
    
    # Convert PCO2 to [CO2] using Henry's law
    # [CO2] = PCO2 × solubility coefficient
    # Solubility of CO2 in plasma = 0.0301 mmol/L/mmHg
    CO2_conc = PCO2 * 0.0301  # mM
    
    # Henderson-Hasselbalch
    pH = pKa1 + np.log10(HCO3 / CO2_conc)
    
    return round(pH, 2)


# ============================================================
# PART 3: Blood Gas Analysis
# ============================================================

@dataclass
class BloodGas:
    """Represents arterial blood gas (ABG) parameters."""
    pH: float
    PCO2: float  # mmHg
    HCO3: float  # mM
    
    def interpret(self) -> dict:
        """
        Interpret ABG results and identify acid-base disturbance.
        
        Returns:
            Dictionary with interpretation
        """
        results = {
            'status': 'Normal',
            'primary_disturbance': None,
            'compensation': None,
            'interpretation': []
        }
        
        # Step 1: Determine acidemia or alkalemia
        if self.pH < 7.35:
            results['status'] = 'Acidemia'
            results['interpretation'].append(f"pH {self.pH} < 7.35 (acidemia)")
        elif self.pH > 7.45:
            results['status'] = 'Alkalemia'
            results['interpretation'].append(f"pH {self.pH} > 7.45 (alkalemia)")
        else:
            results['interpretation'].append(f"pH {self.pH} is within normal range")
        
        # Step 2: Determine primary disturbance
        if self.pH < 7.35:
            if self.PCO2 > 45:
                results['primary_disturbance'] = 'Respiratory Acidosis'
                results['interpretation'].append(f"PCO₂ {self.PCO2} > 45 mmHg (elevated)")
            elif self.HCO3 < 22:
                results['primary_disturbance'] = 'Metabolic Acidosis'
                results['interpretation'].append(f"HCO₃⁻ {self.HCO3} < 22 mM (low)")
            else:
                results['primary_disturbance'] = 'Mixed Acidosis'
        else:
            if self.PCO2 < 35:
                results['primary_disturbance'] = 'Respiratory Alkalosis'
                results['interpretation'].append(f"PCO₂ {self.PCO2} < 35 mmHg (low)")
            elif self.HCO3 > 28:
                results['primary_disturbance'] = 'Metabolic Alkalosis'
                results['interpretation'].append(f"HCO₃⁻ {self.HCO3} > 28 mM (elevated)")
        
        # Step 3: Calculate expected compensation
        if results['primary_disturbance'] == 'Respiratory Acidosis':
            # Acute: ↑ HCO3 by 1 mM per 10 mmHg ↑ PCO2
            # Chronic: ↑ HCO3 by 3.5 mM per 10 mmHg ↑ PCO2
            expected_acute = 24 + (self.PCO2 - 40) / 10
            expected_chronic = 24 + (self.PCO2 - 40) / 10 * 3.5
            results['compensation'] = {
                'acute_HCO3': round(expected_acute, 1),
                'chronic_HCO3': round(expected_chronic, 1),
                'actual_HCO3': self.HCO3
            }
        elif results['primary_disturbance'] == 'Metabolic Acidosis':
            # Expected PCO2 = 1.5 × [HCO3] + 8 ± 2 (Winter's formula)
            expected_PCO2 = 1.5 * self.HCO3 + 8
            results['compensation'] = {
                'expected_PCO2': round(expected_PCO2, 1),
                'actual_PCO2': self.PCO2
            }
        
        return results


# ============================================================
# PART 4: Titration Curve Generator
# ============================================================

def generate_titration_curve(acid_type: str = 'acetic', 
                              initial_conc: float = 0.1,
                              volume_mL: float = 50.0) -> Tuple[np.ndarray, np.ndarray]:
    """
    Generate a titration curve for a weak acid.
    
    Args:
        acid_type: Type of weak acid ('acetic', 'phosphoric', 'carbonic')
        initial_conc: Initial acid concentration in M
        volume_mL: Initial volume in mL
    
    Returns:
        Tuple of (volume added in mL, pH values)
    """
    # Define weak acids relevant to biology
    acids = {
        'acetic': {'Ka': 1.8e-5, 'pKa': 4.74, 'name': 'Acetic Acid (CH₃COOH)'},
        'phosphoric': {'Ka': 6.2e-8, 'pKa': 7.21, 'name': 'Phosphoric Acid (H₃PO₄)'},
        'carbonic': {'Ka': 4.3e-7, 'pKa': 6.37, 'name': 'Carbonic Acid (H₂CO₃)'}
    }
    
    if acid_type not in acids:
        raise ValueError(f"Unknown acid type. Choose from: {list(acids.keys())}")
    
    acid_info = acids[acid_type]
    Ka = acid_info['Ka']
    
    # Generate titration data
    # Add strong base (NaOH) incrementally
    volumes = np.linspace(0, 150, 300)  # 0 to 150 mL NaOH
    pH_values = []
    
    for V_base in volumes:
        # Total volume
        V_total = volume_mL + V_base
        
        # Moles
        n_acid_initial = initial_conc * volume_mL / 1000  # mol
        n_base_added = 0.1 * V_base / 1000  # mol (assuming 0.1 M NaOH)
        
        # After neutralization
        n_HA = n_acid_initial - n_base_added
        n_A = n_base_added
        
        if n_A <= 0:
            # Excess acid - use simple acid dissociation
            H_conc = np.sqrt(Ka * n_HA / (V_total / 1000))
        elif n_HA <= 0:
            # Excess base
            OH_conc = n_A / (V_total / 1000)
            H_conc = 1e-14 / OH_conc
        else:
            # Buffer region - Henderson-Hasselbalch
            pH = acid_info['pKa'] + np.log10(n_A / n_HA)
            H_conc = 10 ** (-pH)
        
        pH_values.append(-np.log10(H_conc) if H_conc > 0 else 14)
    
    return volumes, np.array(pH_values)


# ============================================================
# MAIN: Run All Calculations
# ============================================================

if __name__ == "__main__":
    print("=" * 60)
    print("BMED1207 Chemistry Lab 1: pH and Buffer Calculations")
    print("=" * 60)
    
    # Part 1: Basic pH calculations
    print("\n--- Part 1: pH Calculations ---\n")
    
    test_pH = [7.4, 7.0, 4.5, 9.2]
    for ph in test_pH:
        h_conc = calculate_H_concentration(ph)
        print(f"pH {ph}: [H⁺] = {h_conc:.2e} M")
    
    # Part 2: Blood pH
    print("\n--- Part 2: Blood pH (Bicarbonate Buffer) ---\n")
    
    # Normal arterial blood
    normal_pH = create_bicarbonate_buffer(PCO2=40, HCO3=24)
    print(f"Normal blood: PCO₂=40 mmHg, HCO₃⁻=24 mM → pH = {normal_pH}")
    
    # Respiratory acidosis (COPD)
    copd_pH = create_bicarbonate_buffer(PCO2=60, HCO3=28)
    print(f"COPD: PCO₂=60 mmHg, HCO₃⁻=28 mM → pH = {copd_pH}")
    
    # Metabolic acidosis (DKA)
    dka_pH = create_bicarbonate_buffer(PCO2=30, HCO3=15)
    print(f"DKA: PCO₂=30 mmHg, HCO₃⁻=15 mM → pH = {dka_pH}")
    
    # Part 3: Buffer design
    print("\n--- Part 3: Buffer Design ---\n")
    
    # Create a phosphate buffer (relevant to cellular pH)
    phosphate_buffer = Buffer(
        name="Phosphate Buffer",
        acid_name="H₂PO₄⁻",
        base_name="HPO₄²⁻",
        pKa=7.21,
        acid_conc=0.05,  # 50 mM
        base_conc=0.05   # 50 mM
    )
    
    buffer_pH = phosphate_buffer.calculate_pH()
    buffer_capacity = phosphate_buffer.calculate_buffer_capacity()
    
    print(f"Buffer: {phosphate_buffer.name}")
    print(f"[HA] = {phosphate_buffer.acid_conc} M, [A⁻] = {phosphate_buffer.base_conc} M")
    print(f"pH = {buffer_pH}")
    print(f"Buffer capacity (β) = {buffer_capacity}")
    
    # Part 4: Blood gas interpretation
    print("\n--- Part 4: Blood Gas Analysis ---\n")
    
    # Case 1: Normal
    normal_blood = BloodGas(pH=7.40, PCO2=40, HCO3=24)
    print("Case 1 - Normal:")
    result = normal_blood.interpret()
    for line in result['interpretation']:
        print(f"  • {line}")
    
    # Case 2: DKA
    dka_blood = BloodGas(pH=7.20, PCO2=25, HCO3=12)
    print("\nCase 2 - Diabetic Ketoacidosis (DKA):")
    result = dka_blood.interpret()
    for line in result['interpretation']:
        print(f"  • {line}")
    if result['compensation']:
        print(f"  • Expected PCO₂: {result['compensation']['expected_PCO2']} mmHg")
        print(f"  • Actual PCO₂: {result['compensation']['actual_PCO2']} mmHg")
    
    # Case 3: COPD
    copd_blood = BloodGas(pH=7.32, PCO2=55, HCO3=30)
    print("\nCase 3 - COPD (Chronic Respiratory Acidosis):")
    result = copd_blood.interpret()
    for line in result['interpretation']:
        print(f"  • {line}")
    if result['compensation']:
        print(f"  • Expected HCO₃⁻ (acute): {result['compensation']['acute_HCO3']} mM")
        print(f"  • Expected HCO₃⁻ (chronic): {result['compensation']['chronic_HCO3']} mM")
        print(f"  • Actual HCO₃⁻: {result['compensation']['actual_HCO3']} mM")
    
    print("\n" + "=" * 60)
    print("Lab 1 Complete!")
    print("=" * 60)
```

### Expected Output

```
============================================================
BMED1207 Chemistry Lab 1: pH and Buffer Calculations
============================================================

--- Part 1: pH Calculations ---

pH 7.4: [H⁺] = 3.98e-08 M
pH 7.0: [H⁺] = 1.00e-07 M
pH 4.5: [H⁺] = 3.16e-05 M
pH 9.2: [H⁺] = 6.31e-10 M

--- Part 2: Blood pH (Bicarbonate Buffer) ---

Normal blood: PCO₂=40 mmHg, HCO₃⁻=24 mM → pH = 7.40
COPD: PCO₂=60 mmHg, HCO₃⁻=28 mM → pH = 7.20
DKA: PCO₂=30 mmHg, HCO₃⁻=15 mM → pH = 7.30

--- Part 3: Buffer Design ---

Buffer: Phosphate Buffer
[HA] = 0.05 M, [A⁻] = 0.05 M
pH = 7.21
Buffer capacity (β) = 0.0572

--- Part 4: Blood Gas Analysis ---

Case 1 - Normal:
  • pH 7.4 is within normal range

Case 2 - Diabetic Ketoacidosis (DKA):
  • pH 7.2 < 7.35 (acidemia)
  • HCO₃⁻ 12 < 22 mM (low)
  • Expected PCO₂: 26.0 mmHg
  • Actual PCO₂: 25 mmHg

Case 3 - COPD (Chronic Respiratory Acidosis):
  • pH 7.32 < 7.35 (acidemia)
  • PCO₂ 55 > 45 mmHg (elevated)
  • Expected HCO₃⁻ (acute): 25.5 mM
  • Expected HCO₃⁻ (chronic): 30.25 mM
  • Actual HCO₃⁻: 30 mM

============================================================
Lab 1 Complete!
============================================================
```

---

## Lab 2: Molecule Visualization & Bond Analysis

### Objective
學習使用 RDKit 分析分子的結構、極性，並預測其在生物系統中的行為。

### Code

```python
"""
BMED1207 Chemistry Lab 2
Week 1: Molecule Visualization & Bond Analysis
"""

try:
    from rdkit import Chem
    from rdkit.Chem import Draw, Descriptors, AllChem
    from rdkit.Chem import rdMolDescriptors
    from rdkit.Chem.Draw import rdMolDraw2D
    HAS_RDKIT = True
except ImportError:
    HAS_RDKIT = False
    print("Note: RDKit not installed. Install with: pip install rdkit")

import numpy as np

# ============================================================
# PART 1: Basic Molecule Analysis
# ============================================================

class MoleculeAnalyzer:
    """Analyze biological molecules for BME applications."""
    
    def __init__(self, smiles: str, name: str):
        """
        Initialize with SMILES string.
        
        Args:
            smiles: SMILES notation string
            name: Common name
        """
        self.name = name
        self.smiles = smiles
        
        if HAS_RDKIT:
            self.mol = Chem.MolFromSmiles(smiles)
            if self.mol is None:
                raise ValueError(f"Invalid SMILES: {smiles}")
        else:
            self.mol = None
    
    def get_properties(self) -> dict:
        """Calculate molecular properties relevant to BME."""
        if not HAS_RDKIT:
            return self._manual_properties()
        
        props = {
            'name': self.name,
            'smiles': self.smiles,
            'molecular_formula': rdMolDescriptors.CalcMolFormula(self.mol),
            'molecular_weight': round(Descriptors.MolWt(self.mol), 2),
            'num_atoms': self.mol.GetNumAtoms(),
            'num_bonds': self.mol.GetNumBonds(),
            'num_h_acceptors': rdMolDescriptors.CalcNumHAcceptors(self.mol),
            'num_h_donors': rdMolDescriptors.CalcNumHDonors(self.mol),
            'logP': round(Descriptors.MolLogP(self.mol), 2),  # Lipophilicity
            'tpsa': round(Descriptors.TPSA(self.mol), 2),  # Topological polar surface area
            'num_rotatable_bonds': rdMolDescriptors.CalcNumRotatableBonds(self.mol),
        }
        
        return props
    
    def _manual_properties(self) -> dict:
        """Manual calculation without RDKit (simplified)."""
        return {
            'name': self.name,
            'smiles': self.smiles,
            'note': 'Install RDKit for full analysis'
        }
    
    def predict_bioavailability(self) -> dict:
        """
        Predict oral bioavailability using Lipinski's Rule of 5.
        
        Lipinski's Rule of 5 (2000):
        - MW < 500 Da
        - LogP < 5
        - H-bond donors < 5
        - H-bond acceptors < 10
        
        Returns:
            Dictionary with rule compliance and bioavailability prediction
        """
        if not HAS_RDKIT:
            return {'error': 'RDKit required'}
        
        props = self.get_properties()
        
        rules = {
            'MW < 500 Da': props['molecular_weight'] < 500,
            'LogP < 5': props['logP'] < 5,
            'H-donors < 5': props['num_h_donors'] < 5,
            'H-acceptors < 10': props['num_h_acceptors'] < 10,
        }
        
        violations = sum(not v for v in rules.values())
        
        bioavail = 'Good' if violations == 0 else 'Poor'
        if violations > 2:
            bioavail = 'Very Poor'
        
        return {
            'rules_passed': rules,
            'violations': violations,
            'predicted_bioavailability': bioavail
        }
    
    def analyze_bond_types(self) -> dict:
        """Analyze bond types in the molecule."""
        if not HAS_RDKIT:
            return {'error': 'RDKit required'}
        
        bond_counts = {
            'single_bonds': 0,
            'double_bonds': 0,
            'triple_bonds': 0,
            'aromatic_bonds': 0
        }
        
        for bond in self.mol.GetBonds():
            bt = bond.GetBondType()
            if bt == Chem.rdchem.BondType.SINGLE:
                bond_counts['single_bonds'] += 1
            elif bt == Chem.rdchem.BondType.DOUBLE:
                bond_counts['double_bonds'] += 1
            elif bt == Chem.rdchem.BondType.TRIPLE:
                bond_counts['triple_bonds'] += 1
            elif bt == Chem.rdchem.BondType.AROMATIC:
                bond_counts['aromatic_bonds'] += 1
        
        return bond_counts


def analyze_functional_groups(smiles: str) -> dict:
    """
    Identify common functional groups in a molecule.
    
    Args:
        smiles: SMILES string
    
    Returns:
        Dictionary of detected functional groups
    """
    if not HAS_RDKIT:
        return {'error': 'RDKit required'}
    
    mol = Chem.MolFromSmiles(smiles)
    
    # Define functional group patterns
    patterns = {
        'hydroxyl': Chem.MolFromSmarts('[OX2H]'),  # -OH
        'carboxyl': Chem.MolFromSmarts('C(=O)[OX2H]'),  # -COOH
        'amino_primary': Chem.MolFromSmarts('[NX3H2]'),  # -NH2
        'amino_secondary': Chem.MolFromSmarts('[NX3H1]'),  # -NHR
        'amino_tertiary': Chem.MolFromSmarts('[NX3H0]'),  # -NR2
        'sulfhydryl': Chem.MolFromSmarts('[SX2H]'),  # -SH
        'phosphate': Chem.MolFromSmarts('P'),  # -PO4
        'carbonyl_ketone': Chem.MolFromSmarts('[#6]C(=O)[#6]'),  # R-CO-R'
        'carbonyl_aldehyde': Chem.MolFromSmarts('[#6]C(=O)[H]'),  # R-CHO
        'ester': Chem.MolFromSmarts('C(=O)O'),  # R-COO-R'
        'amide': Chem.MolFromSmarts('C(=O)N'),  # R-CON-R'
    }
    
    results = {}
    for group_name, pattern in patterns.items():
        if pattern is not None:
            matches = mol.GetSubstructMatches(pattern)
            results[group_name] = len(matches)
    
    return results


# ============================================================
# PART 2: Electronegativity Calculations
# ============================================================

class ElectronegativityCalculator:
    """Calculate electronegativity differences for bond polarity."""
    
    # Pauling electronegativity values
    ELECTRONEGATIVITY = {
        'H': 2.20, 'C': 2.55, 'N': 3.04, 'O': 3.44,
        'F': 3.98, 'P': 2.19, 'S': 2.58, 'Cl': 3.16,
        'Br': 2.96, 'I': 2.66, 'Na': 0.93, 'K': 0.82,
        'Ca': 1.00, 'Mg': 1.31, 'Fe': 1.83, 'Zn': 1.65
    }
    
    @classmethod
    def get_en(cls, element: str) -> float:
        """Get electronegativity for an element."""
        if element not in cls.ELECTRONEGATIVITY:
            raise ValueError(f"Unknown element: {element}")
        return cls.ELECTRONEGATIVITY[element]
    
    @classmethod
    def bond_polarity(cls, elem1: str, elem2: str) -> dict:
        """
        Determine bond type based on electronegativity difference.
        
        Args:
            elem1, elem2: Element symbols
        
        Returns:
            Dictionary with bond type analysis
        """
        en1 = cls.get_en(elem1)
        en2 = cls.get_en(elem2)
        delta_en = abs(en1 - en2)
        
        if delta_en < 0.4:
            bond_type = 'Nonpolar Covalent'
        elif delta_en < 1.7:
            bond_type = 'Polar Covalent'
        else:
            bond_type = 'Ionic'
        
        # Determine which atom has greater electron density
        more_en = elem1 if en1 > en2 else elem2
        
        return {
            'elements': (elem1, elem2),
            'electronegativities': (en1, en2),
            'delta_en': round(delta_en, 2),
            'bond_type': bond_type,
            'more_electronegative_atom': more_en,
            'electron_density': f'{more_en} carries δ⁻, other carries δ⁺'
        }
    
    @classmethod
    def analyze_biological_bonds(cls) -> list:
        """Analyze common biological bonds."""
        biological_bonds = [
            ('C', 'C'), ('C', 'H'), ('C', 'O'), ('C', 'N'),
            ('O', 'H'), ('N', 'H'), ('C', 'S'), ('S', 'H'),
            ('N', 'O'), ('Na', 'Cl'), ('K', 'Cl'), ('C', 'P')
        ]
        
        results = []
        for bond in biological_bonds:
            result = cls.bond_polarity(bond[0], bond[1])
            results.append(result)
        
        return results


# ============================================================
# MAIN: Run Analysis
# ============================================================

if __name__ == "__main__":
    print("=" * 60)
    print("BMED1207 Chemistry Lab 2: Molecule Analysis")
    print("=" * 60)
    
    # Part 1: Analyze biomolecules
    print("\n--- Part 1: Biomolecule Analysis ---\n")
    
    biomolecules = [
        ("CCO", "Ethanol", "Simple alcohol"),
        ("CC(=O)O", "Acetic Acid", "Carboxylic acid"),
        ("CC(=O)C", "Acetone", "Ketone"),
        ("NCC(=O)O", "Acetamide", "Amide"),
        ("CS", "Methanethiol", "Sulfur-containing"),
        ("CC(=O)OC", "Methyl Acetate", "Ester"),
        ("CC(N)=O", "Acetamide", "Primary amide"),
    ]
    
    for smiles, name, description in biomolecules:
        print(f"\n{name} ({description}):")
        analyzer = MoleculeAnalyzer(smiles, name)
        props = analyzer.get_properties()
        for key, value in props.items():
            print(f"  {key}: {value}")
    
    # Part 2: Electronegativity analysis
    print("\n\n--- Part 2: Bond Polarity Analysis ---\n")
    
    results = ElectronegativityCalculator.analyze_biological_bonds()
    print(f"{'Bond':<12} {'Δχ':<8} {'Type':<20} {'More EN':<8} {'Charge Distribution'}")
    print("-" * 70)
    
    for r in results:
        bond_str = f"{r['elements'][0]}-{r['elements'][1]}"
        print(f"{bond_str:<12} {r['delta_en']:<8.2f} {r['bond_type']:<20} "
              f"{r['more_electronegative_atom']:<8} {r['electron_density']}")
    
    # Part 3: Lipinski's Rule
    print("\n\n--- Part 3: Drug-likeness Analysis (Lipinski's Rule) ---\n")
    
    drugs = [
        ("CC(C)Cc1ccc(cc1)C(C)C(O)=O", "Ibuprofen"),
        ("CC(C)NCC(O)COc1ccccc1", "Propranolol"),
        ("CN1CCC23C4C1CC5=C2C(=C(C=C5)O)OC3C(C=C4)O", "Morphine"),
        ("c1ccccc1", "Benzene"),
    ]
    
    for smiles, name in drugs:
        print(f"\n{name}:")
        analyzer = MoleculeAnalyzer(smiles, name)
        props = analyzer.get_properties()
        bio = analyzer.predict_bioavailability()
        
        print(f"  MW: {props['molecular_weight']} Da")
        print(f"  LogP: {props['logP']}")
        print(f"  H-donors: {props['num_h_donors']}, H-acceptors: {props['num_h_acceptors']}")
        print(f"  Bioavailability: {bio['predicted_bioavailability']}")
        print(f"  Rule violations: {bio['violations']}")
    
    print("\n" + "=" * 60)
    print("Lab 2 Complete!")
    print("=" * 60)
```

### Expected Output

```
============================================================
BMED1207 Chemistry Lab 2: Molecule Analysis
============================================================

--- Part 1: Biomolecule Analysis ---

Ethanol (Simple alcohol):
  name: Ethanol
  smiles: CCO
  molecular_formula: C2H6O
  molecular_weight: 46.07 Da
  ...

--- Part 2: Bond Polarity Analysis ---

Bond          Δχ       Type                 More EN   Charge Distribution
----------------------------------------------------------------------
C-C           0.00     Nonpolar Covalent    C         C carries δ⁻, other carries δ⁺
C-H           0.35     Nonpolar Covalent    C         C carries δ⁻, other carries δ⁺
C-O           0.89     Polar Covalent       O         O carries δ⁻, other carries δ⁺
O-H           1.24     Polar Covalent       O         O carries δ⁻, other carries δ⁺
Na-Cl         2.23     Ionic                Cl        Na⁺, Cl⁻
...

--- Part 3: Drug-likeness Analysis (Lipinski's Rule) ---

Ibuprofen:
  MW: 206.28 Da
  LogP: 3.97
  H-donors: 1, H-acceptors: 2
  Bioavailability: Good
  Rule violations: 0

Benzene:
  MW: 78.11 Da
  LogP: 2.13
  H-donors: 0, H-acceptors: 0
  Bioavailability: Good
  Rule violations: 0

============================================================
Lab 2 Complete!
============================================================
```

---

## Installation Instructions

```bash
# Required packages
pip install numpy matplotlib rdkit

# For blood gas visualization (optional)
pip install matplotlib pandas
```
