---
name: parallel-tempering-mcmc-sampling
description: Use when fitting Keplerian (or RV+astrometry) models to radial-velocity
  timeseries whose posteriors are highly multi-modal, and you need posterior sampling
  plus ln Z/BIC model comparison across increasing planet counts.
when_to_use_negative:
- Data or analysis goal is not a radial-velocity (or RV+astrometry) timeseries fit
  — EMPEROR's Keplerian/offset/jitter blocks and reddemcee wrapper are built for RV
  orbit retrieval
- The posterior is well-behaved and unimodal, where a plain single-temperature emcee
  sampler is sufficient and parallel tempering's overhead is unnecessary
- The task is a single fixed model fit with no need for automated model comparison
  or evidence estimation across Keplerian counts
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3318
tools:
- name: EMPEROR (astroemperor)
  role: 'Framework that orchestrates the parallel-tempering sampling: loads RV data,
    sets engine config, adds conditions, and runs the autorun model-comparison loop'
  repo: https://github.com/ReddTea/astroemperor
- name: reddemcee
  role: Adaptive Parallel Tempering MCMC ensemble sampler (emcee-based, modified Vousden
    et al. implementation) that samples each model; provides get_evidence_ti/ss/hybrid
  repo: https://github.com/ReddTea/reddemcee
- name: emcee
  role: Affine-invariant ensemble sampler underlying reddemcee; used for autocorrelation
    time calculation
  repo: https://github.com/dfm/emcee
- name: dynesty
  role: DNS nested-sampling engine used as the baseline against which APT effective
    sampling efficiency was compared (3.76x for 51 Pegasi)
- name: celerite2
  role: Gaussian-process noise modelling (e.g. RotationTerm kernel) added to the sampled
    model via switch_celerite
  repo: https://celerite2.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:2cef66f585113fa084976048dbb6bb2f5617e88d3b89d5564c698d0d042d2deb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ReddTea/astroemperor
  tools:
  - EMPEROR (astroemperor)
  - Python
  - celerite2
  - dynesty
  - emcee
  - kepler.py
  - reddemcee
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# parallel-tempering-mcmc-sampling

## Summary

Configure and run adaptive parallel-tempering MCMC posterior sampling (reddemcee/EMPEROR) for radial-velocity exoplanet orbit retrieval, including engine setup, boundary conditions, and evidence-based model comparison across successive Keplerian models.

## When to use

Use when fitting Keplerian (or RV+astrometry) models to radial-velocity timeseries whose posteriors are highly multi-modal, and you need posterior sampling plus ln Z/BIC model comparison across increasing planet counts.

## When NOT to use

- Data or analysis goal is not a radial-velocity (or RV+astrometry) timeseries fit — EMPEROR's Keplerian/offset/jitter blocks and reddemcee wrapper are built for RV orbit retrieval
- The posterior is well-behaved and unimodal, where a plain single-temperature emcee sampler is sufficient and parallel tempering's overhead is unnecessary
- The task is a single fixed model fit with no need for automated model comparison or evidence estimation across Keplerian counts

## Inputs

- RV timeseries catalogue (e.g. 51peg.vels from LICK instrument) placed under datafiles/<Star>/RV/
- Stellar mass (sim.starmass, e.g. 1.12 M_sun for 51 Peg) to derive minimum mass and semi-major axis
- Engine configuration [ntemps, nwalkers, nsweeps, nsteps] and burnin fraction in run_config
- Boundary conditions per parameter: limits, priors, and init_pos via add_condition
- Optional noise-model settings: moav order, GP kernel via switch_celerite (celerite2 RotationTerm)

## Outputs

- Per-run tables stats.tex and values.tex under datalogs/<Star>/run_1/k_1/tables containing ln Z, BIC, and posterior parameter estimates
- best_fit.dat with best-fitting parameters per run
- model_comparison.csv listing each run's statistic with the optimum model identified
- Posterior chains and plots for period, amplitude, eccentricity, longitude, and instrument offsets

## How to apply

