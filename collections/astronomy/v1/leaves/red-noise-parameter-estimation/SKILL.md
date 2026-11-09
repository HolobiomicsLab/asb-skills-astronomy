---
name: red-noise-parameter-estimation
description: Use when fitting a transit light curve whose residuals may contain correlated
  (red) noise, and you need honest parameter uncertainties rather than chi-squared-based
  errors that assume purely white noise.
when_to_use_negative:
- When the residuals are demonstrably white (no correlated component) and classic
  chi-squared uncertainties are adequate — the extra sigma_r parameter is unnecessary.
- When the analysis requires only point estimates of transit parameters (radius, epoch)
  without robust uncertainty quantification — a simpler chi-squared fit suffices.
- When handling data conditions TAP was not designed for, such as needing to phase
  fold and rebin — TAP analyzes multiple light curves of varying cadence, filter,
  and noise conditions without phase folding.
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3295
tools:
- name: Transit Analysis Package (TAP)
  role: Hosts the wavelet-based likelihood and MCMC machinery that lets sigma_w and
    sigma_r evolve freely during transit fitting
  repo: https://github.com/zgazak/TAP
- name: IDL
  role: Language in which TAP and the wavelet likelihood are implemented and driven
    via its graphical user interface
- name: Kepler mission
  role: Source of the high-precision photometry (down to a few parts in 10^5) on which
    the red-noise fit is exercised
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:144848d9005b8168518d7f80471a96266ce8cae32101c9ea771fa62bed590285
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - IDL
  - Kepler mission
  - Transit Analysis Package (TAP)
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# red-noise-parameter-estimation

## Summary

Fits free photometric noise parameters — uncorrelated Gaussian white noise (sigma_w) and 1/f correlated red noise (sigma_r) — via the Carter and Winn (2009) Daubechies-4 wavelet-based likelihood, replacing the classic chi-squared assumption of no correlated noise. Used within TAP's MCMC transit fitting so uncertainty quantification accounts for correlated (red) noise in Kepler-class photometry.

## When to use

Use when fitting a transit light curve whose residuals may contain correlated (red) noise, and you need honest parameter uncertainties rather than chi-squared-based errors that assume purely white noise.

## When NOT to use

- When the residuals are demonstrably white (no correlated component) and classic chi-squared uncertainties are adequate — the extra sigma_r parameter is unnecessary.
- When the analysis requires only point estimates of transit parameters (radius, epoch) without robust uncertainty quantification — a simpler chi-squared fit suffices.
- When handling data conditions TAP was not designed for, such as needing to phase fold and rebin — TAP analyzes multiple light curves of varying cadence, filter, and noise conditions without phase folding.

## Inputs

- Normalized photometric time series of a single transit (e.g., a Kepler mission light curve retrieved from MAST)
- Transit model (Mandel and Agol 2002 analytic model as implemented in TAP/EXOFAST)

## Outputs

- Posterior samples and summaries of sigma_w (white noise scatter) and sigma_r (1/f red noise amplitude) per transit
- Fitted (sigma_w, sigma_r) pair written to wavelet_noise_fit.csv

## How to apply

Retrieve a photometric time series (e.g., one Kepler transit light curve from MAST) and normalize it before fitting. Implement the Daubechies 4th-order discrete wavelet transform likelihood of Carter and Winn (2009) per Eqs. (1)-(3), fixing gamma = 1 and g(1) = (2 ln 2)^-1 while leaving sigma_w and sigma_r free to evolve during the MCMC. Wrap the likelihood in a Metropolis-Hastings MCMC that uses Gibbs sampling to update the noise parameters, constraining sigma_r to be non-negative. Run the chain on the light curve and record posterior summaries of sigma_w and sigma_r (here, per transit). The rationale is that TAP's wavelet-based likelihood parameterizes both uncorrelated and correlated photometric noise, yielding more robust uncertainties than classic chi-squared methods.

## Related tools

- **Transit Analysis Package (TAP)** (Hosts the wavelet-based likelihood and MCMC machinery that lets sigma_w and sigma_r evolve freely during transit fitting) — https://github.com/zgazak/TAP
- **IDL** (Language in which TAP and the wavelet likelihood are implemented and driven via its graphical user interface)
- **Kepler mission** (Source of the high-precision photometry (down to a few parts in 10^5) on which the red-noise fit is exercised)

## Evaluation signals

- Both noise fields are present in the output (e.g., wavelet_noise_fit.csv contains sigma_w and sigma_r columns).
- sigma_w agrees with the light curve's point-to-point scatter within roughly a factor of two.
- sigma_r is non-negative, consistent with its MCMC constraint.
- The module executes end-to-end on a prepared Kepler transit light curve without error.
- Posterior chains for sigma_w and sigma_r show free evolution (Gibbs-updated) rather than fixed values, distinguishing the fit from a chi-squared run.

## Limitations

- Derived transit parameters vary between fitting methods, and confidence quantification depends critically on the treatment of noise, stellar limb darkening, and cadence.
- For long integration observations (as with most Kepler light curves), transit models must be calculated on a finer temporal scale and integrated to the observed cadence to avoid integration distortions.
- The skill was validated against TAP's published behavior (TrES-1b, WASP-4b, WASP-10b, Kepler 4b-8b); results should be compared against those benchmarks rather than assumed transferable.

## Evidence

- [abstract] a wavelet-based likelihood function developed by Carter and Winn (2009), which allows the MCMC to assess parameter uncertainties more robustly than classic χ2 methods by parameterizing uncorrelated: "a wavelet-based likelihood function developed by Carter and Winn (2009), which allows the MCMC to assess parameter uncertainties more robustly than classic χ2 methods by parameterizing uncorrelated"
- [intro] Instead of the classic χ2, TAP uses the wavelet-based likelihood function of [17], which parameterizes both uncorrelated and correlated photometric noise allowing for accurate uncertainties: [evidence span withheld — anchor status: not_found]
- [abstract] uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002): "uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002)"
