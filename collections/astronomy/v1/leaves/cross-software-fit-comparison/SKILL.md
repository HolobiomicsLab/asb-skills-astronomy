---
name: cross-software-fit-comparison
description: Use when a composite X-ray spectral model has been fitted in one framework
  and you need to confirm the posterior parameter estimates are reliable by benchmarking
  them against a blind reference fit (e.g. XSPEC with C-stat) or another software
  approach.
when_to_use_negative:
- The input is not an OGIP-standard X-ray spectrum or the goal lies outside spectral
  fitting (jaxspec's scope is loading OGIP spectra, defining models from implemented
  components, and Bayesian parameter inference)
- You only need a quick chi-square point estimate without a Bayesian posterior; the
  article notes chi-square gives biased parameter estimates at low counts, so this
  comparison framework (Poisson/C-stat likelihood) would be unnecessary or misapplied
- A comprehensive background error budget requiring dispersion propagation is needed
  — the SubtractedBackground model accounts for observed counts without propagating
  any kind of dispersion
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: jaxspec
  role: Python library performing the Bayesian spectral fit with the no U-turn sampler
    whose results are compared across frameworks
  repo: https://github.com/renecotyfanboy/jaxspec
- name: numpyro
  role: Provides the NUTS sampler configuration (num_chains, num_warmup, num_samples,
    enable_x64, set_host_device_count) used for the comparison fit
- name: JAX
  role: Underpins jaxspec's just-in-time compilation and automatic differentiation
    of spectral models, enabling NUTS and the speed advantage
  repo: https://github.com/renecotyfanboy/jaxspec
- name: XSPEC
  role: Established framework used as the independent blind reference fit with C-stat
    for the cross-software comparison (XSPEC 12.15.1 behaviour referenced; Docker
    image available)
  repo: https://hub.docker.com/repository/docker/renecoty/heasoft/general
- name: dill
  role: Serialises the comparison result (result.pickle) for validation and reuse
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202451736
    title: 'jaxspec: A fast and robust Python library for X-ray spectral fitting'
  source_content_hash: sha256:e8747834b4fbc82c0efb69000ee607509c620d01769550323877387309a98680
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-software-fit-comparison

## Summary

Validates a Bayesian X-ray spectral fit by reproducing it in jaxspec (NUTS sampler under a Poisson/C-stat likelihood) and comparing posterior parameter estimates against an independent fit from an established framework such as XSPEC with C-stat, establishing equivalence of results and relative runtime.

## When to use

Use when a composite X-ray spectral model has been fitted in one framework and you need to confirm the posterior parameter estimates are reliable by benchmarking them against a blind reference fit (e.g. XSPEC with C-stat) or another software approach.

## When NOT to use

