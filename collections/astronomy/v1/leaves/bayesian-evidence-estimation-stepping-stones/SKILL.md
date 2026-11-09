---
name: bayesian-evidence-estimation-stepping-stones
description: Use when comparing competing RV models (e.g. number of Keplerians, stellar-activity
  vs planetary signals) or testing whether ln Z is sensitive to prior choices — specifically
  when EMPEROR/reddemcee chains exist and evidence must be computed with 'Evidence'
  as the comparison criterion.
when_to_use_negative:
- When you only need a quick model ranking rather than calibrated evidence values
  — BIC can be selected instead via set_comparison_criteria, which is cheaper than
  stepping stones
- When the MCMC chains have not passed convergence tests or burn-in is insufficient
  — the tempered-sampling evidence estimate will be biased and the comparison meaningless
- When the analysis is not an EMPEROR/reddemcee radial-velocity (or joint RV+astrometry)
  fit; the skill is tied to EMPEROR's Simulation API and its datalogs output
tools:
- name: EMPEROR (astroemperor)
  role: Simulation API that runs the model fits, exposes set_comparison_criteria,
    set_tolerance, add_condition, and writes log Z to the datalogs stats table
  repo: https://github.com/ReddTea/astroemperor
- name: reddemcee
  role: Adaptive parallel-tempering MCMC sampler providing get_evidence_ti, get_evidence_ss
    and get_evidence_hybrid for evidence estimation over tempered chains
  repo: https://github.com/ReddTea/reddemcee
- name: emcee
  role: Base ensemble sampler underlying reddemcee, also used for autocorrelation-time
    diagnostics that gate chain convergence before evidence integration
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:92cd9e77404ed40156ce2afba9043f8823f8bf4a8ae1d889592b5cfb6f55f8da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ReddTea/astroemperor
  tools:
  - EMPEROR (astroemperor)
  - Python
  - emcee
  - reddemcee
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bayesian-evidence-estimation-stepping-stones

## Summary

Estimate the Bayesian evidence (ln Z) of radial-velocity models with EMPEROR by configuring reddemcee parallel-tempering runs and computing evidence via stepping stones (SS) or thermodynamic integration (TI), then use Δln Z between models or prior configurations for model comparison, as demonstrated on HD 55693 where ln Z proved sensitive to prior selection.

## When to use

Use when comparing competing RV models (e.g. number of Keplerians, stellar-activity vs planetary signals) or testing whether ln Z is sensitive to prior choices — specifically when EMPEROR/reddemcee chains exist and evidence must be computed with 'Evidence' as the comparison criterion.

## When NOT to use

- When you only need a quick model ranking rather than calibrated evidence values — BIC can be selected instead via set_comparison_criteria, which is cheaper than stepping stones
- When the MCMC chains have not passed convergence tests or burn-in is insufficient — the tempered-sampling evidence estimate will be biased and the comparison meaningless
- When the analysis is not an EMPEROR/reddemcee radial-velocity (or joint RV+astrometry) fit; the skill is tied to EMPEROR's Simulation API and its datalogs output

## Inputs

- RV time-series folder in /datafiles (e.g. HD 55693 catalogue loaded via sim.load_data)
- Fitted EMPEROR model / chains from a baseline reproduction (Keplerian blocks plus stellar-rotation P_rot and magnetic-cycle P_mag noise terms)
- engine_config['setup'] = [ntemps, nwalkers, nsweeps, nsteps] for reddemcee and run_config burn-in settings
- Alternative prior definitions passed as sim.add_condition([param, 'prior', ...]) / ([param, 'prargs', ...])

## Outputs

- log Z and its uncertainty per model/prior configuration from the datalogs stats table ('log Z' entry)
- Δln Z table relative to the baseline configuration (e.g. prior_evidence_comparison.csv)
- Evidence-based model-selection decision (which Keplerian count / activity model is optimum)

## How to apply

