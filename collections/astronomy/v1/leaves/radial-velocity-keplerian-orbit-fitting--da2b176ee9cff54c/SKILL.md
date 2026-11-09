---
name: radial-velocity-keplerian-orbit-fitting
description: Use when analysing multi-instrument RV timeseries (e.g. 51peg.vels/LICK,
  HIP 21850 AAT/HARPS) to detect planetary signals in highly multi-modal posteriors
  and return the -fitting Keplerian model automatically.
when_to_use_negative:
- The input is not a radial-velocity (or RV+astrometry) timeseries but an already-completed
  feature/parameter table needing downstream statistics
- A single fixed model must be sampled once without automatic model-order selection
  — the autorun loop is designed for iterative model comparison, not one-off fits
- Purely statistical model comparison is expected to be decisive on its own — the
  paper warns that 'purely statistical inference might be insufficient on its own
  for robust exoplanet detection'
edam_operation: http://edamontology.org/operation_2426
tools:
- name: EMPEROR (astroemperor)
  role: Framework that loads RV data, adds Keplerian/noise blocks, runs the autorun
    model-comparison loop and writes stats.tex/values.tex/best_fit.dat outputs
  repo: https://github.com/ReddTea/astroemperor
- name: reddemcee
  role: Adaptive Parallel Tempering MCMC sampler set as the sampling engine via set_engine('reddemcee');
    provides evidence via TI, stepping stones and hybrid methods
  repo: https://github.com/ReddTea/reddemcee
- name: emcee
  role: Base ensemble sampler on which reddemcee is built; used for autocorrelation-time
    diagnostics
  repo: https://github.com/dfm/emcee
- name: kepler.py
  role: Fast and stable Kepler's-equation solver used for the Keplerian parameterisation
    inside EMPEROR
  repo: https://github.com/dfm/kepler.py
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_004
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:2e49f56ae53e567382225cc3e6fb340f7c55fc2bc6e2abf32bff1cd5e29be7a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ReddTea/astroemperor
  tools:
  - EMPEROR (astroemperor)
  - Python
  - dynesty
  - emcee
  - kepler.py
  - reddemcee
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# radial-velocity-keplerian-orbit-fitting

## Summary

Fit Keplerian orbit models to radial-velocity time series (e.g. the 51 Peg LICK RV catalogue) using EMPEROR's adaptive parallel-tempering MCMC (reddemcee) and automatically select the optimum number of Keplerian blocks via BIC or Bayesian evidence comparison.

## When to use

Use when analysing multi-instrument RV timeseries (e.g. 51peg.vels/LICK, HIP 21850 AAT/HARPS) to detect planetary signals in highly multi-modal posteriors and return the best-fitting Keplerian model automatically.

## When NOT to use

- The input is not a radial-velocity (or RV+astrometry) timeseries but an already-completed feature/parameter table needing downstream statistics
- A single fixed model must be sampled once without automatic model-order selection — the autorun loop is designed for iterative model comparison, not one-off fits
- Purely statistical model comparison is expected to be decisive on its own — the paper warns that 'purely statistical inference might be insufficient on its own for robust exoplanet detection'

## Inputs

- Radial-velocity catalogue (e.g. 51peg.vels from LICK) placed in datafiles/<system>/RV
- RV + astrometry datasets for joint modelling (e.g. HIP 21850: AAT, HARPS1, HARPS2, HARPS3)
- Engine configuration [ntemps, nwalkers, nsweeps, nsteps] and run_config burn-in settings
- Parameter conditions (limits, priors, init_pos) for Keplerian, Offset, Jitter and noise blocks

## Outputs

