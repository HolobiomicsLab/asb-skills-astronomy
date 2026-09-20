---
name: bayesian-mcmc-inference-poisson-likelihood
description: Use when an X-ray spectrum (OGIP standard, e.g. an XMM-Newton PHA) needs
  fully Bayesian parameter estimation for a composite model, especially at low counts
  where C-stat is preferred over chi-square to avoid biased estimates.
when_to_use_negative:
- The task requires a comprehensive error budget with background dispersion propagated
  — the SubtractedBackground model does not propagate dispersion, which is clearly
  bad in that case; use BackgroundWithError instead.
- The inference problem involves high-resolution data and a GPU is available where
  variational inference suffices (jaxspec demonstrated VI on the Hitomi Perseus cluster
  data in less than 10 minutes on a GPU).
- A quick chi-square least-squares style fit is the intended output rather than a
  Bayesian posterior — this skill targets state-of-the-art Bayesian approaches with
  explicit priors.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3297
tools:
- name: jaxspec
  role: Provides the spectral model components, ObsConfiguration loaders, MCMCFitter
    wrapper, and plot_corner/plot_ppc/table result methods
  repo: https://github.com/renecotyfanboy/jaxspec
- name: numpyro
  role: Hosts the no U-turn sampler; configured with enable_x64() and set_host_device_count(16)
    for 16 parallel chains in double precision
- name: JAX
  role: Underlying framework providing just-in-time compilation and automatic differentiation
    of the differentiable likelihood functions
- name: XSPEC
  role: Reference framework used for the blind C-stat fit against which posterior
    means are compared to validate jaxspec's results
- name: dill
  role: Serialises the fitted result object to result.pickle for validation/archival
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202451736
    title: 'jaxspec: A fast and robust Python library for X-ray spectral fitting'
  source_content_hash: sha256:41f52cbb542ef2d5c625a57e4228a3d47cccaec4f05dc15f69cf8296ea79311f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/renecotyfanboy/jaxspec
  tools:
  - JAX
  - Python
  - XSPEC
  - dill
  - jaxspec
  - matplotlib
  - numpyro
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3297
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# bayesian-mcmc-inference-poisson-likelihood

## Summary

Bayesian inference of X-ray spectral parameters with jaxspec's MCMCFitter driving the numpyro no U-turn sampler under a Poisson (C-stat-equivalent) likelihood, using differentiable JAX-compiled likelihoods for composite models such as Tbabs()*(Powerlaw() + Blackbodyrad()).

## When to use

Use when an X-ray spectrum (OGIP standard, e.g. an XMM-Newton PHA) needs fully Bayesian parameter estimation for a composite model, especially at low counts where C-stat is preferred over chi-square to avoid biased estimates.

## When NOT to use

- The task requires a comprehensive error budget with background dispersion propagated — the SubtractedBackground model does not propagate dispersion, which is clearly bad in that case; use BackgroundWithError instead.
- The inference problem involves high-resolution data and a GPU is available where variational inference suffices (jaxspec demonstrated VI on the Hitomi Perseus cluster data in less than 10 minutes on a GPU).
- A quick chi-square least-squares style fit is the intended output rather than a Bayesian posterior — this skill targets state-of-the-art Bayesian approaches with explicit priors.

## Inputs

- X-ray spectrum in OGIP standard (e.g. XMM-Newton PHA file loaded via ObsConfiguration.from_pha_file('your.pha', low_energy=0.3, high_energy=12))
- Composite spectral model built from jaxspec components (e.g. Tbabs()*(Powerlaw() + Blackbodyrad()))
- Prior dictionary mapping component parameters (e.g. alpha, kT, nh, norms) to Uniform/LogUniform distributions

## Outputs

- Posterior samples for all model parameters (num_samples=5000 per chain, 16 chains)
- Corner plot of the posterior (result.plot_corner())
- Posterior predictive check plot with model components at n_sigmas=2 (result.plot_ppc)
- LaTeX parameter table (result.table())
- Serialised fit result (e.g. result.pickle via dill)

## How to apply

