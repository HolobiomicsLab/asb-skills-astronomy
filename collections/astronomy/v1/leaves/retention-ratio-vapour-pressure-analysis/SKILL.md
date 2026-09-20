---
name: retention-ratio-vapour-pressure-analysis
description: Use when isothermal GC retention times (40–120°C, 20°C increments) are
  available for a homologous compound series plus a reference compound with known
  ΔHv, and relative vaporization thermodynamics are needed.
when_to_use_negative:
- The analyte is a tetraalkenylstannane lacking boiling point data — hydrocarbon estimation
  methods were less successful for tetraalkenylstannanes and entropy cannot be derived
  without Tb
- The target compound is a triorganostannane — published vapour pressure data vary
  considerably and applicability of the reference-anchored equation is uncertain
- The compound is solid at room temperature (MP > 25°C) and solid-phase vapour pressure
  is needed — Equation (9) data require conversion using entropy of fusion (ΔSf) and
  melting point, an extra input this skill does not supply
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Acquires the isothermal retention time data (40–120°C) on which the retention-ratio
    regressions are based
- name: FID detector
  role: Flame ionization detection of the organostannane mixtures during isothermal
    runs
- name: Chromatopac C-R6A integrator
  role: Integrates retention times from the FID signal
- name: SE-30 capillary column
  role: 2m x 0.55mm ID, 1.2 μm film column used for the isothermal separations of
    the 1.0% w/v octane solutions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:2fd2f5ef2dfd190f000c1feca9d878edd2f8555ac584ed63876b1be4d1a63ab2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chromatopac C-R6A integrator
  - FID detector
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# retention-ratio-vapour-pressure-analysis

## Summary

Derives enthalpies (and entropies) of vaporization for tetraorganostannanes from isothermal capillary GC retention times by regressing ln(retention ratio relative to Me4Sn) against 1/T and anchoring the slope to the Me4Sn reference ΔHv of 31.28 kJ/mol.

## When to use

Use when isothermal GC retention times (40–120°C, 20°C increments) are available for a homologous compound series plus a reference compound with known ΔHv, and relative vaporization thermodynamics are needed.

## When NOT to use

- The analyte is a tetraalkenylstannane lacking boiling point data — hydrocarbon estimation methods were less successful for tetraalkenylstannanes and entropy cannot be derived without Tb
- The target compound is a triorganostannane — published vapour pressure data vary considerably and applicability of the reference-anchored equation is uncertain
- The compound is solid at room temperature (MP > 25°C) and solid-phase vapour pressure is needed — Equation (9) data require conversion using entropy of fusion (ΔSf) and melting point, an extra input this skill does not supply

## Inputs

- Corrected mean retention times for the tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) at 40–120°C in 20°C increments (Table 1)
- Dead-time measurements from natural-gas injections
- Reference compound vapour pressure–temperature data and ΔHv for Me4Sn (31.28 kJ/mol)
- Literature boiling points (Tb) for compounds where available

## Outputs

- Per-compound enthalpies of vaporization ΔHv (expected range 31.3–61.3 kJ/mol) as in Table 4
- Per-compound entropies of vaporization ΔHv/Tb (89.1–93.4 J K-1 mol-1) only where Tb is available
- Ordered table (e.g. vaporization_enthalpies.csv) following Table 4 compound order

## How to apply

Load corrected mean retention times (raw minus natural-gas dead time, averaged over 5 runs; 3 for Allyl4Sn) measured on the SE-30 capillary column from 40°C to 120°C in 20°C increments. For each compound, compute the retention ratio relative to tetramethylstannane (Me4Sn) at each temperature, then regress ln(retention ratio) against 1/T (Kelvin) for the 40–120°C series. Convert each van't Hoff-type slope to an enthalpy of vaporization by anchoring to the Me4Sn reference value ΔHv = 31.28 kJ/mol, which should reproduce ≈31.3 kJ/mol for Me4Sn itself. Where boiling point (Tb) data exist, derive the entropy of vaporization as ΔHv/Tb; omit this for compounds lacking Tb. Rationale: the GC relative-retention method avoids direct vapour pressure measurement and transfers the calibrated vapour pressure–temperature relationship of the reference compound to the analytes.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Acquires the isothermal retention time data (40–120°C) on which the retention-ratio regressions are based)
- **FID detector** (Flame ionization detection of the organostannane mixtures during isothermal runs)
- **Chromatopac C-R6A integrator** (Integrates retention times from the FID signal)
- **SE-30 capillary column** (2m x 0.55mm ID, 1.2 μm film column used for the isothermal separations of the 1.0% w/v octane solutions)

## Examples

```
from scipy.stats import linregress; import numpy as np; r = linregress(1/np.array(T_K), np.log(t_ratio/t_ref)); dHv = 31.28e3 - r.slope*8.314
```

## Evaluation signals

- Recomputed ΔHv for Me4Sn ≈ 31.3 kJ/mol, confirming the reference anchor at 31.28 kJ/mol
- All recomputed ΔHv values fall within the 31.3–61.3 kJ/mol range reported in Table 4
- ΔHv/Tb column is populated only for compounds with available boiling points, with values in the 89.1–93.4 J K-1 mol-1 range
- ln(retention ratio) vs 1/T regressions are linear over the 40–120°C series for each compound
- Output ordering matches Table 4: Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn

## Limitations

- Entropies of vaporization can only be derived where boiling point data exist; lack of appropriate Tb data prevents validating estimation methods for other tetraalkylstannanes and tetraalkenylstannanes
- Tetra-n-butylstannane was the largest compound whose vapour pressure could be determined under the experimental conditions
- Values at 20°C were obtained only by extrapolation from the 40–120°C measurement range
- Existing MW-based hydrocarbon relationships cannot be transferred to tetraorganostannanes; intrinsic molar volume is the descriptor supporting a single unique relationship

## Evidence
