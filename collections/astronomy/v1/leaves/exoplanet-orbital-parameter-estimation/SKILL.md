---
name: exoplanet-orbital-parameter-estimation
description: Use when you have multi-instrument RV timeseries for a star (e.g. GJ
  699, 51 Peg) and need to detect and characterise planetary signals with posteriors,
  evidence-based model selection, and stellar-mass-derived physical parameters.
when_to_use_negative:
- The signal is not an orbital/periodic signal in RV timeseries — EMPEROR is specific
  to RV (and joint astrometry) exoplanet retrieval, not other data types
- A single fixed model is already agreed upon and no automatic model comparison over
  noise treatments (jitter, MA, GP) or Keplerian counts is needed
- You need transit/photometry-only fitting — this workflow is designed for radial-velocity
  (plus optional Hipparcos/Gaia astrometry) data
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: EMPEROR (astroemperor)
  role: 'Main framework: load RV data, configure engines, run autorun model selection,
    output posteriors to datalogs'
  repo: https://github.com/ReddTea/astroemperor
- name: reddemcee
  role: Adaptive parallel-tempering MCMC sampler (PTSampler) with adaptation modes
    and evidence functions (TI, SS, hybrid)
  repo: https://github.com/ReddTea/reddemcee
- name: celerite2
  role: Gaussian Process noise modelling with a RotationTerm kernel via sim.switch_celerite
  repo: https://celerite2.readthedocs.io/en/latest/
- name: kepler.py
  role: Fast and stable Kepler's equation solver used for the orbital solution
  repo: https://github.com/dfm/kepler.py
- name: reddcolors
  role: Colour management for plotting the datalogs results
  repo: https://github.com/ReddTea/reddcolors/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:4631e2c7b254369a9f1a1f02bb9ec009bf78730931fc157fc852bddbdfdd2368
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ReddTea/astroemperor
  tools:
  - EMPEROR (astroemperor)
  - Python
  - celerite2
  - kepler.py
  - reddcolors
  - reddemcee
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# exoplanet-orbital-parameter-estimation

## Summary

Automated retrieval of Keplerian orbital parameters (period, minimum mass, semi-major axis, eccentricity) from radial-velocity timeseries using EMPEROR's adaptive parallel-tempering MCMC with automatic model comparison across noise treatments.

## When to use

Use when you have multi-instrument RV timeseries for a star (e.g. GJ 699, 51 Peg) and need to detect and characterise planetary signals with posteriors, evidence-based model selection, and stellar-mass-derived physical parameters.

## When NOT to use

- The signal is not an orbital/periodic signal in RV timeseries — EMPEROR is specific to RV (and joint astrometry) exoplanet retrieval, not other data types
- A single fixed model is already agreed upon and no automatic model comparison over noise treatments (jitter, MA, GP) or Keplerian counts is needed
- You need transit/photometry-only fitting — this workflow is designed for radial-velocity (plus optional Hipparcos/Gaia astrometry) data

## Inputs

- RV timeseries catalogue files (e.g. 51peg.vels from LICK; GJ 699 RV data under datafiles/GJ699/RV)
- Stellar mass (sim.starmass) for deriving M_sin(i) and semi-major axis
- Optional Hipparcos/Gaia astrometry for joint modelling

## Outputs

- Posterior distributions for Keplerian parameters (period, amplitude, eccentricity, longitude of periastron, offsets)
- Derived physical parameters: minimum mass in M_earth and semi-major axis in AU
- Bayesian evidence (ln Z via thermodynamic integration, stepping stones, or hybrid) per model
- Datalogs tables and plots in the datalogs output folder

## How to apply

