---
name: qspr-transferability-assessment-hydrocarbons-to-organostannanes
description: Use when experimental property data exist for organometallic compounds
  (e.g. tetraorganostannane vapour pressures) and you must decide whether hydrocarbon-derived
  estimation methods are adequate, or when extrapolating GC-measured vapour pressures
  below the measured range.
when_to_use_negative:
- When no appropriate boiling point data exist for the compound or its hydrocarbon
  analogue — the paper notes adequacy for other tetraalkyl/alkenylstannanes is uncertain
  'because of lack of appropriate boiling point data'
- When the relationship of interest depends on molecular weight — the article states
  existing MW-based hydrocarbon relationships 'can not be used with tetraorganostannanes'
- For solid tetraorganostannanes (MP > 25°C) without conversion, since Equation (9)
  data must be transformed to solid vapour pressure using entropy of fusion and melting
  point
edam_operation: http://edamontology.org/operation_2426
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: source of the experimental 40–120°C equilibrium vapour pressures used as ground
    truth for validating hydrocarbon-based estimates
- name: SE-30 capillary column
  role: GC column (2m x 0.55mm ID, 1.2µm film, Alltech) on which retention times for
    vapour pressure determination were measured
- name: Antoine equation
  role: hydrocarbon-based estimation baseline parameterized from the normal boiling
    point of the Sn→C-substituted analogue
- name: Modified Watson correlation
  role: second hydrocarbon-based estimation baseline using the same boiling point
    and molecular-size inputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:b4c2d7f97513a1d32253152f4e0b8f8a177def142e0eb1b80fd41a7237327561
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Antoine equation
  - Modified Watson correlation
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# qspr-transferability-assessment-hydrocarbons-to-organostannanes

## Summary

A method for testing whether QSPR/estimation relationships calibrated on hydrocarbons can predict physicochemical properties (here, 20°C vapour pressures) of organometallic analogues (tetraorganostannanes) by substituting carbon for the central Sn atom and applying hydrocarbon-based estimators (Antoine equation, Modified Watson correlation).

## When to use

Use when experimental property data exist for organometallic compounds (e.g. tetraorganostannane vapour pressures) and you must decide whether hydrocarbon-derived estimation methods are adequate, or when extrapolating GC-measured vapour pressures below the measured range.

## When NOT to use

- When no appropriate boiling point data exist for the compound or its hydrocarbon analogue — the paper notes adequacy for other tetraalkyl/alkenylstannanes is uncertain 'because of lack of appropriate boiling point data'
- When the relationship of interest depends on molecular weight — the article states existing MW-based hydrocarbon relationships 'can not be used with tetraorganostannanes'
- For solid tetraorganostannanes (MP > 25°C) without conversion, since Equation (9) data must be transformed to solid vapour pressure using entropy of fusion and melting point

## Inputs

- List of tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) with boiling points and molecular size inputs
- GC-measured vapour pressure data spanning 40°C to 120°C in 20°C increments (means of 5 runs per compound, 3 for Allyl4Sn)
- Normal boiling points of the carbon-substituted hydrocarbon analogues

## Outputs

- Antoine-equation and Modified Watson vapour pressure estimates extrapolated to 20°C
- Comparison table of estimated vs experimental 20°C vapour pressures (vapour_pressure_estimates.csv)
- Transferability verdict per compound class (tetraalkylstannanes vs tetraalkenylstannanes)

## How to apply

Build hydrocarbon analogues of the organometallic compounds by substituting carbon for the central Sn atom so hydrocarbon-based property estimators become applicable; here, analogues of Et4Sn and Vi4Sn were formed. Implement the Antoine equation baseline parameterized from the normal boiling point of each analogue, and the Modified Watson correlation using the same boiling point and molecular-size inputs. Extrapolate estimates to the target temperature (20°C), which lies below the 40–120°C experimental GC measurement range. Compare computed estimates against reported values (Et4Sn 0.15 kPa; Vi4Sn 0.26 and 0.25 kPa) and experimental 20°C values (Et4Sn 0.16 kPa; Vi4Sn 0.35 kPa), judging transferability by absolute deviation: satisfactory when estimates fall within ~0.01–0.09 kPa (tetraalkylstannanes), inadequate when they systematically underestimate (tetraalkenylstannanes). Write estimated and experimental vapour pressures to vapour_pressure_estimates.csv for audit.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (source of the experimental 40–120°C equilibrium vapour pressures used as ground truth for validating hydrocarbon-based estimates)
- **SE-30 capillary column** (GC column (2m x 0.55mm ID, 1.2µm film, Alltech) on which retention times for vapour pressure determination were measured)
- **Antoine equation** (hydrocarbon-based estimation baseline parameterized from the normal boiling point of the Sn→C-substituted analogue)
- **Modified Watson correlation** (second hydrocarbon-based estimation baseline using the same boiling point and molecular-size inputs)

## Examples

```
python estimate_vapour_pressure.py --compounds Me4Sn,Vi4Sn,Et4Sn,Allyl4Sn,n-Pr4Sn,i-Pr4Sn,n-Bu4Sn --methods antoine,modified_watson --target-temp 20 --output vapour_pressure_estimates.csv
```

## Evaluation signals

- Antoine estimate for Et4Sn (0.15 kPa) matches its reported value within ~0.01 kPa of the experimental 0.16 kPa
- Vi4Sn estimates (0.26 and 0.25 kPa) reproduce the paper's qualitative finding of underestimation relative to the experimental 0.35 kPa
- All estimated and experimental vapour pressures are written to vapour_pressure_estimates.csv without missing compounds
- Estimates extrapolated to 20°C correctly fall below the 40°C lower bound of the GC measurement range
- Qualitative verdicts align with the paper: satisfactory for tetraalkylstannanes, less successful for tetraalkenylstannanes

## Limitations

- Hydrocarbon-based methods systematically underestimate vapour pressures of tetraalkenylstannanes (Vi4Sn: ~0.25–0.26 kPa estimated vs 0.35 kPa experimental)
- Lack of appropriate boiling point data prevents confirming whether the estimation methods are adequate for other tetraalkylstannanes and tetraalkenylstannanes
- Relationships based on molecular weight for hydrocarbons cannot be transferred to tetraorganostannanes; only intrinsic molar volume yields a single unique relationship across both classes
- For solid tetraorganostannanes (MP > 25°C), an extra input (entropy of fusion and melting point) is required to convert to solid vapour pressure

## Evidence