Load the observation as an ObsConfiguration over a defined band (e.g. low_energy=0.3, high_energy=12 keV for the NGC 7793 ULX-4 XMM-Newton PN data), then compose the model from implemented components, e.g. Tbabs()*(Powerlaw() + Blackbodyrad()). Specify explicit priors per component — the paper's example uses Uniform(0,5) on powerlaw_1.alpha, LogUniform(1e-5,1e-2) on its norm, Uniform(0,5) on blackbodyrad_1.kT, LogUniform(1e-2,1e2) on its norm, and Uniform(0,1) on tbabs_1.nh — then call MCMCFitter(model, prior, obs).fit(num_chains=16, num_warmup=1000, num_samples=5000) with numpyro.enable_x64() and numpyro.set_host_device_count(16) so NUTS uses all cores in double precision. The Poisson likelihood (C-stat equivalent) is the deliberate default since the article notes C-stat is necessary at all count rates for unbiased parameters. Inspect posterior distributions via result.plot_corner(), result.plot_ppc(plot_components=True, n_sigmas=2), and result.table(); validate by confirming posterior means reproduce a blind XSPEC fit with C-stat, as jaxspec yields identical results while being ~10x faster.

## Related tools

- **jaxspec** (Provides the spectral model components, ObsConfiguration loaders, MCMCFitter wrapper, and plot_corner/plot_ppc/table result methods) — https://github.com/renecotyfanboy/jaxspec
- **numpyro** (Hosts the no U-turn sampler; configured with enable_x64() and set_host_device_count(16) for 16 parallel chains in double precision)
- **JAX** (Underlying framework providing just-in-time compilation and automatic differentiation of the differentiable likelihood functions)
- **XSPEC** (Reference framework used for the blind C-stat fit against which posterior means are compared to validate jaxspec's results)
- **dill** (Serialises the fitted result object to result.pickle for validation/archival)

## Examples

```
from jaxspec.fit import MCMCFitter; import numpyro; numpyro.enable_x64(); numpyro.set_host_device_count(16); result = MCMCFitter(model, prior, obs).fit(num_chains=16, num_warmup=1000, num_samples=5000)
```

## Evaluation signals

- Recovered posterior means agree with a blind XSPEC fit performed with C-stat (the paper reports identical results between frameworks).
- result.pickle exists after serialisation and both the corner plot and the PPC figures are produced.
- PPC plot drawn with plot_components=True, n_sigmas=2 shows the model and components consistent with the data within 2 sigma.
- Sampling ran with numpyro.enable_x64() and num_chains=16, num_warmup=1000, num_samples=5000 as specified in the workflow.
- Posterior parameter values stay within their explicit priors (e.g. tbabs_1.nh in Uniform(0,1), blackbodyrad_1.kT in Uniform(0,5)).

## Limitations

- jaxspec is still in early release: expect bugs, breaking API changes, undocumented features and lack of functionalities (per README warning).
- The software version of jaxspec itself is not stated in the article (only Python 3.11/3.12 and XSPEC 12.15.1 are referenced), complicating reproducibility.
- Fitting under a Poisson likelihood (C-stat) is required at all count rates; chi-square alternatives can bias parameter estimates at low counts.
- Using SubtractedBackground does not propagate background dispersion, producing an incomplete error budget.

## Evidence

- [abstract] jaxspec yields identical results to other software: "We obtain identical results when compared to other software and approaches, meaning that jaxspec provides reliable results while being ~10 times faster than existing alternatives."
- [methods] Poisson likelihood rationale: "Most recent publications on the subject agree that using the C-stat at all count rates is necessary to ensure an unbiased estimate of the parameters."
- [abstract] NUTS on composite model: "We demonstrate the effectiveness of jaxspec samplers, in particular the no U-turn sampler, using a composite model"
- [methods] fit call parameters: "result = forward.fit(num_chains=16, num_warmup=1000, num_samples=5000)"
- [readme] early release warning: "**jaxspec is still in early release**: expect bugs, breaking API changes, undocumented features and lack of functionalities"
- [intro] JAX foundation: [evidence span withheld — anchor status: external_doc]
