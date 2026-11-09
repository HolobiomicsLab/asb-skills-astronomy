---
name: xray-spectral-model-composition
description: Use when an X-ray spectrum in the OGIP standard must be fitted with a
  composite model (e.g. absorption times sum of continuum components) and parameters
  need to be inferred with Bayesian samplers such as NUTS.
when_to_use_negative:
- The task requires background subtraction with a comprehensive error budget — the
  SubtractedBackground model does not propagate dispersion, so BackgroundWithError
  or SpectralModelBackground should be considered instead.
- A chi-square fit at low count rates is the goal — C-stat/Poisson likelihood is preferred
  to avoid biased parameter estimates at low counts.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3399
tools:
- name: jaxspec
  role: Library providing spectral components (Tbabs, Powerlaw, Blackbodyrad), AdditiveComponent/MultiplicativeComponent
    base classes, ObsConfiguration loading, and MCMCFitter
  repo: https://github.com/renecotyfanboy/jaxspec
- name: JAX
  role: Backend for just-in-time compilation and automatic differentiation of the
    composed spectral model, enabling NUTS sampling; custom components must use JAX-compatible
    primitives
  repo: https://github.com/renecotyfanboy/jaxspec
- name: numpyro
  role: Supplies prior distributions (dist.Uniform, dist.LogUniform) and the no U-turn
    sampler driven via numpyro.enable_x64() and numpyro.set_host_device_count(16)
  repo: https://github.com/renecotyfanboy/jaxspec
- name: Python
  role: Runtime environment; jaxspec is a pure-Python library supporting Python 3.11
    and 3.12
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202451736
    title: 'jaxspec: A fast and robust Python library for X-ray spectral fitting'
  source_content_hash: sha256:ac124aa8398aeb3ed938cf2329e130f9d317a8203d02c0aa97f15a0775429ddc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/renecotyfanboy/jaxspec
  tools:
  - JAX
  - Python
  - XSPEC
  - jaxspec
  - matplotlib
  - numpyro
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# xray-spectral-model-composition

## Summary

Compose X-ray spectral models in jaxspec by combining additive and multiplicative components (built-in or custom subclasses of AdditiveComponent/MultiplicativeComponent) with numpyro priors, for Bayesian fitting under a Poisson likelihood.

## When to use

Use when an X-ray spectrum in the OGIP standard must be fitted with a composite model (e.g. absorption times sum of continuum components) and parameters need to be inferred with Bayesian samplers such as NUTS.

## When NOT to use

- The task requires background subtraction with a comprehensive error budget — the SubtractedBackground model does not propagate dispersion, so BackgroundWithError or SpectralModelBackground should be considered instead.
- A chi-square fit at low count rates is the goal — C-stat/Poisson likelihood is preferred to avoid biased parameter estimates at low counts.

## Inputs

- X-ray spectrum in the OGIP standard (.pha file) or a packaged example observation such as NGC7793_ULX4_PN
- Spectral model expression composed from jaxspec components (e.g. Tbabs()*(Powerlaw() + Blackbodyrad()) or custom AdditiveComponent/MultiplicativeComponent subclasses)
- Prior dict mapping parameter names (e.g. powerlaw_1.alpha, blackbodyrad_1.kT, tbabs_1.nh) to numpyro distributions (Uniform, LogUniform)

## Outputs

- FitResult containing an inference_data posterior with sampled draws for every component parameter
- Corner plot (result.plot_corner()) and posterior predictive plot (result.plot_ppc(plot_components=True, n_sigmas=2))
- LaTeX parameter table (result.table()); serialisable to result.pickle with dill

## How to apply

Build the model by multiplying and adding components, e.g. Tbabs()*(Powerlaw() + Blackbodyrad()), or define custom components by subclassing jaxspec.model.abc.AdditiveComponent (implementing the continuum method, e.g. M(E) = K*sin(E/E0)*exp(-E/E1)) and MultiplicativeComponent (implementing the factor method, e.g. |cos(E/E0)|), declaring parameters such as K, E0, E1 as flax.nnx.Param and using only JAX-compatible primitives so the model is just-in-time compiled and differentiable. Assign numpyro.priors (dist.Uniform or dist.LogUniform) to every parameter in a prior dict keyed by component name (e.g. powerlaw_1.alpha, tbabs_1.nh). Load the observation with jaxspec.data.util.load_example_obsconf('NGC7793_ULX4_PN') or ObsConfiguration.from_pha_file over the 0.3–12 keV band, then run MCMCFitter(model, prior, obs).fit(num_chains=16, num_warmup=1000, num_samples=5000) with numpyro.enable_x64() and numpyro.set_host_device_count(16). Fitting is done under a Poisson (C-stat-equivalent) likelihood because the C-stat at all count rates is necessary to ensure an unbiased estimate of the parameters.

## Related tools

- **jaxspec** (Library providing spectral components (Tbabs, Powerlaw, Blackbodyrad), AdditiveComponent/MultiplicativeComponent base classes, ObsConfiguration loading, and MCMCFitter) — https://github.com/renecotyfanboy/jaxspec
- **JAX** (Backend for just-in-time compilation and automatic differentiation of the composed spectral model, enabling NUTS sampling; custom components must use JAX-compatible primitives) — https://github.com/renecotyfanboy/jaxspec
- **numpyro** (Supplies prior distributions (dist.Uniform, dist.LogUniform) and the no U-turn sampler driven via numpyro.enable_x64() and numpyro.set_host_device_count(16)) — https://github.com/renecotyfanboy/jaxspec
- **Python** (Runtime environment; jaxspec is a pure-Python library supporting Python 3.11 and 3.12)

## Examples

```
from jaxspec import MCMCFitter; result = MCMCFitter(Tbabs()*(Powerlaw() + Blackbodyrad()), prior, obs).fit(num_chains=16, num_warmup=1000, num_samples=5000)
```

## Evaluation signals

- The fitting script runs end-to-end without error and posterior sites for the custom component parameters appear in result.inference_data.posterior with sampled draws, confirming they participate in the inference trace.
- Recovered posterior means agree with a blind XSPEC fit performed with C-stat — jaxspec is reported to give identical results at ~10x lower runtime.
- result.pickle exists after serialisation with dill, and corner and posterior predictive figures are produced.
- All model parameters have priors assigned; missing prior entries for any component parameter will fail the fit definition.

## Limitations

- jaxspec is still in early release: expect bugs, breaking API changes, undocumented features and lack of functionalities.
- Software version number for jaxspec itself is not stated in the article (only Python 3.11/3.12 and XSPEC 12.15.1 are referenced).
- The SubtractedBackground model does not propagate dispersion, which is inadequate for a comprehensive error budget.
- No changelog is available for the repository.

## Evidence

- [intro] custom components: "It can currently load an X-ray spectrum (in the OGIP standard), define a spectral model from the implemented components, and calculate the best parameters using state-of-the-art Bayesian approaches."
- [intro] JAX basis: [evidence span withheld — anchor status: external_doc]
- [methods] Poisson likelihood rationale: "Most recent publications on the subject agree that using the C-stat at all count rates is necessary to ensure an unbiased estimate of the parameters."
- [methods] fit call: "result = forward.fit(num_chains=16, num_warmup=1000, num_samples=5000)"
- [readme] early release caveat: "**jaxspec is still in early release**: expect bugs, breaking API changes, undocumented features and lack of functionalities"
- [abstract] identical results: "We obtain identical results when compared to other software and approaches, meaning that jaxspec provides reliable results while being ~10 times faster than existing alternatives."