Set up a working directory with `datafiles` and `datalogs` subfolders, place the RV catalogue (e.g. under datafiles/GJ699/RV), and load it with astroemperor's `Simulation.load_data`, setting `sim.starmass` so minimum mass and semi-major axis are derived. Configure the reddemcee adaptive parallel-tempering engine via `sim.set_engine('reddemcee')` and `sim.engine_config['setup'] = [ntemps, nwalkers, nsweeps, nsteps]`, tuning the adaptive ladder (adapt_tau, adapt_nu, adapt_mode) which equalises quantities like swap acceptance rate or thermodynamic length. Build candidate models with different noise treatments — jitter, moving-average blocks (sim.moav order), or a GP via `sim.switch_celerite = True` with a RotationTerm kernel through celerite2 — and add parameter conditions (limits, priors) with `add_condition`. Execute `sim.autorun` over increasing Keplerian counts with `sim.set_comparison_criteria('Evidence')` and `set_tolerance`, so successive runs are accepted only when ln Z improves beyond tolerance; constrain subsequent runs using previous posteriors via KDE or Bayesian Gaussian Mixtures. Extract the best model's posteriors (period, amplitude, eccentricity, longitude, offsets) from the datalogs tables and plots.

## Related tools

- **EMPEROR (astroemperor)** (Main framework: load RV data, configure engines, run autorun model selection, output posteriors to datalogs) — https://github.com/ReddTea/astroemperor
- **reddemcee** (Adaptive parallel-tempering MCMC sampler (PTSampler) with adaptation modes and evidence functions (TI, SS, hybrid)) — https://github.com/ReddTea/reddemcee
- **celerite2** (Gaussian Process noise modelling with a RotationTerm kernel via sim.switch_celerite) — https://celerite2.readthedocs.io/en/latest/
- **kepler.py** (Fast and stable Kepler's equation solver used for the orbital solution) — https://github.com/dfm/kepler.py
- **reddcolors** (Colour management for plotting the datalogs results) — https://github.com/ReddTea/reddcolors/

## Examples

```
sim = astroemperor.Simulation(); sim.load_data('GJ699'); sim.set_engine('reddemcee'); sim.set_comparison_criteria('Evidence'); sim.autorun(1, 5)
```

## Evaluation signals

- For Barnard's star (GJ 699), the retrieved Keplerian must reproduce period 3.1536±0.0003 d, minimum mass 0.38±0.03 M_earth, and semi-major axis 0.02315±0.00039 AU within uncertainties
- Each successive model in the autorun is accepted only when ln Z (Evidence) improves by more than the value set with set_tolerance
- Running the shipped test (`test_basic.py` with the 51 Peg RV data) completes and returns tight posteriors for the known 51 Pegasi b signal
- Convergence is confirmed via convergence tests and MCMC autocorrelation times; chains show no stuck multi-modal behaviour thanks to the adaptive tempering ladder

## Limitations

- Evidence estimates (ln Z) are sensitive to prior selection, as demonstrated for HD 55693 — prior choice must be justified
- Purely statistical inference might be insufficient on its own for robust exoplanet detection
- kepler.py's lower-level `solve` returns garbage for eccentricities outside the range zero to one

## Evidence

- [abstract] Barnard's star best model: "The best model shows a period of 3.1536±0.0003 d, minimum mass of 0.38±0.03 M ⊕ , and semi-major axis of 0.02315±0.00039 AU."
- [methods] engine setup: "sim.set_engine('reddemcee')
sim.engine_config['setup'] = [10, 256, 2048, 1]"
- [methods] model comparison: "The comparison statistic can be set with the `set_comparison_criteria` method, and the tolerance changed with the `set_tolerance` method."
- [methods] posterior reuse: "a new prior will be placed upon it. \emp\ includes two approaches for modelling the posterior of each parameter-- Kernel Density Estimation and Bayesian Gaussian Mixtures."
- [methods] working directory: "We need to set up our working directory with two subfolders, `datafiles` and `datalogs`, the former for data input, the later for output."
- [readme] reddemcee overview: [evidence span withheld — anchor status: external_doc]
- [readme] kepler.py caveat: [evidence span withheld — anchor status: external_doc]
- [abstract] prior sensitivity: "the stellar rotation, P rot = 29.72 −0.02 + 0.01 , and magnetic cycle, P mag = 2557.0 −36.7 + 70.1 , were recovered, while demonstrating the sensitivity of ln 𝒵 to prior selection."
