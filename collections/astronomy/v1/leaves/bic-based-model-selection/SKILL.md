---
name: bic-based-model-selection
description: Use when fitting RV time series (or joint RV+astrometry) and the number
  of planetary signals is unknown — decide when to stop adding Keplerians using BIC_old
  - BIC_new > tolerance or Evidence ratios, without manual model comparison.
when_to_use_negative:
- The number of planets is already fixed and externally justified — just run a single
  fixed-model sampling (e.g. sim.autorun(1, 1)) instead of the comparison loop
- The goal is to test a specific non-Keplerian signal against a baseline rather than
  determine model order automatically — configure Blocks manually instead
- The RV data are not compatible with Keplerian plus noise modelling (e.g. pure activity-dominated
  data) — pure statistical inference might be insufficient for robust exoplanet detection,
  so external activity diagnostics should guide the model first
edam_operation: http://edamontology.org/operation_3662
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: EMPEROR (astroemperor)
  role: Runs the fixed autorun control loop that adds Keplerian Blocks and applies
    the BIC/Evidence stopping criterion to return the optimum model
  repo: github.com/ReddTea/astroemperor
- name: reddemcee
  role: Adaptive parallel-tempering MCMC engine used to sample each candidate model
    in the comparison sequence; also provides evidence estimators (TI, stepping stones,
    hybrid)
  repo: github.com/ReddTea/reddemcee
- name: emcee
  role: Base ensemble sampler underlying reddemcee, used for autocorrelation time
    calculations
- name: kepler.py
  role: Fast and stable Kepler's equation solver used in the Keplerian Block forward
    model
  repo: github.com/dfm/kepler.py
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:83ebdc1642bb76eee4550766a3a73c9640228b43198f1ac6a7706c60eadb2f2b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPEROR (astroemperor)
  - Python
  - emcee
  - kepler.py
  - reddemcee
  edam_operation: http://edamontology.org/operation_3662
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# BIC-based model selection

## Summary

Automatically determine the optimum number of Keplerian components describing a radial velocity time series by iteratively adding one Keplerian Block per EMPEROR run and comparing successive models via BIC (or Bayesian Evidence) against a configurable tolerance.

## When to use

Use when fitting RV time series (or joint RV+astrometry) and the number of planetary signals is unknown — decide when to stop adding Keplerians using BIC_old - BIC_new > tolerance or Evidence ratios, without manual model comparison.

## When NOT to use

- The number of planets is already fixed and externally justified — just run a single fixed-model sampling (e.g. sim.autorun(1, 1)) instead of the comparison loop
- The goal is to test a specific non-Keplerian signal against a baseline rather than determine model order automatically — configure Blocks manually instead
- The RV data are not compatible with Keplerian plus noise modelling (e.g. pure activity-dominated data) — pure statistical inference might be insufficient for robust exoplanet detection, so external activity diagnostics should guide the model first

## Inputs

- Radial velocity time series files (e.g. 51peg.vels, LICK; multi-instrument RV data) placed in datafiles/
- Parameter conditions per Block (limits, priors, init_pos) defined via add_condition for Keplerian, Offset and Jitter blocks
- Engine configuration for reddemcee: [ntemps, nwalkers, nsweeps, nsteps] and burnin fraction
- Comparison criteria choice (BIC default, or Evidence) and tolerance value (e.g. 5 for BIC, 10 for Evidence)

## Outputs

- model_comparison.csv listing each run's statistic with the optimum model identified
- Per-run tables stats.tex and values.tex in datalogs/
- best_fit.dat with best-fit parameters of the optimum model
- Terminating autorun: number of Keplerians in the optimum model

## How to apply

Set up the working directory with datafiles/ and datalogs/ subfolders, load the RV time series with Simulation.load_data, and configure the reddemcee adaptive parallel-tempering engine via set_engine (e.g. setup [10, 256, 2048, 1]) with a burnin fraction. Launch the fixed autorun control loop (e.g. sim.autorun(0, 5)), which adds one Keplerian Block per run and samples each model with parallel-tempering MCMC. After each run, compute the comparison statistic — BIC by default with tolerance 5, or Evidence via set_comparison_criteria('Evidence') with set_tolerance(10) (optionally using stepping-stones estimation) — and apply the stopping rule: stop adding components when BIC_old - BIC_new fails to exceed the tolerance (min rule), recording the last accepted model as the optimum. Rationale: successive Keplerian Blocks are only justified if they improve the statistic beyond the tolerance, which guards against overfitting noise; per-run tables (stats.tex, values.tex), best_fit.dat, and model_comparison.csv provide an auditable trail of the selection.

## Related tools

- **EMPEROR (astroemperor)** (Runs the fixed autorun control loop that adds Keplerian Blocks and applies the BIC/Evidence stopping criterion to return the optimum model) — github.com/ReddTea/astroemperor
- **reddemcee** (Adaptive parallel-tempering MCMC engine used to sample each candidate model in the comparison sequence; also provides evidence estimators (TI, stepping stones, hybrid)) — github.com/ReddTea/reddemcee
- **emcee** (Base ensemble sampler underlying reddemcee, used for autocorrelation time calculations)
- **kepler.py** (Fast and stable Kepler's equation solver used in the Keplerian Block forward model) — github.com/dfm/kepler.py

## Examples

```
sim.set_comparison_criteria('Evidence'); sim.set_tolerance(10); sim.autorun(0, 5)
```

## Evaluation signals

- The autorun terminates exactly when the newest model fails to improve the chosen statistic by more than the tolerance (BIC_old - BIC_new <= tol for the min rule)
- model_comparison.csv contains one entry per run with its statistic, and the optimum model is identified as the run before the first failing comparison
- datalogs/ contains stats.tex, values.tex per run and best_fit.dat for the optimum model
- Switching criteria via set_comparison_criteria('Evidence') and set_tolerance(10) changes the stopping run consistently with the recorded statistic values
- Best-fit parameters of the optimum model fall within the priors/limits set via add_condition

## Limitations

- The BIC tolerance default is 5 and Evidence tolerance 10 — models improving by less than the tolerance are rejected, so marginal signals near the threshold may be missed
- Evidence (ln Z) estimates are sensitive to prior selection, as demonstrated for HD 55693
- Purely statistical inference might be insufficient on its own for robust exoplanet detection
- kepler.py's lower-level solve returns garbage for eccentricities outside the range zero to one

## Evidence

- [intro] performs model comparisons automatically to return the optimum model: "performs model comparisons automatically to return the optimum model"
- [methods] comparison statistic and tolerance configuration: "The comparison statistic can be set with the `set_comparison_criteria` method, and the tolerance changed with the `set_tolerance` method."
- [methods] autorun adds one Keplerian per run: "sim.autorun(1, 1)  # (from=1, to=1): just 1 keplerian"
- [methods] evidence via stepping stones: "Curvature-aware Thermodynamic Integration (TI+), Geometric-Bridge Stepping Stones (SS+), and a Hybrid algorithm"
- [abstract] sensitivity of ln Z to priors: "while demonstrating the sensitivity of ln 𝒵 to prior selection"
- [abstract] statistical inference insufficient alone: "Purely statistical inference might be insufficient on its own for robust exoplanet detection."
- [readme] kepler.py eccentricity caveat: [evidence span withheld — anchor status: external_doc]
