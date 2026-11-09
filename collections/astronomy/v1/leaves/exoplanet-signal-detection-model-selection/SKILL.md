---
name: exoplanet-signal-detection-model-selection
description: Use when you have one or more RV timeseries (optionally with Hipparcos/Gaia
  astrometry) and need to automatically decide how many Keplerian signals plus noise
  models (Jitter, magnetic cycle, acceleration) describe the data without manual model
  comparison.
when_to_use_negative:
- The data are not radial-velocity (or joint RV+astrometry) timeseries of a star —
  e.g. transit photometry or already-reduced planet-catalogue tables — since EMPEROR's
  model space is Keplerian + noise blocks fitted to RV.
- You need a single fixed model fitted once (e.g. a known one-planet system with pre-set
  parameters) rather than an iterative model-selection search over increasing Keplerian
  counts.
- Purely statistical model choice is the only criterion you will accept — the paper
  cautions that 'purely statistical inference might be insufficient on its own for
  robust exoplanet detection', and Evidence (ln Z) is demonstrably sensitive to prior
  selection.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: EMPEROR (astroemperor)
  role: 'Main framework: runs the automatic model-selection loop (autorun) that adds
    Keplerians and compares models via BIC or Evidence'
  repo: https://github.com/ReddTea/astroemperor
- name: reddemcee
  role: Adaptive Parallel Tempering MCMC sampler configured via set_engine; provides
    evidence estimates (TI, stepping stones, hybrid)
  repo: https://github.com/ReddTea/reddemcee
- name: emcee
  role: Base ensemble sampler on which reddemcee's parallel-tempering implementation
    is built; used for autocorrelation times
  repo: https://github.com/dfm/emcee
- name: kepler.py
  role: Fast and stable Kepler's equation solver used to compute true-anomaly terms
    for the Keplerian blocks
  repo: https://github.com/dfm/kepler.py
- name: celerite2
  role: Gaussian-process noise model option among EMPEROR's various noise models
  repo: https://celerite2.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:9ca67296b18d3d2c5f227f2b80f34cebaf87a7872df0aea50678c6f7efb3d598
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ReddTea/astroemperor
  tools:
  - EMPEROR (astroemperor)
  - Python
  - celerite2
  - emcee
  - kepler.py
  - reddemcee
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# exoplanet-signal-detection-model-selection

## Summary

Automated detection and characterization of exoplanet signals in radial velocity (and joint astrometry) timeseries using parallel-tempering MCMC in EMPEROR, with an iterative Keplerian-by-Keplerian model-comparison loop that stops at the optimum model via BIC or Evidence tolerance criteria.

## When to use

Use when you have one or more RV timeseries (optionally with Hipparcos/Gaia astrometry) and need to automatically decide how many Keplerian signals plus noise models (Jitter, magnetic cycle, acceleration) best describe the data without manual model comparison.

## When NOT to use

- The data are not radial-velocity (or joint RV+astrometry) timeseries of a star — e.g. transit photometry or already-reduced planet-catalogue tables — since EMPEROR's model space is Keplerian + noise blocks fitted to RV.
- You need a single fixed model fitted once (e.g. a known one-planet system with pre-set parameters) rather than an iterative model-selection search over increasing Keplerian counts.
- Purely statistical model choice is the only criterion you will accept — the paper cautions that 'purely statistical inference might be insufficient on its own for robust exoplanet detection', and Evidence (ln Z) is demonstrably sensitive to prior selection.

## Inputs

- Radial velocity timeseries file(s) placed in datafiles/ (multi-instrument supported, e.g. 51peg.vels from LICK)
- Optional astrometry data (Hipparcos and Gaia) for joint RV+astrometry modelling
- User-set parameter conditions: limits, priors, and init_pos for Keplerian, Offset, Jitter blocks
- Engine setup [ntemps, nwalkers, nsweeps, nsteps] and burnin fraction

## Outputs

- Optimum (best-fitting) model with the automatically determined number of Keplerians and noise components
- Per-run comparison statistics (BIC or Evidence) tracked across successive runs
- Output tables and files in datalogs/: stats.tex, values.tex, best_fit.dat
- Posterior samples and convergence diagnostics from parallel-tempering MCMC

## How to apply

