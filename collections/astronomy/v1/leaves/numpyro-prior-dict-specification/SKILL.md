---
name: numpyro-prior-dict-specification
description: Use when building the prior dict for a jaxspec MCMC fit — e.g. setting
  Uniform(0,1) for nh, Uniform(0,5) for alpha and kT, LogUniform(1e-5,1e-2) for powerlaw
  norm on a 0.3–12 keV XMM-Newton PN spectrum, or background.*-prefixed priors for
  SpectralModelBackground.
when_to_use_negative:
- 'When using SubtractedBackground: it carries no background parameters, so background.*
  prior entries are meaningless there — pick BackgroundWithError or SpectralModelBackground
  if background priors must be sampled.'
- Input is not an OGIP-standard PHA spectrum loadable by ObsConfiguration, since jaxspec's
  loading path expects that format.
- The goal is frequentist chi-square calibration rather than Bayesian inference; jaxspec
  targets state-of-the-art Bayesian approaches under a Poisson likelihood.
edam_operation: http://edamontology.org/operation_2238
tools:
- name: jaxspec
  role: X-ray spectral fitting library whose MCMCFitter consumes the numpyro prior
    dict alongside the model and background model
  repo: https://github.com/renecotyfanboy/jaxspec
- name: JAX
  role: Backend providing differentiable likelihood functions and the numpyro sampling
    ecosystem; enable 64-bit mode for the fit
- name: numpyro
  role: Provides the distribution classes (Uniform, LogUniform) used as prior values
    and the NUTS sampler driven by MCMCFitter
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202451736
    title: 'jaxspec: A fast and robust Python library for X-ray spectral fitting'
  source_content_hash: sha256:b3822555e561a6684a9c3609bed787547ab4308a6599db66ff428ccf98a84c7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/renecotyfanboy/jaxspec
  tools:
  - JAX
  - Python
  - jaxspec
  - numpyro
  edam_operation: http://edamontology.org/operation_2238
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# numpyro-prior-dict-specification

## Summary

Specification of numpyro prior dictionaries (Uniform/LogUniform distributions keyed by dotted component.parameter names) that parameterise jaxspec's MCMCFitter for X-ray spectral models such as Tbabs()*(Powerlaw()+Blackbodyrad()), directly shaping the posterior and error budget under a Poisson (C-stat) likelihood.

## When to use

Use when building the prior dict for a jaxspec MCMC fit — e.g. setting Uniform(0,1) for nh, Uniform(0,5) for alpha and kT, LogUniform(1e-5,1e-2) for powerlaw norm on a 0.3–12 keV XMM-Newton PN spectrum, or background.*-prefixed priors for SpectralModelBackground.

## When NOT to use

- When using SubtractedBackground: it carries no background parameters, so background.* prior entries are meaningless there — pick BackgroundWithError or SpectralModelBackground if background priors must be sampled.
- Input is not an OGIP-standard PHA spectrum loadable by ObsConfiguration, since jaxspec's loading path expects that format.
- The goal is frequentist chi-square calibration rather than Bayesian inference; jaxspec targets state-of-the-art Bayesian approaches under a Poisson likelihood.

## Inputs

- OGIP-standard PHA spectrum loaded via ObsConfiguration.from_pha_file with low_energy=0.3, high_energy=12 (e.g. load_example_obsconf('NGC7793_ULX4_PN'))
- jaxspec spectral model expression, e.g. Tbabs() * (Powerlaw() + Blackbodyrad())
- numpyro distribution instances (Uniform, LogUniform) for each free parameter
- Background model choice: SubtractedBackground(), BackgroundWithError(), or SpectralModelBackground(Tbabs() * Powerlaw())

## Outputs

- Prior dict consumed by MCMCFitter
- Fitted result object with posterior samples and result.table() of parameter means/credible intervals
- Posterior predictive check (plot_ppc) and corner plots, plus background_comparison.png from plot_corner_comparison

## How to apply

