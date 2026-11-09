---
name: gradient-based-nuts-mcmc-sampling
description: Use when you need posterior samples from a differentiable CMB power-spectrum
  likelihood (e.g. candl SPT-3G 2018 TT/TE/EE) and want NUTS sampling driven by exact
  JAX gradients of the log-likelihood.
when_to_use_negative:
- When the likelihood is not JAX-differentiable (e.g. clik-based Planck likelihoods
  without clipy/JAX support) — NUTS requires exact gradients that cannot be obtained.
- When you only need parameter constraints and a gradient-free sampler via Cobaya
  or MontePython suffices, especially if the theory pipeline is not differentiable.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: candl
  role: Provides the automatically differentiable CMB likelihood and the tools (get_params_to_logl_func,
    get_fisher_matrix) that build the log-likelihood function and normalisation scales
  repo: github.com/Lbalkenhol/candl
- name: BlackJAX
  role: Supplies the gradient-based NUTS kernel (blackjax.nuts) used to sample the
    normalised log-likelihood
  repo: https://github.com/blackjax-devs/blackjax
- name: JAX
  role: Powers automatic differentiation, jax.jit compilation, and the jax.lax.scan-based
    inference loop
- name: CosmoPower-JAX
  role: Runs the differentiable emulator mapping cosmological parameters to TT/TE/EE
    spectra inside the sampling loop
  repo: https://github.com/dpiras/cosmopower-jax
- name: Python
  role: Host language for the pipeline; candl is installed via pip (pip install candl-like)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:e03cc733a68f329d4b2a89576b5534dd7af4410331285230dfc70a6bf0da2387
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BlackJAX
  - CosmoPower-JAX
  - JAX
  - Python
  - candl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# gradient-based-nuts-mcmc-sampling

## Summary

Runs gradient-based Hamiltonian Monte Carlo (BlackJAX NUTS) over a JAX-differentiable CMB likelihood (candl) driven by a CosmoPower-JAX emulator, exploiting exact likelihood gradients instead of finite-difference or gradient-free samplers.

## When to use

Use when you need posterior samples from a differentiable CMB power-spectrum likelihood (e.g. candl SPT-3G 2018 TT/TE/EE) and want NUTS sampling driven by exact JAX gradients of the log-likelihood.

## When NOT to use

- When the likelihood is not JAX-differentiable (e.g. clik-based Planck likelihoods without clipy/JAX support) — NUTS requires exact gradients that cannot be obtained.
- When you only need parameter constraints and a gradient-free sampler via Cobaya or MontePython suffices, especially if the theory pipeline is not differentiable.

## Inputs

- candl.Like likelihood object initialised from candl_data.SPT3G_2018_TTTEEE (.yaml likelihood)
- CosmoPower SPT emulator models (TT NN, TE PCAplusNN, EE NN) loaded into CosmoPower-JAX
- Fiducial parameter vector (Planck 18 best-fit: H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0)
- NUTS settings: step_size=0.1, inverse_mass_matrix=np.ones(n_par), number of samples (e.g. 100)

## Outputs

- Array of NUTS posterior samples in physical parameter units, saved as nuts_posterior_samples.npy
- Marginal constraints (mean and standard deviation) per cosmological parameter
- Fisher covariance matrix and parameter ordering from candl.tools.get_fisher_matrix (used for normalisation)

## How to apply

Initialise the likelihood with candl.Like via the candl_data.SPT3G_2018_TTTEEE shortcut, then build a parameters-to-spectra function from the CosmoPower high-accuracy emulators (TT NN, TE PCAplusNN, EE NN) loaded into CosmoPower-JAX using candl.interface.get_CosmoPowerJAX_pars_to_theory_specs_func, and wrap it with candl.tools.get_params_to_logl_func into a single parameters-to-log-likelihood function. Compute the Fisher matrix at the Planck 18 best-fit point (H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0) with candl.tools.get_fisher_matrix, and use its eigenvectors/scales to normalise parameters to zero mean and unit variance. Construct a jax.jit-compiled normalised log-likelihood, define blackjax.nuts(step_size=0.1, inverse_mass_matrix=np.ones(n_par)), and run a jax.lax.scan-based inference loop from a zero starting vector to draw e.g. 100 NUTS samples. Finally denormalise the chains back to physical units using the Fisher scales, compute marginal means and standard deviations, and save the samples to nuts_posterior_samples.npy. The rationale is that JAX-compatibility of candl allows fast and easy computation of gradients of the likelihood, which NUTS requires.

## Related tools

- **candl** (Provides the automatically differentiable CMB likelihood and the tools (get_params_to_logl_func, get_fisher_matrix) that build the log-likelihood function and normalisation scales) — github.com/Lbalkenhol/candl
- **BlackJAX** (Supplies the gradient-based NUTS kernel (blackjax.nuts) used to sample the normalised log-likelihood) — https://github.com/blackjax-devs/blackjax
- **JAX** (Powers automatic differentiation, jax.jit compilation, and the jax.lax.scan-based inference loop)
- **CosmoPower-JAX** (Runs the differentiable emulator mapping cosmological parameters to TT/TE/EE spectra inside the sampling loop) — https://github.com/dpiras/cosmopower-jax
- **Python** (Host language for the pipeline; candl is installed via pip (pip install candl-like))

## Examples

```
import candl, blackjax, jax; like = candl.Like('SPT3G_2018_TTTEEE'); logl = candl.tools.get_params_to_logl_func(like, pars_to_theory_specs); kernel = blackjax.nuts(logl, step_size=0.1, inverse_mass_matrix=np.ones(7))
```

## Evaluation signals

- nuts_posterior_samples.npy exists and loads as a JAX/numpy array with the expected number of parameters (7: H0, ombh2, omch2, logA, ns, tau, yp).
- Recovered marginal means for each parameter fall within a few Fisher standard deviations of the fiducial Planck 18 best-fit values used for normalisation.
- The Fisher matrix computed via candl.tools.get_fisher_matrix succeeds without finite-difference methods, confirming gradient correctness.
- The normalised parameter space has approximately zero mean and unit variance at the fiducial point before sampling.
- The inference loop runs under jax.jit/jax.lax.scan without recompilation errors, indicating the full likelihood chain is JAX-traceable.

## Limitations

- Compatibility with JAX version 0.4.31 was only tested against a single TensorFlow version (TF 2.17.0); version drift may break the differentiable pipeline.
- On some architectures JAX 64-bit computation is unavailable, translating into up to 1e-4 differences in log-likelihood computations (noted for clipy/candl-like likelihoods).
- candl's default behaviour changed so internal priors are now ignored by default; check docs for how to keep them, otherwise posteriors may be unexpectedly prior-free.

## Evidence

- [abstract] gradient-based MCMC: "These also include optimising the band power bin width to minimise parameter errors of a realistic mock data set."
- [intro] JAX gradients: "JAX-compatibility, allowing for fast and easy computation of gradients and Hessians of the likelihoods."
- [methods] BlackJAX repo: "`BlackJAX <https://github.com/blackjax-devs/blackjax>`__ gradient-based samplers for JAX."
- [methods] likelihood wiring: "like = candl.tools.get_params_to_logl_func(candl_like, pars_to_theory_specs)"
- [methods] Fisher tool: "par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars"
- [abstract] Fisher without finite differences: "This facilitates, for example, robust Fisher matrices without finite-difference methods."