- Per-run tables in datalogs/<system>/run/k/tables: stats.tex (ln Z, BIC) and values.tex (posterior parameter estimates)
- best_fit.dat with the best-fitting model parameters
- model_comparison.csv listing each run's comparison statistic with the optimum model identified
- Posterior period/amplitude estimates with credible intervals (e.g. Barnard's star P = 3.1536±0.0003 d, m·sini = 0.38±0.03 M_earth)

## How to apply

Set up a working directory with 'datafiles' and 'datalogs' subfolders, place the RV catalogue under datafiles/<system>/RV, and instantiate astroemperor.Simulation with load_data. Configure the reddemcee adaptive parallel-tempering engine via set_engine with engine_config['setup'] = [ntemps, nwalkers, nsweeps, nsteps] (e.g. [10, 256, 2048, 1]), then constrain parameters with add_condition blocks (limits, priors, init_pos — e.g. Period 1 limits [3, 5] d, Amplitude 1 limits [45, 60] m/s for 51 Peg, starmass 1.12, keplerian_parameterisation 1). Launch sim.autorun(from, to), which iteratively adds one Keplerian Block per run and samples each with parallel-tempering MCMC; subsequent runs are constrained by previous posteriors modelled with KDE or Bayesian Gaussian Mixtures. Choose the comparison statistic with set_comparison_criteria('BIC' or 'Evidence') and set_tolerance (e.g. 5 for BIC, 10 for Evidence; evidence via thermodynamic integration, stepping stones 'ss', or hybrid); the loop stops when BIC_old - BIC_new fails to exceed tolerance, returning the optimum model. Extract posteriors and statistics from datalogs/<system>/run_1/k_1/tables (stats.tex, values.tex, best_fit.dat).

## Related tools

- **EMPEROR (astroemperor)** (Framework that loads RV data, adds Keplerian/noise blocks, runs the autorun model-comparison loop and writes stats.tex/values.tex/best_fit.dat outputs) — https://github.com/ReddTea/astroemperor
- **reddemcee** (Adaptive Parallel Tempering MCMC sampler set as the sampling engine via set_engine('reddemcee'); provides evidence via TI, stepping stones and hybrid methods) — https://github.com/ReddTea/reddemcee
- **emcee** (Base ensemble sampler on which reddemcee is built; used for autocorrelation-time diagnostics) — https://github.com/dfm/emcee
- **kepler.py** (Fast and stable Kepler's-equation solver used for the Keplerian parameterisation inside EMPEROR) — https://github.com/dfm/kepler.py

## Examples

```
sim.set_engine('reddemcee'); sim.engine_config['setup'] = [10, 256, 2048, 1]; sim.set_comparison_criteria('Evidence'); sim.set_tolerance(10); sim.autorun(1, 1)
```

## Evaluation signals

- The autorun loop terminates exactly when the newest model fails to improve the chosen statistic (BIC_old - BIC_new vs tolerance; Evidence with set_tolerance(10))
- Recovered posterior parameters match reported credible intervals, e.g. Barnard's star P = 3.1536±0.0003 d and m·sini = 0.38±0.03 M_earth
- For the 51 Peg APT-vs-DNS check, the effective sampling ratio reaches ~3.76 with tighter APT parameter posteriors
- datalogs/<system>/run/k/tables contains stats.tex and values.tex for each run, and best_fit.dat is written
- model_comparison.csv lists each run's statistic (ln Z or BIC) with the optimum model flagged

## Limitations

- Purely statistical inference might be insufficient on its own for robust exoplanet detection (paper abstract)
- The Bayesian evidence ln Z is sensitive to prior selection (demonstrated on HD 55693), so prior choices via add_condition materially change model comparison outcomes
- kepler.solve 'will return garbage for eccentricities out of the range zero to one' (kepler.py README)

## Evidence

- [intro] EMPEROR is a Python-based algorithm that automatically searches for signals in Radial Velocity timeseries employing Markov chains and parallel tempering methods: "`EMPEROR` (Exoplanet Mcmc Parallel tEmpering for Rv Orbit Retrieval), is a Python-based algorithm that automatically searches for signals in Radial Velocity timeseries"
- [methods] Model comparison statistic and tolerance are user-configurable: "The comparison statistic can be set with the `set_comparison_criteria` method, and the tolerance changed with the `set_tolerance` method."
- [methods] Working directory must contain datafiles and datalogs subfolders: "We need to set up our working directory with two subfolders, `datafiles` and `datalogs`, the former for data input, the later for output."
- [readme] README overview of EMPEROR's automatic model comparison: "performs model comparisons automatically to return the optimum model that best describes the data."
- [readme] reddemcee README describes the adaptive PT sampler: [evidence span withheld — anchor status: not_found]
