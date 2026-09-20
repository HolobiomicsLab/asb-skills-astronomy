---
name: spectrum-binning-with-window-functions
description: Use when a forward model produces unbinned C_ell spectra that must be
  projected onto the binned band powers of a candl likelihood (e.g. SPT-3G 2018 TTTEEE),
  or when generating mock data vectors for forecasting or bin-width optimisation.
when_to_use_negative:
- The input data are already binned band powers from a likelihood whose window-function
  binning has already been applied downstream (binning twice would double-count mode
  weighting)
- The analysis targets unbinned (per-multipole) likelihoods where a window-function
  binning step is not part of the data model
- The likelihood is not a candl-style power-spectrum likelihood (e.g. low-ell commander-style
  likelihoods handled via clipy), since the binned band-power/vector ordering contract
  does not apply
tools:
- name: candl
  role: Provides the likelihood pipeline that bins model spectra with the data set's
    window functions and applies data model transformations
  repo: https://github.com/Lbalkenhol/candl
- name: JAX
  role: Powers the differentiable likelihood so the binned forward model can be differentiated
    for Fisher matrices and gradient-based sampling
  repo: https://github.com/google/jax
- name: candl_data.SPT3G_2018_TTTEEE
  role: Index file supplying the SPT-3G 2018 TTTEEE band powers, covariance, and window
    functions used as a concrete binning example
  repo: https://github.com/Lbalkenhol/candl_data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:d6de42f5edb4fab4d7daaa18bff6dfcce68e6792ea8c0916e1d05b8dd6e4ff27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lbalkenhol/candl
  tools:
  - JAX
  - Python
  - candl
  - candl_data.SPT3G_2018_TTTEEE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# Spectrum binning with window functions

## Summary

Convert long vectors of model CMB spectra (TT/TE/EE/BB etc.) into the binned band-power representation of a given likelihood by applying the data set's window functions, as implemented in candl's data model pipeline. This is a prerequisite step for computing data-model chi-squared and generating realistic mock band powers.

## When to use

Use when a forward model produces unbinned C_ell spectra that must be projected onto the binned band powers of a candl likelihood (e.g. SPT-3G 2018 TTTEEE), or when generating mock data vectors for forecasting or bin-width optimisation.

## When NOT to use

- The input data are already binned band powers from a likelihood whose window-function binning has already been applied downstream (binning twice would double-count mode weighting)
- The analysis targets unbinned (per-multipole) likelihoods where a window-function binning step is not part of the data model
- The likelihood is not a candl-style power-spectrum likelihood (e.g. low-ell commander-style likelihoods handled via clipy), since the binned band-power/vector ordering contract does not apply

## Inputs

- Long vector of unbinned CMB model spectra (e.g. CMB-only TT/TE/EE at fiducial parameters) from a theory code such as CAMB
- candl likelihood object initialised from a data set index .yaml file (e.g. candl_data.SPT3G_2018_TTTEEE) providing window functions and spectra_info bin counts
- Optional data model transformation dictionary from the data set info .yaml

## Outputs

- Binned model band-power vector ordered according to spectra_info spectrum ordering
- Transformed model vector after successive data model transformations, ready for chi-squared evaluation
- Optionally, a mock band-power text file (mock_band_powers.txt) with rows matching spectra_info bin counts

## How to apply

Initialise the likelihood by pointing to the data set's index .yaml file (e.g. candl_data.SPT3G_2018_TTTEEE), which loads the band powers, covariance in spectrum-major order, and window functions declared in spectra_info. Construct the long vector of theory spectra at the chosen parameters (e.g. the fiducial cosmology H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0), then bin the model spectra using the window functions so each band power is the appropriate weighted sum over the fine-grained C_ell vector. After binning, loop over the transformations in the data model and apply them one by one to the model vector, successively, so the model lands in the same space as the observed band powers. The binned (and transformed) model vector is then differenced against the data band powers to compute the log-likelihood/chi-squared, or used as the mean of a multivariate Gaussian draw (with the likelihood's covariance matrix) to write a mock_band_powers.txt file ordered per spectra_info.

## Related tools

- **candl** (Provides the likelihood pipeline that bins model spectra with the data set's window functions and applies data model transformations) — https://github.com/Lbalkenhol/candl
- **JAX** (Powers the differentiable likelihood so the binned forward model can be differentiated for Fisher matrices and gradient-based sampling) — https://github.com/google/jax
- **candl_data.SPT3G_2018_TTTEEE** (Index file supplying the SPT-3G 2018 TTTEEE band powers, covariance, and window functions used as a concrete binning example) — https://github.com/Lbalkenhol/candl_data

## Examples

```
like = candl.tools.get_params_to_logl_func(candl_like, pars_to_theory_specs)
```

## Evaluation signals

- The binned model vector length equals the total bin count summed over the spectra declared in spectra_info
- A mock data vector drawn from the forward-modelled band powers and written to mock_band_powers.txt has the expected number of rows matching spectra_info bin counts and spectrum ordering
- Re-evaluating the chi-squared of the same data vector at the fiducial cosmology reproduces the recorded value with no >0.1% deviation
- Chi-squared at the fiducial cosmology for a Gaussian mock realisation is consistent with the expectation for a realisation of that data vector length given the likelihood's covariance matrix

## Limitations

- JAX support may not be available in 64-bit mode on some architectures, which the clipy README notes can translate into up to 1e-4 differences in log-likelihood computations (relevant when validating chi-squared values near thresholds)
- candl requires the spectrum-major covariance ordering and spectra_info-declared ordering to match the binned vector, so mis-ordered outputs silently break likelihood evaluation
- Data model transformations must be applied in the order given in the data model; applying them out of sequence is not validated automatically

## Evidence

- [methods] Binning is the explicit second step of the candl likelihood evaluation: "#. Bin the model spectra using the window functions."
- [methods] Transformations are applied successively after binning: "Loop over the transformations in the data model and apply them one by one to the model vector."
- [methods] The binned model feeds the chi-squared/log-likelihood computation: "#. Take the difference between data and model band powers and calculate the log likelihood value."
- [abstract] Mock data set generation (forecasting, bin-width optimisation) is a motivating use case: "These also include optimising the band power bin width to minimise parameter errors of a realistic mock data set."
- [methods] Initialisation only requires pointing to the likelihood's .yaml file: "It simply suffices to point to the ``.yaml`` file of a likelihood to initialise it."