Construct a Python dict whose keys are dotted component.parameter names matching the built jaxspec model and whose values are numpyro distributions; for Tbabs()*(Powerlaw()+Blackbodyrad()) fitted to the NGC 7793 ULX-4 PN observation, use nh: Uniform(0,1), alpha: Uniform(0,5), norm: LogUniform(1e-5,1e-2), kT: Uniform(0,5), and the blackbody norm: LogUniform(1e-2,1e2). When the background is modelled with SpectralModelBackground(Tbabs()*Powerlaw()), extend the same dict with background-prefixed keys — background.tbabs_1.nh Uniform(0,1), background.powerlaw_1.alpha Uniform(0,5), background.powerlaw_1.norm LogUniform(1e-8,1e-3) — so background dispersion is sampled jointly with source parameters. Pass the dict to MCMCFitter together with the background model choice and run fit(num_chains=16, num_warmup=1000, num_samples=5000) with 64-bit JAX/numpyro enabled on 16 CPU devices. Rationale: jaxspec fits under a Poisson likelihood (C-stat), so prior support bounds (e.g. alpha, kT in (0,5)) directly constrain the posterior; SubtractedBackground exposes no background parameters, so no background.* priors exist for that variant and its error budget lacks propagated dispersion.

## Related tools

- **jaxspec** (X-ray spectral fitting library whose MCMCFitter consumes the numpyro prior dict alongside the model and background model) — https://github.com/renecotyfanboy/jaxspec
- **JAX** (Backend providing differentiable likelihood functions and the numpyro sampling ecosystem; enable 64-bit mode for the fit)
- **numpyro** (Provides the distribution classes (Uniform, LogUniform) used as prior values and the NUTS sampler driven by MCMCFitter)

## Examples

```
prior = {'nh': Uniform(0, 1), 'alpha': Uniform(0, 5), 'norm': LogUniform(1e-5, 1e-2), 'kT': Uniform(0, 5)}
```

## Evaluation signals

- Every prior key resolves to a component/parameter of the built jaxspec model (e.g. dotted names like background.powerlaw_1.norm for SpectralModelBackground); mismatched keys fail before sampling.
- Posterior estimates for bounded parameters fall strictly inside their prior support: nh in (0,1), alpha and kT in (0,5), norms positive under LogUniform.
- Fit completes with the documented sampler settings (num_chains=16, num_warmup=1000, num_samples=5000) and 64-bit JAX enabled.
- All three background variants yield PPC plots, and background_comparison.png exists with the three labelled variants ('Background with no spread', 'Background with spread', 'Spectral background').
- result.table() returns means/credible intervals for every parameter present in the prior dict.

## Limitations

- jaxspec is in early release: expect bugs, breaking API changes, undocumented features and lack of functionalities (README warning).
- The jaxspec software version is not stated in the article (only Python 3.11/3.12 support is referenced), so prior-dict key syntax may change between releases.
- Specifying priors cannot fix SubtractedBackground's failure to propagate dispersion — that variant remains inadequate for a comprehensive error budget regardless of prior choice.

## Evidence

- [methods] Most recent publications on the subject agree that using the C-stat at all count rates is necessary to ensure an unbiased estimate of the parameters.: "Most recent publications on the subject agree that using the C-stat at all count rates is necessary to ensure an unbiased estimate of the parameters."
- [methods] The `SubtractedBackground` simply account for the observed counts without propagating any kind of dispersion: "The `SubtractedBackground` simply account for the observed counts without propagating any kind of dispersion"
- [intro] load an X-ray spectrum (in the OGIP standard), define a spectral model from the implemented components: "load an X-ray spectrum (in the OGIP standard), define a spectral model from the implemented components"
- [readme] expect bugs, breaking API changes, undocumented features and lack of functionalities: "expect bugs, breaking API changes, undocumented features and lack of functionalities"
- [methods] result = forward.fit(num_chains=16, num_warmup=1000, num_samples=5000): "result = forward.fit(num_chains=16, num_warmup=1000, num_samples=5000)"
