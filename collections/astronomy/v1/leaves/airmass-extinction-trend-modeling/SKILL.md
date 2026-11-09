---
name: airmass-extinction-trend-modeling
description: Use when fitting ground-based transit light curves that show smooth,
  airmass-correlated brightness trends (e.g. from differential extinction during the
  night), and you need unbiased transit parameters and per-transit Tmid via MCMC.
when_to_use_negative:
- Space-based photometry such as Kepler light curves, where long integration distortions
  are handled by computing transit models on a finer temporal scale and integrating
  to the observed cadence, not by an airmass term
- Light curves that have been phase folded or re-binned, since TAP is designed to
  fit raw data with no need to phase fold and rebin
- Observations lacking airmass metadata or with negligible atmospheric coverage across
  the transit, where the extinction trend is unconstrained
tools:
- name: Transit Analysis Package (TAP)
  role: IDL graphical interface used to configure the simultaneous MCMC fit of the
    transit model, wavelet noise parameters, and airmass extinction trend, leaving
    the trend free per transit
  repo: zgazak/TAP
- name: EXOFAST
  role: Provides the Mandel and Agol (2002) analytic transit model that TAP fits via
    MCMC alongside the extinction trend
- name: IDL (Interactive Data Language)
  role: Runtime environment in which the TAP GUI and the airmass-extinction-aware
    fitting are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:92f193f862d275a354fcb4f87088f0a65c930a4646d101c86440d82e716b993e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - IDL (Interactive Data Language)
  - Transit Analysis Package (TAP)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# airmass-extinction-trend-modeling

## Summary

A TAP (Transit Analysis Package) technique for modeling differential extinction with airmass in ground-based transit photometry: an airmass extinction trend is fit simultaneously with the Mandel & Agol (2002) transit model and Carter & Winn (2009) wavelet-based red-noise likelihood, so systematic brightness drifts do not bias transit parameters or mid-transit times.

## When to use

Use when fitting ground-based transit light curves that show smooth, airmass-correlated brightness trends (e.g. from differential extinction during the night), and you need unbiased transit parameters and per-transit Tmid via MCMC.

## When NOT to use

- Space-based photometry such as Kepler light curves, where long integration distortions are handled by computing transit models on a finer temporal scale and integrating to the observed cadence, not by an airmass term
- Light curves that have been phase folded or re-binned, since TAP is designed to fit raw data with no need to phase fold and rebin
- Observations lacking airmass metadata or with negligible atmospheric coverage across the transit, where the extinction trend is unconstrained

## Inputs

- Ground-based differential transit photometry time series (e.g. the two WASP-4b transits from Winn et al. 2009)
- Per-exposure airmass values for each light curve
- Configuration choices in the TAP GUI: shared vs. free parameters, locked eccentricity (e = 0, omega = 0), limb darkening per filter

## Outputs

- Posterior median and 1-sigma uncertainty for transit parameters (e.g. inclination 88.97 +/- 0.65 deg, a/Rs 5.476 +0.023/-0.030, Rp/Rs 0.15398 +/- 0.00055 for WASP-4b)
- Per-transit mid-transit times Tmid with uncertainties (e.g. 0.797484 +/- 0.000054 and 0.650490 +/- 0.000047 days)
- Fitted airmass extinction trend coefficients and red-noise (wavelet) parameters per transit

## How to apply

Load the raw (unfolded, unre-binned) ground-based light curve(s) and per-exposure airmass values into the TAP IDL graphical interface. Configure TAP to fit the Mandel and Agol (2002) analytic transit model via EXOFAST, using the Carter and Winn (2009) wavelet-based likelihood that parameterizes both uncorrelated and correlated (red) photometric noise. Decide which parameters are shared across transits versus free per transit: in the WASP-4b reanalysis, global system parameters (inclination, a/Rs, Rp/Rs) were shared with e = 0 and omega = 0 locked, while the airmass extinction trend, noise parameters, and per-transit mid-transit times were left free. Run the MCMC simultaneously on all light curves without phase folding or rebinning, then extract posterior medians and 1-sigma uncertainties for the transit parameters and extinction-trend coefficients. Rationale: because different fitting methods and noise treatments shift derived quantities, the wavelet likelihood gives robust uncertainties while the free airmass trend absorbs atmospheric extinction so the shared transit model is not distorted.

## Related tools

- **Transit Analysis Package (TAP)** (IDL graphical interface used to configure the simultaneous MCMC fit of the transit model, wavelet noise parameters, and airmass extinction trend, leaving the trend free per transit) — zgazak/TAP
- **EXOFAST** (Provides the Mandel and Agol (2002) analytic transit model that TAP fits via MCMC alongside the extinction trend)
- **IDL (Interactive Data Language)** (Runtime environment in which the TAP GUI and the airmass-extinction-aware fitting are executed)

## Evaluation signals

- Fitted parameters agree with prior publication values within reported 1-sigma errors (paper's criterion of 'good agreement between TAP and prior publications')
- For WASP-4b, inclination, a/Rs, and Rp/Rs match Winn et al. (2009): 88.97 +/- 0.65 deg, 5.476 +0.023/-0.030, 0.15398 +/- 0.00055
- Per-transit Tmid posterior uncertainties reach the ~5e-5 day (tens of ppm) level seen in the reference reanalysis
- Residuals show no remaining airmass-correlated structure, confirming the free extinction trend absorbed the systematic
- Red-noise parameters are free per transit, and uncertainties are robust relative to a classic chi-squared treatment

## Limitations

- The airmass extinction trend is only appropriate for ground-based observations under varying atmospheric conditions; it does not address long-integration distortions in space-based (Kepler) data
- Derived transit parameters are sensitive to noise treatment, stellar limb darkening, and cadence, so results depend on the wavelet-likelihood configuration and limb-darkening choice per filter
- TAP expects raw light curves; phase folding or re-binning the input degrades the intended simultaneous analysis

## Evidence

- [abstract] The software is able to simultaneously analyze multiple transits observed in different conditions (instrument, filter, weather, etc.).: "The software is able to simultaneously analyze multiple transits observed in different conditions (instrument, filter, weather, etc.)."
- [intro] TAP has been developed for the simultaneous analysis of multiple light curves of varying cadence, filter (limb darkening), and noise (observational) conditions with no need to phase fold and rebin: "TAP has been developed for the simultaneous analysis of multiple light curves of varying cadence, filter (limb darkening), and noise (observational) conditions with no need to phase fold and rebin"
- [intro] Instead of the classic chi-squared, TAP uses the wavelet-based likelihood function of [17], which parameterizes both uncorrelated and correlated photometric noise allowing for accurate uncertainties: [evidence span withheld — anchor status: not_found]
