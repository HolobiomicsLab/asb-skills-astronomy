---
name: linear-regression-fit-evaluation
description: Use when combining measured/literature property data for two compound
  classes (e.g. organostannanes + hydrocarbons) and you need to decide whether a molecular
  descriptor (MW vs intrinsic molar volume) collapses the data onto one linear relationship.
when_to_use_negative:
- 'When fitting against molecular weight is the goal: the paper shows two roughly
  parallel but distinct MW relationships for hydrocarbons vs organostannanes, so a
  single combined MW regression is not valid'
- For tetraalkenylstannanes, where hydrocarbon-derived estimation methods were less
  successful and no validated relationship is available
- When predicting other organostannane properties from descriptors other than MW or
  V — little data on such molecular descriptors exists and few published relationships
  use them
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0379
tools:
- name: scipy.stats.linregress
  role: Performs the OLS linear regression of ΔHv (kJ/mol) on intrinsic molar volume
    and returns slope, intercept, and Pearson r for comparison against Equation 11
- name: Shimadzu GC-15A gas chromatograph
  role: Generated the isothermal retention-time data (40–120°C in 20°C increments)
    from which the tetraorganostannane vapour pressures and ΔHv values used in the
    regression were derived
- name: SE-30 capillary column
  role: 2m × 0.55mm ID, 1.2 µm film SE-30 column used for the retention-time measurements
    feeding the ΔHv dataset
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:68d598710ffa715b751cecc2097dac43708f92b55fbd4253eac715d721ee78ae
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chromatopac C-R6A integrator
  - FID detector
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  - scipy.stats.linregress
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0379
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# linear-regression-fit-evaluation

## Summary

Fits and evaluates ordinary least-squares linear regressions of physicochemical properties (lnP, ΔHv) against molecular descriptors (molecular weight, intrinsic molar volume V) to test whether hydrocarbons and tetraorganostannanes share a single relationship.

## When to use

Use when combining measured/literature property data for two compound classes (e.g. organostannanes + hydrocarbons) and you need to decide whether a molecular descriptor (MW vs intrinsic molar volume) collapses the data onto one linear relationship.

## When NOT to use

- When fitting against molecular weight is the goal: the paper shows two roughly parallel but distinct MW relationships for hydrocarbons vs organostannanes, so a single combined MW regression is not valid
- For tetraalkenylstannanes, where hydrocarbon-derived estimation methods were less successful and no validated relationship is available
- When predicting other organostannane properties from descriptors other than MW or V — little data on such molecular descriptors exists and few published relationships use them

## Inputs

- Table 4 enthalpy-of-vaporization values (ΔHv, kJ/mol) for the seven tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn)
- Literature ΔHv values for reference hydrocarbons cited in the discussion
- Intrinsic molar volumes V (cm3/mol) and molecular weights for all compounds
- lnP (kPa) values for vapour-pressure regression (Figure 2 data)

## Outputs

- dhv_intrinsic_volume_regression.csv containing the combined n = 13 dataset with slope, intercept, r, n, and residuals
- Fitted regression parameters (slope ≈ 0.311 kJ mol^-1 per cm3 mol^-1, intercept ≈ 11.83 kJ/mol, r ≈ 0.96)
- Comparison plots of ΔHv or lnP versus molecular weight versus intrinsic molar volume

## How to apply

Assemble the combined dataset (n = 13 for the ΔHv case: seven tetraorganostannanes Me4Sn through n-Bu4Sn from Table 4 plus literature hydrocarbon values) with property (kJ/mol or lnP in kPa) and descriptor (V in cm3/mol) columns. Fit ΔHv (kJ/mol) on V by OLS using scipy.stats.linregress or numpy.polyfit, and compare slope, intercept, and Pearson r against the reported Equation 11 (ΔHv = 3.11e-1·V + 11.83, r = 0.96, n = 13). Rationale: with molecular weight as descriptor the two compound classes split into roughly parallel but distinct lines, so a single unique relationship with r ≈ 0.96 against intrinsic molar volume is the discriminating result. Write the dataset plus slope, intercept, r, n, and residuals (e.g. dhv_intrinsic_volume_regression.csv) so the fit can be re-validated numerically.

## Related tools

- **scipy.stats.linregress** (Performs the OLS linear regression of ΔHv (kJ/mol) on intrinsic molar volume and returns slope, intercept, and Pearson r for comparison against Equation 11)
- **Shimadzu GC-15A gas chromatograph** (Generated the isothermal retention-time data (40–120°C in 20°C increments) from which the tetraorganostannane vapour pressures and ΔHv values used in the regression were derived)
- **SE-30 capillary column** (2m × 0.55mm ID, 1.2 µm film SE-30 column used for the retention-time measurements feeding the ΔHv dataset)

## Examples

```
from scipy.stats import linregress; slope, intercept, r, p, se = linregress(df['V_cm3_mol'], df['dHv_kJ_mol'])
```

## Evaluation signals

- Slope of the ΔHv-versus-V regression ≈ 3.11e-1 kJ mol^-1 per cm3 mol^-1 within numerical tolerance
- Intercept ≈ 11.83 kJ/mol within numerical tolerance
- Pearson correlation coefficient r ≈ 0.96 over the combined n = 13 organostannane-plus-hydrocarbon points
- Output CSV contains one single unique relationship (not two compound-class-specific lines as with the MW plot) and per-point residuals are small and structureless
- Dataset row count equals 13 (7 tetraorganostannanes + 6 literature hydrocarbons)

## Limitations

- Hydrocarbon-based estimation methods were satisfactory for tetraalkylstannanes but less successful for tetraalkenylstannanes, so the combined regression should not be extended to alkenyl compounds
- Whether estimation methods are adequate for other tetraalkylstannanes is uncertain due to lack of appropriate boiling-point data
- The intrinsic-molar-volume relationship is validated only for vapour pressures and enthalpies of vaporization; applicability to triorganostannanes is uncertain because published vapour-pressure values vary considerably
- Lack of molecular descriptors other than MW for organostannanes restricts alternative descriptor regressions

## Evidence
