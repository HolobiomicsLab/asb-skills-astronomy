---
name: bayesian-nested-sampling-inference
description: Use when a transient's photometry or spectrum needs model fitting with
  posterior samples — e.g. fitting SN2011kl with t0_supernova_extinction or a joint
  kilonova spectrum+photometry fit like AT2017gfo.
when_to_use_negative:
- You only need to simulate transient populations or survey lightcurves — redback
  models are callable as functions without data and without running any sampler.
- You need to retrieve or homogenise catalogue data but not infer parameters — the
  get_data interfaces do not require sampling.
- 'The Lasair data getter is involved: the paper warns it may be removed in a future
  release (dependency deprecation risk).'
edam_operation: http://edamontology.org/operation_0282
tools:
- name: redback
  role: Provides fit_model, the priors and likelihoods interfaces, and the homogeneous
    result object with posterior/lightcurve plotting; over 100 transient models available
  repo: https://github.com/nikhil-sarin/redback
- name: bilby
  role: Underlying sampling framework redback uses; enables switching samplers and
    likelihoods
- name: nestle
  role: Nested sampler used in the paper's example fit (nlive=200)
- name: dynesty
  role: Nested sampler used for the joint spectrum+photometry kilonova fit of AT2017gfo
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stae1238
    title: '<scp>redback</scp> : a Bayesian inference software package for electromagnetic
      transients'
  source_content_hash: sha256:b766267c683800f5a00a6344dc684f85920181cf65233f68425f74f458af4bf2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nikhil-sarin/redback
  tools:
  - Python
  - bilby
  - dynesty
  - nestle
  - redback
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bayesian-nested-sampling-inference

## Summary

Bayesian parameter estimation for electromagnetic transient models (kilonovae, supernovae, GRB afterglows, TDEs) using redback's fit_model interface, which wraps bilby and dispatches to nested samplers such as nestle or dynesty to return a homogeneous result object with posterior samples and evidence.

## When to use

Use when a transient's photometry or spectrum needs model fitting with posterior samples — e.g. fitting SN2011kl with t0_supernova_extinction or a joint kilonova spectrum+photometry fit like AT2017gfo.

## When NOT to use

- You only need to simulate transient populations or survey lightcurves — redback models are callable as functions without data and without running any sampler.
- You need to retrieve or homogenise catalogue data but not infer parameters — the get_data interfaces do not require sampling.
- The Lasair data getter is involved: the paper warns it may be removed in a future release (dependency deprecation risk).

## Inputs

- redback transient object built from catalogue photometry/spectra (e.g. Open Transient Catalog data for at2017gfo, SN2011kl flux-density data)
- redback prior specification over model parameters (e.g. t0, host-galaxy extinction, kilonova engine parameters)
- redback/bilby likelihood appropriate to the data mode (photometry, flux density, or joint spectrum+photometry)
- user-supplied or simulated data (e.g. example_data/grb_afterglow.csv, example_data/SN2018ibb_photcat_REDBACK.ascii)

## Outputs

- result object containing posterior samples and evidence for all model parameters
- saved result file (compatible with Zenodo record 8273145 deposition format)
- fitted lightcurve plot overlaying the model on retrieved photometry; corner plots and spectra/photometry evolution plots

## How to apply

Build the redback transient object from retrieved catalogue data (e.g. SN2011kl flux-density mode via redback.get_data, or AT2017gfo via get_kilonova_data_from_open_transient_catalog_data), then define priors over the model parameters (e.g. explosion time t0 plus host-galaxy extinction parameters over the arnett base model). Call redback.fit_model with the transient, priors, likelihood, and a nested sampler — the paper's examples use sampler='nestle' at nlive=200 and dynesty — noting that redback uses bilby under the hood so samplers and likelihoods are easily switched. The call returns a homogeneous result object; save it to a result file (the paper deposits its result files on Zenodo record 8273145) and use the result object's plotting interface to overlay the fitted lightcurve on the retrieved photometry and draw corner/posterior plots.

## Related tools

- **redback** (Provides fit_model, the priors and likelihoods interfaces, and the homogeneous result object with posterior/lightcurve plotting; over 100 transient models available) — https://github.com/nikhil-sarin/redback
- **bilby** (Underlying sampling framework redback uses; enables switching samplers and likelihoods)
- **nestle** (Nested sampler used in the paper's example fit (nlive=200))
- **dynesty** (Nested sampler used for the joint spectrum+photometry kilonova fit of AT2017gfo)

## Examples

```
result = redback.fit_model(transient=sn, prior=priors, likelihood=likelihood, sampler='nestle', nlive=200)
```

## Evaluation signals

- The fit script runs to completion and writes a result file containing posterior samples for ALL parameters of the chosen model (e.g. every t0_supernova_extinction parameter).
- The result object supports posterior plotting and lightcurve plotting without errors.
- The fitted lightcurve qualitatively overlays the retrieved SN2011kl photometry; corner/evolution plots are qualitatively consistent with the paper's plots regenerated from Zenodo record 8273145 result files.
- The nested sampler (nestle/dynesty) completes at the configured nlive (e.g. 200) and the returned object carries the evidence estimate.

## Limitations

- Sampler choice and likelihood defaults depend on the data type; switching is possible but defaults may not suit every dataset.
- The Lasair data getter may be removed in a future release, so data retrieval preceding the fit may break.
- Old flux-density approximation output is retained only for backward compatibility; the new model interface outputs spectra/SNcosmosource/flux, so output mode must be checked when fitting.

## Evidence

- [abstract] redback provides an object-orientated python interface to over 12 different samplers and over 100 different models for kilonovae: "redback provides an object-orientated python interface to over 12 different samplers and over 100 different models for kilonovae, supernovae, gamma-ray burst afterglows, tidal disruption events"
- [results] result = redback.fit_model with nestle nlive=200: "result = redback.fit_model(transient=sn, prior=priors, likelihood=likelihood,
                           sampler='nestle', nlive=200"
- [readme] Redback uses bilby for sampling: [evidence span withheld — anchor status: external_doc]
- [readme] Fitting returns a homogenous result object: [evidence span withheld — anchor status: external_doc]
- [results] Zenodo result files: "The result files required for producing the plots can be found on [Zenodo](https://zenodo.org/record/8273145)."
