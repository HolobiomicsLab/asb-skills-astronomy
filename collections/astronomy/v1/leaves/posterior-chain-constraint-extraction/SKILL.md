---
name: posterior-chain-constraint-extraction
description: Use when you have an array of NUTS samples drawn on Fisher-normalised
  parameters (e.g. from a jax.lax.scan BlackJAX loop) and need marginal means/standard
  deviations in physical cosmological parameter units.
when_to_use_negative:
- Chains were sampled directly in physical parameter space (no Fisher normalisation),
  so denormalisation would double-scale the draws
- You need full posterior shape (non-Gaussianities, credible intervals), not just
  marginal mean/std summaries — NUTS chains support kernel density or quantile estimates
  instead
- The sampler used a non-identity inverse mass matrix or a different normalisation
  scheme than the Fisher scaling used here
edam_operation: http://edamontology.org/operation_0438
tools:
- name: candl
  role: Provides the differentiable JAX likelihood and candl.tools.get_fisher_matrix
    used to obtain the parameter scales for normalising/denormalising the chains
  repo: https://github.com/Lbalkenhol/candl
- name: BlackJAX
  role: Supplies the gradient-based NUTS sampler whose posterior chains are the input
    to the constraint extraction
  repo: https://github.com/blackjax-devs/blackjax
- name: JAX
  role: Runs the jit-compiled likelihood and the jax.lax.scan inference loop that
    generated the sample array
- name: CosmoPower-JAX
  role: Emulator (TT NN, TE PCAplusNN, EE NN) wrapped via candl.interface.get_CosmoPowerJAX_pars_to_theory_specs_func
    to supply theory spectra to the likelihood during sampling
  repo: https://github.com/dpiras/cosmopower-jax
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:935d3935b0a63a7ba83162c34a5592c98354bbe1b96fda574940beb5378d0d8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lbalkenhol/candl
  tools:
  - BlackJAX
  - CosmoPower-JAX
  - JAX
  - Python
  - candl
  edam_operation: http://edamontology.org/operation_0438
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# posterior-chain-constraint-extraction

## Summary

Extracts per-parameter marginal constraints (mean and standard deviation) from gradient-based MCMC posterior chains produced with BlackJAX NUTS on a candl JAX likelihood, by denormalising Fisher-normalised draws back into physical parameter units.

## When to use

Use when you have an array of NUTS samples drawn on Fisher-normalised parameters (e.g. from a jax.lax.scan BlackJAX loop) and need marginal means/standard deviations in physical cosmological parameter units.

## When NOT to use

- Chains were sampled directly in physical parameter space (no Fisher normalisation), so denormalisation would double-scale the draws
- You need full posterior shape (non-Gaussianities, credible intervals), not just marginal mean/std summaries — NUTS chains support kernel density or quantile estimates instead
- The sampler used a non-identity inverse mass matrix or a different normalisation scheme than the Fisher scaling used here

## Inputs

- Posterior sample array (e.g. nuts_posterior_samples.npy), shape (n_samples, n_par), drawn on Fisher-normalised parameters
- Fisher parameter covariance/scales from candl.tools.get_fisher_matrix at the fiducial point (H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0)
- Parameter ordering (par_order) used when building the normalised likelihood

## Outputs

- Per-parameter marginal means in physical units
- Per-parameter marginal standard deviations (1-sigma constraints)
- Denormalised posterior chains in physical parameter units

## How to apply

Load the saved posterior chain array (e.g. nuts_posterior_samples.npy) with numpy/JAX and confirm its shape is (n_samples, n_par) matching the parameter vector used in the Fisher normalisation (H0, ombh2, omch2, logA, ns, tau, yp for the SPT-3G 2018 TT/TE/EE example). Denormalise each draw by multiplying by the Fisher parameter scales — the square roots of the diagonal of the par_cov returned by candl.tools.get_fisher_matrix at the Planck 18 best-fit point — and re-adding the fiducial means. Compute the marginal mean and standard deviation of each parameter over the denormalised chain. Because the chains were sampled in a zero-mean unit-variance space, constraints are trustworthy only if the sampler mixed well (e.g. step_size=0.1 NUTS kernel with identity inverse mass matrix); inspect the chain for stuck walkers before quoting errors.

## Related tools

- **candl** (Provides the differentiable JAX likelihood and candl.tools.get_fisher_matrix used to obtain the parameter scales for normalising/denormalising the chains) — https://github.com/Lbalkenhol/candl
- **BlackJAX** (Supplies the gradient-based NUTS sampler whose posterior chains are the input to the constraint extraction) — https://github.com/blackjax-devs/blackjax
- **JAX** (Runs the jit-compiled likelihood and the jax.lax.scan inference loop that generated the sample array)
- **CosmoPower-JAX** (Emulator (TT NN, TE PCAplusNN, EE NN) wrapped via candl.interface.get_CosmoPowerJAX_pars_to_theory_specs_func to supply theory spectra to the likelihood during sampling) — https://github.com/dpiras/cosmopower-jax

## Examples

```
samples = np.load('nuts_posterior_samples.npy'); chains = samples * fisher_scales + fiducial_means; pars_mean, pars_std = chains.mean(axis=0), chains.std(axis=0)
```

## Evaluation signals

- nuts_posterior_samples.npy exists and loads as a JAX/numpy array with the expected (n_samples, n_par) shape
- Recovered marginal means fall within a few Fisher standard deviations of the fiducial Planck 18 best-fit values used for normalisation
- Recovered marginal standard deviations are consistent with the Fisher matrix prediction (par_cov diagonal) for a Gaussian posterior
- Denormalised constraints are stable across independent NUTS runs from the zero starting vector

## Limitations

- candl compatibility is version-sensitive (e.g. JAX version 0.4.31 noted as tested in the changelog), so chain reproduction may vary across JAX versions
- Marginal mean/std assumes an approximately Gaussian posterior; the Fisher normalisation makes this reasonable but not guaranteed for strongly non-Gaussian directions
- Since v1.5.1 default behaviour ignores candl internal priors, constraints extracted from chains may differ from earlier releases unless priors are re-enabled

## Evidence

- [other] Compute marginal constraints (mean and standard deviation) for each parameter from the denormalised chains and save the sample array to nuts_posterior_samples.npy.: [evidence span withheld — anchor status: not_found]
- [methods] Fisher normalisation grounding: "par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars"
- [abstract] This facilitates, for example, robust Fisher matrices without finite-difference methods.: "This facilitates, for example, robust Fisher matrices without finite-difference methods."
- [intro] Fisher matrices via JAX gradients: "JAX-compatibility, allowing for fast and easy computation of gradients and Hessians of the likelihoods."
- [discussion] candl compatibility caveats: "Compatibility with JAX version 0.4.31 (tested with TF 2.17.0)."