- The input is not an OGIP-standard X-ray spectrum or the goal lies outside spectral fitting (jaxspec's scope is loading OGIP spectra, defining models from implemented components, and Bayesian parameter inference)
- You only need a quick chi-square point estimate without a Bayesian posterior; the article notes chi-square gives biased parameter estimates at low counts, so this comparison framework (Poisson/C-stat likelihood) would be unnecessary or misapplied
- A comprehensive background error budget requiring dispersion propagation is needed — the SubtractedBackground model accounts for observed counts without propagating any kind of dispersion

## Inputs

- X-ray spectrum in OGIP standard (e.g. .pha file) with instrument response, restricted to a defined energy band such as 0.3–12 keV
- Composite spectral model definition (e.g. Tbabs()*(Powerlaw() + Blackbodyrad()) in jaxspec)
- Prior dictionary with explicit Uniform/LogUniform bounds per parameter (e.g. Uniform(0,5) on powerlaw_1.alpha, LogUniform(1e-5,1e-2) on powerlaw_1.norm, Uniform(0,1) on tbabs_1.nh)
- Reference fit from an independent framework (blind XSPEC fit with C-stat)
- MCMC sampler settings (num_chains, num_warmup, num_samples)

## Outputs

- Posterior parameter estimates (means and distributions) for the composite model
- Corner plot (result.plot_corner()) and posterior predictive check plot with components (result.plot_ppc(plot_components=True, n_sigmas=2))
- LaTeX parameter table (result.table())
- Serialised result (result.pickle, via dill)
- Cross-software comparison of parameter estimates and runtime (jaxspec reported ~10x faster than existing alternatives)

## How to apply

Load the target observation as an ObsConfiguration over a defined band (e.g. NGC 7793 ULX-4 XMM-Newton PN, 0.3–12 keV, via load_example_obsconf("NGC7793_ULX4_PN") or ObsConfiguration.from_pha_file with low_energy=0.3, high_energy=12). Build the composite model in jaxspec (e.g. Tbabs()*(Powerlaw() + Blackbodyrad())) and specify explicit priors (Uniform/LogUniform ranges on each component's parameters) so the prior treatment is comparable to the reference fit. Run MCMCFitter(...).fit(num_chains=16, num_warmup=1000, num_samples=5000) with numpyro.enable_x64() and numpyro.set_host_device_count(16) so the no U-turn sampler runs on all cores in double precision under the Poisson (C-stat-equivalent) likelihood; C-stat at all count rates is required for unbiased parameter estimates. Perform an independent blind XSPEC fit (XSPEC 12.15.1; a docker image is available) using C-stat, then compare posterior means against the XSPEC reference values, record runtime, and serialise the result with dill (result.pickle).

## Related tools

- **jaxspec** (Python library performing the Bayesian spectral fit with the no U-turn sampler whose results are compared across frameworks) — https://github.com/renecotyfanboy/jaxspec
- **numpyro** (Provides the NUTS sampler configuration (num_chains, num_warmup, num_samples, enable_x64, set_host_device_count) used for the comparison fit)
- **JAX** (Underpins jaxspec's just-in-time compilation and automatic differentiation of spectral models, enabling NUTS and the speed advantage) — https://github.com/renecotyfanboy/jaxspec
- **XSPEC** (Established framework used as the independent blind reference fit with C-stat for the cross-software comparison (XSPEC 12.15.1 behaviour referenced; Docker image available)) — https://hub.docker.com/repository/docker/renecoty/heasoft/general
- **dill** (Serialises the comparison result (result.pickle) for validation and reuse)

## Examples

```
obs = ObsConfiguration.from_pha_file('your.pha', low_energy=0.3, high_energy=12); result = forward.fit(num_chains=16, num_warmup=1000, num_samples=5000)
```

## Evaluation signals

- Recovered posterior means agree with the blind XSPEC C-stat reference values (the article reports identical results when compared to other software and approaches)
- Runtime is materially lower than the reference framework (jaxspec was ~10 times faster than existing alternatives)
- Both diagnostic figures are produced: the corner plot and the posterior predictive plot (plot_ppc with plot_components=True, n_sigmas=2)
- result.pickle (serialised with dill) exists and reloads the fitted comparison result
- Sampler ran in double precision across 16 chains (numpyro.enable_x64(), set_host_device_count(16)) with the Poisson/C-stat likelihood, not chi-square

## Limitations

- jaxspec is still in early release: expect bugs, breaking API changes, undocumented features and lack of functionalities
- The SubtractedBackground model accounts for observed counts without propagating any kind of dispersion, which is inadequate for a comprehensive error budget
- The jaxspec software version number is not stated (only Python 3.11/3.12 and XSPEC 12.15.1 are referenced) and no changelog was found; chi-square fitting gives biased parameter estimates at low counts, so comparisons must use the C-stat

## Evidence

- [abstract] We obtain identical results when compared to other software and approaches, meaning that jaxspec provides reliable results while being ~10 times faster than existing alternatives.: "We obtain identical results when compared to other software and approaches, meaning that jaxspec provides reliable results while being ~10 times faster than existing alternatives."
- [methods] Most recent publications on the subject agree that using the C-stat at all count rates is necessary to ensure an unbiased estimate of the parameters.: "Most recent publications on the subject agree that using the C-stat at all count rates is necessary to ensure an unbiased estimate of the parameters."
- [methods] result = forward.fit(num_chains=16, num_warmup=1000, num_samples=5000): "result = forward.fit(num_chains=16, num_warmup=1000, num_samples=5000)"
- [readme] It is built on top of JAX to provide just-in-time compilation and automatic differentiation of the spectral models, enabling the use of sampling algorithm such as NUTS.: [evidence span withheld — anchor status: external_doc]
- [readme] **jaxspec is still in early release**: expect bugs, breaking API changes, undocumented features and lack of functionalities: "**jaxspec is still in early release**: expect bugs, breaking API changes, undocumented features and lack of functionalities"
