---
name: gc-derived-vapour-pressure-data-analysis
description: Use when isothermal GC retention-time data (corrected against a reference
  such as natural gas) must be converted to vapour pressures, extrapolated below the
  measured temperature range (e.g. to 20°C from 40–120°C data), or related to molecular
  descriptors such as MW or intrinsic molar volume.
when_to_use_negative:
- 'The compounds are solid at room temperature (MP > 25°C): Equation (9) data must
  additionally be converted to solid vapour pressure using entropy of fusion (ΔSf)
  and melting point, so the liquid-phase regression is not directly applicable.'
- Target compounds are triorganostannanes or other species with little or highly variable
  published vapour pressure data — the paper flags that applicability of the MW relationship
  (Equation 9) to such compounds is uncertain.
- You need to apply hydrocarbon MW-based relationships directly to organostannanes
  — the article states such MW relationships cannot be used with tetraorganostannanes,
  and hydrocarbon estimation methods underestimate tetraalkenylstannanes (e.g. Vi4Sn
  estimated 0.26/0.25 vs 0.35 kPa experimental).
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Instrument used for isothermal determination of organostannane retention times
    from which vapour pressures are calculated
- name: FID detector
  role: Flame ionization detector on the GC-15A used to detect eluting organostannanes
- name: Chromatopac C-R6A integrator
  role: Integration of GC peak retention times interfaced to the GC-15A
- name: SE-30 capillary column
  role: 2 m × 0.55 mm ID, 1.2 µm film column from Alltech Associates used for the
    separations (chosen for low McReynold's constants since C-87 was unavailable)
- name: Antoine equation
  role: Hydrocarbon-based baseline estimator applied to Sn-as-carbon analogues for
    comparison against measured 20°C vapour pressures
- name: Modified Watson correlation
  role: Second hydrocarbon-based estimation method compared with experimental extrapolated
    20°C values
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:07f3abf065142869880a6ecc00b6fcb69427b84238c4af1c55c3e9e02fe65f11
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Antoine equation
  - Chromatopac C-R6A integrator
  - FID detector
  - Modified Watson correlation
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# gc-derived-vapour-pressure-data-analysis

## Summary

A method for converting isothermal capillary GC retention times into equilibrium vapour pressures (relative to a reference compound such as tetramethylstannane), extrapolating them to 20°C, and regressing lnP against molecular weight or intrinsic molar volume for tetraorganostannanes.

## When to use

Use when isothermal GC retention-time data (corrected against a reference such as natural gas) must be converted to vapour pressures, extrapolated below the measured temperature range (e.g. to 20°C from 40–120°C data), or related to molecular descriptors such as MW or intrinsic molar volume.

## When NOT to use

- The compounds are solid at room temperature (MP > 25°C): Equation (9) data must additionally be converted to solid vapour pressure using entropy of fusion (ΔSf) and melting point, so the liquid-phase regression is not directly applicable.
- Target compounds are triorganostannanes or other species with little or highly variable published vapour pressure data — the paper flags that applicability of the MW relationship (Equation 9) to such compounds is uncertain.
- You need to apply hydrocarbon MW-based relationships directly to organostannanes — the article states such MW relationships cannot be used with tetraorganostannanes, and hydrocarbon estimation methods underestimate tetraalkenylstannanes (e.g. Vi4Sn estimated 0.26/0.25 vs 0.35 kPa experimental).

## Inputs

- Isothermal capillary GC retention times for tetraorganostannanes at 40–120°C in 20°C increments (corrected means of 5 runs, 3 for Allyl4Sn), from 1.0% w/v solutions in octane
- Retention time of a natural gas sample for correction
- Reference-compound vapour pressure–temperature data (tetramethylstannane, Me4Sn) from the literature
- Molecular weights of the seven tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn)
- Normal boiling points of hydrocarbon analogues (Sn replaced by carbon) for Antoine / Modified Watson baseline estimates

## Outputs

