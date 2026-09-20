---
name: autodiff-likelihood-fisher-matrix-computation
description: Use when you need cosmological parameter forecast errors from a CMB power-spectrum
  likelihood (TT/TE/EE, BB, phi-phi, kappa-kappa) and want exact Hessian-based Fisher
  matrices without finite differences.
when_to_use_negative:
- You need a full posterior rather than a Gaussian forecast — use gradient-based MCMC
  (e.g. BlackJAX) with the same differentiable likelihood instead
- The likelihood or theory pipeline is not JAX-compatible/differentiable (e.g. a non-differentiable
  external likelihood), in which case autodiff Fisher computation is unavailable and
  finite differences would be required
- 'You intend to switch off candl internal priors (clear_internal_priors: True) while
  still expecting them in the Fisher forecast — priors must be retained for the forecast
  to include them'
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: candl
  role: Differentiable python likelihood for CMB power spectra; provides tools.get_params_to_logl_func
    and tools.get_fisher_matrix used to build and differentiate the Fisher matrix
  repo: https://github.com/Lbalkenhol/candl
- name: JAX
  role: Powers candl's automatic differentiation, enabling gradients and Hessians
    of the likelihood for exact Fisher matrices
  repo: https://github.com/Lbalkenhol/candl
- name: CAMB
  role: Theory code interfaced via candl.interface.get_CAMB_pars_to_theory_specs_func
    to map parameters to CMB spectra including lensing
  repo: https://github.com/cmbant/CAMB
- name: candl_data
  role: Data package shipping the latest SPT and ACT primary and lensing power spectrum
    data (e.g. ACT DR6 lensing) for the likelihood
  repo: https://github.com/Lbalkenhol/candl_data
- name: CosmoPower-JAX
  role: Alternative JAX-native theory emulator interface mapping parameters to CMB
    spectra for the differentiable likelihood
  repo: https://github.com/dpiras/cosmopower-jax
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:dc31062d96b99ea95583d768e98b1256245f39195c63bef16b45512f43bec1f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lbalkenhol/candl
  tools:
  - CAMB
  - CosmoPower-JAX
  - JAX
  - Python
  - candl
  - candl_data
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# autodiff-likelihood-fisher-matrix-computation

## Summary

Compute a Fisher matrix (and hence parameter covariance and 1-sigma errors) by automatic differentiation of a JAX-powered CMB likelihood such as candl, avoiding finite-difference derivative noise entirely.

## When to use

Use when you need cosmological parameter forecast errors from a CMB power-spectrum likelihood (TT/TE/EE, BB, phi-phi, kappa-kappa) and want exact Hessian-based Fisher matrices without finite differences.

## When NOT to use

- You need a full posterior rather than a Gaussian forecast — use gradient-based MCMC (e.g. BlackJAX) with the same differentiable likelihood instead
- The likelihood or theory pipeline is not JAX-compatible/differentiable (e.g. a non-differentiable external likelihood), in which case autodiff Fisher computation is unavailable and finite differences would be required
- You intend to switch off candl internal priors (clear_internal_priors: True) while still expecting them in the Fisher forecast — priors must be retained for the forecast to include them

## Inputs

- candl likelihood object initialised from a .yaml file or index file (e.g. candl_data.ACT_DR6_Lens, spt_candl_data.SPT3G_D1_TnE)
- Theory-code interface function mapping a parameter dictionary to CMB spectra (CAMB via candl.interface.get_CAMB_pars_to_theory_specs_func, or CosmoPower-JAX)
- Fiducial parameter dictionary (Planck-18-style cosmological + nuisance parameters)
- candl internal priors retained in the likelihood definition

## Outputs

- Fisher matrix (parameter covariance from its inverse) with parameter ordering par_order
- 1-sigma parameter errors as sqrt(diag(covariance)) in par_order, written e.g. to act_dr6_pp_fisher.csv

## How to apply

