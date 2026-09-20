---
name: halo-mass-accretion-history-fitting
description: Use when you have halo merger tree catalogs (e.g., TNG, gravity-only
  simulations) and need compact per-halo parametric fits of log halo mass vs. cosmic
  time for host halos or subhalos above 10^11 Msun.
when_to_use_negative:
- Your goal is a population-level or Monte Carlo model of halo histories rather than
  per-halo fits (use the diffmah population-assembly model instead, calibrated on
  these fits).
- Halos have present-day mass below 10^11 Msun or you need accuracy at t < 1 Gyr,
  outside the demonstrated regime.
- You need baryonic physics resolved within halos rather than a smooth parametric
  mass-assembly approximation.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: diffmah
  role: Provides the 3-parameter differentiable power-law model for halo mass assembly
    histories being fitted
  repo: https://github.com/ArgonneCPAC/diffmah/
- name: JAX
  role: Autodiff backend enabling JAX-based gradient optimization of the log-mass
    residuals in diffmah fitting
  repo: https://jax.readthedocs.io/en/latest/
- name: numpy
  role: Array handling for extracting and fitting per-halo mass accretion histories
  repo: https://numpy.org/
- name: mpi4py
  role: Parallel fitting of MAHs for large numbers of simulated halos via history_fitting_script.py
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:8d8b342d8e22be5b5b2dbd3cf01cf8c991906aadc1d571b8b061c1d07432f81d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArgonneCPAC/diffmah
  tools:
  - JAX
  - Python
  - Python >=3.9
  - diffmah
  - mpi4py
  - numpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# halo-mass-accretion-history-fitting

## Summary

Fits the 3-parameter diffmah power-law model (early-time power-law index, transition time, transition smoothness) to individual halo mass accretion histories from cosmological merger trees, achieving ~0.1 dex accuracy in log-mass for t > 1 Gyr for halos above 10^11 Msun.

## When to use

Use when you have halo merger tree catalogs (e.g., TNG, gravity-only simulations) and need compact per-halo parametric fits of log halo mass vs. cosmic time for host halos or subhalos above 10^11 Msun.

## When NOT to use

- Your goal is a population-level or Monte Carlo model of halo histories rather than per-halo fits (use the diffmah population-assembly model instead, calibrated on these fits).
- Halos have present-day mass below 10^11 Msun or you need accuracy at t < 1 Gyr, outside the demonstrated regime.
- You need baryonic physics resolved within halos rather than a smooth parametric mass-assembly approximation.

## Inputs

- Halo merger tree catalogs (host halos and subhalos) from gravity-only or TNG hydrodynamical simulations
- Per-halo mass accretion histories: log halo mass vs. cosmic time arrays (numpy arrays)
- Mass selection threshold: present-day halo mass > 10^11 Msun

## Outputs

- Per-halo best-fit 3-parameter diffmah parameters (early-time power-law index, transition time, transition smoothness)
- Fit diagnostics: log-mass residuals vs. cosmic time (target ~0.1 dex for t > 1 Gyr)
- Assembled fit catalog, e.g., diffmah_fits_tng.csv

## How to apply

Load the merger tree catalogs and select host halos and subhalos with present-day mass greater than 10^11 Msun, extracting each halo's mass accretion history (log halo mass vs. cosmic time). Fit the 3-parameter diffmah power-law model by minimizing log-mass residuals with a JAX-based gradient optimizer — the autodiff library JAX is central to the diffmah implementation, so the loss must stay JAX-traceable. Retain fits meeting the article's acceptance criterion of typical accuracy of ~0.1 dex for t > 1 Gyr. For large halo samples, fit in parallel with mpi4py following the repository's history_fitting_script.py pattern, or use diffmah_fitter_demo.ipynb for a single-halo walkthrough. Assemble best-fit parameters and diagnostics per halo (e.g., diffmah_fits_tng.csv) for downstream population modeling.

## Related tools

- **diffmah** (Provides the 3-parameter differentiable power-law model for halo mass assembly histories being fitted) — https://github.com/ArgonneCPAC/diffmah/
- **JAX** (Autodiff backend enabling JAX-based gradient optimization of the log-mass residuals in diffmah fitting) — https://jax.readthedocs.io/en/latest/
- **numpy** (Array handling for extracting and fitting per-halo mass accretion histories) — https://numpy.org/
- **mpi4py** (Parallel fitting of MAHs for large numbers of simulated halos via history_fitting_script.py)

## Examples

```
conda install -c conda-forge diffmah && jupyter notebook diffmah_fitter_demo.ipynb
```

## Evaluation signals

- Log-mass residuals between fit and halo history are ~0.1 dex or better for t > 1 Gyr (article acceptance criterion).
- All fitted halos have present-day mass > 10^11 Msun, covering both subhalos and host halos.
- The three fitted parameters (early-time power-law index, transition time, transition smoothness) are defined and finite for every halo in the output catalog.
- Residual approximation errors show negligible correlation with the large-scale density field (assembly bias accurately captured).
- Output CSV contains per-halo rows with halo_id, best-fit parameters, and fit diagnostics suitable as a calibration sample for population modeling.

## Limitations

- Accuracy claim is for t > 1 Gyr and present-day masses above 10^11 Msun; behavior outside this regime is not demonstrated.
- The 3-parameter model is a smooth approximation — it does not capture stochastic, burst-like accretion fluctuations of individual halos.
- diffmah requires python>=3.9; installation is via conda-forge or pip from source.
- Fitting large samples benefits from parallelism (mpi4py); serial fitting of big merger-tree catalogs may be slow.

## Evidence

- [abstract] 0.1 dex accuracy claim: "we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$"
- [abstract] JAX autodiff implementation: "Our python implementation is based on the autodiff library JAX"
- [methods] conda-forge installation: "conda install -c conda-forge diffmah"
- [readme] demo notebook for fitting single halo MAH: [evidence span withheld — anchor status: external_doc]
- [readme] parallel fitting with mpi4py: [evidence span withheld — anchor status: external_doc]
