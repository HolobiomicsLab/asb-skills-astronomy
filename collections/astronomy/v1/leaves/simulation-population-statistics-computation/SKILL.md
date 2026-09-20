---
name: simulation-population-statistics-computation
description: Use when you have per-halo growth-history residuals from a model fit
  to merger trees and need to test whether those residuals are uncorrelated with environmental
  variables such as local density.
when_to_use_negative:
- Input is raw halo mass accretion histories rather than residuals from an already-fitted
  model — use the diffmah fitting workflow instead.
- You need to fit or generate Monte Carlo halo assembly histories, not evaluate residual-environment
  correlations.
- The analysis targets hydrodynamical baryonic effects rather than gravity-only halo
  assembly bias residuals.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: numpy
  role: Computes Pearson correlation of fit errors against local density over the
    halo population
  repo: https://github.com/ArgonneCPAC/diffmah
- name: JAX
  role: Autodiff library underlying the diffmah model whose per-halo fit residuals
    are being analyzed
  repo: https://jax.readthedocs.io/en/latest/
- name: diffmah
  role: Differentiable 3-parameter power-law model producing the halo assembly-history
    approximations whose residuals are tested
  repo: https://github.com/ArgonneCPAC/diffmah
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:ac8ca65fed999ebe9ca84d7f994d28d38d21c3b686340881988046680e85c10a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArgonneCPAC/diffmah
  tools:
  - JAX
  - Python
  - diffmah
  - numpy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# simulation-population-statistics-computation

## Summary

Validates that a differentiable halo assembly model (diffmah) faithfully captures halo assembly bias by measuring the correlation between per-halo fit residuals and the large-scale density field in a gravity-only simulation population.

## When to use

Use when you have per-halo growth-history residuals from a model fit to merger trees and need to test whether those residuals are uncorrelated with environmental variables such as local density.

## When NOT to use

- Input is raw halo mass accretion histories rather than residuals from an already-fitted model — use the diffmah fitting workflow instead.
- You need to fit or generate Monte Carlo halo assembly histories, not evaluate residual-environment correlations.
- The analysis targets hydrodynamical baryonic effects rather than gravity-only halo assembly bias residuals.

## Inputs

- Per-halo growth-history residuals from diffmah fits to merger trees (gravity-only simulation, present-day mass > 1e11 solar masses, t > 1 Gyr)
- Halo positions from the gravity-only simulation
- Large-scale density field from the public TNG DMO run

## Outputs

- Pearson correlation coefficients of fit residuals vs local density per population (residual_density_correlation.csv)
- Characterization of residual correlation magnitudes relative to the assembly bias signal

## How to apply

Load the per-halo growth-history residuals (e.g. from diffmah 3-parameter power-law fits to halos of present-day mass > 1e11 solar masses, t > 1 Gyr, in the gravity-only simulation). Map each halo to its local density value using halo positions and the large-scale density field from the public TNG DMO run. With numpy, compute Pearson correlation coefficients of fit errors against local density over the halo population. Record correlation magnitudes and judge them negligible relative to the halo assembly bias signal, writing results to a CSV such as residual_density_correlation.csv.

## Related tools

- **numpy** (Computes Pearson correlation of fit errors against local density over the halo population) — https://github.com/ArgonneCPAC/diffmah
- **JAX** (Autodiff library underlying the diffmah model whose per-halo fit residuals are being analyzed) — https://jax.readthedocs.io/en/latest/
- **diffmah** (Differentiable 3-parameter power-law model producing the halo assembly-history approximations whose residuals are tested) — https://github.com/ArgonneCPAC/diffmah

## Examples

```
python analyze_residuals.py --residuals task_002_resid.h5 --density tng_dmo_density.h5 --out residual_density_correlation.csv
```

## Evaluation signals

- Computed correlation magnitudes must be consistent with the paper's finding of negligible residual correlation with the density field.
- Halo sample matches filter criteria: present-day mass greater than 1e11 solar masses and t > 1 Gyr.
- Each halo maps to exactly one local density value from the TNG DMO density field.
- Output CSV (residual_density_correlation.csv) contains the recorded correlation values.
- Every halo entering the correlation must have residuals produced by a completed diffmah fit (typical accuracy ~0.1 dex).

## Limitations

- Typical accuracy of the underlying diffmah approximation is 0.1 dex only for t > 1 Gyr and halos of present-day mass greater than 1e11 solar masses; residuals outside this regime may behave differently.
- Requires density field from a public DMO run (TNG); correlation results are specific to gravity-only halo populations used in the paper.

## Evidence

- [abstract] the connection between halo assembly and the large-scale density field, known as halo assembly bias, is accurately captured by our model, and that residual errors in our approximations to halo: "the connection between halo assembly and the large-scale density field, known as halo assembly bias, is accurately captured by our model, and that residual errors in our approximations to halo"
- [methods] Pearson correlation of fit errors against local density, evaluated over the halo population: [evidence span withheld — anchor status: not_found]
- [abstract] we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$: "we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$"
- [abstract] Our python implementation is based on the autodiff library JAX: "Our python implementation is based on the autodiff library JAX"