- Corrected mean retention times per compound per temperature (Table 1-style table)
- Calculated equilibrium vapour pressures at 40–120°C and extrapolated 20°C values in kPa
- Linear regression coefficients: slope, intercept, Pearson r for lnP vs MW (e.g. -5.82e-2, 12.27, r = 0.99) and lnP vs intrinsic molar volume
- Per-compound residuals and comparison table of estimated vs experimental 20°C vapour pressures (CSV, e.g. vapour_pressure_regression.csv, vapour_pressure_estimates.csv)
- Derived enthalpies (and entropies, where boiling points exist) of vaporization

## How to apply

Measure retention times isothermally on a Shimadzu GC-15A with FID detector and a 2 m × 0.55 mm ID SE-30 capillary column (1.2 µm film) from 40°C to 120°C in 20°C increments, using 1.0% w/v solutions in octane and averaging over 5 runs per compound (3 for Allyl4Sn). Correct each retention time by subtracting the retention time of a natural gas sample, then calculate vapour pressures for the remaining tetraorganostannanes using Me4Sn as the reference compound with known vapour pressure-temperature data per the relative GC retention theory. Extrapolate the lnP vs 1/T behaviour to 20°C, which lies below the measured range. For QSPR-type analysis, fit ordinary least-squares regressions of lnP (kPa) on molecular weight (n = 7 compounds: Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) — e.g. scipy.stats.linregress — and compare against reported coefficients such as lnP = -5.82e-2 MW + 12.27 with r = 0.99. Optionally compare 20°C extrapolated values against hydrocarbon-based estimates (Antoine equation, Modified Watson correlation) built by treating the central Sn atom as carbon.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Instrument used for isothermal determination of organostannane retention times from which vapour pressures are calculated)
- **FID detector** (Flame ionization detector on the GC-15A used to detect eluting organostannanes)
- **Chromatopac C-R6A integrator** (Integration of GC peak retention times interfaced to the GC-15A)
- **SE-30 capillary column** (2 m × 0.55 mm ID, 1.2 µm film column from Alltech Associates used for the separations (chosen for low McReynold's constants since C-87 was unavailable))
- **Antoine equation** (Hydrocarbon-based baseline estimator applied to Sn-as-carbon analogues for comparison against measured 20°C vapour pressures)
- **Modified Watson correlation** (Second hydrocarbon-based estimation method compared with experimental extrapolated 20°C values)

## Examples

```
from scipy.stats import linregress; slope, intercept, r, p, se = linregress(mw, lnP); print(slope, intercept, r)  # expect -5.82e-2, 12.27, r=0.99
```

## Evaluation signals

- The fitted lnP vs MW regression reproduces lnP (kPa) = -5.82e-2 × MW + 12.27 with r = 0.99 within reasonable numerical tolerance (n = 7 compounds).
- Estimated 20°C vapour pressure for Et4Sn (Antoine, 0.15 kPa) is within ~0.01 kPa of the experimental 0.16 kPa.
- Estimates for Vi4Sn (0.26 and 0.25 kPa vs 0.35 kPa experimental) reproduce the qualitative finding that hydrocarbon methods underestimate tetraalkenylstannanes.
- Retention times are means of 5 runs per compound (3 for Allyl4Sn), and corrected retention times equal raw values minus natural-gas retention time.
- Tetra-n-butylstannane is the largest compound with a determinable vapour pressure under the experimental conditions — any larger compound failing to elute/quantify is expected.

## Limitations

- A C-87 stationary-phase capillary column (identified as most satisfactory by Bidleman) was unavailable, so SE-30 was substituted.
- Lack of appropriate boiling point data prevents confirming whether hydrocarbon estimation methods are adequate for other tetraalkylstannanes and tetraalkenylstannanes.
- Little data exists on organostannane molecular descriptors other than MW, and few published relationships use other descriptors; intrinsic molar volume uniquely unifies organostannanes and hydrocarbons whereas MW does not.
- 20°C values are extrapolations from 40–120°C measurements, not direct measurements.

## Evidence
