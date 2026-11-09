---
name: diffmah-model-residual-analysis
description: 'Use when validating that a fitted halo mass accretion history model
  (e.g., diffmah 3-parameter fits) reproduces assembly bias: i.e., residuals must
  show negligible correlation with large-scale density in simulations like TNG.'
when_to_use_negative:
- When the goal is to fit or refit mass accretion histories themselves (use the diffmah
  history-fitting workflow, e.g., diffmah_fitter_demo.ipynb or history_fitting_script.py)
  rather than to validate residual correlations
- When the halo sample is below the model's demonstrated validity range (t <= 1 Gyr
  or present-day M <= 10^11 Msun), where residual structure may not be interpretable
- When only theoretical/model histories are available without positions or a density
  field — no assembly-bias correlation can be computed
edam_operation: http://edamontology.org/operation_2945
tools:
- name: diffmah
  role: Source of the 3-parameter differentiable halo assembly-history model whose
    fit residuals are analyzed for density-field correlation
  repo: https://github.com/ArgonneCPAC/diffmah
- name: numpy
  role: Loading TNG halo positions/density field, assigning environment, and computing
    the correlation estimators
- name: jax
  role: Autodiff backend underlying the diffmah model implementation
  repo: https://jax.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:4f8eac00958f1c3f3f75400226c3c73208b63fc189bc7d60c65bb8ee88001a03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArgonneCPAC/diffmah
  tools:
  - Python
  - diffmah
  - jax
  - numpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# diffmah-model-residual-analysis

## Summary

Tests whether residual errors in diffmah 3-parameter halo assembly-history approximations remain uncorrelated with the large-scale density field, validating that the model captures halo assembly bias rather than just individual halo growth.

## When to use

Use when validating that a fitted halo mass accretion history model (e.g., diffmah 3-parameter fits) reproduces assembly bias: i.e., residuals must show negligible correlation with large-scale density in simulations like TNG.

## When NOT to use

- When the goal is to fit or refit mass accretion histories themselves (use the diffmah history-fitting workflow, e.g., diffmah_fitter_demo.ipynb or history_fitting_script.py) rather than to validate residual correlations
- When the halo sample is below the model's demonstrated validity range (t <= 1 Gyr or present-day M <= 10^11 Msun), where residual structure may not be interpretable
- When only theoretical/model histories are available without positions or a density field — no assembly-bias correlation can be computed

## Inputs

- diffmah 3-parameter accretion-history fits for simulated halos (e.g., produced in a prior fitting task over merger trees)
- Halo positions and host/subhalo identifiers from the TNG simulation
- Large-scale density field from the TNG simulation
- Present-day halo masses (used to apply the M > 10^11 Msun validity cut)

## Outputs

- assembly_bias_correlations.csv recording assembly-class vs. density-field and residual vs. density-field correlations
- Per-halo fast- vs. slow-accreting population classification derived from the fitted diffmah histories
- Residual correlation estimates demonstrating (or refuting) negligible residual dependence on environment

## How to apply

Load the diffmah 3-parameter accretion-history fits (approximating halo growth to ~0.1 dex for t > 1 Gyr and present-day M > 10^11 Msun) alongside halo positions and the large-scale density field from the TNG simulation using numpy. Assign each halo its large-scale density environment and classify halos into fast- vs. slow-accreting populations based on the fitted assembly histories. Use numpy-based correlation estimators to compute (1) the correlation between halo assembly class and the large-scale density field — the assembly bias signal — and (2) the correlation between diffmah approximation residuals and the density field. The validation decision point is that residual correlations must be negligible while the assembly-class correlations are reproduced, matching the paper's finding that 'halo assembly bias is accurately captured by our model'. Write the measured correlations to assembly_bias_correlations.csv.

## Related tools

- **diffmah** (Source of the 3-parameter differentiable halo assembly-history model whose fit residuals are analyzed for density-field correlation) — https://github.com/ArgonneCPAC/diffmah
- **numpy** (Loading TNG halo positions/density field, assigning environment, and computing the correlation estimators)
- **jax** (Autodiff backend underlying the diffmah model implementation) — https://jax.readthedocs.io/en/latest/

## Examples

```
import numpy as np; r_assembly, r_residual = np.corrcoef(assembly_class, density_field)[0,1], np.corrcoef(fit_residuals, density_field)[0,1]
```

## Evaluation signals

- Diffmah fit residuals vs. large-scale density field correlations are negligible, consistent with the paper's finding of 'negligible residual correlation between approximation errors and the density field'
- Fast- vs. slow-accreting halos reproduce the expected assembly bias signal (correlation of assembly class with density field)
- Halo sample respects the demonstrated model accuracy regime: ~0.1 dex accuracy for t > 1 Gyr and present-day M > 10^11 Msun
- assembly_bias_correlations.csv exists and contains both correlation measurements (assembly class and residuals)
- Fits loaded from the prior fitting step were produced with the diffmah 3-parameter model over the TNG/merger-tree sample, not synthetic growth curves

## Limitations

- Model accuracy and residual-behavior conclusions are demonstrated only for t > 1 Gyr and present-day halo masses above 10^11 Msun, in gravity-only and TNG hydrodynamical simulations
- The published discussion content for this skill is absent; only repository metadata and the abstract-level finding on assembly bias are available as background
- Validation depends on the quality of the upstream diffmah fits; a poor fit (exceeding ~0.1 dex error) can inflate residual correlations spuriously

## Evidence

- [abstract] The model accurately captures halo assembly bias, with negligible residual correlation between approximation errors and the density field.: "the connection between halo assembly and the large-scale density field, known as halo assembly bias, is accurately captured by our model"
- [abstract] 3-parameter model accuracy regime for approximating halo growth.: "we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$"
- [abstract] JAX-based differentiable implementation of the diffmah model.: "Our python implementation is based on the autodiff library JAX"
- [readme] Installation path for diffmah used in the workflow.: [evidence span withheld — anchor status: external_doc]
- [readme] Demo notebook for fitting halo MAHs with diffmah, the upstream step whose residuals this skill analyzes.: [evidence span withheld — anchor status: external_doc]
- [readme] Project data availability for diffmah inputs.: [evidence span withheld — anchor status: external_doc]
