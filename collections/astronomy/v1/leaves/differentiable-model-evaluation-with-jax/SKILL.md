---
name: differentiable-model-evaluation-with-jax
description: Use when you hold per-halo fitted diffmah parameters (early-time, peak-time,
  late-time) and need to evaluate the differentiable model to generate mass assembly
  histories, e.g. Monte Carlo-sampled synthetic halo populations reproducing mean
  growth and assembly diversity.
when_to_use_negative:
- Input is raw halo mass assembly histories / merger trees from a simulation that
  still need to be fit with the 3-parameter model (use the diffmah fitting workflow,
  e.g. diffmah_fitter_demo.ipynb or history_fitting_script.py, instead)
- Target halos have present-day mass below 10^11 solar masses or you need accuracy
  at t <= 1 Gyr, where the 0.1 dex accuracy claim does not apply
- Analysis requires simulating baryonic physics directly (e.g. reproducing TNG hydrodynamical
  dynamics) rather than empirically approximating halo growth
edam_operation: http://edamontology.org/operation_0242
tools:
- name: diffmah
  role: Differentiable 3-parameter power-law model evaluated on parameter sets to
    compute halo mass assembly histories
  repo: github.com/ArgonneCPAC/diffmah
- name: JAX
  role: Autodiff library underpinning model evaluation, so mean and variance of halo
    mass accretion rate are captured self-consistently
  repo: github.com/ArgonneCPAC/diffmah
- name: numpy
  role: Array handling for the calibration sample, Monte Carlo parameter draws, and
    output catalogs
  repo: github.com/ArgonneCPAC/diffmah
- name: Python >=3.9
  role: Runtime environment required by diffmah
  repo: github.com/ArgonneCPAC/diffmah
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:77f824297abbaf0f8da8357760dd816cabc736660596f863ff99f1d92b0f0ce5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JAX
  - Python >=3.9
  - diffmah
  - numpy
  edam_operation: http://edamontology.org/operation_0242
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# differentiable-model-evaluation-with-jax

## Summary

Evaluate the diffmah differentiable halo-assembly model — a 3-parameter power-law MAH model — with JAX autodiff to generate mass assembly histories for individual or Monte Carlo-sampled populations of dark matter halos.

## When to use

Use when you hold per-halo fitted diffmah parameters (early-time, peak-time, late-time) and need to evaluate the differentiable model to generate mass assembly histories, e.g. Monte Carlo-sampled synthetic halo populations reproducing mean growth and assembly diversity.

## When NOT to use

- Input is raw halo mass assembly histories / merger trees from a simulation that still need to be fit with the 3-parameter model (use the diffmah fitting workflow, e.g. diffmah_fitter_demo.ipynb or history_fitting_script.py, instead)
- Target halos have present-day mass below 10^11 solar masses or you need accuracy at t <= 1 Gyr, where the 0.1 dex accuracy claim does not apply
- Analysis requires simulating baryonic physics directly (e.g. reproducing TNG hydrodynamical dynamics) rather than empirically approximating halo growth

## Inputs

- Per-halo 3-parameter diffmah fits (early-time, peak-time, late-time parameters) from fitting halo merger trees in high-resolution cosmological simulations
- Cosmic time grid on which mass assembly histories are evaluated
- Population-level distribution of fitted diffmah parameters across the halo sample

## Outputs

- Synthetic halo population catalog (halo_id, cosmic time, log halo mass), e.g. synthetic_halo_histories.csv
- Mean halo mass accretion rate and its variance across cosmic time for the synthetic population

## How to apply

Install diffmah (python>=3.9, numpy, jax) via `conda install -c conda-forge diffmah` or `pip install .` from the cloned GitHub repository. Load the calibration sample of 3-parameter power-law fits (early-time, peak-time, and late-time parameters) to individual halo merger trees, and construct the population-level distribution of these fitted parameters. Monte Carlo sample parameters from that population distribution so the ensemble reproduces both the average mass growth across time and the diversity with which halos assemble their mass. Evaluate the diffmah model on the sampled parameter sets using JAX autodiff to produce log halo mass as a function of cosmic time for synthetic halos of present-day mass above 10^11 solar masses. Validate that the generated histories approximate halo growth with typical accuracy of 0.1 dex for t > 1 Gyr and that the ensemble scatter faithfully captures the assembly diversity of the calibration sample.

## Related tools

- **diffmah** (Differentiable 3-parameter power-law model evaluated on parameter sets to compute halo mass assembly histories) — github.com/ArgonneCPAC/diffmah
- **JAX** (Autodiff library underpinning model evaluation, so mean and variance of halo mass accretion rate are captured self-consistently) — github.com/ArgonneCPAC/diffmah
- **numpy** (Array handling for the calibration sample, Monte Carlo parameter draws, and output catalogs) — github.com/ArgonneCPAC/diffmah
- **Python >=3.9** (Runtime environment required by diffmah) — github.com/ArgonneCPAC/diffmah

## Examples

```
conda install -c conda-forge diffmah
```

## Evaluation signals

- Generated histories approximate halo growth with typical accuracy of 0.1 dex for t > 1 Gyr
- Validation restricted to halos of present-day mass greater than 10^11 solar masses
- Ensemble scatter of Monte Carlo histories faithfully captures the diversity of halo mass assembly relative to the calibration sample
- Average mass growth across time of the synthetic population matches that of the calibration sample
- Output CSV carries the expected schema: halo_id, cosmic time, log halo mass

## Limitations

- The 3-parameter model's stated 0.1 dex accuracy holds only for t > 1 Gyr and halos of present-day mass greater than 10^11 solar masses
- diffmah requires python>=3.9 plus numpy and jax as dependencies
- Model is empirical — it approximates halo growth rather than simulating it, and was calibrated on gravity-only and TNG hydrodynamical simulations
- No changelog is available for the diffmah repository, so version-specific behavior must be checked against releases directly

## Evidence

- [abstract] abstract: "Our python implementation is based on the autodiff library JAX, and so our model self-consistently captures the mean and variance of halo mass accretion rate across cosmic time."
- [methods] methods: "The latest release of diffmah is available on pip and conda-forge"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
