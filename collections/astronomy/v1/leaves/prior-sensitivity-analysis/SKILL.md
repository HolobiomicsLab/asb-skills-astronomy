---
name: prior-sensitivity-analysis
description: Use when a fitted RV model's Bayesian model comparison (Evidence vs BIC)
  must be checked for robustness to prior choice, or when reporting recovered activity
  signals (rotation, magnetic cycle) alongside an ln Z sensitivity assessment.
when_to_use_negative:
- The comparison statistic of interest is BIC rather than Evidence — ln Z prior sensitivity
  requires Evidence-based comparison set with set_comparison_criteria('Evidence'),
  not BIC.
- The input is not an RV (or joint RV+astrometry) timeseries amenable to EMPEROR's
  Keplerian/noise modelling — this skill is specific to radial-velocity orbit retrieval.
- You need a full automatic model search over keplerian counts (autorun) rather than
  a controlled re-run of one fixed model under varied priors.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3418
tools:
- name: EMPEROR (astroemperor)
  role: Framework for loading RV data, rebuilding the HD 55693 model, applying alternative
    priors via add_condition, and extracting log Z from the datalogs stats table
  repo: https://github.com/ReddTea/astroemperor/
- name: reddemcee
  role: Adaptive Parallel Tempering MCMC engine whose PTSampler setup (ntemps, nwalkers,
    nsweeps, nsteps) is configured in engine_config; provides evidence estimation
    via stepping stones
  repo: https://github.com/ReddTea/reddemcee/
- name: Python
  role: Language in which the sensitivity workflow (loading data, conditions, evidence
    tabulation to CSV) is scripted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:f01ad8d4075a31c0a208ee8e7f28642fe94126d2b4009585d87e8ba5b4b0952d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ReddTea/astroemperor
  tools:
  - EMPEROR (astroemperor)
  - Python
  - reddemcee
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3418
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# prior-sensitivity-analysis

## Summary

Quantifies how Bayesian evidence (ln Z) estimates in an EMPEROR radial-velocity fit depend on prior selection by re-running an identical model (Keplerians plus stellar-activity noise terms) under alternative priors and tabulating Δln Z, as demonstrated on HD 55693 where P_rot = 29.72 d and P_mag = 2557.0 d were recovered.

## When to use

Use when a fitted RV model's Bayesian model comparison (Evidence vs BIC) must be checked for robustness to prior choice, or when reporting recovered activity signals (rotation, magnetic cycle) alongside an ln Z sensitivity assessment.

## When NOT to use

- The comparison statistic of interest is BIC rather than Evidence — ln Z prior sensitivity requires Evidence-based comparison set with set_comparison_criteria('Evidence'), not BIC.
- The input is not an RV (or joint RV+astrometry) timeseries amenable to EMPEROR's Keplerian/noise modelling — this skill is specific to radial-velocity orbit retrieval.
- You need a full automatic model search over keplerian counts (autorun) rather than a controlled re-run of one fixed model under varied priors.

## Inputs

- HD 55693 radial-velocity catalogue in EMPEROR's datafiles folder (multi-instrument RV timeseries)
- Baseline EMPEROR fitted model and MCMC chains for HD 55693 (Keplerian blocks + stellar-rotation/magnetic-cycle noise terms)
- Alternative prior specifications per parameter (e.g. Uniform vs Normal on Period/Amplitude) via add_condition entries

## Outputs

- prior_evidence_comparison.csv tabulating ln Z (with uncertainty) and Δln Z relative to baseline for each prior configuration
- Per-run datalogs stats tables containing the '$\log{Z}$' entry
- Recovered stellar activity periods (P_rot = 29.72 d, P_mag = 2557.0 d) from the baseline model

## How to apply

