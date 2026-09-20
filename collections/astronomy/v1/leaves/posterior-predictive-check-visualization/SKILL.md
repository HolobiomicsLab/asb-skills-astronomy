---
name: posterior-predictive-check-visualization
description: Use when a jaxspec MCMC (NUTS) fit of a composite model such as Tbabs()*(Powerlaw()
  + Blackbodyrad()) to an XMM-Newton spectrum has completed and you must verify the
  posterior actually reproduces the observed 0.3–12 keV counts before reporting parameters.
when_to_use_negative:
- The fit has not converged (e.g. NUTS divergences or chains not mixed) — a PPC on
  a bad posterior is uninformative; check the corner plot first
- You need a quantitative goodness-of-fit statistic rather than a visual check — jaxspec's
  PPC is a plot-level diagnostic, not a formal test
- The spectrum is fitted with a chi-square statistic at low counts, where the article
  notes C-stat/Poisson likelihood is preferred to avoid biased parameter estimates
  — PPC in count space presupposes the Poisson likelihood
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: jaxspec
  role: Provides result.plot_ppc(plot_components=True, n_sigmas=2) and result.plot_corner()
    for posterior predictive and posterior visual checks of the spectral fit
  repo: https://github.com/renecotyfanboy/jaxspec
- name: numpyro
  role: Runs the no U-turn sampler (num_chains=16, num_warmup=1000, num_samples=5000)
    whose posterior samples feed the PPC
- name: matplotlib
  role: Rendering backend underlying the PPC and corner figure generation
- name: XSPEC
  role: Independent C-stat reference fit (via the renecoty/heasoft Docker image) used
    to cross-validate that the PPC-consistent posterior matches established software
- name: dill
  role: Serialises the fitted result (result.pickle) so PPC figures can be regenerated
    without refitting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202451736
    title: 'jaxspec: A fast and robust Python library for X-ray spectral fitting'
  source_content_hash: sha256:89491e65ad994b18df28959f4a7f7edb1fa30c5ffb239e56e046e6a2ef7e0b53
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# posterior-predictive-check-visualization

## Summary

Visually validate a Bayesian X-ray spectral fit by overlaying posterior-predictive spectra (with individual model components and 2-sigma credible bands) on the observed counts, using jaxspec's result.plot_ppc alongside corner plots to confirm the fitted model reproduces the data.

## When to use

Use when a jaxspec MCMC (NUTS) fit of a composite model such as Tbabs()*(Powerlaw() + Blackbodyrad()) to an XMM-Newton spectrum has completed and you must verify the posterior actually reproduces the observed 0.3–12 keV counts before reporting parameters.

## When NOT to use

- The fit has not converged (e.g. NUTS divergences or chains not mixed) — a PPC on a bad posterior is uninformative; check the corner plot first
- You need a quantitative goodness-of-fit statistic rather than a visual check — jaxspec's PPC is a plot-level diagnostic, not a formal test
- The spectrum is fitted with a chi-square statistic at low counts, where the article notes C-stat/Poisson likelihood is preferred to avoid biased parameter estimates — PPC in count space presupposes the Poisson likelihood

## Inputs

- Fitted jaxspec result object returned by MCMCFitter.fit (posterior samples over tbabs_1.nh, powerlaw_1.alpha, powerlaw_1.norm, blackbodyrad_1.kT, blackbodyrad_1.norm)
- ObsConfiguration for the XMM-Newton PN observation (e.g. load_example_obsconf('NGC7793_ULX4_PN') or ObsConfiguration.from_pha_file with low_energy=0.3, high_energy=12)
- Spectral model expression Tbabs()*(Powerlaw() + Blackbodyrad())

## Outputs

- Posterior predictive plot with observed spectrum, predictive realisations, individual model components (plot_components=True) and 2-sigma credible bands
- Corner plot of the posterior (result.plot_corner()) as a complementary diagnostic
- Serialised fit result (result.pickle via dill) for reproducible figure regeneration

