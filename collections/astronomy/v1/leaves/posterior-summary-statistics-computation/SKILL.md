---
name: posterior-summary-statistics-computation
description: Use when you have retained posterior samples from an MCMC run (e.g. CosmoHammer/emcee
  over WMAP 7 Lambda-CDM parameters) and need means, marginal standard deviations,
  and a within-tolerance comparison to a reference sampler.
when_to_use_negative:
- You only need a single best-fit point before sampling (the article instead uses
  particle swarm optimization via ParticleSwarmOptimizer for that purpose), not a
  posterior distribution over samples.
- The chain has not yet converged or burn-in has not been run/determined — summarising
  unconverged samples would violate the ~3.4%-of-sigma agreement criterion.
- You only have single likelihood evaluations (e.g. the WMAP 3/5/7/9-year likelihood
  print placeholders) rather than an ensemble of posterior samples.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CosmoHammer
  role: Python MCMC framework whose CosmoHammerSampler produces the posterior samples
    that are summarised
  repo: https://github.com/cosmo-ethz/CosmoHammer
- name: emcee
  role: Affine invariant Goodman-Weare ensemble sampler embedded in CosmoHammer that
    generates the walker samples
- name: CosmoMC
  role: Reference Metropolis-Hastings sampler whose posterior parameter estimates
    (Table 3) serve as the comparison baseline
- name: CosmoHammerPlugins
  role: Provides the WMAP 7-year likelihood and PyCamb/CambCoreModule modules used
    in the likelihood chain behind the samples
  repo: https://github.com/cosmo-ethz/CosmoHammerPlugins
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:ae331249833a52e0985f7cc7095a4af95957f296e214c2d4c57669ef06152bd0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cosmo-ethz/CosmoHammer
  repo_url: https://github.com/cosmo-ethz/CosmoHammer
  tools:
  - CAMB
  - CosmoHammer
  - CosmoHammerPlugins
  - CosmoMC
  - Python
  - emcee
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# posterior-summary-statistics-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Discard burn-in from an MCMC chain and compute per-parameter posterior means and marginal standard deviations (here for the seven Lambda-CDM parameters fitted to WMAP 7-year data with CosmoHammer/emcee), then validate the estimates against a reference chain such as CosmoMC within the expected statistical-error tolerance of about 3.4% of sigma.

## When to use

Use when you have retained posterior samples from an MCMC run (e.g. CosmoHammer/emcee over WMAP 7 Lambda-CDM parameters) and need means, marginal standard deviations, and a within-tolerance comparison to a reference sampler.

## When NOT to use

- You only need a single best-fit point before sampling (the article instead uses particle swarm optimization via ParticleSwarmOptimizer for that purpose), not a posterior distribution over samples.
- The chain has not yet converged or burn-in has not been run/determined — summarising unconverged samples would violate the ~3.4%-of-sigma agreement criterion.
- You only have single likelihood evaluations (e.g. the WMAP 3/5/7/9-year likelihood print placeholders) rather than an ensemble of posterior samples.

## Inputs

- Retained posterior sample matrix from the CosmoHammer/emcee ensemble sampler (87,500 converged samples after burn-in for 7 Lambda-CDM parameters)
- Burn-in iteration count / retained-sample range from the CosmoHammerSampler configuration
- CosmoMC reference posterior summary (published Table 3 values or a reference chain with N = 35,000 samples)

## Outputs

- posterior_summary.csv with a mean and a marginal standard deviation column for all seven Lambda-CDM parameters
- Per-parameter deviation of CosmoHammer estimates from the CosmoMC Table 3 reference, checked against the ~3.4%-of-sigma tolerance

## How to apply

After the CosmoHammer sampling finishes (e.g. walkers ratio of 50, i.e. 50 x 7 = 350 walkers, with a burn-in phase and enough iterations to accumulate 87,500 converged posterior samples via startSampling()), first discard the burn-in segment so only converged samples are retained. For each of the seven Lambda-CDM parameters, compute the posterior mean and the marginal standard deviation from the retained samples. Extract the same posterior summary from the CosmoMC reference chain (N = 35,000 samples) or take its published Table 3 values. Verify that each CosmoHammer parameter estimate deviates from the reference by no more than the expected statistical error of about 3.4% of the standard deviation, which is the tolerance implied by the chosen sample sizes. Persist the results as a posterior_summary.csv with mean and marginal-standard-deviation columns for all seven parameters.

## Related tools

- **CosmoHammer** (Python MCMC framework whose CosmoHammerSampler produces the posterior samples that are summarised) — https://github.com/cosmo-ethz/CosmoHammer
- **emcee** (Affine invariant Goodman-Weare ensemble sampler embedded in CosmoHammer that generates the walker samples)
- **CosmoMC** (Reference Metropolis-Hastings sampler whose posterior parameter estimates (Table 3) serve as the comparison baseline)
- **CosmoHammerPlugins** (Provides the WMAP 7-year likelihood and PyCamb/CambCoreModule modules used in the likelihood chain behind the samples) — https://github.com/cosmo-ethz/CosmoHammerPlugins

## Examples

```
samples = sampler.getSamples(); retained = samples[burnin:]; means = retained.mean(axis=0); sigmas = retained.std(axis=0)
```

## Evaluation signals

- File posterior_summary.csv exists and contains a mean and a marginal standard deviation column for all seven Lambda-CDM parameters.
- Each CosmoHammer parameter estimate deviates from the CosmoMC Table 3 reference by no more than the reported ~3.4%-of-sigma tolerance.
- Burn-in has been discarded before summarising, so only the retained ~87,500 converged samples enter the mean/sigma computation.
- The number of samples used matches the configured sampling (walkers ratio 50 -> 350 walkers for 7 parameters, consistent with the accumulated sample count).

## Limitations

- Agreement tolerance is statistical: the ~3.4%-of-sigma expectation holds only given the chosen sample sizes (87,500 CosmoHammer samples vs. N = 35,000 CosmoMC samples); smaller chains will not meet it.
- Sampling efficiency degrades when more processes are available than walkers, which can idle processes and leave the chain under-sampled.
- The paper's published code snippets show likelihood print statements only as format placeholders, so likelihood/posterior values must come from your own run, not from the listing.
- WMAP data must be accessible on the local filesystem and CFITSIO installed; otherwise the likelihood chain behind the samples cannot be reproduced.

## Evidence

- [results] print("start sampling")
sampler.startSampling(): "print("start sampling")
sampler.startSampling()"
