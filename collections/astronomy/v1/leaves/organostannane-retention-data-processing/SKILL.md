---
name: organostannane-retention-data-processing
description: Use when corrected capillary-GC retention-time data for tetraorganostannanes
  (Shimadzu GC-15A/FID, SE-30 column) must be converted to vapour pressures via a
  reference compound and Clausius-Clapeyron regression, or combined with hydrocarbon
  literature data into lnP vs intrinsic-molar-volume QSPRs.
when_to_use_negative:
- Estimating vapour pressures of triorganostannanes or organoplumbanes, where little
  data exists, published values vary considerably, and applicability of the lnP–V
  relationship is uncertain
- Solid tetraorganostannanes (melting point > 25°C), whose Equation (9) liquid vapour
  pressures must first be converted to solid vapour pressure using entropy of fusion
  (ΔSf) and melting point
- Transferring hydrocarbon molecular-weight (MW) based relationships to tetraorganostannanes
  — the article shows MW relationships are parallel but distinct, so MW regressions
  cannot be used across the two classes
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Instrument used to determine equilibrium vapour pressures isothermally via
    the capillary GC relative-retention method
- name: FID detector
  role: Flame ionization detection of organostannane peaks during isothermal retention-time
    runs
- name: Chromatopac C-R6A integrator
  role: Integration of FID chromatographic peaks to yield raw retention times
- name: SE-30 capillary column
  role: 2 m × 0.55 mm ID, 1.2 µm film Alltech column used for separations; SE-30 substituted
    because a C-87 stationary-phase column was unavailable
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:b1c4b150b72590b7b17d0cf54062936149e5245474648829a6e678f5e5655cc4
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# organostannane-retention-data-processing

## Summary

Converts corrected GC retention times for tetraorganostannanes into equilibrium vapour pressures (40–120°C, extrapolated to 20°C) using tetramethylstannane (Me4Sn) as reference compound and Clausius-Clapeyron regression, then fits lnP versus intrinsic molar volume across organostannanes plus hydrocarbons.

## When to use

Use when corrected capillary-GC retention-time data for tetraorganostannanes (Shimadzu GC-15A/FID, SE-30 column) must be converted to vapour pressures via a reference compound and Clausius-Clapeyron regression, or combined with hydrocarbon literature data into lnP vs intrinsic-molar-volume QSPRs.

## When NOT to use

- Estimating vapour pressures of triorganostannanes or organoplumbanes, where little data exists, published values vary considerably, and applicability of the lnP–V relationship is uncertain
- Solid tetraorganostannanes (melting point > 25°C), whose Equation (9) liquid vapour pressures must first be converted to solid vapour pressure using entropy of fusion (ΔSf) and melting point
- Transferring hydrocarbon molecular-weight (MW) based relationships to tetraorganostannanes — the article shows MW relationships are parallel but distinct, so MW regressions cannot be used across the two classes

## Inputs

- Corrected mean GC retention times for tetraorganostannanes at 40, 60, 80, 100, 120°C (Table 1; means of 5 runs, 3 for Allyl4Sn)
- Me4Sn reference vapour pressure–temperature data (5 isothermal points)
- Table 3 vapour pressures extrapolated to 20°C for the seven tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn)
- Literature ΔHv and lnP values for reference hydrocarbons (combined n = 13 dataset)
- Intrinsic molar volumes (V, cm3/mol) computed by Hickey and Passino-Reader fragment additivity

## Outputs

- Clausius-Clapeyron fit parameters for Me4Sn (slope, intercept, r, ΔHv) written to clausius_fit_me4sn.csv
- Calculated equilibrium vapour pressures for tetraorganostannanes (40–120°C and extrapolated 20°C values)
- Combined lnP vs intrinsic molar volume regression (slope, intercept, r, n) written to vapour_pressure_regression.csv
- Combined ΔHv vs intrinsic molar volume dataset with regression statistics and residuals written to dhv_intrinsic_volume_regression.csv

## How to apply

Determine retention times isothermally from 1.0% w/v organostannane solutions in octane, ramping 40°C to 120°C in 20°C increments on the SE-30 capillary column, averaging 5 runs per compound (3 for Allyl4Sn). Correct each raw retention time by subtracting the retention time of a natural gas sample, then load the tabulated corrected values (Table 1) with pandas. For the reference compound Me4Sn, convert temperature to 1/T (K^-1) and pressure to log10 P (kPa), fit log P = slope × 1/T + intercept by least squares (numpy.polyfit / scipy.stats.linregress), and derive the enthalpy of vaporization as ΔHv = -slope × 2.303 × R (Me4Sn reference: slope ≈ -1.66e3, intercept ≈ 6.65, r ≈ 1.00, ΔHv ≈ 31.28 kJ/mol). Use this reference relationship to calculate vapour pressures of the remaining six tetraorganostannanes and extrapolate to 20°C. For the QSPR step, compute intrinsic molar volumes by the Hickey and Passino-Reader fragment additivity procedure and regress lnP (kPa) on V (cm3/mol) over the combined n = 13 organostannane-plus-hydrocarbon set, expecting lnP = -9.83e-3 V + 8.99 (r = 0.99); similarly fit ΔHv = 3.11e-1 V + 11.83 (r = 0.96). Write fitted parameters to CSV outputs (e.g. clausius_fit_me4sn.csv, dhv_intrinsic_volume_regression.csv).

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Instrument used to determine equilibrium vapour pressures isothermally via the capillary GC relative-retention method)
- **FID detector** (Flame ionization detection of organostannane peaks during isothermal retention-time runs)
- **Chromatopac C-R6A integrator** (Integration of FID chromatographic peaks to yield raw retention times)
- **SE-30 capillary column** (2 m × 0.55 mm ID, 1.2 µm film Alltech column used for separations; SE-30 substituted because a C-87 stationary-phase column was unavailable)

## Examples

```
from scipy.stats import linregress; s,i,r,p,se = linregress(1/(T_C+273.15), np.log10(P_kPa)); print(-s*2.303*8.314/1000)  # ΔHv kJ/mol for Me4Sn
```

## Evaluation signals

- Me4Sn Clausius-Clapeyron regression reproduces slope ≈ -1.66e3, intercept ≈ 6.65, r ≈ 1.00
- Derived ΔHv for Me4Sn ≈ 31.28 kJ/mol (Table 4 reports 31.3 kJ/mol) with ΔSv ≈ 89.1 J/K/mol
- Corrected retention times equal raw values minus natural gas retention time; every pressure used is a mean of 5 runs (3 for Allyl4Sn)
- Combined lnP (kPa) vs intrinsic molar volume regression over n = 13 gives lnP = -9.83e-3 V + 8.99 with r ≈ 0.99
- Combined ΔHv vs intrinsic molar volume regression over n = 13 gives ΔHv = 3.11e-1 V + 11.83 with r ≈ 0.96

## Limitations

- Under the experimental conditions, tetra-n-butylstannane was the largest compound whose vapour pressure could be determined
- A capillary column with C-87 stationary phase was unavailable, so SE-30 was used instead (relatively low McReynold's constants)
- Vapour pressures for Allyl4Sn rest on only 3 replicate runs rather than 5
- Hydrocarbon estimation methods were less successful for tetraalkenylstannanes, and lack of boiling point data prevents confirming adequacy for other tetraalkylstannanes
- For solid tetraorganostannanes (MP > 25°C), Equation (9) data require conversion with entropy of fusion (ΔSf) and melting point

## Evidence
