---
name: nonparametric-density-hypothesis-testing-planet-populations
description: Use when comparing a continuous property (e.g. bulk density from published
  mass/radius) across two independently labeled exoplanet subpopulations and no assumption
  of normality is justified, as in the savanna-vs-ridge super-Neptune comparison.
when_to_use_negative:
- The two comparison groups are large and demonstrably Gaussian — a parametric t-test
  would be more powerful
- 'The goal is flux contamination of TESS photometry (that is TESS-cont''s scope:
  Gaia DR2/DR3 contaminants, CROWDSAP/FLFRCSAP metrics), not population statistics'
- Planets lack published mass measurements, so mean density rho = 3M / (4*pi*R^3)
  cannot be computed per planet
edam_operation: http://edamontology.org/operation_0403
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: Python
  role: Compute per-planet mean densities from published mass/radius and run the Mann-Whitney
    U / rank-sum hypothesis test on the savanna vs ridge density distributions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:f1c79e4209072267510d7576d0edfde19d39b3806eb35c150b7b79f7a917848a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_0403
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# nonparametric-density-hypothesis-testing-planet-populations

## Summary

Tests whether two exoplanet subpopulations (Neptunian-savanna vs Neptunian-ridge planets) have systematically different mean bulk densities using a nonparametric rank-based test (Mann-Whitney U / rank-sum), and locates the empirical density dividing line between the regimes.

## When to use

Use when comparing a continuous property (e.g. bulk density from published mass/radius) across two independently labeled exoplanet subpopulations and no assumption of normality is justified, as in the savanna-vs-ridge super-Neptune comparison.

## When NOT to use

- The two comparison groups are large and demonstrably Gaussian — a parametric t-test would be more powerful
- The goal is flux contamination of TESS photometry (that is TESS-cont's scope: Gaia DR2/DR3 contaminants, CROWDSAP/FLFRCSAP metrics), not population statistics
- Planets lack published mass measurements, so mean density rho = 3M / (4*pi*R^3) cannot be computed per planet

## Inputs

- NASA Exoplanet Archive planetary parameters table (confirmed transiting super-Neptunes: period, radius, mass, uncertainties)
- Published planetary masses and radii per planet, e.g. TOI-5005 b: Rp = 6.25 +/- 0.24 R_earth, Mp = 32.7 +/- 5.9 M_earth, rho = 0.74 +/- 0.16 g/cm3
- Population labels (Neptunian ridge vs Neptunian savanna) assigned from the period-radius diagram

## Outputs

- Single CSV listing each planet's population label, mass, radius, and computed mean density
- Mann-Whitney U / rank-sum p-value for savanna vs ridge density distributions (expected ~0.0092)
- Empirical density dividing line between regimes (expected near 1 g/cm3)

## How to apply

Query the NASA Exoplanet Archive planetary parameters table for all confirmed transiting super-Neptunes and classify each planet as ridge or savanna from its position in the period-radius diagram. Compute each planet's mean density from published mass and radius using rho = 3M / (4*pi*R^3), propagating published uncertainties, adding key members such as TOI-5005 b (Rp = 6.25 +/- 0.24 R_earth, Mp = 32.7 +/- 5.9 M_earth, rho = 0.74 +/- 0.16 g/cm3) to the savanna subsample. Because the density distributions are small, non-Gaussian samples, apply a nonparametric rank-based test (Mann-Whitney U / rank-sum) in Python rather than a t-test. Report the p-value and inspect the two density distributions to locate the empirical dividing line; in this work the line falls near 1 g/cm3 with p = 0.0092, interpreted as evidence for different evolutionary pathways. Write all per-planet values and test results to a single CSV for validation.

## Related tools

- **Python** (Compute per-planet mean densities from published mass/radius and run the Mann-Whitney U / rank-sum hypothesis test on the savanna vs ridge density distributions)

## Examples

```
from scipy.stats import mannwhitneyu; u, p = mannwhitneyu(savanna_densities, ridge_densities, alternative='two-sided')
```

## Evaluation signals

- Computed p-value matches the reported 0.0092 within the tolerance implied by sample reproducibility
- Empirical dividing-line density falls near 1 g/cm3
- Recomputed density of TOI-5005 b reproduces the published 0.74 +/- 0.16 g/cm3
- Every planet in the CSV carries a valid ridge/savanna label consistent with the period-radius diagram classification
- Uncertainty propagation on densities is consistent with published mass/radius uncertainties

## Limitations

- Nonparametric rank tests compare distributions, not means, so the conclusion is rank-based and sensitive to subsample definition and sample size
- Classification into ridge vs savanna depends on the period-radius diagram boundaries; ambiguous planets near regime edges can shift the p-value and the dividing line
- Contrasting atmospheric mass-loss predictions for TOI-5005 b (0.013 +/- 0.008 vs 0.17 +/- 0.12 M_earth/Gyr^-1 depending on photoevaporation model) illustrate that model choice can affect downstream physical interpretation even when the density statistic is robust

## Evidence

- [abstract] At a population level, we find statistical evidence ( p -value = 0.0092) that planets in the savanna such as TOI-5005 b tend to show lower densities than planets in the ridge: [evidence span withheld — anchor status: not_found]
- [abstract] TOI-5005 b confirmed as a transiting super-Neptune with radius 6.25 ± 0.24 R⊕ and mass 32.7 ± 5.9 M⊕: [evidence span withheld — anchor status: not_found]
- [abstract] Mean density of TOI-5005 b is 0.74 ± 0.16 g cm−3: "corresponds to a mean density of ρ p = 0.74 ± 0.16 g cm −3"
