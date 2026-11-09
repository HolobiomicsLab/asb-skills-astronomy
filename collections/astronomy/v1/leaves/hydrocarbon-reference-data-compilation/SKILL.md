---
name: hydrocarbon-reference-data-compilation
description: Use when fitting a structure-property regression (lnP or ΔHv vs intrinsic
  molar volume) that must merge experimentally measured organostannane data with published
  hydrocarbon values, and when validating against a reported single-relationship equation.
when_to_use_negative:
- The descriptor is molecular weight — the article shows MW-based relationships for
  hydrocarbons cannot be transferred to tetraorganostannanes (three parallel but distinct
  lnP vs MW lines; two distinct ΔHv vs MW lines)
- The target compounds are triorganostannanes or tetraalkenylstannanes — published
  vapour pressure data are sparse or vary considerably, so applicability of Equation
  (9) is uncertain
- The property needed is the vapour pressure of solid tetraorganostannanes (MP > 25°C)
  — Equation (9) values must first be converted using entropy of fusion (ΔSf) and
  melting point
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3369
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Source of the measured 40–120°C equilibrium vapour pressures, extrapolated
    to 20°C, that form the stannane half of the combined dataset
- name: FID detector
  role: Detector used with the Shimadzu GC-15A for retention-time-based vapour pressure
    determination
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:5d83c21755b3b37729fc8c2c7b736bb788f7602039d0b6b4df43f99d57463a8e
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
  - http://edamontology.org/topic_3369
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# hydrocarbon-reference-data-compilation

## Summary

Compiles literature hydrocarbon physicochemical data (vapour pressures, enthalpies of vaporization) together with measured tetraorganostannane values into a combined n = 13 dataset so that a single QSPR against intrinsic molar volume can be fitted and validated against the article's Equations 9 and 11.

## When to use

Use when fitting a structure-property regression (lnP or ΔHv vs intrinsic molar volume) that must merge experimentally measured organostannane data with published hydrocarbon values, and when validating against a reported single-relationship equation.

## When NOT to use

- The descriptor is molecular weight — the article shows MW-based relationships for hydrocarbons cannot be transferred to tetraorganostannanes (three parallel but distinct lnP vs MW lines; two distinct ΔHv vs MW lines)
- The target compounds are triorganostannanes or tetraalkenylstannanes — published vapour pressure data are sparse or vary considerably, so applicability of Equation (9) is uncertain
- The property needed is the vapour pressure of solid tetraorganostannanes (MP > 25°C) — Equation (9) values must first be converted using entropy of fusion (ΔSf) and melting point

## Inputs

- Table 3 vapour pressures (extrapolated to 20°C) for the seven tetraorganostannanes
- Table 4 enthalpies of vaporization for the seven tetraorganostannanes
- Literature hydrocarbon vapour pressure and ΔHv values cited in the article discussion
- Intrinsic molar volumes (cm3/mol) computed via the Hickey and Passino-Reader fragment additivity method

## Outputs

- vapour_pressure_regression.csv — combined lnP vs V dataset with fitted slope, intercept, r, n
- dhv_intrinsic_volume_regression.csv — combined ΔHv vs V dataset with slope, intercept, r, n, residuals

## How to apply

Compile the 13-compound dataset: the seven tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) with vapour pressures extrapolated to 20°C (Table 3) and ΔHv values (Table 4), plus the literature hydrocarbon compounds cited in the discussion. Compute intrinsic molar volumes (V, cm3/mol) for all 13 compounds with the Hickey and Passino-Reader fragment additivity procedure — do NOT use molecular weight, since the article shows lnP vs MW splits into three parallel distinct relationships (stannanes, hydrocarbons, organoplumbanes) whereas V yields one unique line. Fit ordinary least-squares with scipy.stats.linregress or numpy.polyfit: lnP (kPa) = -9.83e-3 V + 8.99 (r = 0.99, n = 13) and ΔHv (kJ/mol) = 3.11e-1 V + 11.83 (r = 0.96, n = 13). Write the combined data plus slope, intercept, r, n and residuals to vapour_pressure_regression.csv and dhv_intrinsic_volume_regression.csv.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Source of the measured 40–120°C equilibrium vapour pressures, extrapolated to 20°C, that form the stannane half of the combined dataset)
- **FID detector** (Detector used with the Shimadzu GC-15A for retention-time-based vapour pressure determination)

## Examples

```
from scipy.stats import linregress; slope, intercept, r, p, se = linregress(V_cm3_mol, lnP_kPa); print(slope, intercept, r, len(V_cm3_mol))
```

## Evaluation signals

- Fitted lnP regression reproduces lnP (kPa) = -9.83e-3 V + 8.99 with r = 0.99 and n = 13 within numerical tolerance
- Fitted ΔHv regression reproduces slope ≈ 3.11e-1 kJ mol^-1 per cm3 mol^-1, intercept ≈ 11.83 kJ/mol, r ≈ 0.96 over n = 13 points
- Dataset contains exactly 13 compounds: 7 tetraorganostannanes plus the cited literature hydrocarbons
- Correlation of lnP with intrinsic molar volume (single line, r ≈ 0.99) is markedly better than any single line against molecular weight, which the paper reports as three parallel but distinct relationships
- Residuals in dhv_intrinsic_volume_regression.csv are small and show no systematic stannane-vs-hydrocarbon split

## Limitations

- Hydrocarbon-based vapour pressure estimation methods are less successful for tetraalkenylstannanes, limiting extension of the combined QSPR to those compounds
- Lack of appropriate boiling point data prevents confirming whether estimation methods are adequate for other tetraalkylstannanes
- Equation (9) applies to liquid-state (subcooled) vapour pressures; for solid tetraorganostannanes (MP > 25°C) conversion via entropy of fusion and melting point is an extra required input
- Only tetra-n-butylstannane could be measured under the experimental conditions; larger stannanes are outside the fitted range

## Evidence
