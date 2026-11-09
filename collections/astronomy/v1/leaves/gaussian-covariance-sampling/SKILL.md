---
name: gaussian-covariance-sampling
description: Use when you need a synthetic TT/TE/EE (or lensing) band-power data vector
  consistent with a candl likelihood's covariance — e.g. for forecasting, Fisher/bin-width
  optimisation studies, or validating a likelihood via chi-squared at the fiducial
  cosmology.
when_to_use_negative:
- When real observed band powers are available — the skill generates synthetic data
  for example calculations, not replacements for measured data vectors
- When the analysis requires non-Gaussian or correlated-survey mock covariance beyond
  the likelihood's own Gaussian covariance matrix (this skill only samples the Gaussian
  realisation)
- When the band-power bin width or spectra_info layout is still being changed, since
  the mock file's ordering and row count must match the declared bin counts
tools:
- name: candl
  role: Provides the likelihood (band powers, spectrum-major covariance, window functions,
    data-model transformations) and the tools (e.g. get_params_to_logl_func) used
    to forward-model the fiducial band powers and compute the mock chi-squared
  repo: https://github.com/Lbalkenhol/candl
- name: candl_data
  role: Supplies the SPT-3G 2018 TT/TE/EE data set (index file, .yaml info, covariance,
    window functions) from which the mock vector is sampled
  repo: https://github.com/Lbalkenhol/candl_data
- name: Python
  role: NumPy random generator draws the multivariate Gaussian realisation and writes
    the mock band powers to text
- name: JAX
  role: Powers candl's differentiable likelihood so the mock data's chi-squared (and
    derived Fisher matrices) can be evaluated and differentiated at the fiducial point
  repo: https://github.com/Lbalkenhol/candl
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:a251d85e06bfc8c4d4a13519a0d930b7c97959a930c16357a9e16212f683aa46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lbalkenhol/candl
  tools:
  - JAX
  - Python
  - candl
  - candl_data
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# gaussian-covariance-sampling

## Summary

Generates realistic mock CMB band-power data vectors by drawing multivariate Gaussian realisations centred on a candl forward-modelled band-power vector, using the likelihood's spectrum-major-ordered covariance matrix. This underpins candl's example analyses such as forecasting and optimising the band power bin width.

## When to use

Use when you need a synthetic TT/TE/EE (or lensing) band-power data vector consistent with a candl likelihood's covariance — e.g. for forecasting, Fisher/bin-width optimisation studies, or validating a likelihood via chi-squared at the fiducial cosmology.

## When NOT to use

- When real observed band powers are available — the skill generates synthetic data for example calculations, not replacements for measured data vectors
- When the analysis requires non-Gaussian or correlated-survey mock covariance beyond the likelihood's own Gaussian covariance matrix (this skill only samples the Gaussian realisation)
- When the band-power bin width or spectra_info layout is still being changed, since the mock file's ordering and row count must match the declared bin counts

## Inputs

- candl likelihood index file (e.g. candl_data.SPT3G_2018_TTTEEE) with band powers, covariance, window functions, and data-model transformations from a .yaml file
- Fiducial cosmological parameter vector (H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0)
- A params-to-bandpowers forward model (candl.tools.get_params_to_logl_func-style tooling)
- spectra_info bin counts defining the expected spectrum ordering and number of band-power rows

## Outputs

- Mock band-power vector written to mock_band_powers.txt with the same spectrum ordering as spectra_info
- Chi-squared value of the mock data vector at the fiducial cosmology
- Realistic mock data set suitable for forecasting and bin-width optimisation example analyses

## How to apply

Initialise the candl likelihood from a data-set index file such as candl_data.SPT3G_2018_TTTEEE, which loads band powers, covariance (in spectrum-major order), window functions, and data-model transformations declared in the info .yaml file. Build the forward model at the fiducial parameters (H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0): construct the long vector of CMB-only spectra, bin it with the likelihood's window functions, then loop over the data-model transformations applying them one by one. Draw the mock vector as a multivariate Gaussian centred on these modelled band powers using the likelihood's covariance matrix (e.g. NumPy's random generator) and write it to mock_band_powers.txt in the spectrum ordering declared in spectra_info. Finally, evaluate the chi-squared of the mock vector at the fiducial cosmology through the candl likelihood and check it against the expectation for a Gaussian realisation of that data-vector length.

## Related tools

- **candl** (Provides the likelihood (band powers, spectrum-major covariance, window functions, data-model transformations) and the tools (e.g. get_params_to_logl_func) used to forward-model the fiducial band powers and compute the mock chi-squared) — https://github.com/Lbalkenhol/candl
- **candl_data** (Supplies the SPT-3G 2018 TT/TE/EE data set (index file, .yaml info, covariance, window functions) from which the mock vector is sampled) — https://github.com/Lbalkenhol/candl_data
- **Python** (NumPy random generator draws the multivariate Gaussian realisation and writes the mock band powers to text)
- **JAX** (Powers candl's differentiable likelihood so the mock data's chi-squared (and derived Fisher matrices) can be evaluated and differentiated at the fiducial point) — https://github.com/Lbalkenhol/candl

## Examples

```
rng = np.random.default_rng(0); mock = rng.multivariate_normal(fid_bandpowers, like.cov); np.savetxt('mock_band_powers.txt', mock)
```

## Evaluation signals

- mock_band_powers.txt exists and its row count equals the expected number of band powers from the spectra_info bin counts
- Spectrum ordering in mock_band_powers.txt matches the ordering declared in spectra_info of the likelihood .yaml
- Chi-squared of the mock vector at the fiducial cosmology is consistent with the expected value for a Gaussian realisation of that data-vector length
- Re-evaluating the chi-squared through the candl likelihood reproduces the recorded value with no >0.1% deviation

## Limitations

- The mock vector is a Gaussian realisation of the likelihood's covariance; the validation expectation (chi-squared ≈ data-vector length) holds only in that Gaussian setting
- The forward model must apply the data-model transformations successively in the loop order of the data model; applying them out of order yields a wrong mock mean vector
- candl's auxiliary mock-data tools are demonstrated for its bundled example data sets (e.g. SPT-3G 2018 TT/TE/EE); other data sets require the same .yaml structure to work directly

## Evidence

- [abstract] These also include optimising the band power bin width to minimise parameter errors of a realistic mock data set.: "These also include optimising the band power bin width to minimise parameter errors of a realistic mock data set."
- [methods] Bin the model spectra using the window functions.: "#. Bin the model spectra using the window functions."
- [methods] Take the difference between data and model band powers and calculate the log likelihood value.: "#. Take the difference between data and model band powers and calculate the log likelihood value."
- [methods] Loop over the transformations in the data model and apply them one by one to the model vector.: "Loop over the transformations in the data model and apply them one by one to the model vector."
- [methods] Simply clone the repo to get the data and run pip install . inside the folder to install.: "Simply clone the repo to get the data and run :code:`pip install .` inside the folder to install."
