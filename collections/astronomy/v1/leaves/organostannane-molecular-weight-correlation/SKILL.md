---
name: organostannane-molecular-weight-correlation
description: Use when you have extrapolated 20°C vapour pressures and molecular weights
  for the seven tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn,
  n-Bu4Sn) and need to test or reproduce the lnP–MW QSPR.
when_to_use_negative:
- Attempting to apply a hydrocarbon-derived MW–vapour-pressure relationship directly
  to tetraorganostannanes — the article states 'any existing relationship involving
  MW for hydrocarbons can not be used with tetraorganostannes'
- Relating lnP to intrinsic molar volume instead of MW — the article treats lnP versus
  intrinsic molar volume as a separate (single, unique) relationship and fits it independently
- Predicting vapour pressures for tetraalkenylstannanes or triorganostannanes with
  this equation — the article reports hydrocarbon estimation methods were less successful
  for tetraalkenylstannanes and applicability to triorganostannanes is uncertain
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Source instrument whose 40–120°C isothermal capillary GC measurements (with
    Me4Sn as reference compound) produced the retention-time data extrapolated to
    20°C vapour pressures used as regression inputs
- name: FID detector
  role: Flame ionization detector on the GC-15A used to acquire the retention times
    underlying the vapour pressure dataset
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:8aa865e5ea6a53e2e32fc8edcbc76869473acc24fd118c606995f6d2ca6fb647
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FID detector
  - Shimadzu GC-15A gas chromatograph
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# organostannane-molecular-weight-correlation

## Summary

Fits an ordinary least-squares linear regression of ln(vapour pressure) versus molecular weight for tetraorganostannanes at 20°C, reproducing Hawker's Equation 8 (lnP(kPa) = -5.82e-2 MW + 12.27, r = 0.99) to quantify the inverse MW–volatility relationship.

## When to use

Use when you have extrapolated 20°C vapour pressures and molecular weights for the seven tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) and need to test or reproduce the lnP–MW QSPR.

## When NOT to use

- Attempting to apply a hydrocarbon-derived MW–vapour-pressure relationship directly to tetraorganostannanes — the article states 'any existing relationship involving MW for hydrocarbons can not be used with tetraorganostannes'
- Relating lnP to intrinsic molar volume instead of MW — the article treats lnP versus intrinsic molar volume as a separate (single, unique) relationship and fits it independently
- Predicting vapour pressures for tetraalkenylstannanes or triorganostannanes with this equation — the article reports hydrocarbon estimation methods were less successful for tetraalkenylstannanes and applicability to triorganostannanes is uncertain

## Inputs

- Table 3 vapour pressures at 20°C (kPa) for seven tetraorganostannanes, extrapolated from 40–120°C GC measurements
- Molecular weights (g/mol) of Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn

## Outputs

- Fitted regression equation lnP(kPa) = slope x MW + intercept with slope, intercept, and Pearson r
- Per-compound residuals table written to vapour_pressure_regression.csv

## How to apply

Load the Table 3 vapour pressures at 20°C (values extrapolated from 40–120°C capillary GC measurements made with a Shimadzu GC-15A with FID detector and SE-30 capillary column) together with the molecular weights of the seven tetraorganostannanes. Transform each vapour pressure to its natural logarithm, lnP (kPa), then run an ordinary least-squares regression of lnP on MW over the n = 7 data points using scipy.stats.linregress or numpy.polyfit. Extract the slope, intercept, and Pearson r and compare to the reported Equation 8 values (slope = -5.82e-2, intercept = 12.27, r = 0.99) within numerical tolerance. Compute per-compound residuals to identify outliers, and write slope, intercept, r, and residuals to vapour_pressure_regression.csv. The rationale is that MW-based relationships for hydrocarbons cannot be transferred to tetraorganostannanes, so the stannane-specific equation must be fitted from measured stannane data.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Source instrument whose 40–120°C isothermal capillary GC measurements (with Me4Sn as reference compound) produced the retention-time data extrapolated to 20°C vapour pressures used as regression inputs)
- **FID detector** (Flame ionization detector on the GC-15A used to acquire the retention times underlying the vapour pressure dataset)

## Examples

```
from scipy.stats import linregress; res = linregress(mw, lnP_kPa); print(res.slope, res.intercept, res.rvalue)  # expect ~ -5.82e-2, 12.27, 0.99
```

## Evaluation signals

- Fitted slope reproduces the reported -5.82 x 10^-2 (lnP per g/mol) within reasonable numerical tolerance
- Fitted intercept reproduces 12.27 within tolerance
- Pearson correlation coefficient equals the reported r = 0.99
- Residuals across the seven compounds show no single compound dominating the fit (per-compound residuals written to vapour_pressure_regression.csv)
- n = 7 data points used, matching the seven tetraorganostannanes from Me4Sn through n-Bu4Sn (tetra-n-butylstannane being the largest compound measurable under the experimental conditions)

## Limitations

- The 20°C vapour pressures are extrapolated, not directly measured, from the 40–120°C range, so regression inputs carry extrapolation uncertainty
- MW relationships for hydrocarbons cannot be substituted for the stannane-specific equation; organostannanes, hydrocarbons, and organoplumbanes form three parallel but distinct lnP–MW relationships
- Few molecular descriptors beyond MW exist for organostannanes, limiting extension of this correlation to other descriptors
- For solid tetraorganostannanes (melting point > 25°C), the derived liquid vapour pressures need conversion using entropy of fusion and melting point

## Evidence
