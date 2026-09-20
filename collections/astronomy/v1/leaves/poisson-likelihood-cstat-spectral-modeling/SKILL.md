---
name: poisson-likelihood-cstat-spectral-modeling
description: Use when fitting X-ray spectra (OGIP PHA/ARF/RMF, e.g. XMM-Newton or
  Hitomi observations) where counts are low and chi-square fitting would bias parameter
  estimates, or when a differentiable likelihood is needed for Bayesian inference.
when_to_use_negative:
- When input spectra are already binned/processed such that a chi-square fit on Gaussian
  errors is standard practice and counts are high — the article's rationale for C-stat
  targets low-count Poisson data
- When a comprehensive error budget is required and the only available background
  option is SubtractedBackground, which does not propagate dispersion
- When a non-Python HEASoft/XSPEC-dependent pipeline must be preserved — jaxspec is
  pure Python with no HEASoft dependency and is still in early release with expected
  API changes
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: jaxspec
  role: Provides the differentiable Poisson-likelihood (C-stat) spectral fitting framework,
    OGIP spectrum loading, spectral model components, background models, and Bayesian
    fitters
  repo: https://github.com/renecotyfanboy/jaxspec
- name: JAX
  role: Underpins jaxspec by providing JIT compilation and automatic differentiation
    of the spectral models and likelihood, enabling NUTS sampling and GPU execution
  repo: https://github.com/renecotyfanboy/jaxspec
- name: numpyro
  role: Supplies prior distributions (dist.Uniform, dist.LogUniform) and the inference
    backends (NUTS sampler, variational inference, GPU platform selection with enable_x64)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202451736
    title: 'jaxspec: A fast and robust Python library for X-ray spectral fitting'
  source_content_hash: sha256:365ef2411aa88c86110932433d897b5abfcafe36529dd2a2e9bfd735ca0d6d90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/renecotyfanboy/jaxspec
  tools:
  - JAX
  - Python
  - jaxspec
  - numpyro
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Poisson-likelihood (C-stat) spectral modeling

## Summary

Fits X-ray spectra under a Poisson likelihood (C-statistic) rather than chi-square, so parameter estimates remain unbiased at low counts; in jaxspec this likelihood is differentiable and JAX-compilable, enabling Bayesian sampling (NUTS) or GPU variational inference.

## When to use

Use when fitting X-ray spectra (OGIP PHA/ARF/RMF, e.g. XMM-Newton or Hitomi observations) where counts are low and chi-square fitting would bias parameter estimates, or when a differentiable likelihood is needed for Bayesian inference.

## When NOT to use

- When input spectra are already binned/processed such that a chi-square fit on Gaussian errors is standard practice and counts are high — the article's rationale for C-stat targets low-count Poisson data
- When a comprehensive error budget is required and the only available background option is SubtractedBackground, which does not propagate dispersion
- When a non-Python HEASoft/XSPEC-dependent pipeline must be preserved — jaxspec is pure Python with no HEASoft dependency and is still in early release with expected API changes

## Inputs

- X-ray spectrum in the OGIP standard (PHA file with associated ARF and RMF response files), e.g. an XMM-Newton observation of NGC 7793 ULX-4 or a Hitomi Perseus cluster observation
- Optional background PHA spectrum for background handling models
- Composite spectral model definition built from jaxspec additive/multiplicative components plus a prior dict (numpyro dist.Uniform, dist.LogUniform, constants)

## Outputs

- Posterior parameter estimates (FitResult) from the Poisson-likelihood Bayesian fit, serialisable with dill
- Corner plot of posterior parameters (plot_corner) and posterior predictive check plot (plot_ppc with components and n_sigmas bands)

## How to apply

