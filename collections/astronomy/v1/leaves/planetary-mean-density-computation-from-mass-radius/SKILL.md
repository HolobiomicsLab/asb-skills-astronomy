---
name: planetary-mean-density-computation-from-mass-radius
description: Use when you have published masses and radii (e.g., R_p = 6.25 ± 0.24
  R_earth, M_p = 32.7 ± 5.9 M_earth) and need mean densities with propagated uncertainties
  for hypothesis testing between planet populations.
when_to_use_negative:
- Planets lack published mass measurements — density cannot be computed without M_p
  and R_p
- The goal is TESS flux-contamination/dilution analysis of a specific target aperture
  — use TESS-cont instead, which computes CROWDSAP and FLFRCSAP metrics rather than
  bulk densities
edam_operation: http://edamontology.org/operation_0306
tools:
- name: Python
  role: Compute rho = 3M/(4*pi*R^3) with uncertainty propagation and run the Mann-Whitney
    U / rank-sum hypothesis test on savanna vs. ridge densities
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:f11b00d49fe561348ef334d28ebf3477c3aa8582cef2b2798aea765e914bb65e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# planetary-mean-density-computation-from-mass-radius

## Summary

Computes each planet's mean bulk density from its published mass and radius using rho = 3M/(4*pi*R^3) with propagated uncertainties, enabling population-level density comparisons (e.g., savanna vs. ridge super-Neptunes).

## When to use

Use when you have published masses and radii (e.g., R_p = 6.25 ± 0.24 R_earth, M_p = 32.7 ± 5.9 M_earth) and need mean densities with propagated uncertainties for hypothesis testing between planet populations.

## When NOT to use

- Planets lack published mass measurements — density cannot be computed without M_p and R_p
- The goal is TESS flux-contamination/dilution analysis of a specific target aperture — use TESS-cont instead, which computes CROWDSAP and FLFRCSAP metrics rather than bulk densities

## Inputs

- NASA Exoplanet Archive planetary parameters table (confirmed transiting super-Neptunes with published mass and radius and their uncertainties)
- Period and radius values for ridge/savanna classification in the period-radius diagram
- Anchor planet parameters, e.g. TOI-5005 b: R_p = 6.25 ± 0.24 R_earth, M_p = 32.7 ± 5.9 M_earth, rho = 0.74 ± 0.16 g/cm^3

## Outputs

- Per-planet mean densities with propagated uncertainties (g cm^-3)
- Mann-Whitney U / rank-sum p-value comparing savanna vs. ridge density distributions (target: 0.0092)
- Empirical density dividing line between the two regimes (~1 g cm^-3)
- Single CSV with each planet's population label, mass, radius, computed mean density, p-value, and dividing-line density

## How to apply

Query the NASA Exoplanet Archive planetary parameters table for all confirmed transiting super-Neptunes and classify each planet as Neptunian ridge or Neptunian savanna by its position in the period-radius diagram. For each planet, compute mean density as rho = 3M/(4*pi*R^3) from the published mass and radius, propagating the published uncertainties (e.g., TOI-5005 b: R_p = 6.25 ± 0.24 R_earth and M_p = 32.7 ± 5.9 M_earth yield rho = 0.74 ± 0.16 g/cm^3). Apply a nonparametric rank-based test (Mann-Whitney U / rank-sum) in Python to the savanna vs. ridge density distributions to test for systematically lower savanna densities, and locate the empirical density dividing line. Write a single CSV listing each planet's population label, mass, radius, computed mean density, plus the test p-value and dividing-line density.

## Related tools

- **Python** (Compute rho = 3M/(4*pi*R^3) with uncertainty propagation and run the Mann-Whitney U / rank-sum hypothesis test on savanna vs. ridge densities)

## Examples

```
import numpy as np; Mp, Rp = 32.7, 6.25; rho = 3*Mp/(4*np.pi*Rp**3)  # earth units -> g/cm3; expect ~0.74 +/- 0.16 for TOI-5005 b
```

## Evaluation signals

- Computed density for TOI-5005 b matches the published 0.74 ± 0.16 g cm^-3
- Mann-Whitney U p-value reproduces the reported 0.0092 within the tolerance implied by sample reproducibility
- Empirical dividing line between savanna and ridge densities falls near 1 g cm^-3
- Uncertainty propagation from published M_p and R_p errors yields density uncertainties consistent with published values
- Output CSV contains population label, mass, radius, and computed mean density for every planet in the subsamples

## Limitations

- Nonparametric rank-based test requires adequate sample sizes in both the savanna and ridge subsamples; the p-value is only reproducible to within the tolerance implied by sample reproducibility
- Uncertainty propagation depends entirely on the quality of published mass and radius values (e.g., M_p = 32.7 ± 5.9 M_earth gives a ~18% mass uncertainty)
- Ridge/savanna labels depend on the period-radius classification boundary, so misclassification shifts both the p-value and the dividing line

## Evidence

- [other] Compute the mean density of each planet from its published mass and radius using rho = 3M / (4*pi*R^3), propagating published uncertainties: [evidence span withheld — anchor status: not_found]
- [abstract] We confirm TOI-5005 b to be a transiting super-Neptune with a radius of R p = 6.25 ± 0.24 R ⊕ ... and a mass of M p = 32.7 ± 5.9 M ⊕: "We confirm TOI-5005 b to be a transiting super-Neptune with a radius of R p = 6.25 ± 0.24 R ⊕"
- [abstract] Planets in the savanna tend to show lower densities than planets in the ridge, with a dividing line around 1 g cm−3 (p-value = 0.0092): [evidence span withheld — anchor status: not_found]
- [other] Apply a nonparametric rank-based hypothesis test (Mann-Whitney U / rank-sum) to the savanna vs. ridge density distributions in Python: [evidence span withheld — anchor status: not_found]
