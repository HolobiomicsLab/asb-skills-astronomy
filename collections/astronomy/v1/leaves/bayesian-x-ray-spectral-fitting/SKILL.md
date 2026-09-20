---
name: bayesian-x-ray-spectral-fitting
description: Use when an X-ray spectrum in the OGIP standard (PHA with ARF/RMF responses)
  must be fitted with a composite spectral model under a fully Bayesian framework
  with priors over spectral parameters.
when_to_use_negative:
- When a purely forward-folding XSPEC-style fit without Bayesian priors is desired
  and a HEASoft/chi-square workflow is already in place — jaxspec's Bayesian framework
  and early-release API may not match that use case
- When the background requires full dispersion propagation with a comprehensive error
  budget — the SubtractedBackground model simply accounts for observed counts without
  propagating any dispersion
- When the environment cannot provide Python 3.11/3.12 (or conda) and JAX compilation
  targets, since jaxspec is a pure-Python JAX-based library
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: jaxspec
  role: 'Core library: loads OGIP-standard spectra, composes spectral models, exposes
    dict_prior, joint_prior_factory, TiedParameter and MCMCFitter/variational fitters'
  repo: https://github.com/renecotyfanboy/jaxspec
- name: JAX
  role: Backend generating differentiable likelihood functions with just-in-time compilation
    and automatic differentiation, runnable on CPU or GPU
- name: numpyro
  role: Provides prior distributions (dist.Uniform, dist.LogUniform, dist.MultivariateNormal),
    the NUTS sampler, and platform/precision configuration (set_platform, enable_x64)
- name: dill
  role: Serialises the FitResult into result.pickle
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202451736
    title: 'jaxspec: A fast and robust Python library for X-ray spectral fitting'
  source_content_hash: sha256:bb1013fabdc88390f279cee24503cd7fd14d1dd46cedbfd8e780ff0c7d0e6024
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# bayesian-x-ray-spectral-fitting

## Summary

Fully Bayesian fitting of X-ray spectra using jaxspec, which is built on JAX to generate differentiable likelihood functions compilable on CPU or GPU, enabling state-of-the-art samplers (no U-turn sampler, MCMC) and variational inference over spectral parameters.

## When to use

Use when an X-ray spectrum in the OGIP standard (PHA with ARF/RMF responses) must be fitted with a composite spectral model under a fully Bayesian framework with priors over spectral parameters.

## When NOT to use

- When a purely forward-folding XSPEC-style fit without Bayesian priors is desired and a HEASoft/chi-square workflow is already in place — jaxspec's Bayesian framework and early-release API may not match that use case
- When the background requires full dispersion propagation with a comprehensive error budget — the SubtractedBackground model simply accounts for observed counts without propagating any dispersion
- When the environment cannot provide Python 3.11/3.12 (or conda) and JAX compilation targets, since jaxspec is a pure-Python JAX-based library

## Inputs

- X-ray spectrum in the OGIP standard (PHA file) with associated ARF/RMF response, e.g. ObsConfiguration.from_pha_file('your.pha', low_energy=0.3, high_energy=12)
- Spectral model composed of jaxspec additive/multiplicative components (e.g. Tbabs() * (Powerlaw() + Powerlaw() + Blackbodyrad()))
- Prior specification: dict_prior entries, joint_prior_factory correlated draws, TiedParameter links
- Packaged example observations such as load_example_obsconf('NGC7793_ULX4_PN') or the Hitomi Perseus cluster observation

## Outputs

- FitResult with result.inference_data.posterior containing numpyro sites named 'forward.spectrum.<obs>.<path>' plus deterministic tie sites
- Posterior plots via result.plot_ppc(plot_components=True, n_sigmas=2) and result.plot_corner()
- Serialised fit object written with dill.dump to result.pickle
- Fitted spectral parameters comparable against published results (e.g. Hitomi Perseus cluster)

## How to apply

