---
name: bayesian-transient-parameter-estimation
description: Use when fitting photometric (lightcurve) or spectroscopic observations
  of an explosive transient with a physical or semi-analytical model, and you need
  posterior samples and evidence rather than a simple chi-square fit.
when_to_use_negative:
- You only need forward simulation of transient populations or survey lightcurves
  (e.g. ZTF/Rubin LSST cadence simulation) without any data to fit — redback models
  can be run standalone without fitting.
- The analysis goal is spectral template classification (SNID/Super-SNID matching),
  not Bayesian parameter estimation of a physical model.
- Your model is not among redback's implemented models and you do not intend to implement
  it as a Python function, since redback expects models as functions.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: redback
  role: Primary package providing transient objects, >100 models, priors/likelihoods,
    fit_model entry point, and plotting
  repo: https://github.com/nikhil-sarin/redback
- name: bilby
  role: Underlying sampling engine; redback uses bilby under the hood so samplers
    (dynesty, nestle) and likelihoods can be switched
- name: dynesty / nestle
  role: Nested samplers invoked through fit_model (e.g. sampler='nestle', nlive=200)
    in the paper's example fits
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stae1238
    title: '<scp>redback</scp> : a Bayesian inference software package for electromagnetic
      transients'
  source_content_hash: sha256:49d04f8f6ce0d492c9948c53839ce36972aa67da9a89f3171593f80102148797
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nikhil-sarin/redback
  tools:
  - Python
  - bilby
  - dynesty / nestle
  - redback
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bayesian-transient-parameter-estimation

## Summary

Perform end-to-end Bayesian parameter estimation for electromagnetic transients (kilonovae, supernovae, GRB afterglows, TDEs, engine-driven transients) using redback, which wraps bilby's samplers and over 100 transient models into a homogeneous fitting interface.

## When to use

Use when fitting photometric (lightcurve) or spectroscopic observations of an explosive transient with a physical or semi-analytical model, and you need posterior samples and evidence rather than a simple chi-square fit.

## When NOT to use

- You only need forward simulation of transient populations or survey lightcurves (e.g. ZTF/Rubin LSST cadence simulation) without any data to fit — redback models can be run standalone without fitting.
- The analysis goal is spectral template classification (SNID/Super-SNID matching), not Bayesian parameter estimation of a physical model.
- Your model is not among redback's implemented models and you do not intend to implement it as a Python function, since redback expects models as functions.

## Inputs

- Transient observations downloaded via redback data getters (e.g. kilonova AT2017gfo from the Open Transient Catalog, Swift/BATSE/OTTER/FINK/LASAIR catalogs)
- User-supplied data files (e.g. example_data/grb_afterglow.csv, example_data/SN2018ibb_photcat_REDBACK.ascii)
- Prior specification from redback's priors interface
- Likelihood configuration from redback's likelihoods / joint_likelihood modules

## Outputs

- Homogeneous bilby result object with posterior samples and evidence
- Fitted lightcurve/spectra evolution plots and corner plots
- Result files comparable to those on Zenodo record 8273145

## How to apply

Obtain the transient object either by downloading catalog data (e.g. redback.get_data.get_kilonova_data_from_open_transient_catalog_data(transient='at2017gfo')) or by loading user-supplied data such as grb_afterglow.csv; redback processes these into a homogeneous transient object. Choose one of redback's >100 models (analytical, semi-analytical, or machine-learning numerical surrogates) and set priors via redback's priors interface. Call redback.fit_model(transient=..., prior=priors, likelihood=likelihood, sampler='nestle', nlive=200) — bilby runs under the hood so samplers like dynesty/nestle can be swapped freely, and the default sampler choice depends on the data type. The call returns a homogeneous result object containing posterior samples and evidence, which supports plotting fitted lightcurves, corner plots, importance sampling, and joint spectrum+photometry (MultiMessengerTransient) fits. Validated configurations from the paper used redback versions 1.0.31–1.17.0 with bilby 2.7.0.dev70+gcde65c51c, and result files for reproducing plots are deposited on Zenodo record 8273145.

## Related tools

- **redback** (Primary package providing transient objects, >100 models, priors/likelihoods, fit_model entry point, and plotting) — https://github.com/nikhil-sarin/redback
- **bilby** (Underlying sampling engine; redback uses bilby under the hood so samplers (dynesty, nestle) and likelihoods can be switched)
- **dynesty / nestle** (Nested samplers invoked through fit_model (e.g. sampler='nestle', nlive=200) in the paper's example fits)

## Examples

```
kne = 'at2017gfo'; data = redback.get_data.get_kilonova_data_from_open_transient_catalog_data(transient=kne); result = redback.fit_model(transient=data, prior=priors, likelihood=likelihood, sampler='nestle', nlive=200)
```

## Evaluation signals

- redback.fit_model completes end-to-end and returns a result object containing posterior samples and evidence
- Posterior/evidence and fitted lightcurve plots render from the result object without errors
- Corner plot and spectra/photometry evolution plots are qualitatively consistent with the paper's demonstration (AT2017gfo joint fit) and with result files on Zenodo record 8273145
- In the mixture-model Arnett fit example, flagged outliers are down-weighted and do not bias the fitted lightcurve
- The documented software environment is reproduced (e.g. redback 1.12.0–1.17.0 with bilby 2.7.0.dev70+gcde65c51c)

## Limitations

- The Lasair data getter may be removed in a future release (dependency deprecation risk).
- Old flux density approximation output is retained only for backward compatibility; the new model interface outputs spectra, SNcosmosource, or flux objects for optical models.
- Default sampler/likelihood selection is made automatically depending on the data, which may not match bespoke analysis setups unless overridden.

## Evidence

- [abstract] redback provides an object-orientated python interface to over 12 different samplers and over 100 different models for kilonovae: "redback provides an object-orientated python interface to over 12 different samplers and over 100 different models for kilonovae"
- [results] fit_model with nestle sampler: "result = redback.fit_model(transient=sn, prior=priors, likelihood=likelihood, sampler='nestle', nlive=200"
- [results] kilonova data getter from Open Transient Catalog: "kne = 'at2017gfo' data = redback.get_data.get_kilonova_data_from_open_transient_catalog_data(transient=kne)"
- [abstract] joint spectrum+photometry kilonova fit capability: "we show how redback can be used to jointly fit the spectrum and photometry of a kilonova"
- [readme] bilby under the hood, homogeneous result object: [evidence span withheld — anchor status: external_doc]
- [discussion] Lasair data getter deprecation risk: "Lasair data getter. This may be removed in a future release."
