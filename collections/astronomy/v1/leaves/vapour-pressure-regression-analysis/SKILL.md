---
name: vapour-pressure-regression-analysis
description: Use when you have measured or extrapolated vapour pressures (e.g. 20°C
  values for Me4Sn through n-Bu4Sn) plus a molecular descriptor (MW or fragment-additivity
  intrinsic molar volume) and need to fit and validate lnP QSPR equations.
when_to_use_negative:
- Applying an MW-based relationship fitted on hydrocarbons to tetraorganostannanes
  — the article states 'any existing relationship involving MW for hydrocarbons can
  not be used with tetraorganostannes'
- Predicting vapour pressures of triorganostannanes or other stannanes where published
  data are scarce or inconsistent, so the validity of Equation (9) is untested
- Estimating vapour pressures of tetraorganostannanes that are solid at room temperature
  (MP > 25°C) without converting Equation (9) output using entropy of fusion (ΔSf)
  and melting point
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Source of the isothermal capillary GC retention times (40–120°C, 1.0% w/v
    solutions in octane, SE-30 column) from which the Table 3 vapour pressures used
    in the regressions were derived
- name: Chromatopac C-R6A integrator
  role: Recorded retention times fed into the corrected-retention-time calculation
    that produced the vapour pressure inputs
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:36e54c511749ce72450829c6c7eeba7e80260ff0b3e21af18e3170e164c69de7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chromatopac C-R6A integrator
  - FID detector
  - Shimadzu GC-15A gas chromatograph
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# vapour-pressure-regression-analysis

## Summary

Fits ordinary least-squares linear regressions of lnP (kPa) on molecular weight (Eq. 8, organostannanes only) and on intrinsic molar volume (Eq. 9, hydrocarbons and tetraorganostannanes combined) to quantify QSPR relationships for 20°C vapour pressures.

## When to use

Use when you have measured or extrapolated vapour pressures (e.g. 20°C values for Me4Sn through n-Bu4Sn) plus a molecular descriptor (MW or fragment-additivity intrinsic molar volume) and need to fit and validate lnP QSPR equations.

## When NOT to use

- Applying an MW-based relationship fitted on hydrocarbons to tetraorganostannanes — the article states 'any existing relationship involving MW for hydrocarbons can not be used with tetraorganostannes'
- Predicting vapour pressures of triorganostannanes or other stannanes where published data are scarce or inconsistent, so the validity of Equation (9) is untested
- Estimating vapour pressures of tetraorganostannanes that are solid at room temperature (MP > 25°C) without converting Equation (9) output using entropy of fusion (ΔSf) and melting point

## Inputs

- Table 3 vapour pressures of tetraorganostannanes extrapolated to 20°C (from capillary GC retention times, 40–120°C range)
- Molecular weights of the seven tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn)
- Literature vapour pressures and molecular weights of hydrocarbon compounds
- Intrinsic molar volumes (V, cm3/mol) computed by the Hickey and Passino-Reader fragment additivity procedure

## Outputs

- vapour_pressure_regression.csv containing fitted slope, intercept, Pearson r and sample size for each regression
- Equation 8: lnP (kPa) = -5.82e-2 MW + 12.27 (r = 0.99, n = 7)
- Equation 9: lnP (kPa) = -9.83e-3 V + 8.99 (r = 0.99, n = 13)
- Per-compound residuals of lnP for each fitted model

## How to apply

Load the Table 3 vapour pressures extrapolated to 20°C for the seven tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) and transform each to lnP (kPa). For Eq. 8, regress lnP on molecular weight over n = 7 points and expect slope ≈ -5.82e-2 kPa per g/mol, intercept ≈ 12.27, r ≈ 0.99. For Eq. 9, compute intrinsic molar volumes (V, cm3/mol) for all 13 compounds (7 stannanes plus literature hydrocarbons) using the Hickey and Passino-Reader fragment additivity procedure, then regress lnP on V to recover lnP = -9.83e-3 V + 8.99 with r ≈ 0.99. The rationale, per the article, is that MW-based relationships form three parallel but distinct lines for hydrocarbons, organostannanes and organoplumbanes, whereas intrinsic molar volume yields a single unique relationship — so descriptor choice is itself a hypothesis under test. Use scipy.stats.linregress or numpy.polyfit, record slope, intercept, r, n and per-compound residuals.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Source of the isothermal capillary GC retention times (40–120°C, 1.0% w/v solutions in octane, SE-30 column) from which the Table 3 vapour pressures used in the regressions were derived)
- **Chromatopac C-R6A integrator** (Recorded retention times fed into the corrected-retention-time calculation that produced the vapour pressure inputs)

## Examples

```
from scipy.stats import linregress; slope, intercept, r, p, se = linregress(V_cm3_per_mol, lnP_kPa)
```

## Evaluation signals

- Eq. 8 refit returns slope ≈ -5.82e-2, intercept ≈ 12.27 with r ≈ 0.99 for n = 7 stannane points within numerical tolerance
- Eq. 9 refit on the combined 13-compound dataset returns slope ≈ -9.83e-3, intercept ≈ 8.99 with r ≈ 0.99
- Intrinsic molar volumes reproduce the Hickey and Passino-Reader fragment additivity values used in the paper (cm3/mol)
- Per-compound lnP residuals show no systematic class separation (alkyl vs alkenyl) in the V-based model, but do so in the MW-based model
- Regression output file contains all required columns (slope, intercept, r, n, residuals) and flags any discrepancy beyond reasonable tolerance

## Limitations

- Hydrocarbon-based estimation methods proved satisfactory for tetraalkylstannanes but less successful for tetraalkenylstannanes
- Lack of appropriate boiling point data prevents confirming whether estimation methods are adequate for other tetraalkylstannanes and tetraalkenylstannanes
- There is little data on molecular descriptors of organostannanes other than MW and few published relationships using other descriptors
- n-Bu4Sn was the largest compound whose vapour pressure could be determined under the experimental conditions, bounding the regression's calibration range

## Evidence
