---
name: mock-data-vector-generation
description: Use when you need a realistic simulated band power vector (e.g. for Fisher
  forecasting, robustness tests, or optimising the band power bin width) matching
  a specific candl data set such as SPT-3G 2018 TT/TE/EE.
when_to_use_negative:
- The goal is fitting or constraining parameters on real measured band powers rather
  than simulating a synthetic data set.
- No covariance matrix or window-function data set in candl format is available for
  the target experiment.
- The mock must be non-Gaussian or include noise realisations outside the band power
  covariance, which this Gaussian-realisation procedure does not model.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: candl
  role: Differentiable likelihood that initialises from a .yaml index file, provides
    the covariance matrix, window-function binning, data model transformations, and
    the chi-squared/log-likelihood evaluation
  repo: https://github.com/Lbalkenhol/candl
- name: candl_data
  role: Supplies the SPT-3G 2018 TT/TE/EE index file, band powers, covariance, window
    functions, and data set info used to centre the mock realisation
  repo: https://github.com/Lbalkenhol/candl_data
- name: Python
  role: Implementation language for the forward model, NumPy multivariate Gaussian
    draw, and mock file writing
- name: JAX
  role: Powers candl's differentiable likelihood; not strictly required for the Gaussian
    draw but enables derivative-based follow-on analyses of the mock data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:e4611568dedf9f74484c32842951b3679221dd7c954eae7fcfac5ce25a1a84a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lbalkenhol/candl
  tools:
  - JAX
  - Python
  - candl
  - candl_data
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# mock-data-vector-generation

## Summary

Generates realistic mock CMB band power data vectors by forward-modelling spectra at fiducial cosmology through the candl likelihood pipeline and drawing a multivariate Gaussian realisation with the data set's covariance matrix, as used in candl's forecasting and bin-width optimisation examples.

## When to use

Use when you need a realistic simulated band power vector (e.g. for Fisher forecasting, robustness tests, or optimising the band power bin width) matching a specific candl data set such as SPT-3G 2018 TT/TE/EE.

## When NOT to use

- The goal is fitting or constraining parameters on real measured band powers rather than simulating a synthetic data set.
- No covariance matrix or window-function data set in candl format is available for the target experiment.
- The mock must be non-Gaussian or include noise realisations outside the band power covariance, which this Gaussian-realisation procedure does not model.

## Inputs

- candl likelihood initialised from a data set index .yaml file (e.g. candl_data.SPT3G_2018_TTTEEE)
- Fiducial cosmological parameter vector (H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0)
- Band power covariance matrix in spectrum-major order from the data set info
- Window functions and data model transformations from the data set info .yaml
- spectra_info declaring the spectrum ordering and bin counts

## Outputs

- mock_band_powers.txt text file of mock band powers ordered as declared in spectra_info
- Chi-squared value of the mock data vector evaluated at the fiducial cosmology through the candl likelihood

## How to apply

Initialise the candl likelihood by pointing to the index .yaml file of the target data set (e.g. candl_data.SPT3G_2018_TTTEEE), which loads band powers, the spectrum-major covariance, window functions, and data model transformations. Build a params-to-bandpowers forward model using candl.tools.get_params_to_logl_func-style tooling: compute CMB-only spectra at the fiducial parameters (H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0), bin them with the window functions, and loop over the data model transformations applying them one by one. Draw a mock data vector as a multivariate Gaussian realisation centred on the forward-modelled band powers using the likelihood's covariance matrix (NumPy random generator). Write the vector to mock_band_powers.txt with the spectrum ordering declared in spectra_info, then evaluate and record the chi-squared at the fiducial cosmology through the candl likelihood.

## Related tools

- **candl** (Differentiable likelihood that initialises from a .yaml index file, provides the covariance matrix, window-function binning, data model transformations, and the chi-squared/log-likelihood evaluation) — https://github.com/Lbalkenhol/candl
- **candl_data** (Supplies the SPT-3G 2018 TT/TE/EE index file, band powers, covariance, window functions, and data set info used to centre the mock realisation) — https://github.com/Lbalkenhol/candl_data
- **Python** (Implementation language for the forward model, NumPy multivariate Gaussian draw, and mock file writing)
- **JAX** (Powers candl's differentiable likelihood; not strictly required for the Gaussian draw but enables derivative-based follow-on analyses of the mock data)

## Examples

```
import candl; like = candl.Candl(candl_data.SPT3G_2018_TTTEEE); mock = np.random.multivariate_normal(model_bandpowers, like.cov)
```

## Evaluation signals

- mock_band_powers.txt exists and its number of band power rows matches the spectra_info bin counts for each spectrum
- The spectrum ordering inside mock_band_powers.txt matches the ordering declared in spectra_info
- The chi-squared at the fiducial cosmology is consistent with the expected value for a Gaussian realisation of the data vector length
- Re-evaluating the chi-squared at the fiducial point shows no >0.1% deviation from the previously recorded value

## Limitations

- The mock is a Gaussian realisation of the covariance matrix, so it inherits the Gaussianity assumption of the likelihood model and cannot represent non-Gaussian covariance features.
- The covariance must be handled in spectrum-major order and the output must follow the spectra_info ordering; mismatched ordering will silently corrupt chi-squared evaluation.
- JAX on some architectures may not provide 64-bit precision, introducing small (up to ~1e-4) differences in log-likelihood computations.

## Evidence

- [abstract] These also include optimising the band power bin width to minimise parameter errors of a realistic mock data set.: "These also include optimising the band power bin width to minimise parameter errors of a realistic mock data set."
- [methods] It simply suffices to point to the .yaml file of a likelihood to initialise it.: "It simply suffices to point to the ``.yaml`` file of a likelihood to initialise it."
- [methods] Loop over the transformations in the data model and apply them one by one to the model vector.: "Loop over the transformations in the data model and apply them one by one to the model vector."
- [methods] Take the difference between data and model band powers and calculate the log likelihood value.: "#. Take the difference between data and model band powers and calculate the log likelihood value."
- [methods] candl comes with clipy, a pure Python implementation of the 2018 Planck likelihoods.: "candl comes with ``clipy``, a pure Python implementation of the 2018 Planck likelihoods."
