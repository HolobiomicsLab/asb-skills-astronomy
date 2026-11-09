---
name: regression-verification-against-reported-statistics
description: Use when a paper reports a fitted equation (e.g. lnP = -5.82e-2 MW +
  12.27, r = 0.99) together with a small table of raw values, and you need to independently
  reproduce the fit before reusing its coefficients.
when_to_use_negative:
- 'Applying an MW-based relationship derived for hydrocarbons to tetraorganostannanes
  — the paper states such relationships form distinct, non-transferable lines: ''any
  existing relationship involving MW for hydrocarbons can not be used with tetraorganostannes'''
- Modelling lnP against intrinsic molar volume instead of MW, where organostannanes
  and hydrocarbons share a single unique relationship — use that skill rather than
  MW-based regression verification
- Estimating vapour pressures of tetraalkenylstannanes with hydrocarbon-based methods,
  which the paper found were 'less successful' for that class
edam_topics:
- http://edamontology.org/topic_2269
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:6b9d0eb0385e230efe42d979094b9c388b33ef496a632420a2fb04485f362725
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FID detector
  - Shimadzu GC-15A gas chromatograph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# regression-verification-against-reported-statistics

## Summary

Refits a published linear regression (here, lnP (kPa) vs molecular weight for seven tetraorganostannanes at 20°C, Equation 8) from the underlying tabulated data and checks the refitted slope, intercept, and Pearson r against the reported values within numerical tolerance.

## When to use

Use when a paper reports a fitted equation (e.g. lnP = -5.82e-2 MW + 12.27, r = 0.99) together with a small table of raw values, and you need to independently reproduce the fit before reusing its coefficients.

## When NOT to use

- Applying an MW-based relationship derived for hydrocarbons to tetraorganostannanes — the paper states such relationships form distinct, non-transferable lines: 'any existing relationship involving MW for hydrocarbons can not be used with tetraorganostannes'
- Modelling lnP against intrinsic molar volume instead of MW, where organostannanes and hydrocarbons share a single unique relationship — use that skill rather than MW-based regression verification
- Estimating vapour pressures of tetraalkenylstannanes with hydrocarbon-based methods, which the paper found were 'less successful' for that class

## Inputs

- Table 3 vapour pressures (kPa) at 20°C for seven tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn)
- Molecular weights of the seven tetraorganostannanes
- Reported regression statistics to verify: slope = -5.82e-2, intercept = 12.27, r = 0.99

## Outputs

- Fitted slope, intercept, and Pearson correlation coefficient for lnP vs MW
- Per-compound residuals from the refitted regression
- Validation flag / discrepancy report comparing refit vs reported statistics
- Results table (e.g. vapour_pressure_regression.csv)

## How to apply

Load the tabulated data (Table 3 vapour pressures at 20°C, extrapolated from 40–120°C capillary GC measurements, for Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) and their molecular weights. Transform each vapour pressure to lnP (kPa) and run an ordinary least-squares regression of lnP on MW over the n = 7 points using scipy.stats.linregress or numpy.polyfit. Extract slope, intercept, and Pearson r, and compare against the reported values (slope = -5.82e-2, intercept = 12.27, r = 0.99) within reasonable numerical tolerance; flag any discrepancy rather than silently accepting the published coefficients. Compute per-compound residuals to reveal which compounds drive any mismatch, and persist all results (e.g. vapour_pressure_regression.csv) for auditability.

## Examples

```
from scipy.stats import linregress; slope, intercept, r, p, se = linregress(MW, lnP)
```

## Evaluation signals

- Refitted slope matches -5.82e-2 within numerical tolerance
- Refitted intercept matches 12.27 within numerical tolerance
- Pearson r reproduces the reported r = 0.99
- Residuals are small and show no single compound dominating the fit (n = 7 data points)
- Regression is performed on natural-log-transformed pressures (lnP in kPa), not raw P or log10 P

## Limitations

- The 20°C inputs are extrapolated, not directly measured — the regression inherits extrapolation error from the 40–120°C capillary GC measurements
- Tetra-n-butylstannane was the largest compound whose vapour pressure could be determined under the experimental conditions, so the MW range covered is bounded
- Retention times were means of 5 runs (3 for Allyl4Sn), so per-compound measurement precision is uneven
- Vapour pressures of solid tetraorganostannanes (MP > 25°C) require conversion using entropy of fusion and melting point, which this regression does not handle

## Evidence
