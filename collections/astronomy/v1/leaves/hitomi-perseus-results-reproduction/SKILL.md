---
name: hitomi-perseus-results-reproduction
description: Use when benchmarking Bayesian X-ray spectral fitting on high-resolution
  calorimeter data (e.g. the Hitomi Perseus cluster observation) and the goal is a
  GPU variational-inference fit that completes in under 10 minutes while matching
  published Hitomi parameters.
when_to_use_negative:
- The input is low-count/low-resolution CCD data where you need an unbiased posterior
  from a full sampler — the C-stat guidance and NUTS sampling are the article's default
  path, while variational inference is the demonstrated approach for high-resolution
  data
- No GPU is available, since the less-than-10-minute runtime claim is specific to
  variational inference on a GPU-compiled likelihood
- The workflow requires HEASoft tooling or exact XSPEC 12.15.1 behaviour, because
  jaxspec has no HEASoft dependency and deliberately departs from XSPEC in places
  while still being in early release
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: jaxspec
  role: Loads the OGIP-standard Hitomi spectrum via ObsConfiguration, defines the
    composite spectral model, and runs the variational-inference fit producing the
    FitResult
  repo: https://github.com/renecotyfanboy/jaxspec
- name: JAX
  role: Provides just-in-time compilation and automatic differentiation of the spectral
    model, yielding differentiable likelihood functions compilable on GPUs
  repo: https://github.com/renecotyfanboy/jaxspec
- name: numpyro
  role: Supplies prior distributions (dist.Uniform, dist.LogUniform), platform configuration
    for GPU and x64, and the variational-inference solver
- name: dill
  role: Serialises the fitted FitResult into result.pickle
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202451736
    title: 'jaxspec: A fast and robust Python library for X-ray spectral fitting'
  source_content_hash: sha256:4b509f3f568a1dd2d53afb24a80aa693432b0a429ad722e00437cd50cd5a9204
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/renecotyfanboy/jaxspec
  tools:
  - JAX
  - Python
  - dill
  - jaxspec
  - numpyro
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# hitomi-perseus-results-reproduction

## Summary

Reproduces the Hitomi collaboration's Perseus cluster high-resolution X-ray spectroscopy results by fitting the Hitomi OGIP observation with jaxspec using variational inference on a GPU, verifying that convincing posteriors are obtained in less than 10 minutes of wall-clock runtime.

## When to use

Use when benchmarking Bayesian X-ray spectral fitting on high-resolution calorimeter data (e.g. the Hitomi Perseus cluster observation) and the goal is a GPU variational-inference fit that completes in under 10 minutes while matching published Hitomi parameters.

## When NOT to use

- The input is low-count/low-resolution CCD data where you need an unbiased posterior from a full sampler — the C-stat guidance and NUTS sampling are the article's default path, while variational inference is the demonstrated approach for high-resolution data
- No GPU is available, since the less-than-10-minute runtime claim is specific to variational inference on a GPU-compiled likelihood
- The workflow requires HEASoft tooling or exact XSPEC 12.15.1 behaviour, because jaxspec has no HEASoft dependency and deliberately departs from XSPEC in places while still being in early release

## Inputs

- Hitomi Perseus cluster observation PHA file in the OGIP standard
- Associated ARF/RMF instrument response files
- numpyro prior dict (e.g. dist.Uniform, dist.LogUniform) over the composite spectral model parameters
- GPU with numpyro/JAX configured (set_platform('gpu'), enable_x64)

## Outputs

- Posterior parameter estimates comparable to the Hitomi collaboration's published Perseus results
- FitResult object serialised with dill as result.pickle
- Wall-clock inference runtime measurement (expected under 10 minutes on GPU)

## How to apply

Install jaxspec into a clean Python 3.12 environment with pip, since jaxspec is pure Python with no HEASoft dependency. Build an ObsConfiguration from the Hitomi Perseus PHA file together with its ARF/RMF response, band-limiting the fit range (e.g. low_energy=0.3, high_energy=12 keV as in the documented pattern). Define a composite spectral model from jaxspec's implemented additive/multiplicative components, attach a prior dict of numpyro distributions such as dist.Uniform and dist.LogUniform, and fix calibrated parameters as constants. Configure the accelerator before any JAX code runs — numpyro.set_platform('gpu') and numpyro.enable_x64() — then solve the inference problem with variational inference on the GPU-compiled likelihood, using a Poisson likelihood (C-stat) since chi-square biases parameter estimates at low counts. Compare the fitted posterior parameters against the Hitomi collaboration's published Perseus results to confirm reproduction, record the wall-clock runtime against the 10-minute threshold, and serialise the FitResult with dill into result.pickle.

## Related tools

- **jaxspec** (Loads the OGIP-standard Hitomi spectrum via ObsConfiguration, defines the composite spectral model, and runs the variational-inference fit producing the FitResult) — https://github.com/renecotyfanboy/jaxspec
- **JAX** (Provides just-in-time compilation and automatic differentiation of the spectral model, yielding differentiable likelihood functions compilable on GPUs) — https://github.com/renecotyfanboy/jaxspec
- **numpyro** (Supplies prior distributions (dist.Uniform, dist.LogUniform), platform configuration for GPU and x64, and the variational-inference solver)
- **dill** (Serialises the fitted FitResult into result.pickle)

## Examples

```
obs = ObsConfiguration.from_pha_file('perseus.pha', low_energy=0.3, high_energy=12)
```

## Evaluation signals

- Fitted posterior parameters match the Hitomi collaboration's published Perseus results (the reproduction criterion)
- Wall-clock inference runtime is below the 10-minute GPU threshold
- numpyro.set_platform('gpu') and numpyro.enable_x64() were called before any JAX code ran, and the platform is confirmed as GPU
- The posterior predictive check (e.g. result.plot_ppc) shows the model accounts for the observed counts in the band-limited range
- result.pickle deserialises with dill into a valid FitResult object

## Limitations

- jaxspec is still in early release: expect bugs, breaking API changes, undocumented features and lack of functionalities
- The SubtractedBackground model does not propagate dispersion, which is inadequate for a comprehensive error budget
- A jaxspec software version number is not stated (only Python 3.11/3.12 and XSPEC 12.15.1 are referenced)
- Fitting must use the Poisson likelihood (C-stat) at all count rates to avoid biased parameter estimates

## Evidence

- [abstract] Variational inference produces convincing results on high-resolution data in less than 10 minutes on a GPU: "we show that variational inference can produce convincing results even on high-resolution data in less than 10 minutes on a GPU."
- [abstract] jaxspec reproduces the Hitomi collaboration's Perseus cluster results using variational inference on a GPU: "reproducing the results of the Hitomi collaboration on the Perseus cluster, while solving the inference problem using variational inference on a GPU."
- [methods] Band-limited ObsConfiguration built from a PHA file: "obs = ObsConfiguration.from_pha_file('your.pha', low_energy=0.3, high_energy=12)"
- [methods] C-stat (Poisson likelihood) required at all count rates: "Most recent publications on the subject agree that using the C-stat at all count rates is necessary to ensure an unbiased estimate of the parameters."
- [readme] jaxspec loads OGIP-standard X-ray spectra and defines spectral models: "It can currently load an X-ray spectrum (in the OGIP standard), define a spectral model from the implemented components, and calculate the best parameters using state-of-the-art Bayesian approaches."
- [readme] jaxspec is in early release with expected bugs and breaking changes: "**jaxspec is still in early release**: expect bugs, breaking API changes, undocumented features and lack of functionalities"
