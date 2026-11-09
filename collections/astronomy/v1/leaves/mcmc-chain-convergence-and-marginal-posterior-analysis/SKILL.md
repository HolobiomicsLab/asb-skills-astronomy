---
name: mcmc-chain-convergence-and-marginal-posterior-analysis
description: Use when a C++/CBL analysis needs posterior sampling of cosmological
  model parameters via MCMC, with a chi-squared likelihood plus Gaussian priors, and
  chain/posterior outputs must be validated.
when_to_use_negative:
- The analysis only needs catalogue handling or correlation-function measurements
  (2PCF/3PCF) rather than posterior sampling — use CBL clustering/statistics classes
  instead
- The likelihood function is not expressible as a chi-squared likelihood with Gaussian
  priors in the cosmobl::MCMC framework and requires a different sampler framework
- Only Python wrapper functions are required — the article notes only a subset of
  the CBL has a Python wrapper, so MCMC work may require C++
edam_operation: http://edamontology.org/operation_2945
edam_topics: []
tools:
- name: Cosmo Bologna Lib
  role: Provides the cosmobl::MCMC framework in which the test posterior is defined
    and the samplers run
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Language in which the posterior, Metropolis-Hastings and stretch-move samplers
    are implemented and linked against the CosmoBolognaLib
- name: GSL
  role: External numerical library (GNU Scientific Library) linked against when running
    the MCMC likelihood sampling
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:97a29bd3063800d2e013765c043e6df0ebbf41fc2e33e41b4b868bd8c7e9c41a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/federicomarulli/CosmoBolognaLib
  tools:
  - C++
  - Cosmo Bologna Lib
  - GSL
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mcmc-chain-convergence-and-marginal-posterior-analysis

## Summary

Runs and post-processes MCMC chains from the CBL statistical analysis toolkit: single-particle Metropolis-Hastings (Hastings 1970) and multi-particle stretch-move (Goodman & Weare 2010) sampling of a user-defined posterior, with burn-in removal and marginalised 1D posteriors written to mcmc_chains.dat.

## When to use

Use when a C++/CBL analysis needs posterior sampling of cosmological model parameters via MCMC, with a chi-squared likelihood plus Gaussian priors, and chain/posterior outputs must be validated.

## When NOT to use

- The analysis only needs catalogue handling or correlation-function measurements (2PCF/3PCF) rather than posterior sampling — use CBL clustering/statistics classes instead
- The likelihood function is not expressible as a chi-squared likelihood with Gaussian priors in the cosmobl::MCMC framework and requires a different sampler framework
- Only Python wrapper functions are required — the article notes only a subset of the CBL has a Python wrapper, so MCMC work may require C++

## Inputs

- User-defined test posterior (chi-squared likelihood plus Gaussian priors on model parameters) implemented in C++ under cosmobl::MCMC
- C++ source linked against CosmoBolognaLib and GSL
- Initial parameter values and chain settings (number of walkers for stretch-move, burn-in length)

## Outputs

- mcmc_chains.dat — chain structure (rows of parameter samples per iteration for both samplers) and marginalised posterior estimates
- Post burn-in parameter samples of every chain
- Marginalised (one-dimensional) posteriors per parameter

## How to apply

Define the test posterior inside the cosmobl::MCMC framework using a chi-squared likelihood combined with Gaussian priors on the model parameters, in C++ linked against the CosmoBolognaLib and GSL. Implement the single-particle Metropolis-Hastings sampler with symmetric proposals and Metropolis acceptance ratio, and the Goodman & Weare (2010) stretch-move sampler, where each walker is updated by stretching toward a randomly chosen walker of the complementary ensemble. Run both samplers on the test posterior, discard burn-in, and store post burn-in parameter samples of every chain. Compute marginalised (one-dimensional) posteriors for each parameter from the chain samples and write the chain structure and posterior estimates to mcmc_chains.dat. Decision point: choose Metropolis-Hastings for single-particle exploration versus stretch-move for multi-particle ensemble exploration of the parameter space.

## Related tools

- **Cosmo Bologna Lib** (Provides the cosmobl::MCMC framework in which the test posterior is defined and the samplers run) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Language in which the posterior, Metropolis-Hastings and stretch-move samplers are implemented and linked against the CosmoBolognaLib)
- **GSL** (External numerical library (GNU Scientific Library) linked against when running the MCMC likelihood sampling)

## Examples

```
g++ -o run_mcmc run_mcmc.cpp -I$CBL_DIR -L$CBL_DIR -lcbl -lgsl -lgslcblas && ./run_mcmc  # posterior defined in cosmobl::MCMC; output written to mcmc_chains.dat
```

## Evaluation signals

- Chain output file mcmc_chains.dat has the expected structure: rows of parameter samples per iteration for both samplers
- Marginalised posteriors of the test posterior converge to the expected unimodal form defined by the Gaussian priors and chi-squared likelihood
- Both the Metropolis-Hastings and stretch-move chains produce post burn-in samples after burn-in is discarded
- Likelihood evaluations use the chi-squared likelihood with Gaussian priors consistently across both sampler implementations

## Limitations

- Convergence is validated only against a test posterior with a unimodal form; multimodal or badly scaled posteriors are not covered by the described validation
- The article's MCMC finding is specific to the CBL implementation (Metropolis-Hastings and Goodman & Weare 2010 stretch-move); other sampler families are out of scope
- Repository README notes 'V4.21 minor bugs fixed in likelihood modules', indicating likelihood-module bugs existed historically

## Evidence

- [methods] Implement the single-particle Metropolis-Hastings sampler (Hastings 1970), proposing symmetric parameter updates and accepting or rejecting each step: [evidence span withheld — anchor status: not_found]
- [methods] Run both samplers on the test posterior, discarding burn-in and storing the post burn-in parameter samples of every chain.: [evidence span withheld — anchor status: not_found]
- [results] verify that the chain output file has the expected structure (rows of parameter samples per iteration for both samplers): [evidence span withheld — anchor status: not_found]
