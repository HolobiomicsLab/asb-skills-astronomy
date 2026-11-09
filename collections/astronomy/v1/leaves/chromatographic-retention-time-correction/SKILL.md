---
name: chromatographic-retention-time-correction
description: Use when isothermal capillary-GC retention times (e.g. 40-120°C in 20°C
  increments on SE-30) must be dead-time corrected, replicate-averaged, and regressed
  against 1/T to derive enthalpies/entropies of vaporization.
when_to_use_negative:
- 'Compounds larger than tetra-n-butylstannane: it was the largest compound whose
  vapour pressure could be determined under these experimental conditions, so retention-based
  extrapolation beyond it is unsupported.'
- 'Tetraalkenylstannanes requiring external vapour pressure estimation: hydrocarbon-based
  estimation methods were less successful for tetraalkenyls, and no explicit alternative
  method is named.'
- 'Solid tetraorganostannanes (mp > 25°C): Equation (9) data must be converted to
  solid vapour pressures using entropy of fusion (ΔSf) and melting point, which this
  retention-correction workflow does not handle.'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Acquires isothermal retention-time data used for dead-time correction and
    van't Hoff regression
- name: FID detector
  role: Detection of organostannane peaks feeding retention-time measurements
- name: Chromatopac C-R6A integrator
  role: Integrates GC peaks to produce the raw retention-time values being corrected
- name: SE-30 capillary column
  role: Separation medium (2m x 0.55mm ID, 1.2µm film) on which retention ratios vs
    Me4Sn are measured; chosen for low McReynold's constants
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:c8d401728f64f1dfde22ed55c9611b4578f320a8f603bc301f7ca6b2c1a6f505
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chromatopac C-R6A integrator
  - FID detector
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# chromatographic-retention-time-correction

## Summary

Corrects raw GC retention times for instrumental dead-time (via natural-gas injection), averages replicate runs, and converts temperature-dependent retention ratios against a reference compound (Me4Sn) into vapour pressures and enthalpies of vaporization via van't Hoff-type regression.

## When to use

Use when isothermal capillary-GC retention times (e.g. 40-120°C in 20°C increments on SE-30) must be dead-time corrected, replicate-averaged, and regressed against 1/T to derive enthalpies/entropies of vaporization.

## When NOT to use

- Compounds larger than tetra-n-butylstannane: it was the largest compound whose vapour pressure could be determined under these experimental conditions, so retention-based extrapolation beyond it is unsupported.
- Tetraalkenylstannanes requiring external vapour pressure estimation: hydrocarbon-based estimation methods were less successful for tetraalkenyls, and no explicit alternative method is named.
- Solid tetraorganostannanes (mp > 25°C): Equation (9) data must be converted to solid vapour pressures using entropy of fusion (ΔSf) and melting point, which this retention-correction workflow does not handle.

## Inputs

- Raw isothermal retention times for tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) from 1.0% w/v solutions in octane, 40-120°C in 20°C increments
- Natural-gas retention times per temperature (dead-time marker)
- Literature vapour pressure-temperature data for Me4Sn (reference compound) and literature Me4Sn ΔHv = 31.28 kJ/mol
- Boiling point (Tb) values where available for entropy-of-vaporization derivation

## Outputs

- Corrected mean retention times per compound per temperature (Table 1 format)
- ln(retention ratio) vs 1/T regression slopes per tetraorganostannane
- Enthalpies of vaporization (ΔHv, 31.3-61.3 kJ/mol range) and entropies of vaporization (89.1-93.4 J K-1 mol-1) as in Table 4, ordered Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn (e.g. vaporization_enthalpies.csv)

## How to apply

Load raw retention times from mixtures of organostannanes (1.0% w/v solutions in octane) measured isothermally from 40°C to 120°C in 20°C increments, and subtract the retention time of a natural-gas sample to obtain corrected retention times (dead-time correction). Average corrected times over 5 separate runs (3 for Allyl4Sn) per compound per temperature. For each compound at each temperature, compute the retention ratio relative to the reference compound Me4Sn, then regress ln(retention ratio) against 1/T (Kelvin) over the 40-120°C series. Convert each regression slope to the compound's enthalpy of vaporization by anchoring to the Me4Sn reference value ΔHv = 31.28 kJ/mol; where boiling point (Tb) data exist, derive the entropy of vaporization as ΔHv/Tb. Rationale: the GC relative-retention method transfers the known vapour pressure-temperature curve of the reference onto the analytes, so dead-time removal and replicate averaging are prerequisite to a linear van't Hoff relationship on the SE-30 column (chosen because it has relatively low McReynold's constants).

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Acquires isothermal retention-time data used for dead-time correction and van't Hoff regression)
- **FID detector** (Detection of organostannane peaks feeding retention-time measurements)
- **Chromatopac C-R6A integrator** (Integrates GC peaks to produce the raw retention-time values being corrected)
- **SE-30 capillary column** (Separation medium (2m x 0.55mm ID, 1.2µm film) on which retention ratios vs Me4Sn are measured; chosen for low McReynold's constants)

## Examples

```
python -c "import numpy as np; R=8.314; slope=np.polyfit(1/T_K, np.log(ratio_vs_Me4Sn), 1)[0]; dHv_kJ = 31.28 + slope*R/1000"  # regress ln(retention ratio) vs 1/T anchored to Me4Sn ΔHv=31.28 kJ/mol
```

## Evaluation signals

- The Me4Sn recomputed ΔHv matches the reference anchor (~31.3 kJ/mol), confirming correct slope-to-enthalpy conversion.
- All derived ΔHv values fall within the 31.3-61.3 kJ/mol Table 4 range for the seven tetraorganostannanes.
- ΔHv/Tb entropies (89.1-93.4 J K-1 mol-1) are populated only for compounds with available boiling points; missing-Tb compounds left blank.
- Corrected retention times equal raw values minus the natural-gas dead time at each temperature, and Allyl4Sn values are means of exactly 3 runs while others are means of 5.
- ln(retention ratio) vs 1/T regressions are linear over the 40-120°C series for every compound before slope conversion.

## Limitations

- A capillary column with C-87 stationary phase (identified as most satisfactory by Bidleman) was unavailable, so SE-30 was substituted.
- Values at 20°C required extrapolation from the 40-120°C measurement range.
- Lack of appropriate boiling point data prevents confirming whether estimation methods are adequate for other tetraalkylstannanes and tetraalkenylstannanes.
- Relationships based on MW for hydrocarbons cannot be used with tetraorganostannanes, constraining downstream QSPR transfer.

## Evidence
