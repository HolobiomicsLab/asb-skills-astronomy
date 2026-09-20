---
name: mcmc-posterior-parameter-estimation
description: Use when you have multi-instrument radial-velocity data (e.g. 51peg.vels
  from LICK) and need posterior parameter estimates plus Bayesian model selection
  (ln Z, BIC) over increasing Keplerian counts.
when_to_use_negative:
- The input is not a radial-velocity (or RV+astrometry) time series — EMPEROR is purpose-built
  for RV orbit retrieval, not generic MCMC curve fitting.
- You need a single fixed model fitted without automatic model comparison — the autorun
  loop targets selection of the optimum number of Keplerians via log Z, not one-off
  fits.
- Eccentricities outside the range zero to one are needed from kepler.solve — the
  README notes it 'will return garbage' out of that range.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0622
tools:
- name: EMPEROR (astroemperor)
  role: Framework that loads RV data, configures the sampler, runs the autorun model-comparison
    loop, and writes stats.tex/values.tex tables
  repo: github.com/ReddTea/astroemperor
- name: reddemcee
  role: Adaptive Parallel Tempering MCMC sampler (APT) set via set_engine('reddemcee');
    provides evidence via TI, SS, and hybrid estimators
  repo: github.com/ReddTea/reddemcee
- name: emcee
  role: Ensemble sampler basis on which reddemcee is built; used for calculating autocorrelation
    times
  repo: https://github.com/dfm/emcee
- name: dynesty
  role: DNS nested-sampling engine used as the comparison baseline for effective sampling
    efficiency
- name: kepler.py
  role: Fast and stable solver for Kepler's equation used in the Keplerian parameterisation
  repo: https://github.com/dfm/kepler.py
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:35347169d92fc2c4dc722879107b28892273771cfcc688552600347c3658971e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPEROR (astroemperor)
  - Python
  - dynesty
  - emcee
  - kepler.py
  - reddemcee
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mcmc-posterior-parameter-estimation

## Summary

Estimate exoplanet orbital parameters and model evidence from radial-velocity time series by running EMPEROR's adaptive parallel-tempering MCMC (reddemcee APT) with automatic Keplerian model comparison via log Z (evidence).

## When to use

Use when you have multi-instrument radial-velocity data (e.g. 51peg.vels from LICK) and need posterior parameter estimates plus Bayesian model selection (ln Z, BIC) over increasing Keplerian counts.

## When NOT to use

- The input is not a radial-velocity (or RV+astrometry) time series — EMPEROR is purpose-built for RV orbit retrieval, not generic MCMC curve fitting.
- You need a single fixed model fitted without automatic model comparison — the autorun loop targets selection of the optimum number of Keplerians via log Z, not one-off fits.
- Eccentricities outside the range zero to one are needed from kepler.solve — the README notes it 'will return garbage' out of that range.

## Inputs

- Radial-velocity catalogue file (e.g. 51peg.vels from LICK) placed under datafiles/51Peg/RV/
- Simulation configuration: engine setup [ntemps, nwalkers, nsweeps, nsteps] = [10, 256, 2048, 1], starmass, keplerian_parameterisation
- Parameter boundary conditions and init_pos (e.g. Period 1 limits [3, 5] d, Amplitude 1 limits [45, 60] m/s)

## Outputs

- Posterior parameter estimates in values.tex under datalogs/51Peg/run_1/k_1/tables
- Model-comparison statistics (ln Z, BIC) in stats.tex under datalogs/51Peg/run_1/k_1/tables
- Automatic selection of the optimum Keplerian model over the autorun model-comparison loop

## How to apply

