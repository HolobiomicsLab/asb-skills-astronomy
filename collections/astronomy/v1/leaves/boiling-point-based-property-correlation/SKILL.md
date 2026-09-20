---
name: boiling-point-based-property-correlation
description: Use when you need to predict vapour pressures of tetraalkyl- or tetraalkenylstannanes
  at temperatures below the experimentally measured range, using only boiling points
  and hydrocarbon-based estimation methods.
when_to_use_negative:
- The compound of interest is a tetraalkenylstannane and high accuracy is required,
  since hydrocarbon-based estimation methods underestimate its vapour pressure (Vi4Sn
  estimated 0.26/0.25 kPa vs 0.35 kPa experimental)
- No appropriate boiling point data are available for the target tetraorganostannane
  — the paper notes adequacy cannot be confirmed without such data
- The target property is a relationship based on molecular weight for tetraorganostannanes,
  since existing MW-based hydrocarbon relationships cannot be used with them
tools:
- name: Antoine equation
  role: Baseline correlation parameterized from the normal boiling point of each hydrocarbon
    analogue to estimate vapour pressure at 20°C
- name: Modified Watson correlation
  role: Second estimation baseline using the same boiling point and molecular-size
    inputs for cross-checking Antoine estimates
- name: Shimadzu GC-15A gas chromatograph
  role: Source of the experimental equilibrium vapour pressures (40–120°C, isothermal
    runs with FID detector) used to validate the estimates
- name: SE-30 capillary column
  role: 2m × 0.55mm ID, 1.2μm film column fitted to the GC-15A for the retention-time
    measurements underlying the reference experimental data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:ae468bded393282dc91be7bd7a92fb5f7adbdb57545e79bb3d0f90ffa60da72e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Antoine equation
  - Modified Watson correlation
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# boiling-point-based-property-correlation

## Summary

Estimates vapour pressures of organometallic compounds (tetraorganostannanes) at temperatures of environmental concern (e.g. 20°C) by treating the central Sn atom as carbon to build hydrocarbon analogues, then applying the Antoine equation and the Modified Watson correlation parameterized from normal boiling points and molecular-size inputs.

## When to use

Use when you need to predict vapour pressures of tetraalkyl- or tetraalkenylstannanes at temperatures below the experimentally measured range, using only boiling points and hydrocarbon-based estimation methods.

## When NOT to use

- The compound of interest is a tetraalkenylstannane and high accuracy is required, since hydrocarbon-based estimation methods underestimate its vapour pressure (Vi4Sn estimated 0.26/0.25 kPa vs 0.35 kPa experimental)
- No appropriate boiling point data are available for the target tetraorganostannane — the paper notes adequacy cannot be confirmed without such data
- The target property is a relationship based on molecular weight for tetraorganostannanes, since existing MW-based hydrocarbon relationships cannot be used with them

## Inputs

- List of tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) with their boiling points and molecular-size inputs
- GC-measured equilibrium vapour pressure data spanning 40°C to 120°C in 20°C increments (corrected mean retention times, Table 1)
- Experimental 20°C vapour pressures obtained by extrapolation (e.g. Et4Sn 0.16 kPa; Vi4Sn 0.35 kPa)

## Outputs

- Antoine-equation vapour pressure estimates per compound at target temperature (e.g. Et4Sn 0.15 kPa; Vi4Sn 0.26 kPa at 20°C)
- Modified Watson correlation estimates per compound (e.g. Vi4Sn 0.25 kPa at 20°C)
- Comparison table written to vapour_pressure_estimates.csv containing estimated and experimental vapour pressures

## How to apply