Load the RV catalogue from the datafiles folder with EMPEROR's Simulation.load_data and reuse the baseline HD 55693 model (Keplerian blocks plus stellar-rotation/magnetic-cycle terms). Configure the reddemcee parallel-tempering engine (ntemps, nwalkers, nsweeps, nsteps in engine_config['setup'], e.g. [10, 256, 2048, 1]) with run_config burn-in matched to the baseline run. Reproduce the baseline ln Z using sim.set_comparison_criteria('Evidence') and sim.evidence_method = 'ss' (stepping stones). Then re-run the identical model under alternative priors applied via sim.add_condition([param, 'prior', ...]) and sim.add_condition([param, 'prargs', ...]) (e.g. Uniform vs Normal on Period/Amplitude), extract log Z and its uncertainty from each run's datalogs stats table ('$​\log{Z}$' entry), and tabulate ln Z and Δln Z relative to baseline per prior configuration (e.g. prior_evidence_comparison.csv). The analysis is validated only if Δln Z between prior configurations exceeds the model-comparison tolerance scale set via set_tolerance (e.g. tolerance 10), demonstrating prior sensitivity as reported for HD 55693.

## Related tools

- **EMPEROR (astroemperor)** (Framework for loading RV data, rebuilding the HD 55693 model, applying alternative priors via add_condition, and extracting log Z from the datalogs stats table) — https://github.com/ReddTea/astroemperor/
- **reddemcee** (Adaptive Parallel Tempering MCMC engine whose PTSampler setup (ntemps, nwalkers, nsweeps, nsteps) is configured in engine_config; provides evidence estimation via stepping stones) — https://github.com/ReddTea/reddemcee/
- **Python** (Language in which the sensitivity workflow (loading data, conditions, evidence tabulation to CSV) is scripted)

## Examples

```
sim.set_engine('reddemcee'); sim.set_comparison_criteria('Evidence'); sim.add_condition(['Period 1', 'prior', 'Uniform']); sim.autorun(1, 1)
```

## Evaluation signals

- The baseline run reproduces the published ln Z for HD 55693 within its stated uncertainty when using sim.evidence_method = 'ss' (stepping stones).
- The comparison table shows non-negligible Δln Z between prior configurations, exceeding the tolerance scale set by set_tolerance (e.g. tolerance 10).
- Each run's datalogs stats table contains a '$\log{Z}$' entry with an associated uncertainty; rows missing it indicate extraction failure.
- Baseline recovered signals match the published values: P_rot = 29.72 d and P_mag = 2557.0 d for HD 55693.
- engine_config['setup'] (ntemps, nwalkers, nsweeps, nsteps) and run_config burn-in match the baseline run so that only priors differ across configurations.

## Limitations

- ln Z sensitivity means model-selection conclusions may be prior-dependent; purely statistical inference might be insufficient on its own for robust exoplanet detection.
- Evidence must be estimated via thermodynamic integration or stepping stones; the choice of method ('ss' vs TI) affects the estimate.
- Conclusions are tied to the specific HD 55693 model and reddemcee sampling configuration; different burn-in or tempering settings would not be comparable to the baseline.

## Evidence

- [abstract] HD 55693 prior sensitivity: "the stellar rotation, P rot = 29.72 −0.02 + 0.01 , and magnetic cycle, P mag = 2557.0 −36.7 + 70.1 , were recovered, while demonstrating the sensitivity of ln 𝒵 to prior selection."
- [methods] comparison criteria and tolerance: "The comparison statistic can be set with the `set_comparison_criteria` method, and the tolerance changed with the `set_tolerance` method."
- [methods] evidence estimation methods: "Curvature-aware Thermodynamic Integration (TI+), Geometric-Bridge Stepping Stones (SS+), and a Hybrid algorithm"
- [methods] condition API: "sim.add_condition(['Period 1', 'limits', [3, 5]])"
- [methods] engine configuration: "sim.set_engine('reddemcee')
sim.engine_config['setup'] = [10, 256, 2048, 1]"
- [methods] posterior-driven prior constraint: "a new prior will be placed upon it. \emp\ includes two approaches for modelling the posterior of each parameter-- Kernel Density Estimation and Bayesian Gaussian Mixtures."
