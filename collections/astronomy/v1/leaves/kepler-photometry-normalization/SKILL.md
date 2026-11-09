---
name: kepler-photometry-normalization
description: Use when you have long-cadence Kepler transit photometry (multiple transits
  of one planet) that must be normalized per segment without phase folding or rebinning
  and fitted simultaneously with shared system parameters.
when_to_use_negative:
- The light curves have already been phase folded and rebinned — TAP is built for
  simultaneous analysis 'with no need to phase fold and rebin', and folding destroys
  the per-transit Tmid information this skill extracts
- The data are not transit photometry of varying cadence/filter/noise conditions (TAP
  targets exoplanet transit light curves, tested on ground-based data and Kepler 4b-8b,
  not generic time-series variability)
- Only a single short-cadence transit is available — the multicurve parameter-locking
  mechanism and its precision gain require multiple curves to fit simultaneously
tools:
- name: Transit Analysis Package (TAP)
  role: 'Runs the simultaneous multicurve MCMC: wavelet (Carter and Winn 2009) likelihood,
    parameter-locking matrix for shared system parameters, and long-integration handling
    for Kepler cadence'
  repo: https://github.com/zgazak/TAP
- name: EXOFAST
  role: Provides the improved implementation of the Mandel and Agol (2002) analytic
    transit model evaluated inside TAP's fit, on a finer temporal scale when long
    integration must be modeled
- name: IDL (Interactive Data Language)
  role: Execution environment for TAP's graphical-user-interface-driven pipeline over
    the normalized Kepler segments
- name: Kepler mission
  role: Source of the long-cadence photometry (precision down to a few parts in 10^5)
    whose integration-time effects the workflow must correct for
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:2da53419d70b26fb30f93d19bcbe22f6e6ef176f1fdecfd9af3521e1c08cc793
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - IDL (Interactive Data Language)
  - Kepler mission
  - Transit Analysis Package (TAP)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# kepler-photometry-normalization

## Summary

A workflow for retrieving long-cadence Kepler transit photometry, normalizing each transit segment without phase folding or rebinning, and feeding the un-phased light curves into TAP's simultaneous multicurve MCMC with shared system parameters. It is grounded in the Transit Analysis Package (TAP), an IDL GUI built on the Mandel and Agol (2002) analytic transit model.

## When to use

Use when you have long-cadence Kepler transit photometry (multiple transits of one planet) that must be normalized per segment without phase folding or rebinning and fitted simultaneously with shared system parameters.

## When NOT to use

- The light curves have already been phase folded and rebinned — TAP is built for simultaneous analysis 'with no need to phase fold and rebin', and folding destroys the per-transit Tmid information this skill extracts
- The data are not transit photometry of varying cadence/filter/noise conditions (TAP targets exoplanet transit light curves, tested on ground-based data and Kepler 4b-8b, not generic time-series variability)
- Only a single short-cadence transit is available — the multicurve parameter-locking mechanism and its precision gain require multiple curves to fit simultaneously

## Inputs

- Long-cadence Kepler photometry of a target planet (raw flux time series retrieved from the MAST archive)
- Per-transit segment definitions so each observed transit can be normalized separately (no phase folding, no rebinning)
- Initial transit parameter guesses: orbital period P, a/Rs, Rp/Rs, inclination i, mid-transit times Tmid, limb-darkening coefficients, noise/trend terms

## Outputs

- Normalized, un-phased per-transit Kepler light curves at native cadence
- Per-transit Tmid values with posterior uncertainties from the wavelet-likelihood MCMC
- Combined (locked) system parameters P, a/Rs, Rp/Rs, i with posterior uncertainties, plus a multicurve vs single-curve uncertainty comparison (e.g. multitransit_results.csv)

## How to apply

Retrieve the long-cadence Kepler photometry of the target planet from the MAST archive and normalize each transit segment individually — TAP is explicitly designed for analysis 'with no need to phase fold and rebin', so keep transit epochs separate so per-transit Tmid values remain recoverable. Configure TAP's parameter-locking matrix so the system parameters P, a/Rs, Rp/Rs, i, and Tmid are forced to evolve together across all N light curves, while limb-darkening, wavelet noise, and trend parameters stay free per curve. Because long integration distorts most Kepler light curves, enable TAP's setting to calculate the Mandel and Agol (2002) transit model (via the EXOFAST implementation) on a finer temporal scale and integrate it to the observed Kepler cadence. Run the MCMC using the Carter and Winn (2009) wavelet-based likelihood, which parameterizes both uncorrelated and correlated photometric noise instead of classic chi-squared, to get accurate uncertainties. Extract per-transit Tmid values plus the combined locked system parameters, and validate by refitting each light curve independently: the multicurve combined-parameter uncertainties should be no larger than the single-curve fit uncertainties.

## Related tools

- **Transit Analysis Package (TAP)** (Runs the simultaneous multicurve MCMC: wavelet (Carter and Winn 2009) likelihood, parameter-locking matrix for shared system parameters, and long-integration handling for Kepler cadence) — https://github.com/zgazak/TAP
- **EXOFAST** (Provides the improved implementation of the Mandel and Agol (2002) analytic transit model evaluated inside TAP's fit, on a finer temporal scale when long integration must be modeled)
- **IDL (Interactive Data Language)** (Execution environment for TAP's graphical-user-interface-driven pipeline over the normalized Kepler segments)
- **Kepler mission** (Source of the long-cadence photometry (precision down to a few parts in 10^5) whose integration-time effects the workflow must correct for)

## Evaluation signals

- Cross-check against published values: TAP reanalysis of TrES-1b, WASP-4b, WASP-10b and Kepler 4b-8b should show good agreement with prior publications
- Refit each light curve independently (single-curve mode) and confirm the multicurve combined-parameter uncertainties are no larger than the single-curve fit uncertainties
- For Kepler long-cadence data, verify the transit model was computed on a finer temporal scale and integrated to the observed cadence rather than evaluated pointwise
- Compare posterior uncertainties against a classic chi-squared fit: the wavelet-based likelihood should give more realistic (larger) uncertainties when correlated red noise is present
- Check that per-transit Tmid values are individually recovered for all N segments, confirming no phase folding or rebinning was applied upstream

## Limitations

- Most Kepler light curves suffer long-integration distortions; if the finer-temporal-scale integration option is not enabled, derived transit parameters will be biased
- Confidence quantification depends critically on the treatment of noise, stellar limb darkening, and cadence — different fitting methods can lead to variations in derived quantities
- Public analysis packages for uniform application of transit models are rare; aside from TAP only alternatives like JKTEBOP exist, so results may not be directly comparable across packages

## Evidence

- [abstract] MCMC fit of analytic transit model: "The Transit Analysis Package (TAP) software uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002)."
- [intro] No phase folding or rebinning: "TAP has been developed for the simultaneous analysis of multiple light curves of varying cadence, filter (limb darkening), and noise (observational) conditions with no need to phase fold and rebin"
- [intro] Long-integration handling for Kepler: "For long integration observations (as is the case with most Kepler light curves) the software can be set to calculate transit models on a finer temporal scale"
- [abstract] Wavelet likelihood for uncertainties: "a wavelet-based likelihood function developed by Carter and Winn (2009), which allows the MCMC to assess parameter uncertainties more robustly than classic χ2 methods by parameterizing uncorrelated"
- [intro] Kepler photometric precision: [evidence span withheld — anchor status: not_found]