Set up a working directory with datafiles/ and datalogs/ subfolders and load the RV timeseries with Simulation.load_data (e.g. the 51 Peg dataset 51peg.vels from LICK). Configure the reddemcee adaptive parallel-tempering engine via set_engine with setup [ntemps, nwalkers, nsweeps, nsteps] (e.g. [10, 256, 2048, 1]) and set the burnin fraction. Define parameter conditions (limits, priors, init_pos) for the Keplerian, Offset, and Jitter blocks using add_condition (e.g. ['Period 1', 'limits', [3, 5]]). Launch the autorun control loop (e.g. sim.autorun(0, 5)), which iteratively adds one Keplerian per run, samples each with parallel tempering MCMC, and constrains new runs using posteriors of previous ones via KDE or Bayesian Gaussian Mixture priors. After each run compute the comparison statistic — BIC by default with tolerance 5, or Evidence via set_comparison_criteria('Evidence') with set_tolerance(10) (estimated by thermodynamic integration, stepping stones, or a hybrid) — and stop adding components when BIC_old - BIC_new fails the tolerance rule; the surviving model is the optimum. Per-run tables (stats.tex, values.tex, best_fit.dat) are written to datalogs for validation.

## Related tools

- **EMPEROR (astroemperor)** (Main framework: runs the automatic model-selection loop (autorun) that adds Keplerians and compares models via BIC or Evidence) — https://github.com/ReddTea/astroemperor
- **reddemcee** (Adaptive Parallel Tempering MCMC sampler configured via set_engine; provides evidence estimates (TI, stepping stones, hybrid)) — https://github.com/ReddTea/reddemcee
- **emcee** (Base ensemble sampler on which reddemcee's parallel-tempering implementation is built; used for autocorrelation times) — https://github.com/dfm/emcee
- **kepler.py** (Fast and stable Kepler's equation solver used to compute true-anomaly terms for the Keplerian blocks) — https://github.com/dfm/kepler.py
- **celerite2** (Gaussian-process noise model option among EMPEROR's various noise models) — https://celerite2.readthedocs.io/en/latest/

## Examples

```
sim = emp.Simulation(); sim.load_data('51Peg'); sim.set_engine('reddemcee'); sim.engine_config['setup'] = [10, 256, 2048, 1]; sim.add_condition(['Period 1', 'limits', [3, 5]]); sim.autorun(0, 5)
```

## Evaluation signals

- The autorun loop terminates exactly when the newest model fails to improve the chosen statistic by more than the tolerance (e.g. BIC_old - BIC_new > 5 for BIC, or tolerance 10 for Evidence).
- Per-run output files (stats.tex, values.tex, best_fit.dat) are present in datalogs/ and each run's statistic is recorded, with the optimum model identifiable from the comparison table.
- Convergence is confirmed by the sampler's convergence tests and autocorrelation-time estimates (get_autocorr_time in reddemcee) before a run's statistics are accepted.
- Recovered physical parameters fall in plausible ranges, as in the paper's benchmarks: Barnard's star period 3.1536±0.0003 d with 0.38±0.03 M_earth minimum mass; 51 Peg sampling efficiency (APT ~3.76x over DNS).
- Running the repo's test (test_basic.py on the 51 Peg or HIP 21850 data) reproduces the expected model-selection outcome for those datasets.

## Limitations

- Evidence (ln Z) estimates are sensitive to prior selection — demonstrated on HD 55693 where priors changed ln Z while P_rot = 29.72 d and P_mag = 2557.0 d were still recovered.
- Purely statistical inference (BIC/Evidence tolerance) might be insufficient on its own for robust exoplanet detection; results warrant astrophysical scrutiny.
- Posterior surfaces are highly multi-modal, so sampling efficiency depends on the adaptive parallel-tempering configuration; poor engine setup can miss signals.

## Evidence

- [intro] performs model comparisons automatically to return the optimum model that best describes the data: "performs model comparisons automatically to return the optimum model that best describes the data"
- [methods] comparison statistic set via set_comparison_criteria and tolerance: "The comparison statistic can be set with the `set_comparison_criteria` method, and the tolerance changed with the `set_tolerance` method."
- [methods] setup of working directory with datafiles and datalogs: "We need to set up our working directory with two subfolders, `datafiles` and `datalogs`, the former for data input, the later for output."
- [readme] EMPEROR overview from README: "employing Markov chains and parallel tempering methods, convergence tests and Bayesian statistics, along with various noise models"
- [abstract] statistical inference caveat: "Purely statistical inference might be insufficient on its own for robust exoplanet detection."
- [methods] posterior modelling for constraining new runs: "a new prior will be placed upon it. \emp\ includes two approaches for modelling the posterior of each parameter-- Kernel Density Estimation and Bayesian Gaussian Mixtures."