Load the spectrum in the OGIP standard into an ObsConfiguration from the PHA file together with its ARF/RMF response, band-limiting the fit range (e.g. 0.3–12 keV). Build a composite spectral model from jaxspec's implemented additive/multiplicative components, attaching priors with numpyro distributions and fixing calibrated parameters as constants. Evaluate the model under a Poisson likelihood (C-stat) instead of chi-square, because the literature consensus is that C-stat is required at all count rates for unbiased parameter estimates. Handle the background explicitly with one of jaxspec's background models (SubtractedBackground, BackgroundWithError, or SpectralModelBackground), preferring BackgroundWithError when a comprehensive error budget matters. Solve the inference with a state-of-the-art Bayesian sampler (e.g. NUTS via MCMCFitter.fit with num_chains, num_warmup, num_samples) or GPU variational inference for high-resolution data, then inspect posteriors with plot_corner and posterior predictive checks with plot_ppc.

## Related tools

- **jaxspec** (Provides the differentiable Poisson-likelihood (C-stat) spectral fitting framework, OGIP spectrum loading, spectral model components, background models, and Bayesian fitters) — https://github.com/renecotyfanboy/jaxspec
- **JAX** (Underpins jaxspec by providing JIT compilation and automatic differentiation of the spectral models and likelihood, enabling NUTS sampling and GPU execution) — https://github.com/renecotyfanboy/jaxspec
- **numpyro** (Supplies prior distributions (dist.Uniform, dist.LogUniform) and the inference backends (NUTS sampler, variational inference, GPU platform selection with enable_x64))

## Examples

```
obs = ObsConfiguration.from_pha_file('your.pha', low_energy=0.3, high_energy=12); result = forward.fit(num_chains=16, num_warmup=1000, num_samples=5000)
```

## Evaluation signals

- Fitted posterior parameters reproduce published reference results (e.g. the Hitomi collaboration's Perseus cluster results or agreement with XSPEC-based fits on the NGC 7793 ULX-4 XMM-Newton observation)
- Posterior predictive check (result.plot_ppc) shows observed counts within the plotted n_sigmas band of the model, including components
- Parameter estimates remain unbiased at low counts — no systematic deviation attributable to chi-square-style fitting, since C-stat should be used at all count rates
- Background treatment is appropriate to the error budget: dispersion is propagated (BackgroundWithError/SpectralModelBackground) rather than simply subtracted when a comprehensive error budget is needed
- Sampler diagnostics indicate convergence (e.g. NUTS chains with num_warmup=1000, num_samples=5000 over 16 chains produce stable posteriors in plot_corner)

## Limitations

- The SubtractedBackground model does not propagate dispersion, which is inadequate when spectral parameters need a comprehensive error budget
- jaxspec is still in early release: expect bugs, breaking API changes, undocumented features and lack of functionalities
- Two behaviours deliberately depart from XSPEC 12.15.1 to keep a sampler alive; results should be compared against XSPEC with this in mind
- No jaxspec version number is stated in the article (only Python 3.11/3.12 support and XSPEC 12.15.1 as reference), and no changelog is available

## Evidence

- [methods] C-stat preferred over chi-square to avoid biased parameter estimates at low counts: "Most recent publications on the subject agree that using the C-stat at all count rates is necessary to ensure an unbiased estimate of the parameters."
- [methods] SubtractedBackground does not propagate dispersion: "The `SubtractedBackground` simply account for the observed counts without propagating any kind of dispersion, which is clearly bad when we want to get spectral parameters with a comprehensive error"
- [intro] jaxspec loads OGIP spectra, defines models, computes best parameters via Bayesian approaches: "It can currently load an X-ray spectrum (in the OGIP standard), define a spectral model from the implemented components, and calculate the best parameters using state-of-the-art Bayesian approaches."
- [readme] JAX gives JIT compilation and autodiff of spectral models enabling NUTS: [evidence span withheld — anchor status: external_doc]
- [abstract] jaxspec produces identical results to other software while ~10x faster: "We obtain identical results when compared to other software and approaches, meaning that jaxspec provides reliable results while being ~10 times faster than existing alternatives."
