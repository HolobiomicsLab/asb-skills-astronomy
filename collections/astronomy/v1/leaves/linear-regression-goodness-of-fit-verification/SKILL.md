---
name: linear-regression-goodness-of-fit-verification
description: Use when tabulated paired measurements (e.g. vapour pressure vs temperature
  at 40-120°C in 20°C increments) must be reduced to a linear model whose slope yields
  a derived physical constant, and the fit must be validated against reported parameters.
when_to_use_negative:
- The input data are not expected to follow a linear model — e.g. the article notes
  vapour pressure relationships with molecular weight are 'parallel but distinct'
  for hydrocarbons vs organostannanes, so a single pooled regression across compound
  classes is inappropriate
- The goal is estimating vapour pressures for tetraalkenylstannanes, for which hydrocarbon-based
  estimation methods were reported as less successful — regression from surrogate
  data should not be trusted
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: numpy.polyfit
  role: least-squares fit of log P vs 1/T to obtain slope and intercept
- name: scipy.stats.linregress
  role: linear regression returning slope, intercept, and Pearson r for goodness-of-fit
    verification
- name: pandas
  role: loading the tabulated corrected retention-time-derived vapour pressure values
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:b919771996e0b829e673982026c4fc588047f0462e6453a23a32523d431ad95d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Shimadzu GC-15A gas chromatograph
  - numpy.polyfit
  - pandas
  - scipy.stats.linregress
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# linear-regression-goodness-of-fit-verification

## Summary

Fits a least-squares linear regression (e.g. log P versus 1/T for a Clausius-Clapeyron plot) and verifies goodness of fit by comparing the recovered slope, intercept, and Pearson r against published reference values before deriving physical quantities such as enthalpy of vaporization.

## When to use

Use when tabulated paired measurements (e.g. vapour pressure vs temperature at 40-120°C in 20°C increments) must be reduced to a linear model whose slope yields a derived physical constant, and the fit must be validated against reported parameters.

## When NOT to use

- The input data are not expected to follow a linear model — e.g. the article notes vapour pressure relationships with molecular weight are 'parallel but distinct' for hydrocarbons vs organostannanes, so a single pooled regression across compound classes is inappropriate
- The goal is estimating vapour pressures for tetraalkenylstannanes, for which hydrocarbon-based estimation methods were reported as less successful — regression from surrogate data should not be trusted

## Inputs

- CSV/table of corrected mean retention times or vapour pressures for tetraorganostannanes at 40-120°C in 20°C increments (Table 1)
- Reference compound (Me4Sn) literature vapour pressure-temperature data
- Universal gas constant R = 8.314 J/K/mol

## Outputs

- Fitted slope, intercept, and Pearson correlation coefficient r of log P vs 1/T
- Derived enthalpy of vaporization AHv (kJ/mol) and, where boiling point data exist, entropy of vaporization (J/K/mol)
- CSV of fitted regression parameters (e.g. clausius_fit_me4sn.csv)

## How to apply

Load the tabulated values (e.g. corrected retention-time-derived vapour pressures for Me4Sn at 40, 60, 80, 100, and 120°C, each the mean of 5 replicate runs) with pandas. Transform the axes as the underlying equation demands — for Clausius-Clapeyron, convert temperatures to reciprocal kelvin (1/T) and take base-10 log of pressure in kPa. Fit with numpy.polyfit or scipy.stats.linregress to obtain slope, intercept, and Pearson r. Convert the slope to the physical quantity using the equation's prefactor, here AHv = -slope x 2.303 x R in kJ/mol. Treat the fit as correct only if slope, intercept, r, and the derived AHv reproduce the published reference values (slope -1.66e3, intercept 6.65, r = 1.00, AHv = 31.28 kJ/mol for Me4Sn, Table 4); write the fitted parameters to an output file such as clausius_fit_me4sn.csv.

## Related tools

- **numpy.polyfit** (least-squares fit of log P vs 1/T to obtain slope and intercept)
- **scipy.stats.linregress** (linear regression returning slope, intercept, and Pearson r for goodness-of-fit verification)
- **pandas** (loading the tabulated corrected retention-time-derived vapour pressure values)

## Examples

```
from scipy.stats import linregress; import numpy as np; slope, intercept, r, p, se = linregress(1/(np.array([40,60,80,100,120])+273.15), np.log10(P_kPa)); Hv = -slope*2.303*8.314/1000
```

## Evaluation signals

- Pearson correlation coefficient r of the log P vs 1/T fit approaches 1.00 (article reports r = 1.00 for Me4Sn)
- Fitted slope matches the published reference value of -1.66e3 and intercept 6.65
- Derived AHv = -slope x 2.303 x R falls at 31.28-31.3 kJ/mol, matching Table 4
- Entropy of vaporization derived where boiling point data exist equals 89.1 J/K/mol for Me4Sn
- Input vapour pressures must come from the mean of 5 separate retention-time runs (3 for Allyl4Sn) — verify replicate counts before fitting

## Limitations

- Vapour pressure values at 20°C were obtained by extrapolation, so regression-derived constants inherit extrapolation uncertainty
- For tetraorganostannanes solid at room temperature (MP > 25°C), Equation (9) data need conversion to solid vapour pressure using entropy of fusion (ASf) and melting point — the liquid-phase regression alone is insufficient
- Lack of appropriate boiling point data prevents validating derived entropies of vaporization for many tetraalkylstannanes and tetraalkenylstannanes

## Evidence
