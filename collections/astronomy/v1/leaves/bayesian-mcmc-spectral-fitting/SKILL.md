---
name: bayesian-mcmc-spectral-fitting
description: Use when an X-ray spectrum (OGIP-standard PHA loaded into an ObsConfiguration)
  must be fitted with a composite additive/multiplicative model and Bayesian posterior
  constraints (credible intervals, corner plots, PPC) are required rather than a simple
  -fit statistic.
when_to_use_negative:
- You only need a quick point estimate for plotting or mock generation — jaxspec itself
  offers fakeit_for_multiple_parameters and ObsConfiguration.mock_from_instrument
  for simulation instead of full MCMC.
- The background contribution is negligible or SubtractedBackground()'s non-propagated
  dispersion is acceptable — but note the article warns this is 'clearly bad when
  we want to get spectral parameters with a comprehensive error budget'.
- jAX/GPU (or multi-core CPU for 16 chains) resources are unavailable, since NUTS
  sampling with 16 chains x 6000 steps per chain is compute-intensive; variational
  inference is the article's faster alternative for high-resolution data.
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: jaxspec
  role: X-ray spectral fitting library providing the model components, ObsConfiguration
    loaders, background models, and MCMCFitter/ FitResult API used for the Bayesian
    fit
  repo: https://github.com/renecotyfanboy/jaxspec
- name: JAX
  role: Backend generating differentiable, JIT-compilable Poisson likelihoods that
    enable the NUTS sampler
- name: Python 3.12
  role: Runtime environment; jaxspec supports Python 3.11 and 3.12 and installs via
    pip or conda
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202451736
    title: 'jaxspec: A fast and robust Python library for X-ray spectral fitting'
  source_content_hash: sha256:8b5856a99832c873cc03d6a3079833e9fc8008c560ff31c736531fc3540648e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/renecotyfanboy/jaxspec
  tools:
  - JAX
  - Python
  - Python 3.12
  - jaxspec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# bayesian-mcmc-spectral-fitting

## Summary

Fit X-ray spectral models to OGIP-standard PHA data with jaxspec's fully Bayesian MCMCFitter, which uses a no U-turn sampler (NUTS) over a differentiable Poisson (C-stat) likelihood built on JAX. This yields posterior parameter distributions with a comprehensive error budget instead of biased chi-square point estimates.

## When to use

Use when an X-ray spectrum (OGIP-standard PHA loaded into an ObsConfiguration) must be fitted with a composite additive/multiplicative model and Bayesian posterior constraints (credible intervals, corner plots, PPC) are required rather than a simple best-fit statistic.

## When NOT to use

- You only need a quick point estimate for plotting or mock generation — jaxspec itself offers fakeit_for_multiple_parameters and ObsConfiguration.mock_from_instrument for simulation instead of full MCMC.
- The background contribution is negligible or SubtractedBackground()'s non-propagated dispersion is acceptable — but note the article warns this is 'clearly bad when we want to get spectral parameters with a comprehensive error budget'.
- jAX/GPU (or multi-core CPU for 16 chains) resources are unavailable, since NUTS sampling with 16 chains x 6000 steps per chain is compute-intensive; variational inference is the article's faster alternative for high-resolution data.

## Inputs

- X-ray spectrum in the OGIP standard (PHA file), loaded as an ObsConfiguration with a specified energy band (e.g. low_energy=0.3, high_energy=12)
- Spectral model composed of jaxspec components (e.g. Tbabs(), Powerlaw(), Blackbodyrad(), or custom AdditiveComponent/MultiplicativeComponent subclasses)
- numpyro prior dictionary assigning dist.Uniform / dist.LogUniform to every free model parameter
- Background-handling choice: SubtractedBackground(), BackgroundWithError(), or SpectralModelBackground with background priors
- Packaged example data such as load_example_obsconf('NGC7793_ULX4_PN')

## Outputs

- FitResult with an inference_data.posterior group containing posterior sites and sampled draws for every model parameter
- Corner plot of posterior marginals (result.plot_corner())
- Posterior predictive check plot (result.plot_ppc(plot_components=True, n_sigmas=2))
- Posterior parameter summary table (result.table()) and comparison figure via plot_corner_comparison
- Serialised FitResult (e.g. pickled with dill)

