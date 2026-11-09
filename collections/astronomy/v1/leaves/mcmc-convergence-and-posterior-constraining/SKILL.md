---
name: mcmc-convergence-and-posterior-constraining
description: Use when a radial-velocity time series contains an unknown number of
  planetary signals and you need the sampler itself to decide how many Keplerian components
  to keep, with posteriors from each run seeding the next and a tolerance-based stopping
  rule.
when_to_use_negative:
- The number of planets or the model structure is already known and fixed — a single
  MCMC run without the autorun comparison loop suffices
- The input is not an RV (or RV+astrometry) time series; EMPEROR's Keplerian/Offset/Jitter
  blocks and comparison loop are built for orbital signal retrieval
- The result will be accepted on statistics alone — the paper notes purely statistical
  inference might be insufficient for robust exoplanet detection
tools:
- name: EMPEROR (astroemperor)
  role: Implements the autorun iterative model-comparison loop, posterior-constraining
    priors (KDE / Bayesian Gaussian Mixtures), and BIC/Evidence stopping criteria
  repo: https://github.com/ReddTea/astroemperor
- name: reddemcee
  role: Adaptive parallel-tempering MCMC sampler (built on emcee) with adaptation
    modes and evidence functions (get_evidence_ti/ss/hybrid) used for each run
  repo: https://github.com/ReddTea/reddemcee
- name: emcee
  role: Ensemble sampler underlying reddemcee, used for autocorrelation-time diagnostics
  repo: https://github.com/dfm/kepler.py
- name: kepler.py
  role: Fast stable Kepler-equation solver providing the Keplerian forward model evaluated
    inside the likelihood
  repo: https://github.com/dfm/kepler.py
- name: celerite2
  role: Gaussian-process noise model available alongside EMPEROR's noise blocks
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:9e23291b8520f5b8bac1da16bc6a838d4d7989b010f298c7c313bf41feba4d23
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
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# mcmc-convergence-and-posterior-constraining

## Summary

An automated iterative MCMC model-selection loop (EMPEROR) that samples each candidate Keplerian model with adaptive parallel tempering, applies convergence tests and Bayesian statistics, and constrains each new run's priors from the previous run's posteriors until the comparison statistic (BIC or Evidence) stops improving.

## When to use

Use when a radial-velocity time series contains an unknown number of planetary signals and you need the sampler itself to decide how many Keplerian components to keep, with posteriors from each run seeding the next and a tolerance-based stopping rule.

## When NOT to use

- The number of planets or the model structure is already known and fixed — a single MCMC run without the autorun comparison loop suffices
- The input is not an RV (or RV+astrometry) time series; EMPEROR's Keplerian/Offset/Jitter blocks and comparison loop are built for orbital signal retrieval
- The result will be accepted on statistics alone — the paper notes purely statistical inference might be insufficient for robust exoplanet detection

## Inputs

- Radial velocity time series files placed in datafiles/ (e.g. 51peg.vels from LICK; multi-instrument AAT/HARPS1-3 data for HIP 21850)
- Parameter conditions per block (limits, priors, init_pos) supplied via add_condition
- Engine setup [ntemps, nwalkers, nsweeps, nsteps] and burnin fraction

## Outputs

- Optimum model (number of Keplerians) selected by the stopping rule
- model_comparison.csv listing each run's statistic with the optimum model identified
- Per-run tables stats.tex and values.tex plus best_fit.dat written to datalogs/
- Evidence estimates via thermodynamic integration, stepping stones, or hybrid, and beta/swap adaptation history from reddemcee

## How to apply

