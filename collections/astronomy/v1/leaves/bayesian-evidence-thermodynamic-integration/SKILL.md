---
name: bayesian-evidence-thermodynamic-integration
description: Use when comparing nested radial-velocity orbit models (successive Keplerian
  counts in EMPEROR's autorun) and you need a Bayesian evidence criterion instead
  of BIC — e.g. when ln Z sensitivity to priors or tight posteriors matter.
when_to_use_negative:
- When only a quick heuristic comparison suffices — BIC is EMPEROR's default statistic
  with tolerance 5 and avoids the extra evidence-estimation cost
- 'When priors are not defensible: the paper shows ln Z is sensitive to prior selection
  (HD 55693 case), so evidence comparisons on poorly justified priors can mislead'
- When sampling a single fixed model rather than comparing successive models — no
  evidence ratio is needed
tools:
- name: EMPEROR (astroemperor)
  role: Orchestrates the autorun model-comparison loop; set_comparison_criteria('Evidence')
    and set_tolerance(10) activate evidence-based selection
  repo: https://github.com/ReddTea/astroemperor
- name: reddemcee
  role: Adaptive parallel-tempering sampler whose PTSampler provides get_evidence_ti,
    get_evidence_ss and get_evidence_hybrid for evidence estimation
  repo: https://github.com/ReddTea/reddemcee
- name: emcee
  role: Base ensemble sampler on which reddemcee is built; used for autocorrelation-time
    calculations
  repo: https://github.com/dfm/emcee
- name: dynesty
  role: DNS nested-sampling engine used as the baseline against which APT's evidence-sampling
    efficiency (3.76x on 51 Pegasi) was measured
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:7411544a03d24b68e27ff661cb72dc2ccd72ea61e1a7822805bdb62b68b5a84a
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# bayesian-evidence-thermodynamic-integration

## Summary

Estimates Bayesian log-evidence (ln Z) from parallel-tempering MCMC chains — via Thermodynamic Integration, Stepping Stones, or a Hybrid — and uses it as the model-comparison statistic in EMPEROR's iterative Keplerian model-selection loop, as demonstrated on the 51 Pegasi RV dataset.

## When to use

Use when comparing nested radial-velocity orbit models (successive Keplerian counts in EMPEROR's autorun) and you need a Bayesian evidence criterion instead of BIC — e.g. when ln Z sensitivity to priors or tight posteriors matter.

## When NOT to use

- When only a quick heuristic comparison suffices — BIC is EMPEROR's default statistic with tolerance 5 and avoids the extra evidence-estimation cost
- When priors are not defensible: the paper shows ln Z is sensitive to prior selection (HD 55693 case), so evidence comparisons on poorly justified priors can mislead
- When sampling a single fixed model rather than comparing successive models — no evidence ratio is needed

## Inputs

- Radial velocity time series (e.g. 51peg.vels LICK catalogue under datafiles/51Peg/RV/)
- EMPEROR Simulation configured with reddemcee engine setup [ntemps, nwalkers, nsweeps, nsteps]
- Parameter conditions (limits, priors, init_pos) for Keplerian, Offset and Jitter blocks
- Parallel-tempered posterior chains from reddemcee PTSampler

## Outputs

- ln Z evidence per model (via thermodynamic integration, stepping stones, or hybrid)
- BIC and posterior parameter estimates in stats.tex and values.tex under datalogs/<target>/run_N/k_M/tables
- Model-comparison verdict identifying the optimum model (e.g. model_comparison.csv listing each run's statistic)
- best_fit.dat with the optimum model's parameters

## How to apply

Run EMPEROR's fixed autorun control loop (e.g. sim.autorun(0, 5)), which adds one Keplerian Block per run and samples each model with the reddemcee adaptive parallel-tempering engine configured via set_engine with setup [ntemps, nwalkers, nsweeps, nsteps] (e.g. [10, 256, 2048, 1]). Switch the comparison statistic from the default BIC (tolerance 5) to evidence with set_comparison_criteria('Evidence') and set_tolerance(10), then compare successive runs by the BIC_old - BIC_new > tol (min) or reversed (max) rule using ln Z. Evidence can be computed by Curvature-aware Thermodynamic Integration (TI+), Geometric-Bridge Stepping Stones (SS+), or a Hybrid algorithm; reddemcee's PTSampler exposes get_evidence_ti and get_evidence_ss on the chain. Stop adding Keplerian components when the newest model fails to improve the statistic by more than the tolerance, and record the optimum model in the per-run tables (stats.tex, values.tex) written to datalogs. On 51 Pegasi this APT-based evidence workflow yielded a 3.76x effective sampling increase over DNS (dynesty) with tighter parameter estimates.

## Related tools

- **EMPEROR (astroemperor)** (Orchestrates the autorun model-comparison loop; set_comparison_criteria('Evidence') and set_tolerance(10) activate evidence-based selection) — https://github.com/ReddTea/astroemperor
- **reddemcee** (Adaptive parallel-tempering sampler whose PTSampler provides get_evidence_ti, get_evidence_ss and get_evidence_hybrid for evidence estimation) — https://github.com/ReddTea/reddemcee
- **emcee** (Base ensemble sampler on which reddemcee is built; used for autocorrelation-time calculations) — https://github.com/dfm/emcee
- **dynesty** (DNS nested-sampling engine used as the baseline against which APT's evidence-sampling efficiency (3.76x on 51 Pegasi) was measured)

## Examples

```
sim.set_engine('reddemcee'); sim.engine_config['setup'] = [10, 256, 2048, 1]; sim.set_comparison_criteria('Evidence'); sim.set_tolerance(10); sim.autorun(0, 5)
```

## Evaluation signals

- The autorun terminates when the newest model fails to improve ln Z by more than the set tolerance (10 for Evidence, 5 for BIC)
- model_comparison.csv lists each run's statistic and identifies the optimum model
- stats.tex and values.tex per run contain ln Z values comparable across successive Keplerian counts
- APT vs DNS cross-check: APT should reproduce the ~3.76 effective sampling increase with tighter posteriors on 51 Pegasi
- Evidence from TI, SS, and Hybrid methods should agree within expectation for well-mixed chains (convergence tests must pass)

## Limitations

- ln Z is sensitive to prior selection, as demonstrated for HD 55693 (P_rot = 29.72 d, P_mag = 2557.0 d recovered but evidence shifted by prior choice)
- Evidence estimation inherits convergence quality from the parallel-tempering run; purely statistical inference might be insufficient on its own for robust exoplanet detection
- kepler.py's lower-level solve returns garbage for eccentricities outside the range zero to one

## Evidence

- [methods] Curvature-aware Thermodynamic Integration (TI+), Geometric-Bridge Stepping Stones (SS+), and a Hybrid algorithm: "Curvature-aware Thermodynamic Integration (TI+), Geometric-Bridge Stepping Stones (SS+), and a Hybrid algorithm"
- [methods] set_comparison_criteria and set_tolerance: "The comparison statistic can be set with the `set_comparison_criteria` method, and the tolerance changed with the `set_tolerance` method."
- [intro] optimum model automatically: "performs model comparisons automatically to return the optimum model that best describes the data"
- [abstract] APT vs DNS efficiency: "For 51 Pegasi, we find APT achieves an effective sampling increase by a factor of 3.76 over DNS, while retrieving tighter parameter estimates."
- [readme] evidence functions in reddemcee: [evidence span withheld — anchor status: external_doc]
- [abstract] ln Z prior sensitivity: "were recovered, while demonstrating the sensitivity of ln 𝒵 to prior selection."