Instantiate the candl likelihood for the data set of interest (e.g. candl.Like(candl_data.ACT_DR6_Lens) with lensing=True and the 'lens_only' variant), then wire a theory interface — candl.interface.get_CAMB_pars_to_theory_specs_func for CAMB or a CosmoPower-JAX interface — that maps a parameter dictionary to CMB spectra including the lensing power spectrum. Compose the parameters-to-log-likelihood function with candl.tools.get_params_to_logl_func(candl_like, pars_to_theory_specs), keeping candl's internal priors (do not set clear_internal_priors: True) so they enter the forecast. Compute the Fisher matrix via JAX automatic differentiation with candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, par_order=None, return_par_order=True) using a Planck-18-style fiducial parameter set — the differentiation is exact Hessian computation, so no finite-difference step sizes are involved. Invert the Fisher matrix, take sqrt(diag) of the covariance as 1-sigma errors, and report errors in the returned par_order ordering.

## Related tools

- **candl** (Differentiable python likelihood for CMB power spectra; provides tools.get_params_to_logl_func and tools.get_fisher_matrix used to build and differentiate the Fisher matrix) — https://github.com/Lbalkenhol/candl
- **JAX** (Powers candl's automatic differentiation, enabling gradients and Hessians of the likelihood for exact Fisher matrices) — https://github.com/Lbalkenhol/candl
- **CAMB** (Theory code interfaced via candl.interface.get_CAMB_pars_to_theory_specs_func to map parameters to CMB spectra including lensing) — https://github.com/cmbant/CAMB
- **candl_data** (Data package shipping the latest SPT and ACT primary and lensing power spectrum data (e.g. ACT DR6 lensing) for the likelihood) — https://github.com/Lbalkenhol/candl_data
- **CosmoPower-JAX** (Alternative JAX-native theory emulator interface mapping parameters to CMB spectra for the differentiable likelihood) — https://github.com/dpiras/cosmopower-jax

## Examples

```
import candl, candl_data; like = candl.Like(candl_data.ACT_DR6_Lens); pars_to_logl = candl.tools.get_params_to_logl_func(like, pars_to_theory_specs); par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, like, fid_pars, par_order=None, return_par_order=True)
```

## Evaluation signals

- Fisher matrix is symmetric (invariant expected from Hessian of a scalar log-likelihood)
- Output table lists one row per parameter in the returned par_order, each with finite, positive 1-sigma error
- No finite-difference step sizes are involved anywhere in the derivative calculation
- Parameter errors change appropriately when candl internal priors are kept versus cleared (clear_internal_priors: True), confirming priors entered the forecast
- Inverting the Fisher matrix yields a positive-definite parameter covariance

## Limitations

- JAX on some architectures lacks 64-bit precision, causing up to 1e-4 differences in log-likelihood computations (noted for clipy/candl-adjacent tooling)
- A breaking change in candl's default behaviour means candl internal priors are now ignored by default; users must consult docs to keep them
- Compatibility of candl v1.5.1 is only stated as tested with a single JAX/TensorFlow version combination (JAX 0.4.31, TF 2.17.0)

## Evidence

- [abstract] autodiff Fisher via tools.get_fisher_matrix: "This facilitates, for example, robust Fisher matrices without finite-difference methods."
- [intro] supported spectra include lensing: "candl supports the analysis of primary CMB and lensing power spectrum data (:math:`TT`, :math:`TE`, :math:`EE`, :math:`BB`, :math:`\phi\phi`, :math:`\kappa\kappa`)."
- [methods] Fisher call signature: "par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars"
- [abstract] JAX powers the differentiation: "candl is powered by JAX, which makes it fast and easy to calculate derivatives of the likelihood."
- [discussion] default prior behaviour changed: "__NOTE:__ Change in default behaviour: candl internal priors now ignored. Check docs for how to keep them"
- [readme] JAX 64-bit caveat: [evidence span withheld — anchor status: external_doc]
