---
name: exoplanet-archive-population-sampling
description: Use when comparing a physical-property distribution (e.g. bulk density)
  between two exoplanet populations split by a diagram position (period–radius), and
  when locating the empirical dividing value between the two regimes.
when_to_use_negative:
- The goal is characterising a single target (e.g. flux contamination in one TESS
  aperture) rather than a two-population statistical comparison
- Planets in the sample lack published masses and radii, so mean densities cannot
  be computed and propagated
- The data are paired or otherwise non-independent between groups, violating the assumptions
  of a rank-sum (Mann-Whitney U) test
edam_operation: http://edamontology.org/operation_3659
tools:
- name: Python
  role: Scripted environment for computing per-planet mean densities with propagated
    uncertainties, running the Mann-Whitney U / rank-sum test, and exporting the results
    CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:b58b762a45c1ba5395988ae361b4b03e5cc29933ccf9e50d9255cea8be9db67f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# exoplanet-archive-population-sampling

## Summary

Population-level sampling and nonparametric comparison of exoplanet bulk densities: query the NASA Exoplanet Archive planetary parameters table for confirmed transiting super-Neptunes, classify each as Neptunian-ridge or Neptunian-savanna by period-radius position, compute mean densities, and rank-test for a density offset with an empirical dividing line near 1 g/cm³.

## When to use

Use when comparing a physical-property distribution (e.g. bulk density) between two exoplanet populations split by a diagram position (period–radius), and when locating the empirical dividing value between the two regimes.

## When NOT to use

- The goal is characterising a single target (e.g. flux contamination in one TESS aperture) rather than a two-population statistical comparison
- Planets in the sample lack published masses and radii, so mean densities cannot be computed and propagated
- The data are paired or otherwise non-independent between groups, violating the assumptions of a rank-sum (Mann-Whitney U) test

## Inputs

- NASA Exoplanet Archive planetary parameters table query: confirmed transiting super-Neptunes with orbital period, radius, and radius uncertainty
- Published planet masses and radii with uncertainties (for density computation via rho = 3M / (4*pi*R^3))
- Published parameters of TOI-5005 b (Rp = 6.25 ± 0.24 R_earth, Mp = 32.7 ± 5.9 M_earth, rho = 0.74 ± 0.16 g/cm3) for inclusion in the savanna subsample

## Outputs

- CSV file listing each planet's population label (ridge vs savanna), mass, radius, and computed mean density
- Mann-Whitney U / rank-sum test p-value comparing savanna vs ridge density distributions (expected ≈ 0.0092)
- Empirical density dividing line between the two regimes (expected near 1 g/cm³)

## How to apply

Query the NASA Exoplanet Archive planetary parameters table for all confirmed transiting super-Neptunes and classify each planet as Neptunian-ridge or Neptunian-savanna from its position in the period-radius diagram. Compute each planet's mean density as rho = 3M / (4*pi*R^3) from published mass and radius, propagating published uncertainties; include TOI-5005 b (Rp = 6.25 ± 0.24 R_earth, Mp = 32.7 ± 5.9 M_earth, rho = 0.74 ± 0.16 g/cm3) in the savanna subsample. Run a nonparametric rank-based hypothesis test (Mann-Whitney U / rank-sum) on the two density distributions in Python and report the p-value (target ≈ 0.0092) together with the empirical dividing-line density (~1 g/cm³). Write a single CSV listing each planet's population label, mass, radius, and computed mean density, plus the p-value and dividing-line density.

## Related tools

- **Python** (Scripted environment for computing per-planet mean densities with propagated uncertainties, running the Mann-Whitney U / rank-sum test, and exporting the results CSV)

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(savanna_rho, ridge_rho, alternative='two-sided')
```

## Evaluation signals

- Computed Mann-Whitney U / rank-sum p-value matches the reported 0.0092 within the tolerance implied by sample reproducibility
- Inferred density dividing line between savanna and ridge falls near 1 g/cm³
- TOI-5005 b density recomputed from Rp = 6.25 ± 0.24 R_earth and Mp = 32.7 ± 5.9 M_earth reproduces 0.74 ± 0.16 g/cm3
- Output CSV contains the required columns: population label, mass, radius, computed mean density, p-value, and dividing-line density
- Every planet in the sample is assigned exactly one population label based on its position in the period-radius diagram

## Limitations

- Population labels depend on where the period-radius diagram boundaries of the Neptunian ridge and savanna are drawn, so the subsamples (and hence the p-value) shift with the adopted definition
- The p-value match is only expected within the tolerance implied by sample reproducibility — the exact planet set pulled from the archive changes the result
- Density uncertainties are propagated from published mass and radius errors, which can be large (e.g. Mp = 32.7 ± 5.9 M_earth for TOI-5005 b) and dominate the spread in each subsample

## Evidence

- [abstract] Population-level statistical comparison of savanna vs ridge planet densities: [evidence span withheld — anchor status: not_found]
- [abstract] Density dividing line near 1 g/cm3: [evidence span withheld — anchor status: not_found]
- [other] Archive query and ridge/savanna classification step: [evidence span withheld — anchor status: not_found]
- [abstract] Per-planet density computation check: "corresponds to a mean density of ρ p = 0.74 ± 0.16 g cm −3"