Set the comparison statistic with sim.set_comparison_criteria('Evidence') and optionally widen acceptance with sim.set_tolerance (e.g. tolerance 10). Select the evidence estimator with sim.evidence_method = 'ss' for geometric-bridge stepping stones, or the thermodynamic-integration / hybrid alternatives implemented in reddemcee. Run the parallel-tempering MCMC (engine_config['setup'] = [ntemps, nwalkers, nsweeps, nsteps], e.g. [10, 256, 2048, 1]) with a suitable burn-in from run_config so the tempered chains are converged before integrating. Re-run the identical model under alternative settings (e.g. Uniform vs Normal priors applied via sim.add_condition([param, 'prior', ...])) and extract log Z with its uncertainty from the datalogs stats table (the '$\log{Z}$' entry). Rationale: stepping-stones estimates are only trustworthy if each temperature level is well sampled, which is why the adaptive parallel-tempering ladder (reddemcee on top of emcee) is used to handle EMPEROR's highly multi-modal posteriors; Δln Z exceeding the tolerance is then used as the decision point between models.

## Related tools

- **EMPEROR (astroemperor)** (Simulation API that runs the model fits, exposes set_comparison_criteria, set_tolerance, add_condition, and writes log Z to the datalogs stats table) — https://github.com/ReddTea/astroemperor
- **reddemcee** (Adaptive parallel-tempering MCMC sampler providing get_evidence_ti, get_evidence_ss and get_evidence_hybrid for evidence estimation over tempered chains) — https://github.com/ReddTea/reddemcee
- **emcee** (Base ensemble sampler underlying reddemcee, also used for autocorrelation-time diagnostics that gate chain convergence before evidence integration)

## Examples

```
sim.set_engine('reddemcee'); sim.engine_config['setup'] = [10, 256, 2048, 1]; sim.set_comparison_criteria('Evidence'); sim.evidence_method = 'ss'
```

## Evaluation signals

- The datalogs stats table contains a finite '$\log{Z}$' entry with an uncertainty for every run
- Re-running the identical baseline model reproduces the baseline ln Z within the stepping-stones uncertainty
- Prior-sensitivity validation: Δln Z between prior configurations exceeds the model-comparison tolerance (e.g. set_tolerance(10) scale), as reported for HD 55693
- Evidence values from alternative estimators (TI vs SS vs hybrid via get_evidence_ti / get_evidence_ss / get_evidence_hybrid) are mutually consistent within errors
- The selected optimum model matches the recovered signals (P_rot = 29.72 d, P_mag = 2557.0 d for HD 55693) and the model-comparison decision is stable under the chosen tolerance

## Limitations

- Purely statistical inference (evidence-based model selection) might be insufficient on its own for robust exoplanet detection — physical plausibility must also be checked
- ln Z is sensitive to prior selection, so evidence comparisons across models with different priors are not directly interpretable without controlling prior settings
- Evidence estimates require well-mixed parallel-tempering chains; reddemcee's solve interface analogy warns that inputs outside valid ranges (e.g. eccentricities outside 0–1) return garbage, so parameter limits must be enforced

## Evidence

- [abstract] For HD 55693, stellar rotation P_rot = 29.72 days and magnetic cycle P_mag = 2557.0 days were recovered, while demonstrating sensitivity of ln Z to prior selection: "the stellar rotation, P rot = 29.72 −0.02 + 0.01 , and magnetic cycle, P mag = 2557.0 −36.7 + 70.1 , were recovered, while demonstrating the sensitivity of ln 𝒵 to prior selection."
- [methods] Evidence estimation methods available in EMPEROR: "Curvature-aware Thermodynamic Integration (TI+), Geometric-Bridge Stepping Stones (SS+), and a Hybrid algorithm"
- [methods] Comparison statistic and tolerance configuration: "The comparison statistic can be set with the `set_comparison_criteria` method, and the tolerance changed with the `set_tolerance` method."
- [readme] reddemcee evidence functions: "| get_evidence_ti       | Calculates evidence by Thermodynamic Integration  |
| get_evidence_ss       | Calculates evidence by Stepping Stones            |"