Set up a working directory with datafiles/ and datalogs/ subfolders, place the RV catalogue under datafiles/51Peg/RV/, and load it with sim.load_data('51Peg'). Configure the sampling engine with set_engine('reddemcee') and engine_config['setup'] = [10, 256, 2048, 1], set starmass = 1.12 and keplerian_parameterisation = 1, then add boundary conditions via add_condition (e.g. 'Period 1' limits [3, 5] d with init_pos [4.1, 4.3]; 'Amplitude 1' limits [45, 60] m/s). Run sim.autorun(1, 1) with set_comparison_criteria('Evidence') and set_tolerance(10) so successive Keplerian models are compared by log Z, computed via Thermodynamic Integration, Stepping Stones, or the Hybrid algorithm. Extract ln Z, BIC, and posterior parameter estimates from stats.tex and values.tex in datalogs/51Peg/run_1/k_1/tables. Convergence relies on parallel tempering with an adaptive temperature ladder, which efficiently searches highly multi-modal posteriors.

## Related tools

- **EMPEROR (astroemperor)** (Framework that loads RV data, configures the sampler, runs the autorun model-comparison loop, and writes stats.tex/values.tex tables) — github.com/ReddTea/astroemperor
- **reddemcee** (Adaptive Parallel Tempering MCMC sampler (APT) set via set_engine('reddemcee'); provides evidence via TI, SS, and hybrid estimators) — github.com/ReddTea/reddemcee
- **emcee** (Ensemble sampler basis on which reddemcee is built; used for calculating autocorrelation times) — https://github.com/dfm/emcee
- **dynesty** (DNS nested-sampling engine used as the comparison baseline for effective sampling efficiency)
- **kepler.py** (Fast and stable solver for Kepler's equation used in the Keplerian parameterisation) — https://github.com/dfm/kepler.py

## Examples

```
sim.load_data('51Peg'); sim.set_engine('reddemcee'); sim.engine_config['setup'] = [10, 256, 2048, 1]; sim.add_condition(['Period 1', 'limits', [3, 5]]); sim.autorun(1, 1)
```

## Evaluation signals

- For 51 Pegasi, the APT run achieves an effective sampling increase by a factor of ~3.76 over DNS, with tighter parameter estimates — reproduce this ratio as the validation target.
- stats.tex and values.tex files exist under datalogs/51Peg/run_1/k_1/tables and contain ln Z, BIC, and posterior parameter estimates.
- The autorun model-comparison loop stops per set_tolerance(10) once successive Keplerian models are not distinguished by log Z.
- Posterior period for 51 Pegasi falls within the imposed boundary [3, 5] days and amplitude within [45, 60] m/s.
- reddemcee chain shape matches (ntemps, nsweeps*nsteps, nwalkers, ndim) = (10, 2048, 256, ndim) with discard/thin options honoured.

## Limitations

- Purely statistical inference might be insufficient on its own for robust exoplanet detection.
- ln Z (evidence) estimates are sensitive to prior selection, as demonstrated for HD 55693.
- kepler.solve returns garbage for eccentricities outside the range zero to one.
- Constraining subsequent runs via posterior-based priors (KDE / Bayesian Gaussian Mixtures, HDI ranges) can influence later model comparisons.

## Evidence

- [abstract] APT achieves an effective sampling increase by a factor of 3.76 over DNS for the 51 Pegasi system, and the APT run retrieves tighter parameter estimates than DNS.: "For 51 Pegasi, we find APT achieves an effective sampling increase by a factor of 3.76 over DNS, while retrieving tighter parameter estimates."
- [intro] EMPEROR searches RV time series with MCMC and parallel tempering, performing automatic model comparisons.: "is a Python-based algorithm that automatically searches for signals in Radial Velocity timeseries"
- [methods] Engine setup and comparison criteria configuration via set_engine, set_comparison_criteria, and set_tolerance.: "sim.set_engine('reddemcee')
sim.engine_config['setup'] = [10, 256, 2048, 1]"
- [methods] The comparison statistic can be set with set_comparison_criteria and the tolerance with set_tolerance.: "The comparison statistic can be set with the `set_comparison_criteria` method, and the tolerance changed with the `set_tolerance` method."
- [readme] reddemcee provides evidence via Thermodynamic Integration and Stepping Stones, is based on emcee, and stores chains queryable with discard/thin options.: [evidence span withheld — anchor status: not_found]
- [readme] kepler.solve returns garbage for eccentricities outside [0, 1].: [evidence span withheld — anchor status: external_doc]