## How to apply

After running MCMCFitter(model, prior, obs).fit(num_chains=16, num_warmup=1000, num_samples=5000) on an ObsConfiguration built from the NGC 7793 ULX-4 PN observation over 0.3–12 keV, call result.plot_ppc(plot_components=True, n_sigmas=2) to draw posterior predictive realisations with each model component (powerlaw, blackbodyrad) shown separately and credible bands at 2 sigma. Inspect whether the observed PN counts fall within the predictive envelope across the full band; systematic residuals at specific energies indicate model mis-specification rather than sampling failure. Pair the PPC with result.plot_corner() to confirm posterior unimodality and with a blind XSPEC 12.15.1 C-stat fit, since jaxspec is reported to give identical results to other software. Because the fit uses the Poisson likelihood (C-stat equivalent), which is necessary for unbiased parameter estimates at all count rates, the PPC should be checked in count space rather than on chi-square residuals. Serialise the fitted result (e.g. with dill to result.pickle) so the PPC can be regenerated without re-running the sampler.

## Related tools

- **jaxspec** (Provides result.plot_ppc(plot_components=True, n_sigmas=2) and result.plot_corner() for posterior predictive and posterior visual checks of the spectral fit) — https://github.com/renecotyfanboy/jaxspec
- **numpyro** (Runs the no U-turn sampler (num_chains=16, num_warmup=1000, num_samples=5000) whose posterior samples feed the PPC)
- **matplotlib** (Rendering backend underlying the PPC and corner figure generation)
- **XSPEC** (Independent C-stat reference fit (via the renecoty/heasoft Docker image) used to cross-validate that the PPC-consistent posterior matches established software)
- **dill** (Serialises the fitted result (result.pickle) so PPC figures can be regenerated without refitting)

## Examples

```
result = MCMCFitter(Tbabs()*(Powerlaw() + Blackbodyrad()), prior, obs).fit(num_chains=16, num_warmup=1000, num_samples=5000); result.plot_ppc(plot_components=True, n_sigmas=2); result.plot_corner()
```

## Evaluation signals

- Observed NGC 7793 ULX-4 PN counts in 0.3–12 keV lie within the 2-sigma posterior predictive envelope across the band
- Individual components (powerlaw and blackbodyrad) rendered by plot_components=True sum to a predictive curve consistent with the data
- Corner plot shows unimodal posteriors whose means match the blind XSPEC C-stat reference values
- result.pickle exists so the PPC can be regenerated from the stored posterior
- Posterior parameter estimates reproduce the reported identical results at ~10x lower runtime than existing alternatives

## Limitations

- jaxspec is in early release: expect bugs, breaking API changes, undocumented features and lack of functionalities
- The SubtractedBackground model does not propagate dispersion, which is inadequate for a comprehensive error budget and can distort PPC error bands when background is subtracted rather than modelled
- A jaxspec software version number is not stated in the article (only Python 3.11/3.12 and XSPEC 12.15.1 are referenced), so reproducibility of the PPC depends on pinning the installed version yourself
- PPC is a visual diagnostic; the article validates results by agreement with XSPEC rather than by a formal PPC statistic

## Evidence

- [methods] Posterior predictive plot with components: "result.plot_ppc(plot_components=True, n_sigmas=2)"
- [methods] Poisson likelihood rationale: "Most recent publications on the subject agree that using the C-stat at all count rates is necessary to ensure an unbiased estimate of the parameters."
- [methods] Sampling configuration feeding the PPC: "result = forward.fit(num_chains=16, num_warmup=1000, num_samples=5000)"
- [abstract] Cross-framework validation claim: "We obtain identical results when compared to other software and approaches, meaning that jaxspec provides reliable results while being ~10 times faster"
- [readme] Early-release caveat: "**jaxspec is still in early release**: expect bugs, breaking API changes, undocumented features and lack of functionalities"
