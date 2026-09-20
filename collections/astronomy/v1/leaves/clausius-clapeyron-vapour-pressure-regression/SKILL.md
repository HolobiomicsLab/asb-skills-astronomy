---
name: clausius-clapeyron-vapour-pressure-regression
description: Use when you have a compound's vapour pressure measured at multiple isothermal
  temperatures (e.g. 40-120°C in 20°C increments) and need the derived enthalpy of
  vaporization from the log P vs 1/T slope.
when_to_use_negative:
- Input vapour pressures are extrapolated values (e.g. the 20°C values obtained by
  extrapolation rather than measured 40-120°C isothermal data) and a measured-quality
  fit is required
- The compound is a tetraalkenylstannane, for which hydrocarbon-based vapour pressure
  estimation methods were shown to be less successful
- You need vapour pressure prediction from molecular descriptors (MW or intrinsic
  molar volume relationships) rather than derivation of AHv from measured P-T data
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0203
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: 'Source instrument: isothermal capillary GC with FID that generated the corrected
    retention times underlying the vapour pressure-temperature data being regressed'
- name: pandas
  role: Load the tabulated corrected retention-time-derived vapour pressure values
    per temperature point
- name: numpy.polyfit / scipy.stats.linregress
  role: Least-squares linear fit of log P vs 1/T returning slope, intercept, and r
    used to compute AHv = -slope x 2.303 x R
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:081a54ab7e46b2c6950b7204a9adfeeb34618103e6ecd66b9ba3bbcf8e8f56ae
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Shimadzu GC-15A gas chromatograph
  - numpy.polyfit / scipy.stats.linregress
  - pandas
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# clausius-clapeyron-vapour-pressure-regression

## Summary

Derives the enthalpy (and entropy) of vaporization of a compound by least-squares fitting of log P versus 1/T, following the Clausius-Clapeyron treatment used for tetramethylstannane (Me4Sn) reference data in Hawker's capillary-GC study of tetraorganostannane vapour pressures.

## When to use

Use when you have a compound's vapour pressure measured at multiple isothermal temperatures (e.g. 40-120°C in 20°C increments) and need the derived enthalpy of vaporization from the log P vs 1/T slope.

## When NOT to use

- Input vapour pressures are extrapolated values (e.g. the 20°C values obtained by extrapolation rather than measured 40-120°C isothermal data) and a measured-quality fit is required
- The compound is a tetraalkenylstannane, for which hydrocarbon-based vapour pressure estimation methods were shown to be less successful
- You need vapour pressure prediction from molecular descriptors (MW or intrinsic molar volume relationships) rather than derivation of AHv from measured P-T data

## Inputs

- Tabulated vapour pressure values (kPa) at 5 isothermal temperatures (40-120°C), derived from corrected retention times via a reference compound (Me4Sn)
- Mean retention times from 5 replicate isothermal GC runs per temperature (Shimadzu GC-15A, SE-30 capillary column)

## Outputs

- Regression parameters: slope, intercept, Pearson correlation coefficient r
- Enthalpy of vaporization AHv in kJ/mol (and entropy of vaporization where boiling point data exist)
- Fitted parameters written to a CSV file (e.g. clausius_fit_me4sn.csv)

## How to apply

Load the tabulated vapour pressure values measured at isothermal temperature points (here 40, 60, 80, 100, 120°C, each the mean of 5 replicate GC runs). Convert temperatures to reciprocal absolute temperature (1/T in K^-1) and take the base-10 logarithm of pressure in kPa. Fit log P vs 1/T with a least-squares regression (numpy.polyfit or scipy.stats.linregress) to obtain slope, intercept, and Pearson r. Derive the enthalpy of vaporization as AHv = -slope x 2.303 x R and report it in kJ/mol; with boiling point data available, the entropy of vaporization can also be derived. Validate against reference values — for Me4Sn the expected fit is slope -1.66e3, intercept 6.65, r = 1.00, AHv = 31.28 kJ/mol (reported 31.3 kJ/mol, 89.1 J/K/mol in Table 4).

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Source instrument: isothermal capillary GC with FID that generated the corrected retention times underlying the vapour pressure-temperature data being regressed)
- **pandas** (Load the tabulated corrected retention-time-derived vapour pressure values per temperature point)
- **numpy.polyfit / scipy.stats.linregress** (Least-squares linear fit of log P vs 1/T returning slope, intercept, and r used to compute AHv = -slope x 2.303 x R)

## Examples

```
import numpy as np, pandas as pd; d = pd.read_csv('me4sn_vp.csv'); invT = 1/(d.temp_C + 273.15); slope, intercept, r, p, se = __import__('scipy.stats', fromlist=['linregress']).linregress(invT, np.log10(d.P_kPa)); AHv = -slope * 2.303 * 8.314 / 1000
```

## Evaluation signals

- Fitted slope for Me4Sn should match the reported reference value of -1.66e3 (intercept 6.65, r = 1.00)
- Derived AHv should reproduce 31.28 kJ/mol (reported 31.3 kJ/mol) via AHv = -slope x 2.303 x R
- Entropy of vaporization should match the reported 89.1 J/K/mol for Me4Sn (Table 4)
- Residuals of log P vs 1/T should be near-zero (Pearson r = 1.00 as reported)
- Output CSV (clausius_fit_me4sn.csv) contains slope, intercept, r, and AHv consistent with the regression

## Limitations

- Vapour pressures at 20°C were obtained only by extrapolation from the measured 40-120°C range, so fits anchored to extrapolated points carry extra uncertainty
- Whether estimation methods are adequate for other tetraalkylstannanes and tetraalkenylstannanes is uncertain because of lack of appropriate boiling point data, limiting entropy-of-vaporization derivation
- For solid tetraorganostannanes (MP > 25°C), Equation (9) data must be converted to solid vapour pressure using entropy of fusion (ASf) and melting point, an extra required input
- Published vapour pressure data for some organostannanes are sparse or vary considerably, complicating validation

## Evidence
