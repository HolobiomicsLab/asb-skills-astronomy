---
name: metropolis-hastings-sampler-implementation
description: Use when you need to sample a multi-parameter posterior defined by a
  chi-squared likelihood with Gaussian priors in C++/CosmoBolognaLib, producing MCMC
  chains and marginalised posteriors.
when_to_use_negative:
- The posterior is not expressible as a chi-squared likelihood with Gaussian priors
  in the CBL parameter class, or the model is not implementable as a C++ likelihood
  function linked to CBL/GSL
- You need a Python-only workflow — only a subset of the CBL has a Python wrapper,
  and the article notes full wrapping is deferred to a future version
- You need an archival, version-pinned MCMC implementation — the CBL download was
  distributed as a .tar archive from a personal institutional URL without a fixed
  release identifier
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Cosmo Bologna Lib
  role: Provides the cosmobl::MCMC framework in which the posterior is defined and
    the Metropolis-Hastings and stretch-move samplers are implemented; distributed
    as C++ libraries at github.com/federicomarulli/CosmoBolognaLib
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Language in which the test posterior and both samplers are implemented and
    compiled against CBL
- name: GSL
  role: External numerical library (GNU Scientific Library) that the C++ likelihood/sampler
    code is linked against; installed as an external dependency of CBL
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:bd9a44d5be6f4964dca39eef7fe5ff6adc13fecc972a0ab0c5f193da7e00494b
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
yes_rate: 1.0
n_claims: 7
---

# metropolis-hastings-sampler-implementation

## Summary

Implements and runs the single-particle Metropolis-Hastings (Hastings, 1970) MCMC sampler — alongside the Goodman and Weare (2010) stretch-move algorithm — inside the cosmobl::MCMC framework of CosmoBolognaLib to sample a chi-squared posterior with Gaussian priors and estimate marginalised parameter posteriors.

## When to use

Use when you need to sample a multi-parameter posterior defined by a chi-squared likelihood with Gaussian priors in C++/CosmoBolognaLib, producing MCMC chains and marginalised posteriors.

## When NOT to use

- The posterior is not expressible as a chi-squared likelihood with Gaussian priors in the CBL parameter class, or the model is not implementable as a C++ likelihood function linked to CBL/GSL
- You need a Python-only workflow — only a subset of the CBL has a Python wrapper, and the article notes full wrapping is deferred to a future version
- You need an archival, version-pinned MCMC implementation — the CBL download was distributed as a .tar archive from a personal institutional URL without a fixed release identifier

## Inputs

- Test posterior defined in C++ within the cosmobl::MCMC framework using a chi-squared likelihood combined with Gaussian priors on the model parameters
- C++ executable linked against the CosmoBolognaLib and GSL libraries
- Sampler configuration (number of walkers for stretch-move, burn-in length, proposal symmetries)

## Outputs

- mcmc_chains.dat containing the chain structure (rows of parameter samples per iteration for both samplers) and posterior estimates
- Marginalised one-dimensional posteriors for each model parameter computed from the post burn-in chain samples

## How to apply

Define the test posterior inside the cosmobl::MCMC framework as a chi-squared likelihood combined with Gaussian priors on the model parameters, written in C++ and linked against CosmoBolognaLib and GSL (installed alongside FFTW, OpenMP and Numerical Recipes). Implement the Metropolis-Hastings sampler with symmetric parameter proposals, accepting or rejecting each step via the Metropolis acceptance ratio; optionally run the multi-particle stretch-move sampler, updating each walker by stretching toward a randomly chosen walker of the complementary ensemble. Run both samplers on the test posterior, discard the burn-in, and store the post burn-in parameter samples of every chain. Compute marginalised one-dimensional posteriors for each parameter from the chain samples and write the chain structure and posterior estimates to mcmc_chains.dat. Validate by checking the file structure and that the marginalised posteriors converge to the expected unimodal form set by the Gaussian priors.

## Related tools

- **Cosmo Bologna Lib** (Provides the cosmobl::MCMC framework in which the posterior is defined and the Metropolis-Hastings and stretch-move samplers are implemented; distributed as C++ libraries at github.com/federicomarulli/CosmoBolognaLib) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Language in which the test posterior and both samplers are implemented and compiled against CBL)
- **GSL** (External numerical library (GNU Scientific Library) that the C++ likelihood/sampler code is linked against; installed as an external dependency of CBL)

## Examples

```
cd Cosmo_Bologna_Lib/ && make  # compile CBL libraries, then build and run the C++ MCMC executable linked against libFUNC and GSL
```

## Evaluation signals

- mcmc_chains.dat has the expected structure: rows of parameter samples per iteration for both the Metropolis-Hastings and stretch-move samplers
- The marginalised (one-dimensional) posteriors of the test posterior converge to the expected unimodal form defined by the Gaussian priors and chi-squared likelihood
- Post burn-in parameter samples are stored for every chain, with the burn-in segment correctly discarded
- Each Metropolis-Hastings proposal step is accepted or rejected according to the Metropolis acceptance ratio applied to symmetric updates
- The stretch-move updates target a randomly chosen walker of the complementary ensemble (Goodman and Weare 2010 scheme)

## Limitations

- Only a subset of the CBL has a Python wrapper; the MCMC skill must be exercised in C++ code linked against CBL and GSL
- The README notes 'minor bugs fixed in likelihood modules' at version V4.21, so pre-V4.21 releases may contain likelihood-module defects affecting posterior sampling
- No fixed version/release identifier is given for the CBL download (only a .tar archive at a personal institutional URL), and the repository contains no changelog for the MCMC modules

## Evidence

- [other] The CBL implements the Metropolis-Hastings algorithm (Hastings, 1970), which performs a single-particle sampling of the parameter space (alongside a stretch-move algorithm, Goodman and Weare 2010).: [evidence span withheld — anchor status: not_found]
- [intro] CBL is fully C++ and can be wrapped into scripting languages.: [evidence span withheld — anchor status: not_found]