Load the observation into an ObsConfiguration (e.g. ObsConfiguration.from_pha_file('your.pha', low_energy=0.3, high_energy=12)) band-limited to the fit range, and define a composite model such as Tbabs() * (Powerlaw() + Powerlaw() + Blackbodyrad()) from jaxspec's implemented additive/multiplicative components. Build a prior dict with jaxspec.fit.dict_prior using numpyro distributions (e.g. Uniform(0,1) for spectrum.tbabs_1.nh, LogUniform(1e-5,1e-2) for per-observation norm entries with '[*]' suffix); add correlated draws via jaxspec.fit.joint_prior_factory (e.g. sampling alpha and norm jointly from dist.MultivariateNormal) and parameter ties via TiedParameter with a scaling function such as lambda x: 0.5 * x. Fit under a Poisson likelihood (C-stat) rather than chi-square, since C-stat is necessary at all count rates to ensure unbiased parameter estimates. Run MCMCFitter with the NUTS sampler (e.g. num_chains=16, num_warmup=1000, num_samples=5000), or use variational inference on a GPU (after numpyro.set_platform('gpu') and numpyro.enable_x64()) for high-resolution data. Inspect result.inference_data.posterior for expected numpyro site names (per-observation sites under 'forward.spectrum.<obs>.<path>' and deterministic tie sites), then serialise the FitResult with dill.dump into result.pickle.

## Related tools

- **jaxspec** (Core library: loads OGIP-standard spectra, composes spectral models, exposes dict_prior, joint_prior_factory, TiedParameter and MCMCFitter/variational fitters) — https://github.com/renecotyfanboy/jaxspec
- **JAX** (Backend generating differentiable likelihood functions with just-in-time compilation and automatic differentiation, runnable on CPU or GPU)
- **numpyro** (Provides prior distributions (dist.Uniform, dist.LogUniform, dist.MultivariateNormal), the NUTS sampler, and platform/precision configuration (set_platform, enable_x64))
- **dill** (Serialises the FitResult into result.pickle)

## Examples

```
pip install jaxspec --upgrade; obs = ObsConfiguration.from_pha_file('your.pha', low_energy=0.3, high_energy=12); result = forward.fit(num_chains=16, num_warmup=1000, num_samples=5000)
```

## Evaluation signals

- result.inference_data.posterior contains the expected numpyro site names: per-observation sites under 'forward.spectrum.<obs>.<path>' and deterministic sites for tied parameters
- Fitted posteriors are identical to those from other software/approaches while running ~10 times faster
- Variational inference on the Hitomi Perseus high-resolution data reproduces the Hitomi collaboration's published results in under 10 minutes on a GPU
- Posterior predictive check via result.plot_ppc(plot_components=True, n_sigmas=2) shows model counts consistent with observed counts
- result.pickle exists and reloads the complete FitResult after dill.dump

## Limitations

- jaxspec is still in early release: expect bugs, breaking API changes, undocumented features and lack of functionalities
- The SubtractedBackground model does not propagate dispersion, which is inadequate for a comprehensive error budget
- C-stat (Poisson likelihood) must be used at all count rates; chi-square fitting biases parameter estimates at low counts
- Software version number for jaxspec is not stated (only Python 3.11/3.12 and XSPEC 12.15.1 are referenced)

## Evidence

- [abstract] jaxspec generates differentiable likelihoods with JAX-backed priors and NUTS sampling: "Based on the JAX ecosystem, jaxspec allows the generation of differentiable likelihood functions compilable on core or graphical process units (GPUs)"
- [abstract] NUTS demonstrated on a composite model: "We demonstrate the effectiveness of jaxspec samplers, in particular the no U-turn sampler, using a composite model"
- [intro] jaxspec loads OGIP spectra and fits with Bayesian approaches: "It can currently load an X-ray spectrum (in the OGIP standard), define a spectral model from the implemented components, and calculate the best parameters using state-of-the-art Bayesian approaches."
- [methods] C-stat required for unbiased estimates: "Most recent publications on the subject agree that using the C-stat at all count rates is necessary to ensure an unbiased estimate of the parameters."
- [methods] MCMCFitter call signature: "result = forward.fit(num_chains=16, num_warmup=1000, num_samples=5000)"
- [readme] jaxspec installs from PyPI in a fresh Python 3.12 environment: [evidence span withheld — anchor status: external_doc]
- [readme] early-release caveat: "**jaxspec is still in early release**: expect bugs, breaking API changes, undocumented features and lack of functionalities"