Build a hydrocarbon analogue of each tetraorganostannane by substituting carbon for the central Sn atom so that hydrocarbon-based property estimators become applicable. Implement the Antoine equation baseline parameterized from the normal boiling point of each analogue, and the Modified Watson correlation using the same boiling point and molecular-size inputs. Because GC-measured data span only 40°C to 120°C in 20°C increments, extrapolate both estimates down to 20°C. Compare computed values against GC determinations (means of 5 runs per compound, 3 for Allyl4Sn, using Me4Sn as the reference compound with corrected retention times) and against reported 20°C values such as Et4Sn 0.16 kPa and Vi4Sn 0.35 kPa. Judge success by reproducing the paper's finding: Antoine gave Et4Sn 0.15 kPa (satisfactory vs 0.16 kPa experimental), while Vi4Sn estimates of 0.26 and 0.25 kPa (Antoine and Modified Watson) underestimate the 0.35 kPa experimental value, showing the methods work for tetraalkylstannanes but are less successful for tetraalkenylstannanes.

## Related tools

- **Antoine equation** (Baseline correlation parameterized from the normal boiling point of each hydrocarbon analogue to estimate vapour pressure at 20°C)
- **Modified Watson correlation** (Second estimation baseline using the same boiling point and molecular-size inputs for cross-checking Antoine estimates)
- **Shimadzu GC-15A gas chromatograph** (Source of the experimental equilibrium vapour pressures (40–120°C, isothermal runs with FID detector) used to validate the estimates)
- **SE-30 capillary column** (2m × 0.55mm ID, 1.2μm film column fitted to the GC-15A for the retention-time measurements underlying the reference experimental data)

## Examples

```
python antoine_watson_vapour_pressure.py --compounds Me4Sn,Vi4Sn,Et4Sn,Allyl4Sn,n-Pr4Sn,i-Pr4Sn,n-Bu4Sn --T 20 --out vapour_pressure_estimates.csv
```

## Evaluation signals

- Antoine estimate for Et4Sn equals its reported value of 0.15 kPa, within ~0.01 kPa of the experimental 0.16 kPa
- Vi4Sn estimates from Antoine (0.26 kPa) and Modified Watson (0.25 kPa) both fall below the experimental 0.35 kPa, reproducing the qualitative underestimate for tetraalkenylstannanes
- All estimated and experimental values are written to vapour_pressure_estimates.csv with compound, method, temperature, and pressure columns
- Extrapolation target (20°C) lies below the 40–120°C experimental measurement window, so estimates must be flagged as extrapolated rather than interpolated
- Estimates reproduce the paper's conclusion that hydrocarbon-based methods are satisfactory for tetraalkylstannanes but less successful for tetraalkenylstannanes

## Limitations

- Hydrocarbon-based estimation methods are less successful for tetraalkenylstannanes; no explicit alternative method is named for these compounds
- Adequacy of the methods for other tetraalkylstannanes and tetraalkenylstannanes is uncertain because of lack of appropriate boiling point data
- For solid tetraorganostannanes (MP > 25°C), Equation (9) data must be converted to solid vapour pressure using entropy of fusion and melting point, an extra required input
- Experimental 20°C values are themselves extrapolations from 40–120°C GC measurements, not direct determinations
- A capillary column with C-87 stationary phase (most satisfactory per Bidleman) was unavailable, so SE-30 was used instead

## Evidence

- [abstract] hydrocarbon estimators satisfactory for alkyl but not alkenyl stannanes: "Current vapour pressure estimation methods for hydrocarbons proved satisfactory for tetraalkylstannanes, but less successful for tetraalkenylstannanes."
- [abstract] GC measurement range and 20°C extrapolation: "Equilibrium vapour pressures (40 ° to 120°C) of tetraorganostannes have been measured by a capillary gas chromatography method. By extrapolation, values at 20°C have also been determined."
- [methods] GC instrument used for validation data: "determined isothermally using a Shimadzu GC-15A gas chromatograph with FID detector"
- [methods] retention time averaging protocol: "All retention times for organostannanes used in vapour pressure determinations were the means of 5 separate runs, except for those of Allyl4Sn which were the mean of 3."
- [discussion] boiling point data gap limiting method validation: "Whether these estimation methods prove adequate for other tetraaikylstannanes, but less satisfactory for tetraalkenylstannanes is uncertain because of lack of appropriate boiling point data"
