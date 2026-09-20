---
name: exoplanet-transit-parameter-estimation
description: Use when unbinned, un-phase-folded transit photometry (ground-based or
  Kepler) must yield inclination, a/Rs, Rp/Rs, and per-transit mid-transit times with
  uncertainties that correctly account for red noise.
when_to_use_negative:
- The input is not transit photometry (e.g. radial velocities or spectra); TAP is
  designed specifically for exoplanet transit light curves
- The workflow requires phase-folded or rebinned light curves; TAP is built to fit
  simultaneous, unbinned curves of varying cadence and filter
- A quick chi-squared point estimate is desired without honest uncertainty quantification;
  TAP's value lies in the wavelet-based white/red noise treatment, not in speed
tools:
- name: Transit Analysis Package (TAP)
  role: IDL graphical user interface that drives the MCMC transit fitting and wavelet-likelihood
    noise treatment
  repo: zgazak/TAP
- name: IDL
  role: Interactive Data Language runtime environment in which the TAP GUI package
    is executed
  repo: zgazak/TAP
- name: EXOFAST
  role: Provides the improved implementation of the Mandel & Agol (2002) analytic
    transit model used inside TAP
  repo: zgazak/TAP
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:dd0d46aac82865a874b65babd7b02b622f7badd12c8e82fffd18c50afc959920
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - Transit Analysis Package (TAP)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# exoplanet-transit-parameter-estimation

## Summary

Fit exoplanet transit light curves with TAP, an IDL GUI package that drives EXOFAST's Mandel & Agol (2002) analytic model via MCMC, using the Carter & Winn (2009) wavelet-based likelihood to parameterize uncorrelated and correlated (red) photometric noise for accurate parameter uncertainties.

## When to use

Use when unbinned, un-phase-folded transit photometry (ground-based or Kepler) must yield inclination, a/Rs, Rp/Rs, and per-transit mid-transit times with uncertainties that correctly account for red noise.

## When NOT to use

- The input is not transit photometry (e.g. radial velocities or spectra); TAP is designed specifically for exoplanet transit light curves
- The workflow requires phase-folded or rebinned light curves; TAP is built to fit simultaneous, unbinned curves of varying cadence and filter
- A quick chi-squared point estimate is desired without honest uncertainty quantification; TAP's value lies in the wavelet-based white/red noise treatment, not in speed

## Inputs

- Ground-based or Kepler transit light curves (time-series photometry), loaded directly without phase folding or rebinning
- Fitting configuration: Mandel & Agol (2002) limb-darkening model per filter, locked eccentricity (e = 0, omega = 0), shared global system parameters across transits, free per-transit Tmid, noise parameters, and airmass extinction trend

## Outputs

- Posterior median and 1-sigma uncertainties for inclination (deg), a/Rs, Rp/Rs, and per-transit mid-transit times Tmid (days)
- MCMC chain results reporting white and red noise parameters per transit
- A comparison table (Table 3-style) of TAP-fitted parameters against previously published values

## How to apply

Load the transit light curves into the TAP IDL graphical interface and configure it to fit the Mandel and Agol (2002) analytic transit model as implemented in EXOFAST. Use the Carter and Winn (2009) wavelet-based likelihood instead of classic chi-squared so the MCMC parameterizes both uncorrelated and correlated photometric noise, which is what makes the reported 1-sigma uncertainties trustworthy. Where the science allows, lock orbital eccentricity and argument of periastron (e = 0, omega = 0) and share the global system parameters across all transits, while leaving per-transit mid-transit times, per-transit noise parameters, and airmass extinction trends free. Run the TAP MCMC simultaneously on all light curves without phase folding or rebinning, since TAP is explicitly built to analyze multiple curves of varying cadence, filter (limb darkening), and noise conditions as observed. For long-integration data such as Kepler photometry, set TAP to calculate transit models on a finer temporal scale and integrate to the observed cadence. Extract posterior medians and 1-sigma errors for inclination, a/Rs, Rp/Rs, and each Tmid, then compare against the prior publication's values and uncertainties.

## Related tools

- **Transit Analysis Package (TAP)** (IDL graphical user interface that drives the MCMC transit fitting and wavelet-likelihood noise treatment) — zgazak/TAP
- **IDL** (Interactive Data Language runtime environment in which the TAP GUI package is executed) — zgazak/TAP
- **EXOFAST** (Provides the improved implementation of the Mandel & Agol (2002) analytic transit model used inside TAP) — zgazak/TAP

## Evaluation signals

- Each fitted parameter and its 1-sigma uncertainty is consistent with the prior publication's value (e.g. for WASP-4b per Winn et al. 2009: inclination 88.97 +/- 0.65 deg, a/Rs 5.476 +0.023/-0.030, Rp/Rs 0.15398 +/- 0.00055)
- Per-transit Tmid values match published values within errors (WASP-4b: 0.797484 +/- 0.000054 and 0.650490 +/- 0.000047 days)
- Fitting was performed simultaneously on multiple light curves without phase folding or rebinning, with per-transit Tmid left free
- Uncertainties come from the wavelet-based likelihood (white + red noise), not a classic chi-squared assumption
- Reanalysis reproduces published parameters with 'good agreement between TAP and prior publications', as reported for TrES-1b, WASP-4b, WASP-10b, and Kepler 4b-8b

## Limitations

- Different fitting methods lead to variations in derived quantities, and confidence in derived parameters depends critically on the treatment of noise, stellar limb darkening, and cadence
- Long-integration observations (e.g. most Kepler light curves) require setting TAP to calculate models on a finer temporal scale and integrate to the observed cadence, or distortions appear
- Locking eccentricity (e = 0, omega = 0) assumes a circular orbit; this configuration will not capture eccentric-transit systematics

## Evidence

- [abstract] uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002): "The Transit Analysis Package (TAP) software uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002)."
- [abstract] wavelet-based likelihood of Carter and Winn (2009) parameterizing white and red noise: "a wavelet-based likelihood function developed by Carter and Winn (2009), which allows the MCMC to assess parameter uncertainties more robustly than classic χ2 methods"
- [intro] simultaneous analysis of multiple light curves without phase folding or rebinning: "TAP has been developed for the simultaneous analysis of multiple light curves of varying cadence, filter (limb darkening), and noise (observational) conditions with no need to phase fold and rebin"
- [intro] TAP uses EXOFAST's implementation of the analytic transit model: "The TAP software package uses EXOFAST [23], an improved implementation of the [15] analytic transit model"
- [intro] different fitting methods give varying derived quantities; confidence depends on noise, limb darkening, cadence: [evidence span withheld — anchor status: not_found]
- [abstract] good agreement between TAP and prior publications: [evidence span withheld — anchor status: not_found]
