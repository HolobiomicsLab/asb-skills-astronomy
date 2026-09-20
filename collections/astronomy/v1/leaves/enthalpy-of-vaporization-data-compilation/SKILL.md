---
name: enthalpy-of-vaporization-data-compilation
description: Use when combining derived or literature ΔHv data for organostannanes
  and reference hydrocarbons into one dataset and testing descriptor choice (molecular
  weight vs intrinsic molar volume) via linear regression.
when_to_use_negative:
- When the target property is vapour pressure (lnP vs V) rather than enthalpy of vaporization
  — use the Equation 9 / vapour-pressure regression workflow instead
- When the descriptor of interest is molecular weight — the article shows MW relationships
  for hydrocarbons are distinct from (and inapplicable to) tetraorganostannanes
- When the dataset contains tetraalkenylstannanes only and no hydrocarbon reference
  points, since hydrocarbon-based estimation methods were less successful for tetraalkenylstannanes
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scipy.stats.linregress
  role: OLS fit of ΔHv (kJ/mol) on intrinsic molar volume V (cm³/mol) over the combined
    n = 13 organostannane-plus-hydrocarbon dataset
- name: Shimadzu GC-15A gas chromatograph
  role: Source instrument for the capillary GC vapour-pressure measurements from which
    the tetraorganostannane ΔHv values were derived
- name: SE-30 capillary column
  role: 2m × 0.55mm ID, 1.2µm film Alltech SE-30 column used for the isothermal retention-time
    measurements underlying the ΔHv derivation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:adc717e802aa83a2a0cac3fdaa4c4f4d818d17d2db349fc05a72d993eb2f2630
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chromatopac C-R6A integrator
  - FID detector
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  - scipy.stats.linregress
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# enthalpy-of-vaporization-data-compilation

## Summary

Compiles enthalpies of vaporization (ΔHv) for the seven tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) with literature hydrocarbon ΔHv values, and fits a single linear regression of ΔHv (kJ/mol) on intrinsic molar volume V (cm³/mol) to test whether organostannanes and hydrocarbons share one QSPR relationship.

## When to use

Use when combining derived or literature ΔHv data for organostannanes and reference hydrocarbons into one dataset and testing descriptor choice (molecular weight vs intrinsic molar volume) via linear regression.

## When NOT to use

- When the target property is vapour pressure (lnP vs V) rather than enthalpy of vaporization — use the Equation 9 / vapour-pressure regression workflow instead
- When the descriptor of interest is molecular weight — the article shows MW relationships for hydrocarbons are distinct from (and inapplicable to) tetraorganostannanes
- When the dataset contains tetraalkenylstannanes only and no hydrocarbon reference points, since hydrocarbon-based estimation methods were less successful for tetraalkenylstannanes

## Inputs

- Table 4 enthalpy-of-vaporization values (kJ/mol) for seven tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn)
- Intrinsic molar volumes V (cm³/mol) for the organostannanes and hydrocarbons
- Literature ΔHv values for reference hydrocarbons discussed in the article (to reach n = 13 combined points)

## Outputs

- dhv_intrinsic_volume_regression.csv containing the combined n = 13 dataset with regression statistics (slope, intercept, r, n) and per-point residuals

## How to apply

Load the Table 4 ΔHv values for the seven tetraorganostannanes and their intrinsic molar volumes into a pandas DataFrame, then append literature ΔHv values for the cited reference hydrocarbons to form a combined n = 13 dataset. Fit an ordinary least-squares regression of ΔHv (kJ/mol) on V (cm³/mol) with scipy.stats.linregress or numpy.polyfit, and compare slope, intercept, and Pearson r against the reported relationship ΔHv = 3.11e-1·V + 11.83 with r = 0.96. The rationale is that ΔHv forms two roughly parallel but distinct relationships against molecular weight, but a single unique relationship against intrinsic molar volume, so V is the correct descriptor for cross-family prediction. Write the combined data plus slope, intercept, r, n, and residuals to dhv_intrinsic_volume_regression.csv.

## Related tools

- **scipy.stats.linregress** (OLS fit of ΔHv (kJ/mol) on intrinsic molar volume V (cm³/mol) over the combined n = 13 organostannane-plus-hydrocarbon dataset)
- **Shimadzu GC-15A gas chromatograph** (Source instrument for the capillary GC vapour-pressure measurements from which the tetraorganostannane ΔHv values were derived)
- **SE-30 capillary column** (2m × 0.55mm ID, 1.2µm film Alltech SE-30 column used for the isothermal retention-time measurements underlying the ΔHv derivation)

## Examples

```
from scipy.stats import linregress; res = linregress(V_cm3_mol, dhv_kJ_mol); assert abs(res.slope-0.311)<0.01 and abs(res.intercept-11.83)<0.5
```

## Evaluation signals

- Fitted slope ≈ 3.11e-1 kJ mol^-1 per cm³ mol^-1 within numerical tolerance
- Fitted intercept ≈ 11.83 kJ/mol within numerical tolerance
- Pearson correlation coefficient r ≈ 0.96
- Combined dataset size equals n = 13 (7 tetraorganostannanes + 6 hydrocarbons) with no missing V or ΔHv entries
- Residuals written per-point to dhv_intrinsic_volume_regression.csv are approximately zero-mean with no systematic family split (stannane vs hydrocarbon) in sign

## Limitations

- ΔHv values derive from vapour pressures measured only over 40–120 °C; tetra-n-butylstannane was the largest compound whose vapour pressure could be determined under the experimental conditions
- Entropies of vaporization could be derived only where boiling point information was available; lack of appropriate boiling point data limits validation for other tetraalkylstannanes and tetraalkenylstannanes
- Vapour pressures at 20 °C were obtained by extrapolation, and estimation methods based on hydrocarbons were less successful for tetraalkenylstannanes

## Evidence
