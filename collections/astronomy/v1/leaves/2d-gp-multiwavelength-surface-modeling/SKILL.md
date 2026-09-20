---
name: 2d-gp-multiwavelength-surface-modeling
description: Use when you have multiwavelength broadband photometry (e.g., a space-separated
  CSV like example_data/grb_afterglow.csv with GRB afterglow data) and need a joint,
  data-driven model over both time and frequency axes rather than fitting each band
  separately.
when_to_use_negative:
- You need physically motivated afterglow model parameters (e.g., jet or circumstellar
  medium parameters) rather than a non-parametric, data-driven interpolation surface.
- Data are single-band or otherwise unsuited to joint 2D time–frequency GP regression;
  fit a 1D lightcurve model instead.
- Input is not multiwavelength photometry (e.g., spectra or X-ray spectral data, which
  redback routes to spectral template matching / xray_spectral_fitting workflows).
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: redback
  role: Provides the 2D GP regression setup over time and wavelength axes, redback.fit_model
    entry point, and plotting utilities for the lightcurve surface
  repo: https://github.com/nikhil-sarin/redback
- name: george
  role: Kernel interface supplying the ConstantKernel x Matern32Kernel used for the
    joint time–frequency GP
- name: bilby
  role: Under-the-hood sampling framework wrapping samplers (nestle/dynesty) that
    redback.fit_model calls for GP hyperparameter inference
- name: pandas
  role: read_csv loading of the space-separated grb_afterglow.csv multiwavelength
    photometry
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/mnras/stae1238
    title: '<scp>redback</scp> : a Bayesian inference software package for electromagnetic
      transients'
  source_content_hash: sha256:37edd623ce380b539224bdc058a69bbf46955297948a746315f3f7c3081b9c11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nikhil-sarin/redback
  tools:
  - Python
  - bilby
  - george
  - pandas
  - redback
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# 2D Gaussian-process multiwavelength surface modeling with redback/george

## Summary

Fit a 2D Gaussian Process jointly over time and frequency/wavelength axes to multiwavelength photometry using redback's george kernel interface (ConstantKernel x Matern32Kernel), sampling GP hyperparameters with bilby-wrapped samplers such as nestle/dynesty. This recovers a smooth lightcurve/spectrum surface (e.g., for GRB afterglow broadband data) while quantifying uncertainty across all bands simultaneously.

## When to use

Use when you have multiwavelength broadband photometry (e.g., a space-separated CSV like example_data/grb_afterglow.csv with GRB afterglow data) and need a joint, data-driven model over both time and frequency axes rather than fitting each band separately.

## When NOT to use

- You need physically motivated afterglow model parameters (e.g., jet or circumstellar medium parameters) rather than a non-parametric, data-driven interpolation surface.
- Data are single-band or otherwise unsuited to joint 2D time–frequency GP regression; fit a 1D lightcurve model instead.
- Input is not multiwavelength photometry (e.g., spectra or X-ray spectral data, which redback routes to spectral template matching / xray_spectral_fitting workflows).

## Inputs

- Multiwavelength broadband photometry CSV (space-separated, e.g. example_data/grb_afterglow.csv — GRB170817A afterglow broadband data)
- GP kernel specification: george ConstantKernel x Matern32Kernel over time and frequency axes
- Prior specification for GP hyperparameters
- redback 1.17.0 environment with bilby 2.7.0.dev70+gcde65c51c

## Outputs

- gp_2d_lightcurve_surface.png — fitted GP lightcurve surface across all photometric bands
- gp_loglikelihood.txt — final GP log-likelihood (reported value: 22.542225396255056)
- bilby result object with posterior samples of GP hyperparameters

## How to apply

Load the multiwavelength broadband data with pandas (e.g., pd.read_csv('example_data/grb_afterglow.csv') for space-separated afterglow photometry). Construct a 2D GP regression setup in redback using the george kernel interface, combining a ConstantKernel with a Matern32Kernel operating jointly on the time and wavelength/photometric-band axes. Run the fit via redback.fit_model with the bilby-wrapped sampler (nestle or dynesty, e.g., sampler='nestle', nlive=200) to optimize the GP hyperparameters against the multiwavelength photometry; redback returns a homogeneous result object. Generate the fitted GP lightcurve surface across all bands with redback's plotting utilities (saved as e.g. gp_2d_lightcurve_surface.png) and record the final GP log-likelihood to gp_loglikelihood.txt. Validate by checking the reproduced final log-likelihood against the reported reference value 22.542225396255056 in the redback 1.17.0 / bilby 2.7.0.dev70+gcde65c51c environment.

## Related tools

- **redback** (Provides the 2D GP regression setup over time and wavelength axes, redback.fit_model entry point, and plotting utilities for the lightcurve surface) — https://github.com/nikhil-sarin/redback
- **george** (Kernel interface supplying the ConstantKernel x Matern32Kernel used for the joint time–frequency GP)
- **bilby** (Under-the-hood sampling framework wrapping samplers (nestle/dynesty) that redback.fit_model calls for GP hyperparameter inference)
- **pandas** (read_csv loading of the space-separated grb_afterglow.csv multiwavelength photometry)

## Examples

```
data = pd.read_csv('example_data/grb_afterglow.csv'); result = redback.fit_model(transient=data, kernel=george.kernels.ConstantKernel * george.kernels.Matern32Kernel, sampler='nestle', nlive=200)
```

## Evaluation signals

- Reproduced final GP log-likelihood equals the reported value 22.542225396255056 (redback 1.17.0 with bilby 2.7.0.dev70+gcde65c51c).
- gp_2d_lightcurve_surface.png renders a smooth GP surface spanning all photometric bands with no plotting errors.
- gp_loglikelihood.txt contains a single numeric log-likelihood consistent with the result object.
- redback.fit_model returns a homogeneous result object with posterior samples for the GP hyperparameters (ConstantKernel and Matern32Kernel terms).
- Fit runs through the bilby-wrapped sampler (nestle/dynesty) to completion without sampler warnings on the loaded dataset.

## Limitations

- Result depends on environment/versions (redback 1.17.0, bilby 2.7.0.dev70+gcde65c51c.d20260325); other versions may not reproduce the log-likelihood to reported precision.
- The GP is a non-parametric surrogate: it interpolates the multiwavelength surface but does not yield physical afterglow parameters.
- redback's default sampler/likelihood choice depends on the data, so deviating from nestle/dynesty defaults may change results.

## Evidence

- [results] 2D GP fit achieved a final log-likelihood of 22.542225396255056: [evidence span withheld — anchor status: not_found]
- [results] data loading snippet: "data = pd.read_csv('example_data/grb_afterglow.csv')"
- [results] fit_model call pattern: "result = redback.fit_model(transient=sn, prior=priors, likelihood=likelihood,
                           sampler='nestle', nlive=200"
- [readme] redback uses bilby under the hood: [evidence span withheld — anchor status: external_doc]
- [abstract] jointly fit spectrum and photometry: "we show how redback can be used to jointly fit the spectrum and photometry of a kilonova"
