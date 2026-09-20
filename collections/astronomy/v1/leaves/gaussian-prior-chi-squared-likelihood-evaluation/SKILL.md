---
name: gaussian-prior-chi-squared-likelihood-evaluation
description: 'Use when a posterior must be defined numerically before MCMC sampling
  in CBL: a chi-squared likelihood from model-data comparison plus Gaussian priors
  on cosmological parameters, to be explored by Metropolis-Hastings or stretch-move
  samplers.'
when_to_use_negative:
- The analysis goal is clustering measurement (2PCF/3PCF) or catalogue handling rather
  than parameter inference — use CBL's clustering and catalogue classes instead
- 'A Python-only workflow is required: only a subset of CBL has a Python wrapper,
  so a full posterior/MCMC workflow may need to stay in C++'
- The posterior likelihood is not expressible as a chi-squared with Gaussian priors,
  and a different likelihood form is required
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: Cosmo Bologna Lib
  role: Provides the cosmobl::MCMC framework and statistical analysis classes in which
    the chi-squared + Gaussian-prior posterior is defined and sampled
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Language in which the test posterior and samplers are written and linked against
    CBL
- name: GSL
  role: External dependency (GNU Scientific Library) that CBL must be installed against
    for the likelihood/MCMC code to compile and run
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:19f5485d4b8c6dbee43fdc300775b4e7fb8b571d82b18b771db9bb0eb175bd08
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
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gaussian-prior-chi-squared-likelihood-evaluation

## Summary

Defines a test posterior inside the cosmobl::MCMC framework by combining a chi-squared likelihood with Gaussian priors on model parameters, written in C++ and linked against CosmoBolognaLib and GSL, so that MCMC samplers (Metropolis-Hastings, Goodman-Weare stretch move) can explore it.

## When to use

Use when a posterior must be defined numerically before MCMC sampling in CBL: a chi-squared likelihood from model-data comparison plus Gaussian priors on cosmological parameters, to be explored by Metropolis-Hastings or stretch-move samplers.

## When NOT to use

- The analysis goal is clustering measurement (2PCF/3PCF) or catalogue handling rather than parameter inference — use CBL's clustering and catalogue classes instead
- A Python-only workflow is required: only a subset of CBL has a Python wrapper, so a full posterior/MCMC workflow may need to stay in C++
- The posterior likelihood is not expressible as a chi-squared with Gaussian priors, and a different likelihood form is required

## Inputs

- C++ source file defining the test posterior (chi-squared likelihood plus Gaussian priors) for the cosmobl::MCMC framework
- Cosmology model parameter vector (e.g. Omega_B, Omega_M, Omega_DE, sigma8, h) with Gaussian prior means and widths
- Observational data or catalogue statistics to compare against the model via the chi-squared term
- Linked CosmoBolognaLib and GSL installations

## Outputs

- Chain samples per iteration for Metropolis-Hastings and stretch-move samplers (rows of parameter samples per iteration)
- mcmc_chains.dat file containing the chain structure and posterior estimates
- Marginalised (one-dimensional) posteriors for each model parameter

## How to apply

Write the test posterior in C++ inside the cosmobl::MCMC framework, computing a chi-squared likelihood and combining it with Gaussian priors on each model parameter, then link the code against the CosmoBolognaLib and GSL. The likelihood term enters the Metropolis acceptance ratio used by the single-particle Metropolis-Hastings sampler (Hastings 1970) and by the multi-particle Goodman-Weare (2010) stretch-move sampler, in which each walker is stretched toward a randomly chosen walker of the complementary ensemble. After running either sampler, discard burn-in and keep the post burn-in parameter samples of every chain. Compute marginalised (one-dimensional) posteriors for each parameter from the chain samples and write the chain structure to mcmc_chains.dat. The Gaussian priors jointly with the chi-squared likelihood define the expected unimodal posterior shape used for validation.

## Related tools

- **Cosmo Bologna Lib** (Provides the cosmobl::MCMC framework and statistical analysis classes in which the chi-squared + Gaussian-prior posterior is defined and sampled) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Language in which the test posterior and samplers are written and linked against CBL)
- **GSL** (External dependency (GNU Scientific Library) that CBL must be installed against for the likelihood/MCMC code to compile and run)

## Examples

```
cosm.set_OmegaB(0.049); cosm.set_OmegaM(0.319); cosm.set_OmegaDE(0.681); cosm.set_sigma8(0.83); cosm.set_hh(0.67);
```

## Evaluation signals

- The chain output file mcmc_chains.dat has the expected structure: rows of parameter samples per iteration for both the Metropolis-Hastings and stretch-move samplers
- Marginalised (one-dimensional) posteriors of the test posterior converge to the expected unimodal form defined by the Gaussian priors and chi-squared likelihood
- Metropolis-Hastings steps are accepted/rejected with the correct Metropolis acceptance ratio on the chi-squared + Gaussian-prior posterior
- Burn-in is discarded and stored samples come only from the post burn-in portion of every chain
- The stretch-move updates each walker by stretching toward a randomly chosen walker of the complementary ensemble (Goodman and Weare 2010 scheme)

## Limitations

- Only a subset of the CBL has a Python wrapper; likelihood/MCMC work must stay in C++ for full coverage
- No fixed version/release identifier is given for the CBL download (only a .tar archive at a personal institutional URL), so likelihood-module behaviour may vary between builds
- Example codes depend on local input files (e.g. cat.dat, random/, pairs/, output/ directories) not described or hosted, so reproducing the posterior setup requires those inputs

## Evidence

- [intro] CBL written in C++, wrappable: [evidence span withheld — anchor status: not_found]
