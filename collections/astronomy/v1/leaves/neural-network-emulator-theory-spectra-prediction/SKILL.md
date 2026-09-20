---
name: neural-network-emulator-theory-spectra-prediction
description: Use when sampling or differentiating through a CMB likelihood requires
  fast, JAX-native theory spectra (e.g. gradient-based NUTS with BlackJAX) and a trained
  CosmoPower emulator for the target data set exists.
when_to_use_negative:
- The target parameter space or survey configuration has no pre-trained CosmoPower
  emulator — training a new emulator is a separate task
- Accurate Boltzmann-level spectra (e.g. via CAMB with candl.interface.get_CAMB_pars_to_theory_specs_func)
  are required and emulator accuracy is not acceptable
- The analysis pipeline is not JAX-based and CPU/GPU autodiff is unnecessary — a standard
  sampler with a non-differentiable theory code suffices
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: candl
  role: Provides candl.interface.get_CosmoPowerJAX_pars_to_theory_specs_func and candl.tools.get_params_to_logl_func
    to wrap the emulator output into a differentiable log-likelihood
  repo: github.com/Lbalkenhol/candl
- name: CosmoPower-JAX
  role: JAX reimplementation of CosmoPower neural-network emulators that evaluates
    TT/TE/EE theory spectra differentiably
  repo: https://github.com/dpiras/cosmopower-jax
- name: JAX
  role: Backend providing differentiation, jit compilation, and hardware acceleration
    for emulator evaluation and the composed likelihood
  repo: https://github.com/google/jax
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:4199c79e0910ddf0b2fd85f89ac9f8acbdcca5545a3986066c1e6396483c2ee7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BlackJAX
  - CosmoPower-JAX
  - JAX
  - Python
  - candl
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# neural-network-emulator-theory-spectra-prediction

## Summary

Replace an expensive Boltzmann-solver call with a pre-trained neural-network emulator (CosmoPower TT NN, TE PCAplusNN, EE NN) evaluated in JAX via CosmoPower-JAX, and wire it into candl's likelihood through candl.interface.get_CosmoPowerJAX_pars_to_theory_specs_func so cosmological parameters map directly to differentiable CMB power spectra.

## When to use

Use when sampling or differentiating through a CMB likelihood requires fast, JAX-native theory spectra (e.g. gradient-based NUTS with BlackJAX) and a trained CosmoPower emulator for the target data set exists.

## When NOT to use

- The target parameter space or survey configuration has no pre-trained CosmoPower emulator — training a new emulator is a separate task
- Accurate Boltzmann-level spectra (e.g. via CAMB with candl.interface.get_CAMB_pars_to_theory_specs_func) are required and emulator accuracy is not acceptable
- The analysis pipeline is not JAX-based and CPU/GPU autodiff is unnecessary — a standard sampler with a non-differentiable theory code suffices

## Inputs

- Pre-trained CosmoPower emulator models (TT NN, TE PCAplusNN, EE NN) for the SPT-3G 2018 TT/TE/EE data set
- Cosmological parameter vector (e.g. H0, ombh2, omch2, logA, ns, tau, yp)
- candl.Like likelihood instance initialised from candl_data.SPT3G_2018_TTTEEE

## Outputs

- JAX-differentiable parameters-to-theory-spectra function (CMB TT/TE/EE power spectra)
- Single parameters-to-log-likelihood function suitable for jax.jit, gradients, and MCMC

## How to apply

Load the pre-trained CosmoPower emulator models for the target data set (for the SPT-3G 2018 example: the TT neural network, the TE PCAplusNN model, and the EE neural network) into CosmoPower-JAX, which makes the emulator evaluation differentiable in JAX. Build the parameters-to-spectra function with candl.interface.get_CosmoPowerJAX_pars_to_theory_specs_func, then compose it with the candl likelihood via candl.tools.get_params_to_logl_func to obtain a single parameters-to-log-likelihood function. Optionally compute the Fisher matrix with candl.tools.get_fisher_matrix at a fiducial point (e.g. Planck 18 best fit: H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0) to normalise parameters to zero mean and unit variance before sampling. The rationale is that emulator evaluation inside JAX keeps the whole likelihood differentiable, enabling gradients, Hessians, and Fisher matrices without finite differences.

## Related tools

- **candl** (Provides candl.interface.get_CosmoPowerJAX_pars_to_theory_specs_func and candl.tools.get_params_to_logl_func to wrap the emulator output into a differentiable log-likelihood) — github.com/Lbalkenhol/candl
- **CosmoPower-JAX** (JAX reimplementation of CosmoPower neural-network emulators that evaluates TT/TE/EE theory spectra differentiably) — https://github.com/dpiras/cosmopower-jax
- **JAX** (Backend providing differentiation, jit compilation, and hardware acceleration for emulator evaluation and the composed likelihood) — https://github.com/google/jax

## Examples

```
import candl, candl_data; like = candl.Like(candl_data.SPT3G_2018_TTTEEE); pars_to_specs = candl.interface.get_CosmoPowerJAX_pars_to_theory_specs_func(cp_model); pars_to_logl = candl.tools.get_params_to_logl_func(like, pars_to_specs)
```

## Evaluation signals

- At the fiducial parameter point (Planck 18 best fit), the emulated spectra fed into candl yield a log-likelihood consistent with the expected chi-squared near its minimum
- The composed parameters-to-logl function is jax.jit-compilable and differentiable (jax.grad returns finite values for all emulator-supplied parameters)
- Fisher matrix from candl.tools.get_fisher_matrix is finite and positive-definite at the fiducial point
- Posterior marginal means from NUTS sampling with the emulated likelihood fall within a few Fisher standard deviations of the fiducial values

## Limitations

- Emulator accuracy is bounded by the training set; spectra outside the emulator's training domain may bias the likelihood
- JAX 64-bit precision must be available; on architectures without it, log-likelihood differences up to the 1e-4 level can arise (documented for the analogous clipy setup)
- Version compatibility matters — candl's changelog notes compatibility tied to specific JAX versions (e.g. JAX 0.4.31)

## Evidence

- [other] Load the CosmoPower SPT high-accuracy emulator models (TT NN, TE PCAplusNN, EE NN) into CosmoPower-JAX and build the parameters-to-spectra function with candl.interface.get_CosmoPowerJAX_pars_to_theory_specs_func: [evidence span withheld — anchor status: not_found]
- [methods] get_params_to_logl_func: "like = candl.tools.get_params_to_logl_func(candl_like, pars_to_theory_specs)"
- [intro] JAX-compatibility enabling differentiable emulator-based likelihoods: "JAX-compatibility, allowing for fast and easy computation of gradients and Hessians of the likelihoods."
- [methods] get_fisher_matrix at fiducial parameters: "par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars"
- [abstract] candl is powered by JAX, which makes it fast and easy to calculate derivatives of the likelihood.: "candl is powered by JAX, which makes it fast and easy to calculate derivatives of the likelihood."
