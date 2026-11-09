---
name: dark-matter-halo-residual-quantification
description: Use when you have per-halo fit residuals from a halo assembly-history
  model (e.g. diffmah fit errors) plus halo positions and a large-scale density field
  from a simulation like TNG, and need to test whether approximation errors remain
  uncorrelated with environment — i.
when_to_use_negative:
- You only need halo mass accretion history fits themselves, not an environment-correlation
  test — use the diffmah fitting workflow (history_fitting_script.py) instead
- The halo catalog lacks positions or a matching density field, so no local environment
  can be assigned per halo
- The residuals come from a model other than diffmah whose accuracy regime (e.g. <
  1e11 Msun halos or t < 1 Gyr) has not been established
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: numpy
  role: Computes Pearson correlation coefficients between per-halo fit residuals and
    mapped local density values
  repo: https://github.com/ArgonneCPAC/diffmah
- name: diffmah
  role: Differentiable model whose 3-parameter power-law fits produce the growth-history
    residuals under test
  repo: https://github.com/ArgonneCPAC/diffmah/
- name: JAX
  role: Autodiff backend on which the diffmah model is implemented, generating the
    approximations whose errors are analyzed
  repo: https://jax.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:351c50ab5e600136e49ea2f21fbfff34ff577034d8ddfd7bafa4bbef43964a37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArgonneCPAC/diffmah
  tools:
  - JAX
  - Python
  - diffmah
  - numpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# dark-matter-halo-residual-quantification

## Summary

Quantifies whether residual errors in diffmah approximations to halo mass accretion histories correlate with the large-scale density field, testing whether the differentiable model captures halo assembly bias. Positive results indicate the 3-parameter power-law fit leaves no unmodeled connection between halo assembly and environment.

## When to use

Use when you have per-halo fit residuals from a halo assembly-history model (e.g. diffmah fit errors) plus halo positions and a large-scale density field from a simulation like TNG, and need to test whether approximation errors remain uncorrelated with environment — i.e. whether assembly bias is captured.

## When NOT to use

- You only need halo mass accretion history fits themselves, not an environment-correlation test — use the diffmah fitting workflow (history_fitting_script.py) instead
- The halo catalog lacks positions or a matching density field, so no local environment can be assigned per halo
- The residuals come from a model other than diffmah whose accuracy regime (e.g. < 1e11 Msun halos or t < 1 Gyr) has not been established

## Inputs

- Per-halo growth-history fit residuals from diffmah approximations to halo mass accretion histories
- Halo positions from a cosmological simulation merger-tree catalog (gravity-only or TNG hydrodynamical run)
- Large-scale density field from the public TNG DMO run

## Outputs

- Pearson correlation coefficients of fit errors vs. local density over the halo population
- CSV table of residual-density correlation values (residual_density_correlation.csv)

## How to apply

Load the per-halo growth-history residuals (e.g. from a prior diffmah fitting pass on merger trees of halos with present-day mass > 1e11 Msun and t > 1 Gyr, as in the paper's sample). Load halo positions and the large-scale density field from the public TNG DMO run and map each halo to its local density value via nearest-neighbor or grid interpolation. Compute Pearson correlation coefficients between the fit errors and local density over the full halo population with numpy. Record the correlation magnitudes and judge them negligible relative to the halo assembly bias signal itself; write results to a table such as residual_density_correlation.csv. The validation criterion follows the paper: residual errors show negligible correlation with the density field while the model itself captures assembly bias.

## Related tools

- **numpy** (Computes Pearson correlation coefficients between per-halo fit residuals and mapped local density values) — https://github.com/ArgonneCPAC/diffmah
- **diffmah** (Differentiable model whose 3-parameter power-law fits produce the growth-history residuals under test) — https://github.com/ArgonneCPAC/diffmah/
- **JAX** (Autodiff backend on which the diffmah model is implemented, generating the approximations whose errors are analyzed) — https://jax.readthedocs.io/en/latest/

## Examples

```
import numpy as np; r = np.corrcoef(fit_errors, local_density)[0, 1]
```

## Evaluation signals

- Correlation magnitudes between residual errors and local density are negligible relative to the halo assembly bias signal, per the paper's finding
- The residual set corresponds to the paper's validated regime: halos with present-day mass > 1e11 Msun and t > 1 Gyr
- The computed coefficients are consistent with the published claim that assembly bias is accurately captured by the model
- Output file residual_density_correlation.csv contains the correlation values with a documented halo-population scope

## Limitations

- The residual-correlation null result was demonstrated for halos of present-day mass greater than 10^11 solar masses with t > 1 Gyr; behavior outside this regime is not established
- The test relies on the density field being mapped from the same simulation population (TNG DMO run) as the halos; mismatched catalogs would invalidate the correlation
- Typical diffmah fit accuracy is ~0.1 dex; residuals at this scale may be too small to detect weak environment correlations

## Evidence

- [abstract] Residual-density correlation finding: "the connection between halo assembly and the large-scale density field, known as halo assembly bias, is accurately captured by our model, and that residual errors in our approximations to halo"
- [abstract] Model accuracy regime: "we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$"
- [abstract] TNG dataset used: "as well as in the TNG hydrodynamical simulation"
- [methods] Implementation stack: "Our python implementation is based on the autodiff library JAX, and so our model self-consistently captures the mean and variance of halo mass accretion rate across cosmic time."
- [readme] Data availability: [evidence span withheld — anchor status: external_doc]
