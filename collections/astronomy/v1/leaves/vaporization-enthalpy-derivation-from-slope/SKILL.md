---
name: vaporization-enthalpy-derivation-from-slope
description: Use when you have vapour pressure measurements at multiple isothermal
  temperatures (e.g. 40-120°C in 20°C increments) and need the enthalpy of vaporization
  from the Clausius-Clapeyron slope.
when_to_use_negative:
- Input is a single vapour pressure at one temperature rather than a pressure-temperature
  series — the slope requires multiple isothermal points
- The goal is to estimate vapour pressures of unknown tetraorganostannanes from a
  reference compound (Equation 9 theory section) rather than to derive thermodynamic
  quantities from an existing P-T series
- The compound is solid at room temperature (MP > 25°C) and solid-state vapour pressure
  is needed via entropy of fusion correction, rather than the liquid-form Clausius-Clapeyron
  enthalpy
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: 'Source instrument: determines equilibrium vapour pressures isothermally (40-120°C)
    via corrected retention times that feed the Clausius-Clapeyron regression'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:660bd0e8d2fb15c65f5fef09abd7439575fbec98d8d46dc357f4d321b55576ab
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Shimadzu GC-15A gas chromatograph
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# vaporization-enthalpy-derivation-from-slope

## Summary

Derives the enthalpy of vaporization (AHv) of a compound by fitting a least-squares linear regression of log10(vapour pressure, kPa) versus reciprocal absolute temperature (1/T, K^-1) and converting the slope via AHv = -slope x 2.303 x R, as done for tetramethylstannane (Me4Sn) in Hawker's 1992 Chemosphere study of tetraorganostannane vapour pressures.

## When to use

Use when you have vapour pressure measurements at multiple isothermal temperatures (e.g. 40-120°C in 20°C increments) and need the enthalpy of vaporization from the Clausius-Clapeyron slope.

## When NOT to use

- Input is a single vapour pressure at one temperature rather than a pressure-temperature series — the slope requires multiple isothermal points
- The goal is to estimate vapour pressures of unknown tetraorganostannanes from a reference compound (Equation 9 theory section) rather than to derive thermodynamic quantities from an existing P-T series
- The compound is solid at room temperature (MP > 25°C) and solid-state vapour pressure is needed via entropy of fusion correction, rather than the liquid-form Clausius-Clapeyron enthalpy

## Inputs

- Table of corrected retention-time-derived equilibrium vapour pressures (kPa) per compound at 40-120°C in 20°C increments (e.g. Table 1/2 data for Me4Sn, the reference compound)
- Isothermal temperature points in °C at which each pressure was measured (40, 60, 80, 100, 120°C)

## Outputs

- Clausius-Clapeyron regression parameters: slope, intercept, Pearson r (expected for Me4Sn: -1.66e3, 6.65, 1.00)
- Derived enthalpy of vaporization AHv in kJ/mol (Me4Sn: 31.3 kJ/mol) and entropy of vaporization in J/K/mol (89.1 J/K/mol)
- Fitted-parameter CSV file, e.g. clausius_fit_me4sn.csv

## How to apply

Load the tabulated corrected retention-time-derived vapour pressures (e.g. Me4Sn values measured isothermally at 40, 60, 80, 100 and 120°C, each the mean of 5 replicate GC runs) using pandas. Convert temperatures to reciprocal absolute temperature (1/T in K^-1) and compute log10 of pressure in kPa. Fit log P versus 1/T with numpy.polyfit or scipy.stats.linregress to obtain slope, intercept and Pearson r. Compute AHv = -slope x 2.303 x R and report in kJ/mol; the Me4Sn reference fit gives slope -1.66e3, intercept 6.65, r = 1.00 and AHv = 31.28 kJ/mol (entropy of vaporization 89.1 J/K/mol, Table 4). Write fitted parameters to a CSV (e.g. clausius_fit_me4sn.csv) and compare against reference values to validate the fit.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Source instrument: determines equilibrium vapour pressures isothermally (40-120°C) via corrected retention times that feed the Clausius-Clapeyron regression)

## Examples

```
import numpy as np; slope, intercept = np.polyfit(1/(T_C+273.15), np.log10(P_kPa), 1); AHv = -slope*2.303*8.314/1000  # kJ/mol, Me4Sn -> 31.28
```

## Evaluation signals

- Fitted slope for Me4Sn should match the reference value of -1.66e3 (log10 kPa vs K^-1 units)
- Fitted intercept should match the reference value of 6.65
- Pearson correlation coefficient should be 1.00 (r ~ 1) for the Me4Sn fit
- Derived AHv should equal -slope x 2.303 x 8.314 / 1000 ≈ 31.28 kJ/mol, consistent with Table 4's 31.3 kJ/mol
- Input data must be log base-10 pressure in kPa and 1/T in K^-1; using ln P or °C temperatures changes the slope and breaks the 2.303 conversion factor

## Limitations

- Vapour pressures between 40 and 120°C were measured by capillary GC with corrected retention times (means of 5 runs, 3 for Allyl4Sn); values at 20°C are extrapolations and carry additional uncertainty
- For solid tetraorganostannanes (MP > 25°C), Equation (9) data must be converted to solid vapour pressures using entropy of fusion (ASf) and melting point before thermodynamic interpretation
- Only where boiling point information was available could entropies of vaporization be derived (Table 4)

## Evidence
