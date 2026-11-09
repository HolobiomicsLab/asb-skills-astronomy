---
name: mcmc-convergence-diagnostic-analysis
description: Use when comparing two sampler backends (e.g. emcee ensemble MCMC vs
  stan HMC/NUTS) on the same visibility likelihood, or when deciding whether a posterior
  run has converged before trusting parameter estimates.
when_to_use_negative:
- Input is a single already-converged posterior with no alternative sampler run to
  compare against and only qualitative trace inspection is needed
- The model uses the emcee-only --astrom independent astrometry fitting option, which
  has no stan counterpart and would make sampler comparison invalid
- Likelihood evaluation is so cheap that ESS efficiency gains do not matter (e.g.
  tiny u,v-averaged datasets where emcee already completes quickly)
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: arviz
  role: Computes R-hat, effective sample size, and trace stability diagnostics for
    both the emcee and HMC posteriors
  repo: github.com/drgmk/vis-r
- name: stan
  role: Gradient-based HMC/NUTS sampler backend of vis-r, initialized with Pathfinder,
    whose output is validated against the emcee run
  repo: github.com/drgmk/vis-r
- name: emcee
  role: Reference ensemble-MCMC sampler backend of vis-r whose posterior and ESS serve
    as the baseline in the comparison
  repo: github.com/drgmk/vis-r
- name: vis-r
  role: Provides the visibility likelihood and the emcee and stan sampling front ends
    being compared
  repo: github.com/drgmk/vis-r
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:53e1b12900cd7570158daf9fbf3d7c0d8e1faf24a0eafdbec9c50761509a4be9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - arviz
  - conda
  - emcee
  - stan
  - vis-r
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# MCMC convergence diagnostic analysis

## Summary

Compute convergence diagnostics (R-hat, effective sample size, trace stability) with arviz for MCMC posteriors fitted with vis-r, specifically to validate swapping the emcee sampler for the gradient-based stan HMC/NUTS implementation at equal warmup/sampling step budgets.

## When to use

Use when comparing two sampler backends (e.g. emcee ensemble MCMC vs stan HMC/NUTS) on the same visibility likelihood, or when deciding whether a posterior run has converged before trusting parameter estimates.

## When NOT to use

- Input is a single already-converged posterior with no alternative sampler run to compare against and only qualitative trace inspection is needed
- The model uses the emcee-only --astrom independent astrometry fitting option, which has no stan counterpart and would make sampler comparison invalid
- Likelihood evaluation is so cheap that ESS efficiency gains do not matter (e.g. tiny u,v-averaged datasets where emcee already completes quickly)

## Inputs

- Posterior samples from the vis-r emcee fit (hr4796.selfcal.npy visibilities, -g geometry and -p radial profile parameters)
- Posterior samples from the vis-r stan fit (--stan flag, Pathfinder-initialized HMC/NUTS)
- arviz installed in the vis-r conda environment

## Outputs

- R-hat values per profile parameter for both runs
- Effective sample size per parameter for both runs
- Posterior trace stability assessment
- Per-parameter posterior comparison (credible-interval agreement emcee vs HMC)

## How to apply

Run the reference vis-r fit on the self-calibrated visibilities (e.g. hr4796.selfcal.npy) with the emcee sampler at the default number of warmup/sampling steps, supplying disc geometry parameters via -g and radial profile parameters via -p. Rerun the same fit with the --stan flag, which first estimates parameter scales with the Pathfinder algorithm and then samples with the gradient-based HMC/NUTS sampler at the same step count. Load both posterior sets into arviz and compute R-hat, effective sample size (ESS), and posterior trace stability for each run. Accept the HMC run if R-hat is near 1 and its ESS is substantially higher than emcee's at equal step count, and if its per-parameter posteriors agree with the emcee estimates within the fitted credible intervals. Rationale: the README notes both versions default to the same number of warmup/sampling steps but 'stan will yield many more independent samples for a given number of steps', so ESS at equal step budget is the key comparison metric.

## Related tools

- **arviz** (Computes R-hat, effective sample size, and trace stability diagnostics for both the emcee and HMC posteriors) — github.com/drgmk/vis-r
- **stan** (Gradient-based HMC/NUTS sampler backend of vis-r, initialized with Pathfinder, whose output is validated against the emcee run) — github.com/drgmk/vis-r
- **emcee** (Reference ensemble-MCMC sampler backend of vis-r whose posterior and ESS serve as the baseline in the comparison) — github.com/drgmk/vis-r
- **vis-r** (Provides the visibility likelihood and the emcee and stan sampling front ends being compared) — github.com/drgmk/vis-r

## Examples

```
vis-r -v data/hr4796.selfcal.npy --stan -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01
```

## Evaluation signals

- R-hat for every parameter in the HMC run is near 1
- Effective sample size for the HMC run is substantially higher than for the emcee run at the same number of warmup/sampling steps
- HMC parameter posteriors fall within the emcee fitted credible intervals per radial profile parameter
- Posterior traces show no drift or stickiness across chains for either run

## Limitations

- The stan and python/emcee versions of vis-r are 'generally the same, but with minor differences': only the emcee version implements --astrom, and each set of visibilities can have independent offset parameters in the emcee version
- Independent offsets per visibility dataset are not available in the stan version, constraining which emcee fits can be reproduced with HMC
- The vis-r radial-profile method assumes a single disk plane and fixed vertical scale height with radius, limiting what the posterior parameters can represent

## Evidence

- [methods] stan first estimates parameter scales with Pathfinder, then HMC/NUTS samples; default step counts equal but stan yields many more independent samples: "The `stan` version first estimates the parameter scales with the Pathfinder algorithm"
- [readme] Equal default steps, but stan yields many more independent samples for a given step budget: "Both default to the same number of 
warmup/sampling steps, but `stan` will yield many more independent samples
 for a given number of steps."
- [intro] stan and python/emcee versions are generally the same with minor differences: "There are `stan` and `python/emcee` 
versions that are generally the same, but with minor differences."
- [methods] Only the emcee version implements --astrom independent astrometry per dataset: "only the `emcee` version has independent astrometry fitting for 
 each visibility dataset `--astrom` implemented"
- [abstract] vis-r likelihood supports gradient-based fitting methods such as Hamiltonian Monte-Carlo: "is sufficient for most ALMA observations of debris discs, and can be used with gradient-based fitting methods such as Hamiltonian Monte-Carlo."
