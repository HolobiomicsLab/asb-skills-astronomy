---
name: mcmc-gibbs-sampling
description: Use when fitting Kepler or ground-based transit light curves where classic
  chi-squared underestimates parameter uncertainties because the photometric noise
  contains a correlated (red) component.
when_to_use_negative:
- When the photometric noise is known to be purely uncorrelated (white) — the classic
  chi-squared likelihood suffices and the extra sigma_r parameter is unnecessary.
- When the input is already a set of published transit parameters or a derived table
  rather than a raw/normalized photometric time series that needs light-curve fitting.
tools:
- name: Transit Analysis Package (TAP)
  role: IDL GUI package whose MCMC engine fits light curves with the Mandel and Agol
    (2002) model under the wavelet-based likelihood, updating sigma_w and sigma_r
    via Gibbs sampling
  repo: zgazak/TAP
- name: EXOFAST
  role: Improved implementation of the Mandel and Agol analytic transit model that
    TAP calls during MCMC fitting
- name: IDL
  role: Interactive Data Language environment in which TAP's MCMC/wavelet likelihood
    is implemented and executed
- name: Kepler mission
  role: Source of high-precision (few parts in 10^5) photometric time series on which
    the MCMC fit is run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:738ce183e6b5341f9496f8e99b436627cfd35a337dba82830d3e83cdd989664f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - Kepler mission
  - Transit Analysis Package (TAP)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# mcmc-gibbs-sampling

## Summary

Runs TAP's Markov Chain Monte Carlo fitting of exoplanet transit light curves, in which a wavelet-based likelihood (Carter and Winn 2009) lets uncorrelated white noise (sigma_w) and 1/f correlated red noise (sigma_r) evolve as free parameters via Gibbs sampling instead of assuming noiseless correlations as in classic chi-squared.

## When to use

Use when fitting Kepler or ground-based transit light curves where classic chi-squared underestimates parameter uncertainties because the photometric noise contains a correlated (red) component.

## When NOT to use

- When the photometric noise is known to be purely uncorrelated (white) — the classic chi-squared likelihood suffices and the extra sigma_r parameter is unnecessary.
- When the input is already a set of published transit parameters or a derived table rather than a raw/normalized photometric time series that needs light-curve fitting.

## Inputs

- Normalized photometric transit light-curve time series (e.g. Kepler photometry retrieved from MAST)
- Transit model parameters for the Mandel and Agol (2002) analytic model
- Initial values/bounds for the noise parameters sigma_w and sigma_r (sigma_r constrained non-negative)

## Outputs

- Posterior samples and posterior summaries of transit parameters per light curve
- Fitted (sigma_w, sigma_r) noise pair per transit written to wavelet_noise_fit.csv

## How to apply

Retrieve and normalize a Kepler transit light curve from MAST, then evaluate the Daubechies 4th-order discrete wavelet transform likelihood of Carter and Winn (2009), fixing gamma = 1 and g(1) = (2 ln 2)^-1 while leaving sigma_w and sigma_r free. Wrap the likelihood in a Metropolis-Hastings MCMC that fits the analytic transit model of Mandel and Agol (2002) (implemented in TAP via EXOFAST). Use Gibbs sampling to update the noise parameters at each step, constraining sigma_r to be non-negative. Run the chain on the light curve, record posterior summaries of sigma_w and sigma_r per transit, and write the fitted (sigma_w, sigma_r) pair to wavelet_noise_fit.csv. The rationale is that quantifying confidence in transit parameters depends critically on noise treatment, so parameterizing both uncorrelated and correlated photometric noise yields more robust uncertainties than chi-squared.

## Related tools

- **Transit Analysis Package (TAP)** (IDL GUI package whose MCMC engine fits light curves with the Mandel and Agol (2002) model under the wavelet-based likelihood, updating sigma_w and sigma_r via Gibbs sampling) — zgazak/TAP
- **EXOFAST** (Improved implementation of the Mandel and Agol analytic transit model that TAP calls during MCMC fitting)
- **IDL** (Interactive Data Language environment in which TAP's MCMC/wavelet likelihood is implemented and executed)
- **Kepler mission** (Source of high-precision (few parts in 10^5) photometric time series on which the MCMC fit is run)

## Evaluation signals

- The module executes and wavelet_noise_fit.csv contains both noise fields (sigma_w and sigma_r).
- The fitted sigma_w agrees with the light curve's point-to-point scatter within roughly a factor of two.
- The sigma_r posterior respects the non-negativity constraint (sigma_r >= 0).
- Reanalysis of benchmark systems (ground-based TrES-1b, WASP-4b, WASP-10b and space-based Kepler 4b-8b) shows good agreement between TAP and prior publications.
- Parameter uncertainties are reported per transit without phase folding or rebinning of multiple light curves of varying cadence.

## Limitations

- The likelihood assumes 1/f correlated (red) noise with gamma fixed to 1; other correlation structures are not parameterized.
- sigma_r must be constrained to be non-negative, which can force the red-noise term against a hard boundary at zero.
- Quantification of confidence in derived parameters depends critically on the treatment of noise, stellar limb darkening, and cadence, so results vary with those choices.

## Evidence

- [abstract] TAP MCMC light-curve fitting with Mandel & Agol model: "uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002)"
- [abstract] Wavelet likelihood beats classic chi-squared on uncertainty robustness: "a wavelet-based likelihood function developed by Carter and Winn (2009), which allows the MCMC to assess parameter uncertainties more robustly than classic χ2"
- [intro] TAP replaces chi-squared with wavelet likelihood parameterizing white and red noise: [evidence span withheld — anchor status: not_found]
- [intro] Simultaneous multi-cadence, multi-noise analysis without phase folding: "the simultaneous analysis of multiple light curves of varying cadence, filter (limb darkening), and noise (observational) conditions with no need to phase fold and rebin"
