---
name: cosmological-simulation-merger-tree-analysis
description: Use when you have halo merger tree catalogs (e.g., from TNG or gravity-only
  simulations) and need to model, fit, or evaluate individual halo mass accretion
  histories log(Mhalo) vs. cosmic time.
when_to_use_negative:
- The input is a population-level mean halo growth or statistical model of mass accretion
  rather than individual halo merger-tree histories — use the paper's halo-population
  assembly model instead
- You need accurate fits at very early times (t < 1 Gyr); the reported 0.1 dex typical
  accuracy applies only for t > 1 Gyr
- Halos have present-day mass below 10^11 solar masses, which are outside the validated
  sample
edam_operation: http://edamontology.org/operation_2239
tools:
- name: diffmah
  role: Differentiable 3-parameter power-law model for mass assembly histories of
    individual and populations of dark matter halos; provides the fitting machinery
  repo: github.com/ArgonneCPAC/diffmah
- name: JAX
  role: Autodiff library underlying the gradient-based optimization of diffmah log-mass
    residuals
  repo: github.com/ArgonneCPAC/diffmah
- name: numpy
  role: Array handling for merger-tree histories and aggregation of per-halo residual
    statistics
- name: Python
  role: Runtime environment (diffmah requires python>=3.9); example fitting scripts
    use Python with mpi4py for parallel fitting
  repo: github.com/ArgonneCPAC/diffmah
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:a6172fe7801596be94a10da0df0f466d4d0af4dc448ea708ead708e510079825
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JAX
  - Python
  - Python >=3.9
  - diffmah
  - numpy
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cosmological-simulation-merger-tree-analysis

## Summary

Fitting 3-parameter diffmah power-law models to individual halo mass assembly histories (host halos and subhalos) extracted from cosmological simulation merger trees, using JAX-based gradient optimization to achieve ~0.1 dex typical accuracy in log-mass approximation.

## When to use

Use when you have halo merger tree catalogs (e.g., from TNG or gravity-only simulations) and need to model, fit, or evaluate individual halo mass accretion histories log(Mhalo) vs. cosmic time.

## When NOT to use

- The input is a population-level mean halo growth or statistical model of mass accretion rather than individual halo merger-tree histories — use the paper's halo-population assembly model instead
- You need accurate fits at very early times (t < 1 Gyr); the reported 0.1 dex typical accuracy applies only for t > 1 Gyr
- Halos have present-day mass below 10^11 solar masses, which are outside the validated sample

## Inputs

- Halo merger tree catalogs from cosmological simulations (gravity-only or TNG hydrodynamical), containing log halo mass vs. cosmic time histories
- Halo selection filter: present-day mass > 10^11 solar masses, including subhalos and host halos
- diffmah project data from https://portal.nersc.gov/project/hacc/aphearin/diffmah_data/

## Outputs

- Best-fit 3-parameter diffmah parameters (early-time power-law index, transition time, transition smoothness) per halo (e.g., diffmah_fits_tng.csv)
- Fit diagnostics: pointwise dex residuals and aggregated accuracy statistics such as median absolute residual for t > 1 Gyr (e.g., residual_accuracy_report.csv)
- Monte Carlo realizations of cosmologically representative halo histories generated from fitted parameters

## How to apply

Load halo merger tree catalogs and select host halos and subhalos with present-day mass greater than 10^11 solar masses, extracting each halo's mass accretion history (log halo mass vs. cosmic time) with numpy array handling. Fit the 3-parameter diffmah power-law model — parameterized by early-time power-law index, transition time, and transition smoothness — to each history by minimizing log-mass residuals with a JAX-based gradient optimizer. Restrict fit-quality evaluation to t > 1 Gyr and compute pointwise residuals in dex (log10 of fitted/simulated mass ratio). Judge success by a typical (median) absolute residual of ~0.1 dex across the sample; flag results if the median exceeds 0.1 dex. Use the published project data at portal.nersc.gov/project/hacc/aphearin/diffmah_data/ and the diffmah_fitter_demo.ipynb notebook or history_fitting_script.py (mpi4py parallel fitting) as reference implementations.

## Related tools

- **diffmah** (Differentiable 3-parameter power-law model for mass assembly histories of individual and populations of dark matter halos; provides the fitting machinery) — github.com/ArgonneCPAC/diffmah
- **JAX** (Autodiff library underlying the gradient-based optimization of diffmah log-mass residuals) — github.com/ArgonneCPAC/diffmah
- **numpy** (Array handling for merger-tree histories and aggregation of per-halo residual statistics)
- **Python** (Runtime environment (diffmah requires python>=3.9); example fitting scripts use Python with mpi4py for parallel fitting) — github.com/ArgonneCPAC/diffmah

## Examples

```
conda install -c conda-forge diffmah && python history_fitting_script.py
```

## Evaluation signals

- Median absolute residual between fitted and simulated log mass is ~0.1 dex or below for t > 1 Gyr across the selected sample
- All fitted halos have present-day mass > 10^11 Msun and include both subhalos and host halos
- Residual errors show negligible correlation with the large-scale density field (assembly bias is captured)
- Output CSVs contain best-fit 3-parameter values (early-time index, transition time, transition smoothness) plus per-halo diagnostics for every selected halo
- Fitted curves reproduce both average mass growth across time and the diversity of halo assembly histories

## Limitations

- The 0.1 dex typical accuracy claim is restricted to t > 1 Gyr; early-time approximation errors are not covered by the reported criterion
- Validation covers halos of present-day mass greater than 10^11 Msun in gravity-only and TNG hydrodynamical simulations; extrapolation to other regimes is untested here
- Requires python>=3.9 and JAX; large-sample fitting over many halos relies on mpi4py parallelization (history_fitting_script.py)

## Evidence

- [abstract] we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$: "we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater"
- [readme] The `diffmah_fitter_demo.ipynb` notebook demonstrates how to fit the MAH of a simulated halo with a diffmah approximation.: [evidence span withheld — anchor status: external_doc]
- [methods] diffmah requires python>=3.9, and has the following dependencies: "diffmah requires python>=3.9, and has the following dependencies"
- [abstract] Using large samples of halo merger trees taken from high-resolution cosmological simulations: "Using large samples of halo merger trees taken from high-resolution cosmological simulations"