## How to apply

Load the observation into an ObsConfiguration (e.g. from a PHA file with an energy band such as 0.3–12 keV) and compose a model from jaxspec components, e.g. Tbabs() * (Powerlaw() + Blackbodyrad()), declaring every parameter in a numpyro prior dict (dist.Uniform for e.g. nh or alpha, dist.LogUniform for norms spanning orders of magnitude). Prefer the Poisson likelihood (C-stat) over chi-square, since C-stat at all count rates is necessary to ensure an unbiased estimate of the parameters at low counts. Fit with MCMCFitter.fit using the demonstrated settings num_chains=16, num_warmup=1000, num_samples=5000, enabling 64-bit JAX/numpyro across multiple CPU devices (or a GPU) since JAX provides just-in-time compilation and automatic differentiation that enable NUTS. Choose the background model deliberately: SubtractedBackground() does not propagate dispersion and gives an incomplete error budget, so use BackgroundWithError() (each background bin a Poisson realisation) or SpectralModelBackground(Tbabs() * Powerlaw()) with its own priors when background spread matters. Inspect result.plot_corner() and result.plot_ppc(plot_components=True, n_sigmas=2), and compare variants with plot_corner_comparison from jaxspec.analysis.compare.

## Related tools

- **jaxspec** (X-ray spectral fitting library providing the model components, ObsConfiguration loaders, background models, and MCMCFitter/ FitResult API used for the Bayesian fit) — https://github.com/renecotyfanboy/jaxspec
- **JAX** (Backend generating differentiable, JIT-compilable Poisson likelihoods that enable the NUTS sampler)
- **Python 3.12** (Runtime environment; jaxspec supports Python 3.11 and 3.12 and installs via pip or conda)

## Examples

```
from jaxspec.fit import MCMCFitter; result = forward.fit(num_chains=16, num_warmup=1000, num_samples=5000); result.plot_corner()
```

## Evaluation signals

- MCMCFitter.fit completes with num_chains=16, num_warmup=1000, num_samples=5000 without error under the Poisson likelihood.
- result.inference_data.posterior contains sampled draws for every free parameter site (including any custom component parameters).
- Posterior predictive check plot (plot_ppc with plot_components=True, n_sigmas=2) is produced and shows acceptable model-data agreement.
- Parameter estimates are unbiased/identical to alternative software: the paper reports 'identical results when compared to other software and approaches' (reproducing Hitomi Perseus results), so cross-check against XSPEC-style fits where available.
- When comparing background variants, plot_corner_comparison shows the three labelled variants ('Background with no spread', 'Background with spread', 'Spectral background') and per-variant PPC plots exist.

## Limitations

- jaxspec is still in early release: expect bugs, breaking API changes, undocumented features and lack of functionalities.
- SubtractedBackground accounts for observed counts without propagating any dispersion, which is inadequate for a comprehensive error budget.
- MCMC is compute-heavy (16 chains, 1000 warmup + 5000 samples); the paper's alternative is variational inference, which produced convincing results on high-resolution data in less than 10 minutes on a GPU.
- No software version number for jaxspec is stated in the article, and no changelog was found, so reproducibility pinning must be done manually (e.g. via pip freeze).

## Evidence

- [abstract] Bayesian inference with NUTS and differentiable likelihood: "Based on the JAX ecosystem, jaxspec allows the generation of differentiable likelihood functions compilable on core or graphical process units (GPUs)"
- [methods] Poisson likelihood / C-stat preference: "Most recent publications on the subject agree that using the C-stat at all count rates is necessary to ensure an unbiased estimate of the parameters."
- [methods] Demonstrated fit configuration: "result = forward.fit(num_chains=16, num_warmup=1000, num_samples=5000)"
- [readme] Skill scope statement: "It can currently load an X-ray spectrum (in the OGIP standard), define a spectral model from the implemented components, and calculate the best parameters using state-of-the-art Bayesian approaches."
- [readme] Early-release caveat: "> :warning: **jaxspec is still in early release**: expect bugs, breaking API changes, undocumented features and lack of functionalities"
