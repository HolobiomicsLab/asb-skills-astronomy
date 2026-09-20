---
name: intrinsic-molar-volume-qspr-verification
description: Use when testing whether a property of organometallic compounds (e.g.
  ΔHv or lnP of tetraorganostannanes) is better predicted by intrinsic molar volume
  than by molecular weight, or when validating that hydrocarbon-derived QSPRs transfer
  to organostannanes.
when_to_use_negative:
- When the compound set includes tetraalkenylstannanes only with no reference hydrocarbons
  — the paper shows hydrocarbon-derived estimation methods are 'less successful for
  tetraalkenylstannanes', so the single-relationship assumption may not hold
- When molecular weight is the intended descriptor — the paper explicitly states 'any
  existing relationship involving MW for hydrocarbons can not be used with tetraorganostannes'
- When predicting solid-state vapour pressures of tetraorganostannanes with MP > 25°C,
  which additionally require entropy of fusion (ΔSf) and melting point to convert
  Equation (9) data
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Source of the isothermal retention-time data from which the tetraorganostannane
    vapour pressures and derived ΔHv values were determined
- name: FID detector
  role: Flame ionization detection on the GC-15A used for retention-time measurement
    of organostannane mixtures
- name: Chromatopac C-R6A integrator
  role: Integration of FID peaks for the retention-time measurements underlying the
    ΔHv dataset
- name: SE-30 capillary column
  role: 2 m x 0.55 mm ID, 1.2 μm film Alltech SE-30 column used instead of the unavailable
    C-87 phase for the GC determinations
- name: scipy.stats.linregress
  role: Ordinary least-squares linear regression of ΔHv on intrinsic molar volume
    specified in the reproduction workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:197a5a66b785bf51cf17594916de9b1b6a308918050c08aaf03518ff6dc534e3
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# intrinsic-molar-volume-qspr-verification

## Summary

Reproduces and verifies the paper's central QSPR claim that enthalpies of vaporization (and ln vapour pressures) of tetraorganostannanes and reference hydrocarbons collapse onto a single linear relationship when regressed on intrinsic molar volume (V, cm^3/mol), unlike molecular-weight plots which yield parallel-but-distinct lines per compound class.

## When to use

Use when testing whether a property of organometallic compounds (e.g. ΔHv or lnP of tetraorganostannanes) is better predicted by intrinsic molar volume than by molecular weight, or when validating that hydrocarbon-derived QSPRs transfer to organostannanes.

## When NOT to use

- When the compound set includes tetraalkenylstannanes only with no reference hydrocarbons — the paper shows hydrocarbon-derived estimation methods are 'less successful for tetraalkenylstannanes', so the single-relationship assumption may not hold
- When molecular weight is the intended descriptor — the paper explicitly states 'any existing relationship involving MW for hydrocarbons can not be used with tetraorganostannes'
- When predicting solid-state vapour pressures of tetraorganostannanes with MP > 25°C, which additionally require entropy of fusion (ΔSf) and melting point to convert Equation (9) data

## Inputs

- Table 4 enthalpy-of-vaporization values (kJ/mol) for the seven tetraorganostannanes Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn
- Intrinsic molar volumes V (cm^3/mol) for the seven tetraorganostannanes
- Literature ΔHv values and intrinsic molar volumes for the reference hydrocarbons cited in the discussion (to reach n = 13)
- Molecular weights of all 13 compounds (for the contrast regression)

## Outputs

- dhv_intrinsic_volume_regression.csv containing the combined n = 13 dataset with regression statistics (slope, intercept, r, n, residuals)
- Fitted coefficients ΔHv = 3.11e-1 * V + 11.83 (kJ/mol vs cm^3/mol) with Pearson r ≈ 0.96
- Contrast regression of ΔHv on molecular weight showing two parallel but distinct hydrocarbon/organostannane relationships

## How to apply

Load the Table 4 ΔHv values for the seven tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) measured via GC relative to tetramethylstannane as reference, plus their intrinsic molar volumes, into a pandas DataFrame. Append literature ΔHv values for the reference hydrocarbons discussed in the paper to reach n = 13 points. Fit an ordinary least-squares regression of ΔHv (kJ/mol) on V using scipy.stats.linregress or numpy.polyfit, then check slope ≈ 3.11e-1 kJ mol^-1 per cm^3 mol^-1, intercept ≈ 11.83 kJ/mol, and Pearson r ≈ 0.96. As a contrast, regress the same ΔHv data on molecular weight and confirm two roughly parallel but distinct hydrocarbon/organostannane lines appear — the single-relationship result with V is the paper's key finding. Write the combined dataset with slope, intercept, r, n and residuals to dhv_intrinsic_volume_regression.csv.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Source of the isothermal retention-time data from which the tetraorganostannane vapour pressures and derived ΔHv values were determined)
- **FID detector** (Flame ionization detection on the GC-15A used for retention-time measurement of organostannane mixtures)
- **Chromatopac C-R6A integrator** (Integration of FID peaks for the retention-time measurements underlying the ΔHv dataset)
- **SE-30 capillary column** (2 m x 0.55 mm ID, 1.2 μm film Alltech SE-30 column used instead of the unavailable C-87 phase for the GC determinations)
- **scipy.stats.linregress** (Ordinary least-squares linear regression of ΔHv on intrinsic molar volume specified in the reproduction workflow)

## Examples

```
import pandas as pd; from scipy.stats import linregress; df = pd.read_csv('dhv_intrinsic_volume_regression.csv'); slope, intercept, r, p, se = linregress(df.V_cm3_mol, df.dHv_kJ_mol)
```

## Evaluation signals

- Slope of the ΔHv vs V regression ≈ 3.11e-1 kJ mol^-1 per cm^3 mol^-1 within numerical tolerance
- Intercept of the ΔHv vs V regression ≈ 11.83 kJ/mol
- Pearson correlation coefficient r ≈ 0.96 over the combined n = 13 points
- Dataset row count equals 13 (7 tetraorganostannanes + 6 reference hydrocarbons)
- The contrast regression of ΔHv on molecular weight yields two roughly parallel but distinct hydrocarbon/organostannane lines rather than one, confirming the descriptor comparison

## Limitations

- The regression covers only the seven tetraorganostannanes for which vapour pressures could be measured — tetra-n-butylstannane was 'the largest compound whose vapour pressure could be determined under the experimental conditions'
- Vapour pressures at 20°C were obtained by extrapolation from the 40–120°C measurement range, adding uncertainty to derived ΔHv values
- There is little data on molecular descriptors of organostannanes other than MW, and few published relationships using other descriptors, limiting independent verification of the intrinsic-volume coefficients
- Whether hydrocarbon estimation methods are adequate for other tetraalkylstannanes and tetraalkenylstannanes is uncertain due to lack of appropriate boiling point data

## Evidence
