---
name: goodman-weare-ensemble-stretch-move-sampling
description: Use when you need to sample a Bayesian posterior over cosmological model
  parameters with an ensemble of walkers, e.g. a chi-squared likelihood with Gaussian
  priors inside the cosmobl::MCMC framework in C++.
when_to_use_negative:
- The target is a single-chain, single-particle random walk — use the Metropolis-Hastings
  (Hastings 1970) sampler instead
- The posterior is not expressible inside the CBL chi-squared-plus-Gaussian-prior
  likelihood framework and cannot be coded in C++ against CBL/GSL
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Cosmo Bologna Lib
  role: Provides the cosmobl::MCMC framework and the Goodman-Weare stretch-move sampler
    implementation
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Language in which the test posterior and the MCMC driver are written and linked
    against the CBL libraries
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: GSL
  role: External numerical library that must be installed and linked to compile and
    run the CBL MCMC code
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:29cc10bee75d34f5ad921524a829a5f5ac72ff4ca305d6adf11c4fc3e6ade93b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/federicomarulli/CosmoBolognaLib
  tools:
  - C++
  - Cosmo Bologna Lib
  - GSL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# goodman-weare-ensemble-stretch-move-sampling

## Summary

Multi-particle ensemble posterior sampling via the Goodman & Weare (2010) stretch-move algorithm as implemented in the CosmoBolognaLib (CBL) MCMC framework, complementing the single-particle Metropolis-Hastings sampler.

## When to use

Use when you need to sample a Bayesian posterior over cosmological model parameters with an ensemble of walkers, e.g. a chi-squared likelihood with Gaussian priors inside the cosmobl::MCMC framework in C++.

## When NOT to use

- The target is a single-chain, single-particle random walk — use the Metropolis-Hastings (Hastings 1970) sampler instead
- The posterior is not expressible inside the CBL chi-squared-plus-Gaussian-prior likelihood framework and cannot be coded in C++ against CBL/GSL

## Inputs

- Test posterior defined in C++ within the cosmobl::MCMC framework (chi-squared likelihood with Gaussian priors on model parameters)
- Compiled CosmoBolognaLib libraries (linked with GSL) built from the CosmoBolognaLib/ source tree

## Outputs

- Chain structure written to mcmc_chains.dat (rows of parameter samples per iteration for both samplers)
- Marginalised one-dimensional posteriors for each parameter computed from post burn-in chain samples

## How to apply

Define the target posterior inside the cosmobl::MCMC framework using a chi-squared likelihood combined with Gaussian priors on the model parameters, written in C++ and linked against CosmoBolognaLib and GSL. Build the required CBL libraries with make (e.g. 'make lib*' compiles the individual .so libraries). Run the multi-particle stretch-move sampler, in which each walker is updated by stretching toward a randomly chosen walker of the complementary ensemble, contrasting it with the single-particle Metropolis-Hastings acceptance-ratio sampler. Discard burn-in and store the post burn-in parameter samples of every chain, then compute marginalised (one-dimensional) posteriors for each parameter from the chain samples.

## Related tools

- **Cosmo Bologna Lib** (Provides the cosmobl::MCMC framework and the Goodman-Weare stretch-move sampler implementation) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Language in which the test posterior and the MCMC driver are written and linked against the CBL libraries) — https://github.com/federicomarulli/CosmoBolognaLib
- **GSL** (External numerical library that must be installed and linked to compile and run the CBL MCMC code)

## Evaluation signals

- mcmc_chains.dat has the expected structure: rows of parameter samples per iteration for both the Metropolis-Hastings and stretch-move samplers
- Marginalised one-dimensional posteriors of the test posterior converge to the expected unimodal form defined by the Gaussian priors and chi-squared likelihood
- Post burn-in samples of every chain are stored and burn-in iterations are properly discarded
- The CBL library links successfully against GSL and the code compiles (e.g. 'make lib*' produces the .so library)

## Limitations

- Version history shows likelihood modules have had bugs fixed in later releases (V4.21: 'minor bugs fixed in likelihood modules'); verify the CBL version in use
- The stretch-move sampler is documented only alongside the single-particle Metropolis-Hastings algorithm; comparative convergence diagnostics between the two samplers are not specified in the article

## Evidence

- [other] The CBL implements the Metropolis-Hastings algorithm (Hastings, 1970), which performs a single-particle sampling of the parameter space: [evidence span withheld — anchor status: not_found]
- [intro] The CBL are fully written in C++. They can be included either in C++ codes or, alternatively, in high-level scripting languages through wrapping.: [evidence span withheld — anchor status: not_found]
