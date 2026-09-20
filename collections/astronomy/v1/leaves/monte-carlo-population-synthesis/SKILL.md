---
name: monte-carlo-population-synthesis
description: Use when you have per-halo diffmah parameter fits (early-time, peak-time,
  late-time power-law parameters) and need a synthetic population of halo histories
  matching the ensemble's mean growth and scatter across cosmic time.
when_to_use_negative:
- You need to fit individual simulated halo MAHs to obtain diffmah parameters — that
  is the history-fitting step (e.g. diffmah_fitter_demo.ipynb / history_fitting_script.py),
  not population Monte Carlo sampling
- The target population includes halos below 10^11 solar masses, where the 3-parameter
  model's stated ~0.1 dex accuracy for t > 1 Gyr has not been demonstrated
- You need baryonic/hydrodynamical halo physics beyond what the empirical diffmah
  parameterization captures (TNG was used only as a validation simulation)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: diffmah
  role: Empirical differentiable model whose 3-parameter population distribution is
    Monte Carlo sampled to generate synthetic halo assembly histories
  repo: https://github.com/ArgonneCPAC/diffmah/
- name: jax
  role: Autodiff library underpinning diffmah; evaluates the model on sampled parameter
    sets to generate mass assembly histories
  repo: https://jax.readthedocs.io/en/latest/
- name: numpy
  role: Array/numerical backend used for building parameter distributions and sampling
    synthetic populations
  repo: https://numpy.org/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:5ec1dc098753ce505c5c367d30c8b4b62f4725f7ac5ae646ed95122f23cfd0c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArgonneCPAC/diffmah
  tools:
  - JAX
  - Python >=3.9
  - diffmah
  - jax
  - numpy
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# monte-carlo-population-synthesis

## Summary

Monte Carlo sampling of diffmah parameters from a population-level distribution to generate cosmologically representative synthetic halo mass assembly histories that reproduce both mean growth and assembly diversity.

## When to use

Use when you have per-halo diffmah parameter fits (early-time, peak-time, late-time power-law parameters) and need a synthetic population of halo histories matching the ensemble's mean growth and scatter across cosmic time.

## When NOT to use

- You need to fit individual simulated halo MAHs to obtain diffmah parameters — that is the history-fitting step (e.g. diffmah_fitter_demo.ipynb / history_fitting_script.py), not population Monte Carlo sampling
- The target population includes halos below 10^11 solar masses, where the 3-parameter model's stated ~0.1 dex accuracy for t > 1 Gyr has not been demonstrated
- You need baryonic/hydrodynamical halo physics beyond what the empirical diffmah parameterization captures (TNG was used only as a validation simulation)

## Inputs

- Per-halo diffmah parameter fits (3-parameter power-law: early-time, peak-time, late-time parameters) for the calibration halo sample
- Halo sample restricted to present-day mass > 10^11 solar masses (host halos and subhalos)

## Outputs

- Synthetic population catalog (halo_id, cosmic time, log halo mass) e.g. synthetic_halo_histories.csv
- Ensemble of Monte Carlo halo mass assembly histories spanning cosmic time

## How to apply

Load the 3-parameter diffmah fits (early-time, peak-time, and late-time parameters) from the calibration sample (e.g., per-halo fits from a prior fitting task) and construct the population-level distribution of these parameters. Monte Carlo sample parameter sets from this distribution so that the ensemble reproduces both the average mass growth across time and the diversity with which halos assemble their mass. Evaluate the differentiable diffmah model on the sampled parameters using JAX autodiff to produce mass assembly histories for synthetic halos, targeting present-day masses above 10^11 solar masses. Write the synthetic population catalog (halo_id, cosmic time, log halo mass) to a CSV such as synthetic_halo_histories.csv. The diffmah codebase is designed exactly for this: its publicly available source code can be used to generate Monte Carlo realizations of cosmologically representative halo histories.

## Related tools

- **diffmah** (Empirical differentiable model whose 3-parameter population distribution is Monte Carlo sampled to generate synthetic halo assembly histories) — https://github.com/ArgonneCPAC/diffmah/
- **jax** (Autodiff library underpinning diffmah; evaluates the model on sampled parameter sets to generate mass assembly histories) — https://jax.readthedocs.io/en/latest/
- **numpy** (Array/numerical backend used for building parameter distributions and sampling synthetic populations) — https://numpy.org/

## Examples

```
conda create -c conda-forge -n diffit python=3.9 numpy jax pytest ipython jupyter matplotlib scipy h5py diffmah
```

## Evaluation signals

- Monte Carlo histories approximate halo growth with typical accuracy of 0.1 dex for t > 1 Gyr for halos of present-day mass > 10^11 solar masses
- Ensemble scatter of synthetic histories faithfully captures the diversity of halo mass assembly relative to the calibration sample
- Ensemble average mass growth across time matches the calibration population's average
- Output CSV contains the expected schema: halo_id, cosmic time, log halo mass per synthetic halo
- Synthetic present-day masses span above 10^11 solar masses as specified

## Limitations

- Typical 0.1 dex accuracy is demonstrated only for t > 1 Gyr and halos of present-day mass greater than 10^11 solar masses; behavior outside this regime is not validated
- diffmah requires python>=3.9 and depends on numpy and jax; install via conda-forge or pip from source
- Model is empirical (3-parameter power-law) — it captures assembly statistics, not physical baryonic processes

## Evidence

- [abstract] Monte Carlo source code capability: "Our publicly available source code can be used to generate Monte Carlo realizations of cosmologically representative halo histories"
- [abstract] 3-parameter accuracy claim: "our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$"
- [abstract] JAX-based implementation: "Our python implementation is based on the autodiff library JAX"
- [methods] Installation via conda-forge: "conda install -c conda-forge diffmah"
- [methods] Python version requirement: "diffmah requires python>=3.9, and has the following dependencies"
- [readme] Demo fitter notebook reference: [evidence span withheld — anchor status: external_doc]
