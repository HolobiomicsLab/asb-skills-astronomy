---
name: likelihood-chisq-evaluation
description: Use when a mock or real band power vector (e.g. SPT-3G 2018 TT/TE/EE)
  must be scored against a forward model at fixed cosmological parameters (H0=67.37,
  ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0) to obtain
  a chi-squared/log-likelihood value.
when_to_use_negative:
- When parameters must be inferred rather than fixed — use gradient-based MCMC sampling
  or Fisher matrix tools (candl.tools.get_fisher_matrix) instead of a single-point
  chi-squared evaluation
- When the data are not CMB power spectrum band powers (e.g. raw time-stream or map
  data) — candl operates on TT, TE, EE, BB, phi-phi, kappa-kappa power spectrum measurements
- When working with Planck clik likelihood files outside candl — use clipy, which
  is the pure Python implementation of the 2018 Planck likelihoods
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3394
tools:
- name: candl
  role: Differentiable likelihood that takes CMB spectra and parameter values and
    returns a log-likelihood value; provides candl.tools.get_params_to_logl_func for
    the params-to-bandpowers forward model and chi-squared evaluation
  repo: github.com/Lbalkenhol/candl
- name: JAX
  role: Powers candl's automatic differentiation, enabling fast and easy computation
    of likelihood derivatives during evaluation
  repo: https://github.com/google/jax
- name: Python
  role: Implementation language for the likelihood evaluation, mock data generation
    (NumPy random generator), and text output
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:7ce69750b7123b78d70fb93bb233b19af7b2841c4c5100890ba911d3976c4a3d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JAX
  - Python
  - candl
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3394
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# likelihood-chisq-evaluation

## Summary

Evaluate the chi-squared (log-likelihood) of a CMB band power data vector at a fixed fiducial cosmology using a candl likelihood, e.g. as the validation step after generating a mock data vector from the likelihood's covariance matrix.

## When to use

Use when a mock or real band power vector (e.g. SPT-3G 2018 TT/TE/EE) must be scored against a forward model at fixed cosmological parameters (H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0) to obtain a chi-squared/log-likelihood value.

## When NOT to use

- When parameters must be inferred rather than fixed — use gradient-based MCMC sampling or Fisher matrix tools (candl.tools.get_fisher_matrix) instead of a single-point chi-squared evaluation
- When the data are not CMB power spectrum band powers (e.g. raw time-stream or map data) — candl operates on TT, TE, EE, BB, phi-phi, kappa-kappa power spectrum measurements
- When working with Planck clik likelihood files outside candl — use clipy, which is the pure Python implementation of the 2018 Planck likelihoods

## Inputs

- candl likelihood initialised from a data set info .yaml index file (e.g. candl_data.SPT3G_2018_TTTEEE)
- Band power data vector in the spectrum ordering declared in spectra_info (e.g. mock_band_powers.txt)
- Covariance matrix in spectrum-major order shipped with the likelihood
- Fiducial cosmological parameter set (H0, ombh2, omch2, logA, ns, tau, yp)

## Outputs

- Chi-squared / log-likelihood value of the data vector at the fiducial cosmology
- Forward-modelled band power vector at the fiducial parameters

## How to apply

Initialise the candl likelihood from the candl_data.SPT3G_2018_TTTEEE index file, which loads band powers, the spectrum-major-order covariance, window functions, and data-model transformations declared in the data set info .yaml file. Build the forward model by constructing the long vector of CMB-only spectra at the fiducial parameters, binning with the window functions, and applying the data-model transformations successively ('Loop over the transformations in the data model and apply them one by one to the model vector'). Compute the chi-squared by taking the difference between data and model band powers and evaluating the log-likelihood through candl (optionally via candl.tools.get_params_to_logl_func, which returns a parameters-to-log-likelihood function powered by JAX). For a mock data vector drawn as a multivariate Gaussian realisation of the covariance at the fiducial cosmology, the resulting chi-squared should be statistically consistent with the data vector length, so re-evaluating at the same point must reproduce the value with no >0.1% deviation.

## Related tools

- **candl** (Differentiable likelihood that takes CMB spectra and parameter values and returns a log-likelihood value; provides candl.tools.get_params_to_logl_func for the params-to-bandpowers forward model and chi-squared evaluation) — github.com/Lbalkenhol/candl
- **JAX** (Powers candl's automatic differentiation, enabling fast and easy computation of likelihood derivatives during evaluation) — https://github.com/google/jax
- **Python** (Implementation language for the likelihood evaluation, mock data generation (NumPy random generator), and text output)

## Examples

```
import candl; like = candl.Likelihood('candl_data/SPT3G_2018_TTTEEE/.../info.yaml'); logl = like.logl(mock_band_powers, fid_pars)
```

## Evaluation signals

- The mock band power file exists and its number of rows matches the bin counts declared in spectra_info
- The chi-squared at the fiducial cosmology is consistent with the expected value for a Gaussian realisation of a data vector of that length
- Re-evaluating the likelihood at the fiducial point reproduces the recorded chi-squared with no >0.1% deviation (deterministic forward model)
- The data vector ordering in mock_band_powers.txt matches the spectrum ordering declared in spectra_info

## Limitations

- On some architectures JAX 64-bit computation may be unavailable, introducing up to ~1e-4 differences in log-likelihood values (noted for clipy/candl-style likelihoods)
- The forward model requires a theory pipeline (e.g. CAMB) supplying spectra at the fiducial parameters; chi-squared validity depends on correct spectrum-major covariance ordering and window-function binning
- Chi-squared evaluation at a single point does not itself constrain parameters; it is a validation/diagnostic step, e.g. before bin-width optimisation of a mock data set

## Evidence

- [methods] Forward model: construct long spectra vector, bin with window functions, apply data-model transformations: "#. Bin the model spectra using the window functions."
- [methods] Data-model transformations applied successively before the likelihood difference: "Loop over the transformations in the data model and apply them one by one to the model vector."
- [methods] Chi-squared is computed from the data-model band power difference: "#. Take the difference between data and model band powers and calculate the log likelihood value."
- [methods] Likelihood interface: a function of CMB spectra and parameters returning a log-likelihood value: "The likelihood is a function that takes a set of CMB spectra and parameter values returns a log-likelihood value."
- [intro] Supports primary CMB and lensing power spectra, defining the applicable data types: [evidence span withheld — anchor status: not_found]
