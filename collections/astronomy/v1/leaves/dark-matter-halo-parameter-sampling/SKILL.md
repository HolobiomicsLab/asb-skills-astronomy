---
name: dark-matter-halo-parameter-sampling
description: Use when you have per-halo diffmah parameter fits (early-time, peak-time,
  late-time power-law parameters) and need synthetic Monte Carlo realizations of halo
  mass histories whose ensemble reproduces population-average growth and assembly
  diversity.
when_to_use_negative:
- You need to fit diffmah parameters to individual simulated halos from merger trees
  in the first place — that is the upstream calibration/fitting task, not population
  sampling.
- You require accurate halo growth approximations for t < 1 Gyr or for halos below
  present-day mass 10^11 solar masses, outside the demonstrated validity range.
- You need physically simulated halo evolution (e.g. from gravity-only or TNG hydrodynamical
  simulations) rather than an empirical, statistically representative realization.
tools:
- name: diffmah
  role: Differentiable empirical model providing the parameterization of halo mass
    assembly histories and the machinery to generate Monte Carlo realizations of cosmologically
    representative halo histories
  repo: https://github.com/ArgonneCPAC/diffmah/
- name: JAX
  role: Autodiff library used to evaluate the differentiable diffmah model on Monte
    Carlo sampled parameter sets
  repo: https://jax.readthedocs.io/en/latest/
- name: numpy
  role: Numerical dependency for constructing population distributions of fitted parameters
    and writing the synthetic catalog
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:93a1a74814a168726b181e8f1e7a54d31f29aaae93746a8e0213e1c6606dc9ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArgonneCPAC/diffmah
  tools:
  - JAX
  - Python >=3.9
  - diffmah
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# dark-matter-halo-parameter-sampling

## Summary

Monte Carlo sampling of diffmah model parameters to generate cosmologically representative populations of synthetic dark matter halo mass assembly histories. The diffmah source code is explicitly designed to produce such realizations, reproducing both mean growth and the diversity of halo assembly.

## When to use

Use when you have per-halo diffmah parameter fits (early-time, peak-time, late-time power-law parameters) and need synthetic Monte Carlo realizations of halo mass histories whose ensemble reproduces population-average growth and assembly diversity.

## When NOT to use

- You need to fit diffmah parameters to individual simulated halos from merger trees in the first place — that is the upstream calibration/fitting task, not population sampling.
- You require accurate halo growth approximations for t < 1 Gyr or for halos below present-day mass 10^11 solar masses, outside the demonstrated validity range.
- You need physically simulated halo evolution (e.g. from gravity-only or TNG hydrodynamical simulations) rather than an empirical, statistically representative realization.

## Inputs

- Per-halo diffmah parameter fits from halo merger-tree fitting (early-time, peak-time, and late-time parameters of the 3-parameter power-law model)
- Population-level distribution of the 3 fitted diffmah parameters across the calibration halo sample
- Cosmic time array over which mass assembly histories are evaluated

## Outputs

- Synthetic population catalog with columns halo_id, cosmic time, and log halo mass (e.g. synthetic_halo_histories.csv)
- Monte Carlo ensemble of diffmah mass assembly histories for cosmologically representative halos

## How to apply

Load the calibration sample of per-halo 3-parameter diffmah power-law fits (early-time, peak-time, late-time parameters) and construct the population-level distribution of these parameters. Monte Carlo sample parameter sets from this distribution so that the ensemble reproduces both the average mass growth across cosmic time and the diversity with which halos assemble their mass — the explicit goal of the diffmah population model. Evaluate the differentiable diffmah model on the sampled parameter sets using JAX autodiff to generate mass assembly histories for synthetic halos spanning present-day masses above 10^11 solar masses. Write the synthetic population catalog as (halo_id, cosmic time, log halo mass) rows, e.g. to synthetic_halo_histories.csv. The population model is judged by whether the ensemble scatter matches the calibration sample and individual histories approximate true growth with ~0.1 dex accuracy for t > 1 Gyr.

## Related tools

- **diffmah** (Differentiable empirical model providing the parameterization of halo mass assembly histories and the machinery to generate Monte Carlo realizations of cosmologically representative halo histories) — https://github.com/ArgonneCPAC/diffmah/
- **JAX** (Autodiff library used to evaluate the differentiable diffmah model on Monte Carlo sampled parameter sets) — https://jax.readthedocs.io/en/latest/
- **numpy** (Numerical dependency for constructing population distributions of fitted parameters and writing the synthetic catalog)

## Examples

```
conda create -c conda-forge -n diffit python=3.9 numpy jax pytest ipython jupyter matplotlib scipy h5py diffmah
```

## Evaluation signals

- Typical accuracy of ~0.1 dex between Monte Carlo diffmah histories and halo growth for t > 1 Gyr for halos of present-day mass greater than 10^11 solar masses
- Ensemble average mass growth across time matches the task_001 calibration sample
- Ensemble scatter faithfully captures the diversity with which halos assemble their mass relative to the calibration sample
- Output catalog schema contains halo_id, cosmic time, and log halo mass, and spans present-day masses above 10^11 solar masses

## Limitations

- The 0.1 dex accuracy demonstration applies only for t > 1 Gyr and halos of present-day mass greater than 10^11 solar masses; behavior outside this regime is not established.
- diffmah is an empirical model, so realizations are statistically representative of halo populations rather than physically simulated trajectories.
- Environment constraints: diffmah requires python>=3.9 with numpy and jax dependencies; the project data is hosted externally at portal.nersc.gov and no changelog is available for the repository.

## Evidence

- [abstract] Monte Carlo realizations of cosmologically representative halo histories: "Our publicly available source code can be used to generate Monte Carlo realizations of cosmologically representative halo histories"
- [abstract] 0.1 dex accuracy threshold: "we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$"
- [abstract] population model goal: "which not only reproduces average mass growth across time, but also faithfully captures the diversity with which halos assemble their mass"
- [readme] installation and environment: [evidence span withheld — anchor status: external_doc]
