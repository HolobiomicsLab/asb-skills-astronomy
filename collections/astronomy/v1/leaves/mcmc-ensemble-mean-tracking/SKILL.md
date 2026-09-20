---
name: mcmc-ensemble-mean-tracking
description: Use when you need to quantify how many MCMC iterations are required before
  chains lose their initialisation bias, especially when comparing ensemble samplers
  with large walker counts L against Metropolis–Hastings chains.
when_to_use_negative:
- When chains are Metropolis–Hastings without an accessible ensemble of independent
  walkers, the predicted error sigma/sqrt(L) must be reinterpreted per chain rather
  than per walker ensemble
- When samples are taken from the self-tuning CosmoMC proposal-tuning phase (~2000
  steps), as those samples are not yet usable for inference
- When the goal is autocorrelation-time estimation itself rather than burn-in determination
  — use acor (dfm/acor) for tau estimation instead
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3452
tools:
- name: CosmoHammer
  role: Generates the emcee ensemble chains whose walker-mean trajectories are tracked
    to determine burn-in
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: emcee
  role: Affine invariant ensemble sampler producing the walker ensemble (e.g. 300
    walkers) used for the X̄_t analysis
  repo: https://github.com/cosmo-ethz/CosmoHammer
- name: CosmoMC
  role: Reference Metropolis–Hastings sampler whose chains are tracked with the same
    method after ~2000 steps of proposal tuning
- name: acor
  role: Python port of Goodman's ACOR routine used to estimate autocorrelation times
    of the chains, relevant to interpreting differing burn-in behaviour
  repo: https://github.com/dfm/acor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:263b2b90fb3fe471d0f85e34ed11caf86b712d99fd59dc733d25160e584a1a36
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cosmo-ethz/CosmoHammer
  tools:
  - CosmoHammer
  - CosmoMC
  - Python
  - acor
  - emcee
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# mcmc-ensemble-mean-tracking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Determines MCMC burn-in length by monitoring the ensemble mean of sequential walker steps X̄_t against the predicted standard error of the mean sigma/sqrt(L), identifying when initialisation bias vanishes. Applied to CosmoHammer (emcee) and fine-tuned CosmoMC chains for six WMAP7 cosmological parameters.

## When to use

Use when you need to quantify how many MCMC iterations are required before chains lose their initialisation bias, especially when comparing ensemble samplers with large walker counts L against Metropolis–Hastings chains.

## When NOT to use

- When chains are Metropolis–Hastings without an accessible ensemble of independent walkers, the predicted error sigma/sqrt(L) must be reinterpreted per chain rather than per walker ensemble
- When samples are taken from the self-tuning CosmoMC proposal-tuning phase (~2000 steps), as those samples are not yet usable for inference
- When the goal is autocorrelation-time estimation itself rather than burn-in determination — use acor (dfm/acor) for tau estimation instead

## Inputs

- Posterior sample chains from CosmoHammer (emcee) runs for six WMAP7 cosmological parameters
- Posterior sample chains from fine-tuned CosmoMC (Metropolis–Hastings) runs, after the ~2000-step proposal-tuning phase
- Number of walkers L (e.g. 300 walkers from a walkers ratio of 50) or number of chains

## Outputs

- Trajectories of the ensemble mean sequential step X̄_t vs iteration t, with the sigma/sqrt(L) band overlaid (burnin_convergence_analysis.png)
- Determined burn-in lengths table (burnin_summary.csv), e.g. 250 steps for both CosmoHammer configurations and ~2000 tuning steps for self-tuning CosmoMC

## How to apply

Load posterior samples from the CosmoHammer/emcee chains (e.g. walkers ratio 50 launching 300 walkers for six WMAP7 parameters) and compute the ensemble mean sequential step X̄_t as a function of iteration t across all walkers. Estimate the scatter of X̄_t across realisations and compare it with the predicted standard error sigma/sqrt(L), where L is the number of walkers or chains. The burn-in length is the iteration at which X̄_t first fluctuates within the sigma/sqrt(L) band — 250 steps was found as a safe burn-in for both CosmoHammer configurations. Repeat the same tracking for self-tuning CosmoMC Metropolis–Hastings chains, which require roughly 2000 steps of proposal tuning before samples become usable. Plot the X̄_t trajectories with the sigma/sqrt(L) band to visualise where the initialisation bias vanishes. The rationale is that parallelised ensemble samplers with large L allow a tight predicted standard error, making burn-in detectable despite different autocorrelation times between the two samplers.

## Related tools

- **CosmoHammer** (Generates the emcee ensemble chains whose walker-mean trajectories are tracked to determine burn-in) — http://github.com/cosmo-ethz/CosmoHammer
- **emcee** (Affine invariant ensemble sampler producing the walker ensemble (e.g. 300 walkers) used for the X̄_t analysis) — https://github.com/cosmo-ethz/CosmoHammer
- **CosmoMC** (Reference Metropolis–Hastings sampler whose chains are tracked with the same method after ~2000 steps of proposal tuning)
- **acor** (Python port of Goodman's ACOR routine used to estimate autocorrelation times of the chains, relevant to interpreting differing burn-in behaviour) — https://github.com/dfm/acor

## Examples

```
chain = LikelihoodComputationChain(min=params[:,1], max=params[:,2]); sampler = CosmoHammerSampler(params=params, chain=chain, filePrefix='wmap7_', walkersRatio=50, burnin=250, sampleIterations=100); sampler.startSampling()
```

## Evaluation signals

- The determined burn-in length must be 250 steps for both CosmoHammer configurations, matching the paper's reported value
- Self-tuning CosmoMC must require approximately 2000 steps of proposal tuning before samples become usable
- After burn-in, X̄_t must remain fluctuating within the sigma/sqrt(L) band across iterations
- The scatter of X̄_t across realisations must be consistent with the predicted standard error sigma/sqrt(L) once initialisation bias is lost
- The saved outputs must be burnin_convergence_analysis.png (trajectories with error band) and burnin_summary.csv (burn-in lengths table)

## Limitations

- Scaling saturates: once there are more processes than walkers, processes idle, so the walker count L bounds effective parallelism and thus the achievable sigma/sqrt(L)
- Doubling cores incurs ~11% scaling overhead (alpha = 0.89), so wall-time savings during the tuning/burn-in phase are not perfectly linear
- CosmoMC chains cannot be used before the ~2000-step proposal-tuning phase completes; samples from that phase must be discarded

## Evidence

- [other] By monitoring the mean sequential step of the walkers against the predicted standard error of the mean, a safe burn-in period of 250 iterations is found: [evidence span withheld — anchor status: not_found]
- [readme] emcee as embedded sampler: [evidence span withheld — anchor status: not_found]
- [intro] CosmoMC description: [evidence span withheld — anchor status: not_found]
