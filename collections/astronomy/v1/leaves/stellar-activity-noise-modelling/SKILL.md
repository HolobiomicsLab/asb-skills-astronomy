---
name: stellar-activity-noise-modelling
description: Use when an RV time series (e.g. HD 55693 multi-instrument data) shows
  activity-correlated periodicities that could masquerade as planets, and you must
  co-fit Keplerians with magnetic-cycle/sinusoid, SAI, or MOAV noise terms before
  comparing models by ln Z.
when_to_use_negative:
- The goal is a purely Keplerian fit to a dataset with no activity-index or rotation
  diagnostics available — the magnetic-cycle/sinusoid and SAI noise terms are unwarranted
  there
- You need an explicit physical GP stellar-activity kernel (e.g. celerite2 quasi-periodic)
  rather than EMPEROR's parametric sinusoid/magnetic-cycle/SAI/MOAV noise components
- The RV data contain a clear single Keplerian with well-separated period from any
  known rotation period, and model comparison by evidence is unnecessary
tools:
- name: EMPEROR (astroemperor)
  role: Simulation framework providing load_data, add_magnetic_cycle/add_sinusoid/add_sai/add_moav/add_jitter,
    add_condition priors, and evidence-based autorun model selection
  repo: https://github.com/ReddTea/astroemperor
- name: reddemcee
  role: Adaptive parallel-tempering MCMC sampler used as EMPEROR's engine; supplies
    stepping-stones/therodynamic-integration evidence estimation
  repo: https://github.com/ReddTea/reddemcee
- name: emcee
  role: Base ensemble sampler on which reddemcee is built; used for autocorrelation
    times
  repo: https://github.com/dfm/emcee
- name: celerite2
  role: Optional Gaussian-process library listed among EMPEROR's method dependencies
  repo: https://celerite2.readthedocs.io/en/latest/
- name: kepler.py
  role: Fast Kepler's-equation solver used for the Keplerian signal components
  repo: https://github.com/dfm/kepler.py
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:d44dfb9ece59b82d662320038ae793796fa548740b51ee74e16caaf69e6ce0c6
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# stellar-activity-noise-modelling

## Summary

Model stellar activity (rotation, magnetic cycle, activity-index correlations) as noise terms alongside Keplerians in EMPEROR's Bayesian RV framework, so that P_rot ~29.7 d and P_mag ~2557 d signals are separated from planetary orbits and evidence is computed for each configuration.

## When to use

Use when an RV time series (e.g. HD 55693 multi-instrument data) shows activity-correlated periodicities that could masquerade as planets, and you must co-fit Keplerians with magnetic-cycle/sinusoid, SAI, or MOAV noise terms before comparing models by ln Z.

## When NOT to use

- The goal is a purely Keplerian fit to a dataset with no activity-index or rotation diagnostics available — the magnetic-cycle/sinusoid and SAI noise terms are unwarranted there
- You need an explicit physical GP stellar-activity kernel (e.g. celerite2 quasi-periodic) rather than EMPEROR's parametric sinusoid/magnetic-cycle/SAI/MOAV noise components
- The RV data contain a clear single Keplerian with well-separated period from any known rotation period, and model comparison by evidence is unnecessary

## Inputs

- Radial-velocity catalogue (e.g. HD 55693 RV files) placed in datafiles/<star>/RV, multi-instrument
- Stellar activity index time series (for add_sai terms)
- Baseline EMPEROR fit chains/datalogs for reuse
- engine_config['setup'] values [ntemps, nwalkers, nsweeps, nsteps] for reddemcee

## Outputs

- Posterior estimates of P_rot and P_mag with credible intervals (e.g. 29.72 (+0.01/−0.02) d and 2557.0 (+70.1/−36.7) d)
- datalogs output tables (best_fit.dat, values.tex, stats.tex) including '$\log{Z}$' entries with uncertainties
- Prior-evidence comparison table (e.g. prior_evidence_comparison.csv with ln Z and Δln Z per prior configuration)

## How to apply

