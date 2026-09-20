---
name: corner-plot-posterior-comparison
description: Use when you have two or more completed jaxspec Bayesian fits (MCMCFitter
  results with posterior samples) and need a visual, side-by-side comparison of their
  posterior parameter distributions, such as comparing background-handling models
  on the same observation.
when_to_use_negative:
- You have only a single fit — use result.plot_corner() directly instead of the comparison
  overlay
- The fits come from different observations or energy bands, so posterior differences
  are not attributable to the model choice being compared
- You need a quantitative model-comparison metric (e.g. Bayes factor) rather than
  a visual posterior overlay
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: jaxspec
  role: Provides plot_corner_comparison (jaxspec.analysis.compare), result.plot_corner()
    and result.table() used to visualise and compare posterior distributions of the
    fitted variants
  repo: https://github.com/renecotyfanboy/jaxspec
- name: JAX
  role: Backend enabling the differentiable likelihoods and 64-bit MCMC sampling whose
    posteriors are compared in the corner plots
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202451736
    title: 'jaxspec: A fast and robust Python library for X-ray spectral fitting'
  source_content_hash: sha256:b3ae5488860829a4605a33f42d4d4f355f4c0fdd07913a0d48a87378dc431a1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/renecotyfanboy/jaxspec
  tools:
  - JAX
  - Python
  - jaxspec
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# corner-plot-posterior-comparison

## Summary

Generate per-fit corner plots of posterior parameter distributions from jaxspec MCMC fits and overlay them with plot_corner_comparison to compare, e.g., the SubtractedBackground, BackgroundWithError and SpectralModelBackground variants fitted to the XMM-Newton NGC 7793 ULX-4 PN spectrum.

## When to use

Use when you have two or more completed jaxspec Bayesian fits (MCMCFitter results with posterior samples) and need a visual, side-by-side comparison of their posterior parameter distributions, such as comparing background-handling models on the same observation.

## When NOT to use

- You have only a single fit — use result.plot_corner() directly instead of the comparison overlay
- The fits come from different observations or energy bands, so posterior differences are not attributable to the model choice being compared
- You need a quantitative model-comparison metric (e.g. Bayes factor) rather than a visual posterior overlay

## Inputs

- Two or more jaxspec FitterResult objects from MCMCFitter.fit runs on the same ObsConfiguration (e.g. NGC7793_ULX4_PN, 0.3–12 keV band), one per background model variant
- Variant labels for the comparison legend
- Optional: variant name list matching the fits in a fixed order

## Outputs

- A corner-plot comparison figure (background_comparison.png) overlaying posterior distributions from all fitted variants with labels
- Per-variant corner plots from result.plot_corner()
- Summary tables of posterior parameter estimates (means/credible intervals) per variant via result.table()

## How to apply

For each fitted variant, call result.plot_corner() on the FitterResult to render the marginal and pairwise posterior distributions of the spectral parameters (e.g. nh, alpha, norm, kT). Then pass the multiple results to plot_corner_comparison from jaxspec.analysis.compare to overlay the posteriors of all variants in one figure with per-variant labels ('Background with no spread', 'Background with spread', 'Spectral background'). Save the comparison as background_comparison.png and supplement it with result.table() summaries of posterior means and credible intervals per variant. Rationale: the SubtractedBackground model propagates no background dispersion, so corner-level comparison against BackgroundWithError (each background bin treated as a Poisson realisation) and SpectralModelBackground (background modelled as Tbabs()*Powerlaw() with its own priors) reveals how the error budget shifts with background treatment.

## Related tools

- **jaxspec** (Provides plot_corner_comparison (jaxspec.analysis.compare), result.plot_corner() and result.table() used to visualise and compare posterior distributions of the fitted variants) — https://github.com/renecotyfanboy/jaxspec
- **JAX** (Backend enabling the differentiable likelihoods and 64-bit MCMC sampling whose posteriors are compared in the corner plots)

## Examples

```
from jaxspec.analysis.compare import plot_corner_comparison; plot_corner_comparison(result_subtracted, result_bgerr, result_spectral, labels=['Background with no spread', 'Background with spread', 'Spectral background'])
```

## Evaluation signals

- background_comparison.png exists and displays three labelled variants ('Background with no spread', 'Background with spread', 'Spectral background')
- Each per-variant posterior predictive check (PPC) plot is produced alongside the corner comparison
- The corner plots show posteriors for all five source parameters (nh, alpha, norm, kT, blackbody norm) and, for SpectralModelBackground, the background.tbabs_1.nh, background.powerlaw_1.alpha and background.powerlaw_1.norm priors
- result.table() returns means/credible intervals for all three variants, and the SubtractedBackground variant is checked for an under-dispersed (narrower) error budget relative to variants that propagate background dispersion
- Each fit used the documented sampling settings (num_chains=16, num_warmup=1000, num_samples=5000) so posteriors are comparable

## Limitations

- jaxspec is still in early release: expect bugs, breaking API changes, undocumented features and lack of functionalities
- Corner comparison is visual; the article does not provide a quantitative posterior-comparison metric for deciding between background variants
- Posteriors are only comparable if priors (and sampling settings) are held constant across variants except for the background-handling choice

## Evidence

- [methods] plot_corner_comparison import: "from jaxspec.analysis.compare import plot_corner_comparison"
- [methods] SubtractedBackground limitation: "The `SubtractedBackground` simply account for the observed counts without propagating any kind of dispersion, which is clearly bad when we want to get spectral parameters with a comprehensive error"
- [methods] corner plotting API: "result.plot_corner()"
- [readme] library scope: "`jaxspec` is an X-ray spectral fitting library built in pure Python. It can currently load an X-ray spectrum (in the OGIP standard), define a spectral model from the implemented components, and"
