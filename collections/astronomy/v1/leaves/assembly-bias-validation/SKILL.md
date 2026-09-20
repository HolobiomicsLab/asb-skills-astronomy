---
name: assembly-bias-validation
description: Decision support for testing whether an empirical halo-assembly model
  reproduces not only mean halo mass growth but the per-halo diversity of assembly
  histories, and whether residual assembly-bias correlations remain.
when_to_use_negative:
- You need physically simulated merger trees or hydrodynamical evolution (e.g. full
  TNG runs) rather than an empirical, differentiable approximation to MAHs
- Your halos fall outside the validated regime — present-day mass below 10^11 solar
  masses or cosmic times t < 1 Gyr, where the stated 0.1 dex accuracy does not hold
- Input is already a synthetic diffmah population catalog and the goal is downstream
  galaxy-halo connection rather than validating the assembly model itself
edam_operation: http://edamontology.org/operation_0282
tools:
- name: diffmah
  role: Differentiable empirical model used to evaluate mass assembly histories for
    Monte Carlo-sampled halo parameters
  repo: https://github.com/ArgonneCPAC/diffmah/
- name: JAX
  role: Autodiff library underlying diffmah; evaluates the model and self-consistently
    captures mean and variance of halo mass accretion rate across cosmic time
  repo: https://jax.readthedocs.io/en/latest/
- name: Python >=3.9
  role: Runtime environment requirement for diffmah (installed with numpy and jax
    dependencies)
- name: numpy
  role: Dependency used for constructing population-level distributions of fitted
    diffmah parameters and Monte Carlo sampling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:ef7161ca8397048febc95bdfb54c5111de4257b41f4bb8f8958328b0e724116e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArgonneCPAC/diffmah
  tools:
  - JAX
  - Python >=3.9
  - diffmah
  - numpy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# assembly-bias-validation

## Summary

Generates and validates Monte Carlo realizations of cosmologically representative halo assembly histories using the differentiable diffmah 3-parameter model, checking that both the average mass growth across cosmic time and the diversity (and large-scale-structure dependence, i.e. assembly bias) of halo mass assembly are faithfully reproduced.

## When to use

Decision support for testing whether an empirical halo-assembly model reproduces not only mean halo mass growth but the per-halo diversity of assembly histories, and whether residual assembly-bias correlations remain.

## When NOT to use

- You need physically simulated merger trees or hydrodynamical evolution (e.g. full TNG runs) rather than an empirical, differentiable approximation to MAHs
- Your halos fall outside the validated regime — present-day mass below 10^11 solar masses or cosmic times t < 1 Gyr, where the stated 0.1 dex accuracy does not hold
- Input is already a synthetic diffmah population catalog and the goal is downstream galaxy-halo connection rather than validating the assembly model itself

## Inputs

- Per-halo diffmah parameter fits (3-parameter power-law: early-time, peak-time, late-time parameters) from a calibration sample of halo merger trees
- Halo mass accretion histories (MAHs) from cosmological simulations (e.g. gravity-only or TNG hydrodynamical merger trees)
- Large-scale density field / environment labels for assembly-bias residual checks

## Outputs

- Synthetic population catalog of Monte Carlo halo histories (halo_id, cosmic time, log halo mass), e.g. synthetic_halo_histories.csv
- Validation diagnostics: per-halo dex-level accuracy vs calibration sample and ensemble-scatter comparison

## How to apply

Install diffmah (pip or conda-forge; requires python>=3.9 with numpy and jax dependencies). Load per-halo diffmah parameter fits (the 3-parameter power-law fit with early-time, peak-time, and late-time parameters) from your calibration sample of merger trees, then construct the population-level distribution of those fitted parameters and Monte Carlo sample from it so the synthetic ensemble reproduces both the mean mass growth across time and the scatter of individual assembly histories. Evaluate the diffmah model on the sampled parameters using JAX autodiff to produce mass assembly histories (halo_id, cosmic time, log halo mass) for synthetic halos of present-day mass above 10^11 solar masses. Validate by (a) checking per-halo approximation accuracy of ~0.1 dex for t > 1 Gyr against the calibration sample, and (b) confirming that residual errors in the approximated histories show negligible correlation with the large-scale density field, i.e. assembly bias is captured. The diffmah_fitter_demo.ipynb notebook and history_fitting_script.py (mpi4py) in the repository demonstrate fitting MAHs for single halos and large halo samples in parallel.

## Related tools

- **diffmah** (Differentiable empirical model used to evaluate mass assembly histories for Monte Carlo-sampled halo parameters) — https://github.com/ArgonneCPAC/diffmah/
- **JAX** (Autodiff library underlying diffmah; evaluates the model and self-consistently captures mean and variance of halo mass accretion rate across cosmic time) — https://jax.readthedocs.io/en/latest/
- **Python >=3.9** (Runtime environment requirement for diffmah (installed with numpy and jax dependencies))
- **numpy** (Dependency used for constructing population-level distributions of fitted diffmah parameters and Monte Carlo sampling)

## Examples

```
conda create -c conda-forge -n diffit python=3.9 numpy jax pytest ipython jupyter matplotlib scipy h5py diffmah
```

## Evaluation signals

- Per-halo synthetic MAHs approximate calibration histories with typical accuracy of 0.1 dex for t > 1 Gyr
- All validated halos have present-day mass greater than 10^11 solar masses
- Ensemble scatter of Monte Carlo histories faithfully captures the diversity of halo mass assembly relative to the task_001 calibration sample
- Residual errors in approximated halo histories show negligible correlation with the large-scale density field (assembly bias captured)
- Output CSV contains the required columns (halo_id, cosmic time, log halo mass) with monotonically or physically plausible mass growth

## Limitations

- Validated accuracy (0.1 dex for t > 1 Gyr) applies only to halos of present-day mass greater than 10^11 solar masses; behavior outside this regime is not guaranteed
- The model is an empirical 3-parameter power-law approximation; it reproduces statistics of assembly (mean, diversity, assembly bias) rather than resolving physical merger physics
- JAX is a hard dependency of diffmah, so environments without JAX-compatible accelerators/python>=3.9 cannot run the model

## Evidence

- [abstract] A 3-parameter power-law model approximates halo growth with typical accuracy of 0.1 dex for t > 1 Gyr for halos of present-day mass greater than 10^11 solar masses: "we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$"
- [abstract] Assembly bias is captured with negligible residual correlations: "the connection between halo assembly and the large-scale density field, known as halo assembly bias, is accurately captured by our model"
- [abstract] Monte Carlo realizations of representative halo histories are supported: "Our publicly available source code can be used to generate Monte Carlo realizations of cosmologically representative halo histories"
- [methods] diffmah installation and dependencies: "diffmah requires python>=3.9, and has the following dependencies"
- [readme] Demo notebook for fitting individual halo MAHs: [evidence span withheld — anchor status: external_doc]