Set up a working directory with 'datafiles' and 'datalogs' subfolders, place the RV catalogue under datafiles/HD55693/RV, and instantiate astroemperor.Simulation with load_data. Configure the reddemcee parallel-tempering engine via set_engine('reddemcee') and engine_config['setup'] = [ntemps, nwalkers, nsweeps, nsteps] (the 51 Peg demo uses [10, 256, 2048, 1]). Build the model with EMPEROR's noise-model API — add_sinusoid, add_magnetic_cycle, add_sai (stellar activity indices), add_moav, add_offset, add_jitter, add_acceleration — plus Keplerian blocks, constraining priors/limits/initial positions through add_condition (e.g. priors centred near P_rot = 29.72 d and P_mag = 2557.0 d). Enable evidence-based model comparison with set_comparison_criteria('Evidence'), set_tolerance(10), and evidence_method = 'ss' (stepping stones), then run sim.autorun so successive models are accepted only when log-evidence improves beyond tolerance. Extract posteriors and ln Z (with uncertainties) from the datalogs tables (best_fit.dat, stats.tex) and reuse the fitted chains to test alternative priors — the article reports ln Z is sensitive to prior selection, so tabulate Δln Z per configuration against the tolerance scale.

## Related tools

- **EMPEROR (astroemperor)** (Simulation framework providing load_data, add_magnetic_cycle/add_sinusoid/add_sai/add_moav/add_jitter, add_condition priors, and evidence-based autorun model selection) — https://github.com/ReddTea/astroemperor
- **reddemcee** (Adaptive parallel-tempering MCMC sampler used as EMPEROR's engine; supplies stepping-stones/therodynamic-integration evidence estimation) — https://github.com/ReddTea/reddemcee
- **emcee** (Base ensemble sampler on which reddemcee is built; used for autocorrelation times) — https://github.com/dfm/emcee
- **celerite2** (Optional Gaussian-process library listed among EMPEROR's method dependencies) — https://celerite2.readthedocs.io/en/latest/
- **kepler.py** (Fast Kepler's-equation solver used for the Keplerian signal components) — https://github.com/dfm/kepler.py

## Examples

```
sim.set_engine('reddemcee'); sim.engine_config['setup'] = [10, 256, 2048, 1]; sim.load_data('HD55693'); sim.set_comparison_criteria('Evidence'); sim.set_tolerance(10)
```

## Evaluation signals

- Recovered P_rot falls within 29.72 (+0.01/−0.02) d and P_mag within 2557.0 (+70.1/−36.7) d credible intervals for HD 55693
- Δln Z between prior configurations exceeds the set_tolerance(10) model-comparison scale, confirming prior sensitivity of the evidence
- Evidence values are computed with evidence_method = 'ss' (stepping stones) under set_comparison_criteria('Evidence'), not BIC
- Autocorrelation times from the PT chains are acceptable (parallel tempering showed ~3.76x effective sampling gain over DNS on 51 Peg)
- datalogs contains stats.tex/best_fit.dat entries for every noise term added (sinusoid, magnetic cycle, SAI, jitter, offsets)

## Limitations

- Purely statistical inference (evidence alone) might be insufficient on its own for robust exoplanet detection — activity signals can mimic planets, so ln Z results need physical interpretation
- Evidence ln Z is sensitive to prior selection (e.g. Uniform vs Normal on Period/Amplitude), so conclusions depend on the prior configuration chosen via add_condition
- kepler.solve returns garbage for eccentricities outside [0, 1], corrupting eccentric Keplerian components if limits are not enforced

## Evidence

- [abstract] HD 55693 recovered periods: "the stellar rotation, P rot = 29.72 −0.02 + 0.01 , and magnetic cycle, P mag = 2557.0 −36.7 + 70.1 , were recovered, while demonstrating the sensitivity of ln 𝒵 to prior selection."
- [methods] working directory setup: "We need to set up our working directory with two subfolders, `datafiles` and `datalogs`, the former for data input, the later for output."
- [methods] model comparison configuration: "The comparison statistic can be set with the `set_comparison_criteria` method, and the tolerance changed with the `set_tolerance` method."
- [methods] engine configuration: "sim.set_engine('reddemcee')
sim.engine_config['setup'] = [10, 256, 2048, 1]"
- [intro] sampling rationale: "employing Markov chains and parallel tempering methods, convergence tests and Bayesian statistics, along with various noise models"
- [readme] noise-model API: "It has a series of configuration commands that will amaze you
  - Advanced Noise Model"
- [readme] emcee basis: [evidence span withheld — anchor status: external_doc]
