---
name: qspr-property-relationship-fitting
description: Use when you have a small (n≈7–13) compound set with measured or extrapolated
  vapour pressures plus a molecular descriptor (MW or intrinsic molar volume) and
  need to fit and validate a linear lnP–descriptor relationship.
when_to_use_negative:
- The target property involves triorganostannanes — published vapour pressure values
  vary considerably and applicability of Equation (9) to triorganostannanes is uncertain
- The compounds are solid at room temperature (MP > 25°C) — Equation (9) data need
  conversion to solid vapour pressure using entropy of fusion (ΔSf) and melting point,
  an extra required input
- You need an MW-based relationship transferable between hydrocarbons and tetraorganostannanes
  — existing MW relationships for hydrocarbons cannot be used with tetraorganostannanes
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Source instrument whose isothermal capillary GC measurements (40–120°C, FID
    detection) provide the retention-time-derived vapour pressures regressed here
- name: SE-30 capillary column
  role: 2m x 0.55mm ID column (1.2µm film, Alltech) used instead of the unavailable
    C-87 phase; produces the corrected retention times underlying the lnP data
- name: scipy.stats.linregress
  role: Statistical library specified for the ordinary least-squares fit of lnP on
    MW or V
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:d6ac200c75a3a91949a9aef8604835c99234835f5f9c5777cfa4da676fc6a05a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FID detector
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  - scipy.stats.linregress
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# qspr-property-relationship-fitting

## Summary

Fits ordinary least-squares QSPR regressions of lnP (kPa) against molecular weight and intrinsic molar volume for tetraorganostannanes plus hydrocarbons, reproducing Hawker (1992) Equations 8 and 9 from GC-derived vapour pressure data.

## When to use

Use when you have a small (n≈7–13) compound set with measured or extrapolated vapour pressures plus a molecular descriptor (MW or intrinsic molar volume) and need to fit and validate a linear lnP–descriptor relationship.

## When NOT to use

- The target property involves triorganostannanes — published vapour pressure values vary considerably and applicability of Equation (9) to triorganostannanes is uncertain
- The compounds are solid at room temperature (MP > 25°C) — Equation (9) data need conversion to solid vapour pressure using entropy of fusion (ΔSf) and melting point, an extra required input
- You need an MW-based relationship transferable between hydrocarbons and tetraorganostannanes — existing MW relationships for hydrocarbons cannot be used with tetraorganostannanes

## Inputs

- Table 3 vapour pressures at 20°C (extrapolated from 40–120°C capillary GC measurements) for seven tetraorganostannanes
- Molecular weights of the tetraorganostannanes and literature hydrocarbons (n=7 or n=13 datasets)
- Intrinsic molar volumes (V, cm3/mol) computed via the Hickey and Passino-Reader fragment additivity procedure

## Outputs

- Fitted regression equation (slope, intercept, Pearson r, n) for lnP vs MW (Equation 8)
- Fitted regression equation (slope, intercept, Pearson r, n) for lnP vs intrinsic molar volume (Equation 9)
- Per-compound residuals written to vapour_pressure_regression.csv

## How to apply

Compile the compound set (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn with 20°C vapour pressures extrapolated from 40–120°C capillary GC measurements, plus literature hydrocarbons), transform each vapour pressure to lnP (kPa), and compute intrinsic molar volumes via the Hickey and Passino-Reader fragment additivity procedure. Fit ordinary least-squares regressions of lnP on MW (Equation 8, reported slope -5.82e-2, intercept 12.27, r=0.99, n=7) and of lnP on V (Equation 9, lnP = -9.83e-3 V + 8.99, r=0.99, n=13) using scipy.stats.linregress or numpy.polyfit. The rationale is that MW-based relationships are parallel but distinct for hydrocarbons vs organostannanes, whereas a single unique relationship exists with intrinsic molar volume, so Equation 9 combines both classes in one regression. Extract slope, intercept, Pearson r and per-compound residuals, and flag any deviation from the reported coefficients beyond numerical tolerance.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Source instrument whose isothermal capillary GC measurements (40–120°C, FID detection) provide the retention-time-derived vapour pressures regressed here)
- **SE-30 capillary column** (2m x 0.55mm ID column (1.2µm film, Alltech) used instead of the unavailable C-87 phase; produces the corrected retention times underlying the lnP data)
- **scipy.stats.linregress** (Statistical library specified for the ordinary least-squares fit of lnP on MW or V)

## Examples

```
from scipy.stats import linregress; slope, intercept, r, p, se = linregress(MW, lnP)  # Equation 8: expect slope=-5.82e-2, intercept=12.27, r=0.99
```

## Evaluation signals

- Equation 8 check: fitted lnP vs MW regression on the 7 tetraorganostannanes reproduces slope -5.82e-2, intercept 12.27, r = 0.99 within numerical tolerance
- Equation 9 check: fitted lnP vs intrinsic molar volume on the combined 13 compounds reproduces lnP = -9.83e-3 V + 8.99 with r = 0.99
- Sample-size invariant: n = 7 for the stannane-only MW regression and n = 13 for the combined hydrocarbon + stannane V regression
- Residual check: per-compound residuals written to vapour_pressure_regression.csv show no systematic class separation in the Equation 9 fit (single unique relationship)

## Limitations

- Current hydrocarbon vapour pressure estimation methods proved satisfactory for tetraalkylstannanes but less successful for tetraalkenylstannanes (e.g. Vi4Sn, Allyl4Sn)
- There is little data on other molecular descriptors of organostannanes besides MW and few published relationships using other descriptors
- For some compounds little vapour pressure data is available while published values vary considerably, limiting generalization of Equation (9)

## Evidence
