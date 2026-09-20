---
name: markov-chain-monte-carlo-posterior-sampling
description: Use when a cosmological model defined by a chi-squared likelihood with
  Gaussian priors needs parameter-space posterior sampling, chain output, and marginalised
  (one-dimensional) posteriors.
when_to_use_negative:
- The analysis goal is measuring one-, two- or three-point correlation statistics
  (2PCF/3PCF) from catalogues rather than sampling a posterior over model parameters
- A fixed, versioned release of the MCMC backend is required for reproducibility —
  the article's download is only a .tar archive from a personal institutional URL
  with no DOI/zenodo identifier
- Full Python-based workflow is required with no C++ compilation — only a subset of
  CBL functions has a Python wrapper, with full coverage deferred to a future version
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Cosmo Bologna Lib
  role: Provides the cosmobl::MCMC framework implementing the Metropolis-Hastings
    (Hastings 1970) and stretch-move (Goodman and Weare 2010) samplers
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Language in which the test posterior and sampler driver are written and linked
    against CBL
- name: GSL
  role: External numerical library linked against the C++ MCMC code; also installed
    as a CBL external dependency
- name: Python
  role: Alternative high-level language for including CBL via wrapping (subset of
    functions)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:8f3a3a3451bb03f66ceee89fcd77c78a5d0155f36348cd8e78c314159890f192
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/federicomarulli/CosmoBolognaLib
  tools:
  - C++
  - Cosmo Bologna Lib
  - GSL
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# markov-chain-monte-carlo-posterior-sampling

## Summary

Bayesian posterior sampling of model parameters using the two MCMC samplers implemented in the CosmoBolognaLib statistical analysis toolkit: the single-particle Metropolis-Hastings algorithm (Hastings 1970) and the multi-particle stretch-move algorithm (Goodman and Weare 2010).

## When to use

Use when a cosmological model defined by a chi-squared likelihood with Gaussian priors needs parameter-space posterior sampling, chain output, and marginalised (one-dimensional) posteriors.

## When NOT to use

- The analysis goal is measuring one-, two- or three-point correlation statistics (2PCF/3PCF) from catalogues rather than sampling a posterior over model parameters
- A fixed, versioned release of the MCMC backend is required for reproducibility — the article's download is only a .tar archive from a personal institutional URL with no DOI/zenodo identifier
- Full Python-based workflow is required with no C++ compilation — only a subset of CBL functions has a Python wrapper, with full coverage deferred to a future version

## Inputs

- Test posterior defined in C++ as a chi-squared likelihood with Gaussian priors on the model parameters
- Initial parameter vectors / walker positions for the chains
- Sampler settings (number of iterations, burn-in length, number of walkers for stretch-move)

## Outputs

- mcmc_chains.dat containing rows of parameter samples per iteration for both the Metropolis-Hastings and stretch-move samplers
- Post burn-in parameter samples of every chain
- Marginalised (one-dimensional) posteriors for each parameter

## How to apply

Define the test posterior within the cosmobl::MCMC framework using a chi-squared likelihood combined with Gaussian priors on the model parameters, written in C++ and linked against the CosmoBolognaLib and GSL. For Metropolis-Hastings, propose symmetric single-particle parameter updates and accept or reject each step with the Metropolis acceptance ratio; for stretch-move, update each walker by stretching toward a randomly chosen walker of the complementary ensemble. Run both samplers on the test posterior, discard burn-in, and store the post burn-in parameter samples of every chain. Compute marginalised (one-dimensional) posteriors for each parameter from the chain samples and write the chain structure and posterior estimates to mcmc_chains.dat. The rationale is that CBL provides both a single-particle (Metropolis-Hastings) and a multi-particle (Goodman-Weare stretch-move) exploration strategy, so ensemble-based sampling can be compared against independent-chain sampling on the same posterior.

## Related tools

- **Cosmo Bologna Lib** (Provides the cosmobl::MCMC framework implementing the Metropolis-Hastings (Hastings 1970) and stretch-move (Goodman and Weare 2010) samplers) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Language in which the test posterior and sampler driver are written and linked against CBL)
- **GSL** (External numerical library linked against the C++ MCMC code; also installed as a CBL external dependency)
- **Python** (Alternative high-level language for including CBL via wrapping (subset of functions))

## Evaluation signals

- mcmc_chains.dat has the expected structure: rows of parameter samples per iteration for both the Metropolis-Hastings and stretch-move samplers
- Marginalised (one-dimensional) posteriors of the test posterior converge to the expected unimodal form defined by the Gaussian priors and chi-squared likelihood
- Burn-in samples are discarded before the stored chain statistics are computed
- Metropolis-Hastings steps are accepted/rejected via the symmetric Metropolis acceptance ratio; stretch-move updates use walkers from the complementary ensemble
- CBL version history should be checked against the README since V4.21 fixed 'minor bugs fixed in likelihood modules', so likelihood behaviour may differ across versions

## Limitations

- Python wrapper covers only a subset of CBL functions; full coverage is deferred to a future version, so MCMC workflows may require C++
- No fixed version/release identifier is given for the CBL download; only a .tar archive at a personal institutional URL (no DOI/zenodo)
- Likelihood modules had minor bugs fixed as recently as V4.21 per the repository README version history

## Evidence

- [methods] MCMC samplers implemented in CBL: [evidence span withheld — anchor status: not_found]
- [intro] CBL language and inclusion modes: [evidence span withheld — anchor status: not_found]