Set up a working directory with datafiles/ and datalogs/, load the RV time series via Simulation.load_data (e.g. 51Peg), and configure the reddemcee adaptive parallel-tempering engine with set_engine('reddemcee') and setup [ntemps, nwalkers, nsweeps, nsteps] such as [10, 256, 2048, 1], plus a burnin fraction. Define limits, priors and init_pos with add_condition for the Keplerian, Offset and Jitter blocks, then launch the autorun control loop (e.g. sim.autorun(0, 5)), which adds one Keplerian Block per run and samples it with parallel-tempering MCMC. After each run, compute BIC (default tolerance 5) or switch to Evidence via set_comparison_criteria('Evidence') with set_tolerance(10) — estimated by thermodynamic integration, stepping stones or the hybrid method — and stop when BIC_old - BIC_new fails to beat the tolerance. Between runs, constrain new priors using the previous run's posterior modelled by Kernel Density Estimation or Bayesian Gaussian Mixtures, optionally restricted to HDI ranges.

## Related tools

- **EMPEROR (astroemperor)** (Implements the autorun iterative model-comparison loop, posterior-constraining priors (KDE / Bayesian Gaussian Mixtures), and BIC/Evidence stopping criteria) — https://github.com/ReddTea/astroemperor
- **reddemcee** (Adaptive parallel-tempering MCMC sampler (built on emcee) with adaptation modes and evidence functions (get_evidence_ti/ss/hybrid) used for each run) — https://github.com/ReddTea/reddemcee
- **emcee** (Ensemble sampler underlying reddemcee, used for autocorrelation-time diagnostics) — https://github.com/dfm/kepler.py
- **kepler.py** (Fast stable Kepler-equation solver providing the Keplerian forward model evaluated inside the likelihood) — https://github.com/dfm/kepler.py
- **celerite2** (Gaussian-process noise model available alongside EMPEROR's noise blocks)

## Examples

```
sim.set_engine('reddemcee'); sim.engine_config['setup'] = [10, 256, 2048, 1]; sim.set_comparison_criteria('Evidence'); sim.set_tolerance(10); sim.autorun(0, 5)
```

## Evaluation signals

- The autorun loop terminates exactly when the newest model fails to improve the chosen statistic by more than the tolerance (BIC tol 5, Evidence tol 10), and no extra Keplerian is added
- model_comparison.csv lists every run's statistic value and flags the optimum model, consistent with stats.tex, values.tex and best_fit.dat in datalogs/
- Evidence estimates from thermodynamic integration and stepping-stones (get_evidence_ti / get_evidence_ss) agree within their reported uncertainties
- Posterior-constrained runs sample without divergent walkers: chains pass convergence/autocorrelation checks (get_autocorr_time) and the adaptive temperature ladder equalises swap statistics (e.g. SAR mode)
- Recovered parameters for benchmark datasets match published values (e.g. 51 Peg APT run with tighter estimates than DNS; Barnard's star P = 3.1536±0.0003 d)

## Limitations

- Evidence (ln Z) is sensitive to prior selection, as demonstrated on HD 55693 — comparison outcomes can flip with prior choices
- Purely statistical inference might be insufficient on its own for robust exoplanet detection
- kepler.solve returns garbage for eccentricities outside the range 0 to 1
- BIC is the default criterion with tolerance 5; using a too-tight or too-loose tolerance changes when the loop stops adding components

## Evidence

- [intro] Convergence tests and Bayesian statistics with model comparison: "convergence tests and Bayesian statistics, along with various noise models"
- [intro] Automatic model comparison returning the optimum model: "performs model comparisons automatically to return the optimum model that best describes the data"
- [methods] Posterior-constraining of new runs via KDE or Bayesian Gaussian Mixtures: "a new prior will be placed upon it. \emp\ includes two approaches for modelling the posterior of each parameter-- Kernel Density Estimation and Bayesian Gaussian Mixtures."
- [methods] Configurable comparison statistic and tolerance: "The comparison statistic can be set with the `set_comparison_criteria` method, and the tolerance changed with the `set_tolerance` method."
- [readme] reddemcee is an adaptive parallel-tempering MCMC sampler based on emcee: [evidence span withheld — anchor status: external_doc]
- [readme] Adaptation modes target equalising temperature-swap statistics: [evidence span withheld — anchor status: external_doc]
