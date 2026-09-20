---
name: neural-network-emulator-likelihood-pipelining
description: Use when you need a parameters-to-log-likelihood function for CMB power-spectrum
  analyses (e.g. SPT-3G 2018 TT/TE/EE) that is end-to-end differentiable with JAX,
  avoiding slow Boltzmann solvers via a trained emulator.
when_to_use_negative:
- When the theory model cannot be emulated accurately by a trained neural-network
  emulator (no validated CosmoPower-JAX models for the parameter space) — use a Boltzmann
  solver interface such as candl.interface.get_CAMB_pars_to_theory_specs_func instead
- When the analysis requires exact Boltzmann-solver spectra rather than emulator-approximated
  C_l, since emulator truncation errors propagate into the likelihood
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: candl
  role: Provides the differentiable CMB likelihood and the tools to compose parameters-to-log-likelihood
    functions and Fisher matrices
  repo: github.com/Lbalkenhol/candl
- name: JAX
  role: Powers automatic differentiation of the emulated likelihood chain (gradients
    and Hessians)
- name: CosmoPower-JAX
  role: Neural-network emulator supplying differentiable parameters-to-theory-spectra
    mapping
  repo: https://github.com/dpiras/cosmopower-jax
- name: pip
  role: Installation channel for candl (pip install candl-like) and the data repository
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:7238c5cca59f9ed9234a7665103c5e82ed1f42846ff2f3b17579e65eca0a81ae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CosmoPower-JAX
  - JAX
  - Python
  - candl
  - pip
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# neural-network-emulator-likelihood-pipelining

## Summary

Chains a differentiable neural-network emulator (CosmoPower-JAX) into a differentiable CMB likelihood (candl) so that a single JAX-differentiable function maps cosmological parameters to a log-likelihood, enabling Hessian-based Fisher forecasts and gradient-based sampling.

## When to use

Use when you need a parameters-to-log-likelihood function for CMB power-spectrum analyses (e.g. SPT-3G 2018 TT/TE/EE) that is end-to-end differentiable with JAX, avoiding slow Boltzmann solvers via a trained emulator.

## When NOT to use

- When the theory model cannot be emulated accurately by a trained neural-network emulator (no validated CosmoPower-JAX models for the parameter space) — use a Boltzmann solver interface such as candl.interface.get_CAMB_pars_to_theory_specs_func instead
- When the analysis requires exact Boltzmann-solver spectra rather than emulator-approximated C_l, since emulator truncation errors propagate into the likelihood

## Inputs

- candl likelihood index .yaml file (e.g. candl_data.SPT3G_2018_TTTEEE)
- Trained CosmoPower-JAX emulator files (TT: cmb_spt_TT_NN, TE: cmb_spt_TE_PCAplusNN, EE: cmb_spt_EE_NN) placed in cosmopower_jax/trained_models
- Fiducial cosmological parameter dictionary (H0, ombh2, omch2, logA, ns, tau, yp)

## Outputs

- Differentiable parameters-to-log-likelihood Python function
- Log-likelihood values and JAX gradients/Hessians at any parameter point
- Fisher matrix covariance (par_cov) and parameter ordering (par_order) via candl.tools.get_fisher_matrix

## How to apply

First train or obtain CosmoPower-JAX emulators for the desired spectra and place the trained models (e.g. cmb_spt_TT_NN for TT, cmb_spt_TE_PCAplusNN for TE, cmb_spt_EE_NN for EE) in cosmopower_jax/trained_models. Initialise the likelihood by pointing candl.Like at the relevant index .yaml file, e.g. candl_data.SPT3G_2018_TTTEEE. Build the parameters-to-theory-spectra mapping with candl.interface.get_CosmoPowerJAX_pars_to_theory_specs_func using the emulator filenames per spectrum. Assemble the differentiable parameters-to-log-likelihood function with candl.tools.get_params_to_logl_func(candl_like, pars_to_theory_specs); because both the emulator and the likelihood are JAX-compatible, the whole chain supports gradients and Hessians. Evaluate the chain at a fiducial parameter vector (e.g. H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0) before downstream use (Fisher matrices, gradient-based MCMC via BlackJAX, or optimisation with Optax).

## Related tools

- **candl** (Provides the differentiable CMB likelihood and the tools to compose parameters-to-log-likelihood functions and Fisher matrices) — github.com/Lbalkenhol/candl
- **JAX** (Powers automatic differentiation of the emulated likelihood chain (gradients and Hessians))
- **CosmoPower-JAX** (Neural-network emulator supplying differentiable parameters-to-theory-spectra mapping) — https://github.com/dpiras/cosmopower-jax
- **pip** (Installation channel for candl (pip install candl-like) and the data repository)

## Examples

```
pars_to_theory_specs = candl.interface.get_CosmoPowerJAX_pars_to_theory_specs_func(candl_like); pars_to_logl = candl.tools.get_params_to_logl_func(candl_like, pars_to_theory_specs); par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, par_order=None, return_par_order=True)
```

## Evaluation signals

- The Fisher covariance par_cov returned by candl.tools.get_fisher_matrix must be symmetric positive semi-definite
- Square-root diagonal entries of par_cov must give parameter uncertainties consistent with those quoted for the SPT-3G 2018 TT/TE/EE analysis
- The assembled parameters-to-log-likelihood function must evaluate at the fiducial parameter dictionary without error and be JAX-differentiable (grad/hessian calls succeed)
- Emulator outputs must be validated against reference spectra before being fed into the likelihood chain

## Limitations

- JAX on some architectures lacks 64-bit support, translating into up to 1e-4 differences in log-likelihood computations (noted for clipy, a sibling JAX-compatible likelihood package)
- candl v1.5.1 compatibility was only stated as tested with a single TensorFlow version (TF 2.17.0) / JAX 0.4.31, so emulator-likelihood chains may be version-sensitive
- A change in default behaviour means candl internal priors are now ignored, which affects the parameter-to-log-likelihood chain unless priors are re-enabled

## Evidence

- [methods] Workflow step: build emulator-to-likelihood mapping: [evidence span withheld — anchor status: not_found]
- [abstract] Fisher matrix without finite differences: "This facilitates, for example, robust Fisher matrices without finite-difference methods."
- [intro] JAX gradients and Hessians: "JAX-compatibility, allowing for fast and easy computation of gradients and Hessians of the likelihoods."
- [methods] Likelihood initialisation from yaml: "It simply suffices to point to the ``.yaml`` file of a likelihood to initialise it."
- [readme] candl pip-installable: "`candl` can be installed with pip::

    pip install candl-like"