Set up a working directory with datafiles/ and datalogs/ subfolders and load the RV catalogue (e.g. 51peg.vels LICK) with Simulation.load_data. Configure the sampler via sim.set_engine('reddemcee') and engine_config['setup'] = [ntemps, nwalkers, nsweeps, nsteps] (paper example: [10, 256, 2048, 1]), tuning the adaptive ladder through adapt_tau, adapt_nu, and adapt_mode (modes SAR, SMD, SGG, GAO, ETL equalise different swap statistics). Constrain parameters with add_condition limits/priors/init_pos (e.g. Period 1 limits [3, 5] d with init_pos [4.1, 4.3]; Amplitude 1 limits [45, 60] m/s), then launch sim.autorun which iteratively adds one Keplerian block per run. Compare successive models with set_comparison_criteria('Evidence') (or BIC default) and set_tolerance(10), stopping when the newest model fails to improve the statistic beyond tolerance; extract ln Z, BIC, and posteriors from stats.tex and values.tex in datalogs/<star>/run_1/k_1/tables. The rationale is that parallel tempering with an adaptive temperature ladder efficiently explores multi-modal posteriors where single-temperature emcee chains stall, and evidence-based comparison returns the optimum model without manual intervention.

## Related tools

- **EMPEROR (astroemperor)** (Framework that orchestrates the parallel-tempering sampling: loads RV data, sets engine config, adds conditions, and runs the autorun model-comparison loop) — https://github.com/ReddTea/astroemperor
- **reddemcee** (Adaptive Parallel Tempering MCMC ensemble sampler (emcee-based, modified Vousden et al. implementation) that samples each model; provides get_evidence_ti/ss/hybrid) — https://github.com/ReddTea/reddemcee
- **emcee** (Affine-invariant ensemble sampler underlying reddemcee; used for autocorrelation time calculation) — https://github.com/dfm/emcee
- **dynesty** (DNS nested-sampling engine used as the baseline against which APT effective sampling efficiency was compared (3.76x for 51 Pegasi))
- **celerite2** (Gaussian-process noise modelling (e.g. RotationTerm kernel) added to the sampled model via switch_celerite) — https://celerite2.readthedocs.io/en/latest/

## Examples

```
sim.set_engine('reddemcee'); sim.engine_config['setup'] = [10, 256, 2048, 1]; sim.set_comparison_criteria('Evidence'); sim.set_tolerance(10); sim.autorun(0, 5)
```

## Evaluation signals

- The APT-over-DNS effective sampling ratio for 51 Pegasi reproduces the reported ~3.76 with APT returning tighter parameter posteriors
- The autorun loop terminates exactly when the newest model's ln Z (or BIC) fails to improve by more than set_tolerance (e.g. 10 for Evidence, 5 for BIC default)
- model_comparison.csv lists each run's statistic and correctly flags the optimum model
- Extracted posteriors match independent estimates within uncertainties (e.g. Barnard's star period 3.1536±0.0003 d, mass 0.38±0.03 M_earth, a = 0.02315±0.00039 AU)
- stats.tex reports ln Z with its uncertainty and chain convergence passes autocorrelation/convergence tests from reddemcee

## Limitations

- Purely statistical inference might be insufficient on its own for robust exoplanet detection
- The Bayesian evidence ln Z is sensitive to prior selection (demonstrated for HD 55693 with Uniform vs Normal priors on Period/Amplitude), so comparison results depend on prior choices
- kepler.solve returns garbage for eccentricities outside the range zero to one, so eccentricity priors must keep samples in the valid range

## Evidence

- [abstract] APT achieves an effective sampling increase by a factor of 3.76 over DNS while retrieving tighter parameter estimates: "For 51 Pegasi, we find APT achieves an effective sampling increase by a factor of 3.76 over DNS, while retrieving tighter parameter estimates."
- [intro] EMPEROR performs automated model comparison over RV timeseries with MCMC and parallel tempering: "performs model comparisons automatically to return the optimum model that best describes the data"
- [methods] Engine configuration via set_engine and engine_config: "sim.set_engine('reddemcee')
sim.engine_config['setup'] = [10, 256, 2048, 1]"
- [methods] Model comparison statistic and tolerance are configurable: "The comparison statistic can be set with the `set_comparison_criteria` method, and the tolerance changed with the `set_tolerance` method."
- [readme] reddemcee is an adaptive parallel-tempering sampler built on emcee and Vousden et al., usable standalone: [evidence span withheld — anchor status: external_doc]
