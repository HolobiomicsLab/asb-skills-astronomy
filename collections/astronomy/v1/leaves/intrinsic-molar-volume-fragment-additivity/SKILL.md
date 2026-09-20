---
name: intrinsic-molar-volume-fragment-additivity
description: Use when you have vapour pressure (lnP, kPa) data for chemically heterogeneous
  compound classes (e.g. hydrocarbons plus organometallics like tetraorganostannanes)
  and need a single unified QSPR descriptor or a structural predictor of P.
when_to_use_negative:
- 'When the descriptor of interest is molecular weight: the article shows hydrocarbons
  and organostannanes (and organoplumbanes) form three parallel but distinct lnP–MW
  relationships, so any existing MW-based relationship for hydrocarbons cannot be
  used with tetraorganostannanes.'
- 'When estimating vapour pressures of solid tetraorganostannanes (MP > 25°C) directly:
  Equation (9) values must first be converted to solid vapour pressures using entropy
  of fusion (ΔSf) and melting point.'
- 'When the target compounds are tetraalkenylstannanes: current hydrocarbon-based
  estimation methods proved less successful for these, and the paper provides no validated
  alternative.'
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Source of the measured equilibrium vapour pressures (40–120°C) for the tetraorganostannanes
    that, after extrapolation to 20°C, form the stannane half of the lnP–V dataset
- name: scipy.stats.linregress
  role: Ordinary least-squares fit of lnP (kPa) on intrinsic molar volume V for the
    13 combined compounds
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:20b48d68ff18262e509086699bef73ee30c0e82fa894c745799ae5cf5f799357
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Shimadzu GC-15A gas chromatograph
  - scipy.stats.linregress
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# intrinsic-molar-volume-fragment-additivity

## Summary

Computes intrinsic molar volumes (V, cm3/mol) for compounds via the Hickey and Passino-Reader fragment additivity procedure and demonstrates that lnP of both hydrocarbons and tetraorganostannanes collapses onto a single linear QSPR in V, unlike molecular weight which yields parallel-but-distinct relationships.

## When to use

Use when you have vapour pressure (lnP, kPa) data for chemically heterogeneous compound classes (e.g. hydrocarbons plus organometallics like tetraorganostannanes) and need a single unified QSPR descriptor or a structural predictor of P.

## When NOT to use

- When the descriptor of interest is molecular weight: the article shows hydrocarbons and organostannanes (and organoplumbanes) form three parallel but distinct lnP–MW relationships, so any existing MW-based relationship for hydrocarbons cannot be used with tetraorganostannanes.
- When estimating vapour pressures of solid tetraorganostannanes (MP > 25°C) directly: Equation (9) values must first be converted to solid vapour pressures using entropy of fusion (ΔSf) and melting point.
- When the target compounds are tetraalkenylstannanes: current hydrocarbon-based estimation methods proved less successful for these, and the paper provides no validated alternative.

## Inputs

- Vapour pressure values (P in kPa, converted to lnP) for seven tetraorganostannanes extrapolated to 20°C from capillary GC measurements (40–120°C, 20°C increments, means of 5 runs; 3 for Allyl4Sn)
- Literature vapour pressures for the hydrocarbon compounds cited in the article
- Molecular structures of all 13 compounds (fragment inventory for Hickey and Passino-Reader additivity)

## Outputs

- Intrinsic molar volumes V (cm3/mol) for all 13 compounds
- Fitted regression parameters: slope -9.83e-3, intercept 8.99, r = 0.99, n = 13
- vapour_pressure_regression.csv recording slope, intercept, correlation coefficient and sample size

## How to apply

First assemble measured lnP values: for the seven tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) use capillary GC-derived vapour pressures extrapolated to 20°C, with Me4Sn as the reference compound; add literature hydrocarbon values to give n = 13. Next, compute each compound's intrinsic molar volume using the Hickey and Passino-Reader fragment additivity procedure, summing fragment contributions over the structure (Sn-containing fragments for the stannanes, hydrocarbon fragments otherwise). Then fit an ordinary least-squares linear regression of lnP (kPa) on V in Python (scipy.stats.linregress or numpy.polyfit). The regression should recover lnP (kPa) = -9.83 x 10^-3 V + 8.99 with r = 0.99, confirming a single unique lnP–intrinsic-molar-volume relationship across both compound classes.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Source of the measured equilibrium vapour pressures (40–120°C) for the tetraorganostannanes that, after extrapolation to 20°C, form the stannane half of the lnP–V dataset)
- **scipy.stats.linregress** (Ordinary least-squares fit of lnP (kPa) on intrinsic molar volume V for the 13 combined compounds)

## Examples

```
from scipy.stats import linregress; res = linregress(V_cm3_mol, lnP_kPa); print(res.slope, res.intercept, res.rvalue)  # expect -9.83e-3, 8.99, ~0.99
```

## Evaluation signals

- Regression slope matches -9.83 x 10^-3 (per cm3/mol) within numerical tolerance
- Regression intercept matches 8.99 (lnP in kPa) within numerical tolerance
- Correlation coefficient r ≈ 0.99 and sample size n = 13 (7 stannanes + 6 hydrocarbons)
- Intrinsic molar volumes computed additively must be in cm3/mol so the lnP–V slope is dimensionally consistent
- Separate lnP–MW fits for hydrocarbons and stannanes should yield distinct parallel lines, confirming V (not MW) is the unifying descriptor

## Limitations

- Equation (9) applies to subcooled-liquid vapour pressures; for tetraorganostannanes solid at room temperature (MP > 25°C), conversion to solid vapour pressure requires entropy of fusion (ΔSf) and melting point as extra inputs.
- Applicability to triorganostannanes is uncertain because little vapour pressure data exists and published values vary considerably.
- Lack of appropriate boiling point data prevents confirming whether the estimation approach is adequate for other tetraalkylstannanes and tetraalkenylstannanes.
- Tetra-n-butylstannane was the largest compound whose vapour pressure could be determined under the experimental conditions, bounding the calibration range.

## Evidence
