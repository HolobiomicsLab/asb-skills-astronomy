---
name: vapour-pressure-estimation-organostannanes
description: Use when estimating ambient-temperature (20°C) vapour pressures of tetraalkyl-
  or tetraalkenylstannanes (e.g. Et4Sn, Vi4Sn) and boiling-point data are available
  for hydrocarbon analogues.
when_to_use_negative:
- The compound is a tetraorganostannane lacking appropriate boiling point data — the
  paper states estimation adequacy is uncertain 'because of lack of appropriate boiling
  point data'
- The target is a triorganostannane — little vapour pressure data exists and published
  values vary considerably, so applicability is uncertain
- The tetraorganostannane is solid at room temperature (MP > 25°C) — Equation (9)
  data need conversion to solid vapour pressure using entropy of fusion and melting
  point, an extra required input not covered here
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Experimental determination of equilibrium vapour pressures (40–120°C) used
    as validation data for the estimation methods
- name: SE-30 capillary column
  role: 2 m × 0.55 mm ID capillary column (1.2 µm film) fitted to the GC for the retention-time-based
    vapour pressure measurements
- name: Chromatopac C-R6A integrator
  role: Integration of FID detector signals for retention-time determination
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:eeaad8f2565b3ca8e263a586e817b1cec0c13ee6a610e13180a7b83ff83cc0e2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chromatopac C-R6A integrator
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# vapour-pressure-estimation-organostannanes

## Summary

Estimates 20°C vapour pressures of tetraorganostannanes by building hydrocarbon analogues (central Sn replaced by carbon) and applying hydrocarbon-based estimators — the Antoine equation and the Modified Watson correlation — parameterized from normal boiling points, then validating against GC-measured experimental values.

## When to use

Use when estimating ambient-temperature (20°C) vapour pressures of tetraalkyl- or tetraalkenylstannanes (e.g. Et4Sn, Vi4Sn) and boiling-point data are available for hydrocarbon analogues.

## When NOT to use

- The compound is a tetraorganostannane lacking appropriate boiling point data — the paper states estimation adequacy is uncertain 'because of lack of appropriate boiling point data'
- The target is a triorganostannane — little vapour pressure data exists and published values vary considerably, so applicability is uncertain
- The tetraorganostannane is solid at room temperature (MP > 25°C) — Equation (9) data need conversion to solid vapour pressure using entropy of fusion and melting point, an extra required input not covered here

## Inputs

- List of tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) with normal boiling points and molecular-size inputs
- GC-measured equilibrium vapour pressures at 40–120°C in 20°C increments (capillary GC with SE-30 column, Me4Sn as reference compound)
- Experimental 20°C vapour pressures obtained by extrapolation (Et4Sn 0.16 kPa; Vi4Sn 0.35 kPa)

## Outputs

- Antoine-equation 20°C vapour pressure estimates for each hydrocarbon analogue (kPa)
- Modified Watson correlation 20°C vapour pressure estimates (kPa)
- vapour_pressure_estimates.csv containing estimated and experimental vapour pressures for comparison

## How to apply

Load the tetraorganostannane list (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) with boiling points, molecular-size inputs, and the GC-measured vapour pressures spanning 40°C to 120°C in 20°C increments (means of 5 GC runs per compound, 3 for Allyl4Sn). Build the hydrocarbon analogue by substituting carbon for the central tin atom so hydrocarbon-based property estimators become applicable. Implement the Antoine equation baseline parameterized from the analogue's normal boiling point, and the Modified Watson correlation using the same boiling point and molecular-size inputs, then extrapolate both down to 20°C — which lies below the 40–120°C experimental range. Validate against the reported estimates (Et4Sn 0.15 kPa; Vi4Sn 0.26 and 0.25 kPa) and the experimental 20°C values (Et4Sn 0.16 kPa; Vi4Sn 0.35 kPa), writing all estimated and experimental values to vapour_pressure_estimates.csv. The rationale, per Hawker (1992), is that hydrocarbon-based estimation methods proved satisfactory for tetraalkylstannanes but less successful for tetraalkenylstannanes.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Experimental determination of equilibrium vapour pressures (40–120°C) used as validation data for the estimation methods)
- **SE-30 capillary column** (2 m × 0.55 mm ID capillary column (1.2 µm film) fitted to the GC for the retention-time-based vapour pressure measurements)
- **Chromatopac C-R6A integrator** (Integration of FID detector signals for retention-time determination)

## Examples

```
python estimate_vapour_pressure.py --compounds Et4Sn,Vi4Sn --methods antoine,modified_watson --temperature 20 --output vapour_pressure_estimates.csv
```

## Evaluation signals

- Antoine estimate for Et4Sn (0.15 kPa) matches its reported value within ~0.01 kPa of the experimental 0.16 kPa
- Estimates for Vi4Sn (Antoine 0.26 kPa, Modified Watson 0.25 kPa vs experimental 0.35 kPa) reproduce the paper's qualitative finding of underestimation for tetraalkenylstannanes
- Extrapolated 20°C estimates fall below the 40–120°C experimental measurement range and are flagged as extrapolations
- vapour_pressure_estimates.csv contains both estimated and experimental values for all compounds with consistent units (kPa)
- Hydrocarbon analogues correctly substitute carbon for the central tin atom before applying the estimators

## Limitations

- Hydrocarbon-based estimation methods were less successful for tetraalkenylstannanes (Vi4Sn estimated 0.26/0.25 kPa vs 0.35 kPa experimental)
- Adequacy for other tetraalkylstannanes is uncertain due to lack of appropriate boiling point data
- Relationships based on molecular weight for hydrocarbons cannot be used with tetraorganostannanes (MW-based curves are parallel but distinct); a single unique relationship exists only with intrinsic molar volume
- 20°C values are extrapolations below the 40–120°C range actually measured by GC

## Evidence
