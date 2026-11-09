---
name: parallel-tempering-sampler-configuration
description: Use when fitting radial velocity (RV) time series with EMPEROR and multi-modal
  posteriors or several candidate Keplerian/activity signals (e.g. HD 55693 rotation
  at ~29.72 d and magnetic cycle at ~2557 d) require adaptive parallel tempering rather
  than plain emcee/DNS sampling.
when_to_use_negative:
- The posterior is well-behaved and unimodal, where a plain emcee or DNS run suffices
  and the temperature ladder adds unnecessary cost
- The problem is not Bayesian posterior sampling of RV signals (e.g. you need a feature
  table or classical periodogram-only analysis without model comparison)
- You need kepler.solve for eccentricities outside the range zero to one, which the
  kepler.py README explicitly states will return garbage
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: EMPEROR (astroemperor)
  role: Simulation framework that wraps the sampler; set_engine and engine_config['setup']
    configure reddemcee inside sim.autorun model searches
  repo: github.com/ReddTea/astroemperor
- name: reddemcee
  role: Adaptive Parallel Tempering MCMC sampler (PTSampler) providing run_mcmc, get_evidence_ti,
    get_evidence_ss, get_evidence_hybrid and adaptation modes
  repo: github.com/ReddTea/reddemcee
- name: emcee
  role: Base ensemble sampler on which reddemcee is built; also used for autocorrelation
    time calculation
  repo: github.com/dfm/emcee
- name: kepler.py
  role: Fast Kepler's equation solver used within the RV forward model evaluated at
    each sampler step
  repo: github.com/dfm/kepler.py
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:5728a7db2c46dd033fab58d96dbe40b375c0ffec56f47a0eee58da868a4f1038
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPEROR (astroemperor)
  - Python
  - emcee
  - kepler.py
  - reddemcee
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# parallel-tempering-sampler-configuration

## Summary

Configuring EMPEROR's adaptive parallel tempering (APT) MCMC engine, reddemcee, via set_engine and engine_config['setup'] = [ntemps, nwalkers, nsweeps, nsteps] so that highly multi-modal radial velocity posteriors can be sampled efficiently and log-evidence (ln Z) estimated through thermodynamic integration or stepping stones.

## When to use

Use when fitting radial velocity (RV) time series with EMPEROR and multi-modal posteriors or several candidate Keplerian/activity signals (e.g. HD 55693 rotation at ~29.72 d and magnetic cycle at ~2557 d) require adaptive parallel tempering rather than plain emcee/DNS sampling.

## When NOT to use

- The posterior is well-behaved and unimodal, where a plain emcee or DNS run suffices and the temperature ladder adds unnecessary cost
- The problem is not Bayesian posterior sampling of RV signals (e.g. you need a feature table or classical periodogram-only analysis without model comparison)
- You need kepler.solve for eccentricities outside the range zero to one, which the kepler.py README explicitly states will return garbage

## Inputs

- Radial velocity time-series catalogue placed under datafiles/<star>/RV (e.g. HD 55693, 51 Peg, HIP 21850 RV+astrometry)
- Sampler setup tuple [ntemps, nwalkers, nsweeps, nsteps] for reddemcee
- Parameter conditions (limits, priors, initial positions) defined via add_condition, e.g. priors around P_rot ~ 29.72 d and P_mag ~ 2557 d

## Outputs

- Posterior chains with shape (ntemps, nsweeps*nsteps, nwalkers, ndim) accessible via get_chain (optionally flattened/discarded/thinned)
- Evidence estimates from thermodynamic integration (TI+), geometric-bridge stepping stones (SS+), or the hybrid algorithm
- datalogs output files: best_fit.dat, values.tex, stats.tex with posterior period estimates and uncertainties

## How to apply

Instantiate astroemperor.Simulation and load the RV catalogue with load_data (e.g. 'HD55693' read from datafiles/), then call sim.set_engine('reddemcee') and set the ladder geometry via sim.engine_config['setup'] = [ntemps, nwalkers, nsweeps, nsteps] (the paper's 51 Peg demo uses [10, 256, 2048, 1], while the HD 55693 activity analysis is run with the same four-element setup). The reddemcee PTSampler adapts the temperature ladder to equalise quantities such as uniform swap acceptance rate (SAR) or thermodynamic length (ETL), controlled through adapt_mode, adapt_tau, and adapt_nu; starting positions p0 must have shape (ntemps, nwalkers, ndim). Evidence is computed with get_evidence_ti, get_evidence_ss, or the hybrid algorithm so model comparison via set_comparison_criteria('Evidence') and set_tolerance(10) can accept successive models only when ln Z improves by more than the tolerance. The rationale is that APT achieves an effective sampling increase of a factor of 3.76 over Diffusive Nested Sampling (DNS) on 51 Pegasi while returning tighter parameter estimates.

## Related tools

- **EMPEROR (astroemperor)** (Simulation framework that wraps the sampler; set_engine and engine_config['setup'] configure reddemcee inside sim.autorun model searches) — github.com/ReddTea/astroemperor
- **reddemcee** (Adaptive Parallel Tempering MCMC sampler (PTSampler) providing run_mcmc, get_evidence_ti, get_evidence_ss, get_evidence_hybrid and adaptation modes) — github.com/ReddTea/reddemcee
- **emcee** (Base ensemble sampler on which reddemcee is built; also used for autocorrelation time calculation) — github.com/dfm/emcee
- **kepler.py** (Fast Kepler's equation solver used within the RV forward model evaluated at each sampler step) — github.com/dfm/kepler.py

## Examples

```
sim.set_engine('reddemcee'); sim.engine_config['setup'] = [10, 256, 2048, 1]; sim.autorun(1, 1)
```

## Evaluation signals

- Recovered posteriors reproduce the reported HD 55693 signals within credible intervals: P_rot = 29.72 (+0.01/-0.02) d and P_mag = 2557.0 (+70.1/-0.02... +36.7) d — i.e. 29.72 (+0.01/-0.02) d and 2557.0 (+70.1/-36.7) d
- Chain shape matches (ntemps, nsweeps*nsteps, nwalkers, ndim); e.g. sampler.get_chain(discard=100, flat=True) returns shape (5, 5000, 2) for the README's 5-temperature, 2-parameter example
- Successive models are accepted only when the log-evidence improves by more than the set tolerance (set_tolerance(10) with set_comparison_criteria('Evidence'))
- For 51 Pegasi, effective sample size increases by a factor of ~3.76 over DNS with tighter parameter estimates

## Limitations

- The recovered ln Z is sensitive to prior selection, as demonstrated in the HD 55693 analysis, so model-comparison results depend on the priors chosen via add_condition
- kepler.solve will return garbage for eccentricities out of the range zero to one
- Purely statistical inference might be insufficient on its own for robust exoplanet detection

## Evidence

- [methods] APT setup on 51 Peg: "sim.set_engine('reddemcee')
sim.engine_config['setup'] = [10, 256, 2048, 1]"
- [abstract] APT advantage over DNS: "For 51 Pegasi, we find APT achieves an effective sampling increase by a factor of 3.76 over DNS, while retrieving tighter parameter estimates."
- [readme] Sampler purpose and lineage: [evidence span withheld — anchor status: external_doc]
- [methods] Evidence estimators: "Curvature-aware Thermodynamic Integration (TI+), Geometric-Bridge Stepping Stones (SS+), and a Hybrid algorithm"
- [readme] Kepler solver validity range: [evidence span withheld — anchor status: external_doc]
