---
name: fisher-matrix-parameter-normalisation
description: Use when you need a parameter covariance estimate for a differentiable
  likelihood (e.g. SPT-3G 2018 TT/TE/EE via candl) without finite differences, or
  when preparing a normalised parameter space for a gradient-based sampler such as
  BlackJAX NUTS.
when_to_use_negative:
- The likelihood is not differentiable/JAX-compatible and you would have to fall back
  to finite-difference derivatives
- You need full posterior contours rather than a Gaussian Fisher approximation (use
  MCMC instead)
- The theory pipeline is non-JAX (e.g. standard CAMB without candl's differentiable
  interface), where the differentiable Fisher computation does not apply
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: candl
  role: Differentiable likelihood framework; candl.tools.get_fisher_matrix computes
    the Fisher matrix from automatic derivatives of the likelihood
  repo: https://github.com/Lbalkenhol/candl
- name: JAX
  role: Provides the automatic gradients/Hessians of the likelihood that make the
    Fisher matrix computation fast and finite-difference-free
- name: CosmoPower-JAX
  role: Supplies the differentiable parameters-to-spectra function that is plugged
    into the Fisher computation
  repo: https://github.com/dpiras/cosmopower-jax
- name: BlackJAX
  role: Consumes the Fisher-normalised parameter space as inverse mass matrix / normalisation
    for NUTS sampling
  repo: https://github.com/blackjax-devs/blackjax
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:9ee9e552380bb3fb0255c9e5a9a240d386637509a265d5318a0c5b34ee4feb2a
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# fisher-matrix-parameter-normalisation

## Summary

Compute the Fisher matrix of a JAX-differentiable CMB likelihood with candl.tools.get_fisher_matrix and use its square root (parameter scales) to normalise the parameter vector to zero mean and unit variance. This whitening is the standard preconditioning step that makes gradient-based sampling of the likelihood numerically well-behaved.

## When to use

Use when you need a parameter covariance estimate for a differentiable likelihood (e.g. SPT-3G 2018 TT/TE/EE via candl) without finite differences, or when preparing a normalised parameter space for a gradient-based sampler such as BlackJAX NUTS.

## When NOT to use

- The likelihood is not differentiable/JAX-compatible and you would have to fall back to finite-difference derivatives
- You need full posterior contours rather than a Gaussian Fisher approximation (use MCMC instead)
- The theory pipeline is non-JAX (e.g. standard CAMB without candl's differentiable interface), where the differentiable Fisher computation does not apply

## Inputs

- candl.Like likelihood instance initialised from a .yaml file (e.g. candl_data.SPT3G_2018_TTTEEE)
- parameters-to-theory-spectra function (e.g. candl.interface.get_CosmoPowerJAX_pars_to_theory_specs_func over TT NN, TE PCAplusNN, EE NN emulators)
- fiducial parameter vector and ordering (e.g. Planck 18 best-fit values for H0, ombh2, omch2, logA, ns, tau, yp)

## Outputs

- Parameter covariance matrix par_cov (inverse of the Fisher matrix)
- Parameter ordering par_order
- Per-parameter Fisher standard deviations used as normalisation scales
- Zero-mean, unit-variance normalised parameter space for downstream sampling

## How to apply

Initialise the candl likelihood (e.g. candl_data.SPT3G_2018_TTTEEE), supply a parameters-to-theory-spectra function (e.g. a CosmoPower-JAX emulator wrapped with candl.tools.get_params_to_logl_func), and choose a fiducial parameter point — in the paper's example the Planck 18 best fit (H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0). Call candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars) which returns the parameter covariance par_cov and the parameter ordering par_order; because candl is JAX-differentiable this uses automatic derivatives rather than finite differences, giving a robust Fisher matrix. Invert par_cov to obtain the Fisher matrix itself and use the Fisher standard deviations as the scale vector to map the physical parameter vector to zero mean and unit variance (and back, when denormalising posterior samples). Verify by checking that the recovered marginal means of downstream samples fall within a few Fisher standard deviations of the fiducial values used for normalisation.

## Related tools

- **candl** (Differentiable likelihood framework; candl.tools.get_fisher_matrix computes the Fisher matrix from automatic derivatives of the likelihood) — https://github.com/Lbalkenhol/candl
- **JAX** (Provides the automatic gradients/Hessians of the likelihood that make the Fisher matrix computation fast and finite-difference-free)
- **CosmoPower-JAX** (Supplies the differentiable parameters-to-spectra function that is plugged into the Fisher computation) — https://github.com/dpiras/cosmopower-jax
- **BlackJAX** (Consumes the Fisher-normalised parameter space as inverse mass matrix / normalisation for NUTS sampling) — https://github.com/blackjax-devs/blackjax

## Examples

```
par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars)
```

## Evaluation signals

- par_cov returned by candl.tools.get_fisher_matrix must be symmetric and positive definite
- par_order must match the parameter names of the fiducial vector passed in
- Marginal means of posterior samples denormalised with the Fisher scales should fall within a few Fisher standard deviations of the fiducial Planck 18 best-fit values
- Fisher constraint sizes should be consistent with published SPT-3G 2018 TT/TE/EE marginalised uncertainties
- Normalised parameter vector should have zero mean and unit variance by construction

## Limitations

- Fisher matrices are Gaussian approximations; they characterise constraints at the chosen fiducial point only
- The fiducial point choice (e.g. the Planck 18 best fit) directly sets the normalisation scales; a poor fiducial choice degrades the whitening
- On some architectures JAX 64-bit precision may be unavailable, producing small (~1e-4) differences in likelihood values that propagate to derivatives

## Evidence

- [abstract] This facilitates, for example, robust Fisher matrices without finite-difference methods.: "This facilitates, for example, robust Fisher matrices without finite-difference methods."
- [methods] get_fisher_matrix API call: "par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars"
- [intro] JAX-compatibility enabling gradient and Hessian computation: "JAX-compatibility, allowing for fast and easy computation of gradients and Hessians of the likelihoods."
- [methods] Fisher matrix use in the NUTS workflow: [evidence span withheld — anchor status: not_found]
