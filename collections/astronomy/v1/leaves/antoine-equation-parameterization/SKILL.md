---
name: antoine-equation-parameterization
description: Use when estimating a vapour pressure at a temperature below the measured
  range (e.g. 20°C) for an organometallic compound whose hydrocarbon analogue (Sn→C
  substitution) has known boiling point data, and no direct measurement is available.
when_to_use_negative:
- When the compound's hydrocarbon analogue lacks appropriate boiling point data —
  the paper notes adequacy for other tetraalkyl/alkenylstannanes is uncertain 'because
  of lack of appropriate boiling point data'.
- When the target property relationship relies on molecular weight — the paper states
  existing MW-based relationships for hydrocarbons cannot be used with tetraorganostannanes;
  use the intrinsic molar volume relationship instead.
- When direct measured vapour pressures within the 40–120°C range are available and
  no sub-range extrapolation is needed — measured GC values should be used directly.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3397
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Source of experimental retention-time data from which the benchmark 20°C vapour
    pressures for comparison were derived
- name: SE-30 capillary column
  role: Capillary GC column (2 m × 0.55 mm ID, 1.2 µm film, Alltech) used for the
    isothermal retention-time measurements underlying the experimental vapour pressures
- name: Modified Watson correlation
  role: Second estimation baseline implemented alongside the Antoine equation, using
    the same boiling point and molecular-size inputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:e807ff56e9dfd39ff127067a3b27f8c0692a7d38bf9d66f2857cfe3059f8162a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Antoine equation
  - Modified Watson correlation
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3397
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# antoine-equation-parameterization

## Summary

Parameterizes the Antoine equation from the normal boiling point of a hydrocarbon analogue of an organostannane (central Sn treated as carbon) to extrapolate 20°C vapour pressures below the 40–120°C GC measurement range, then compares estimates with experimental values.

## When to use

Use when estimating a vapour pressure at a temperature below the measured range (e.g. 20°C) for an organometallic compound whose hydrocarbon analogue (Sn→C substitution) has known boiling point data, and no direct measurement is available.

## When NOT to use

- When the compound's hydrocarbon analogue lacks appropriate boiling point data — the paper notes adequacy for other tetraalkyl/alkenylstannanes is uncertain 'because of lack of appropriate boiling point data'.
- When the target property relationship relies on molecular weight — the paper states existing MW-based relationships for hydrocarbons cannot be used with tetraorganostannanes; use the intrinsic molar volume relationship instead.
- When direct measured vapour pressures within the 40–120°C range are available and no sub-range extrapolation is needed — measured GC values should be used directly.

## Inputs

- List of tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) with boiling points and molecular size inputs
- Normal boiling points of the hydrocarbon analogues (Sn→C substitution)
- Experimental GC-measured vapour pressures spanning 40–120°C in 20°C increments, plus extrapolated 20°C values (e.g. Et4Sn 0.16 kPa, Vi4Sn 0.35 kPa)

## Outputs

- Antoine-equation 20°C vapour pressure estimates per compound (kPa)
- Modified Watson 20°C vapour pressure estimates per compound (kPa)
- Comparison table written to vapour_pressure_estimates.csv containing estimated and experimental values

## How to apply

1) Build the hydrocarbon analogue by substituting carbon for the central Sn atom of the target compound (e.g. Et4Sn → hydrocarbon analogue of tetraethylstannane), so hydrocarbon-based property estimators become applicable. 2) Parameterize the Antoine equation using the normal boiling point of the analogue. 3) Independently implement the Modified Watson correlation with the same boiling point and molecular-size inputs as a second baseline. 4) Extrapolate both estimates down to 20°C — note 20°C lies below the 40–120°C experimental range measured in 20°C increments, so this is an extrapolation, not interpolation. 5) Compare computed values against experimental 20°C values and validate against reported benchmarks (Et4Sn estimated 0.15 kPa vs 0.16 kPa experimental; Vi4Sn estimated 0.26 and 0.25 kPa vs 0.35 kPa experimental). 6) Write estimated and experimental vapour pressures to vapour_pressure_estimates.csv. Rationale: the paper establishes that hydrocarbon estimation methods are satisfactory for tetraalkylstannanes but underestimate tetraalkenylstannanes.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Source of experimental retention-time data from which the benchmark 20°C vapour pressures for comparison were derived)
- **SE-30 capillary column** (Capillary GC column (2 m × 0.55 mm ID, 1.2 µm film, Alltech) used for the isothermal retention-time measurements underlying the experimental vapour pressures)
- **Modified Watson correlation** (Second estimation baseline implemented alongside the Antoine equation, using the same boiling point and molecular-size inputs)

## Examples

```
python estimate_vp.py --analogues hydrocarbon_boiling_points.csv --methods antoine,modified_watson --extrapolate 20C --out vapour_pressure_estimates.csv
```

## Evaluation signals

- Antoine estimate for Et4Sn (0.15 kPa) matches its reported value within ~0.01 kPa of the experimental 0.16 kPa
- Vi4Sn estimates (0.26 and 0.25 kPa) reproduce the paper's qualitative finding of underestimation relative to the experimental 0.35 kPa
- Both Antoine and Modified Watson estimates fall below the experimental value for tetraalkenylstannanes and are close for tetraalkylstannanes
- Output file vapour_pressure_estimates.csv contains both estimated and experimental values for every compound processed
- Estimates are computed at 20°C, confirming the extrapolation lies outside the 40–120°C measurement range

## Limitations

- Hydrocarbon-based estimation methods are less successful for tetraalkenylstannanes; no alternative estimation method is named in the article.
- Adequacy for other tetraalkylstannanes and tetraalkenylstannanes is uncertain due to lack of appropriate boiling point data.
- The 20°C estimate is an extrapolation below the 40–120°C GC measurement range, compounding parameterization uncertainty.
- Relationships involving molecular weight for hydrocarbons cannot be used with tetraorganostannanes; molecular descriptors other than MW are scarce for organostannanes.

## Evidence

- [abstract] Current vapour pressure estimation methods for hydrocarbons proved satisfactory for tetraalkylstannanes, but less successful for tetraalkenylstannanes.: "Current vapour pressure estimation methods for hydrocarbons proved satisfactory for tetraalkylstannanes, but less successful for tetraalkenylstannanes."
- [abstract] By extrapolation, values at 20°C have also been determined.: "By extrapolation, values at 20°C have also been determined."
- [methods] The experimental temperature range was from 40°C to 120°C in 20°C increments.: "The experimental temperature range was from 40°C to 120°C in 20°C increments."
- [methods] All retention times for organostannanes used in vapour pressure determinations were the means of 5 separate runs: "All retention times for organostannanes used in vapour pressure determinations were the means of 5 separate runs, except for those of Allyl4Sn which were the mean of 3."
- [discussion] Whether these estimation methods prove adequate for other tetraalkylstannanes, but less satisfactory for tetraalkenylstannanes is uncertain because of lack of appropriate boiling point data: [evidence span withheld — anchor status: not_found]
