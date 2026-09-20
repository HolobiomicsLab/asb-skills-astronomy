---
name: mcmc-transit-parameter-inference
description: Use when you have raw, non-phase-folded transit photometry (e.g. Kepler
  long-cadence or ground-based light curves) and need posterior transit parameters
  (P, a/Rs, Rp/Rs, i, Tmid, limb darkening) with robust uncertainty estimates.
when_to_use_negative:
- The light curves have already been phase-folded and rebinned — TAP is explicitly
  designed to analyze multiple transits with 'no need to phase fold and rebin'
- The data are not transit photometry (e.g. RV time series or non-transit flux calibrations)
  — the Mandel and Agol (2002) transit model would not apply
- Only a classic chi-squared uncertainty budget is required and red-noise handling
  is explicitly out of scope for the downstream comparison
tools:
- name: Transit Analysis Package (TAP)
  role: Runs the MCMC transit fitting, wavelet likelihood, multicurve parameter locking,
    and long-integration handling
  repo: zgazak/TAP
- name: EXOFAST
  role: Provides the improved implementation of the Mandel and Agol (2002) analytic
    transit model used inside TAP's MCMC
- name: IDL
  role: Runtime environment hosting the TAP graphical user interface for the fitting
    workflow
- name: Kepler mission
  role: Source of long-cadence photometry tested with TAP; long-integration mode integrates
    the model to the observed Kepler cadence
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:6aa85e0e07559e6e647f077b2391aa9451ae7f7529d288253d7c6c0ce3109f32
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
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mcmc-transit-parameter-inference

## Summary

Fit exoplanet transit light curves with MCMC in TAP (IDL), using the Mandel and Agol (2002) analytic transit model via EXOFAST and the Carter and Winn (2009) wavelet likelihood to parameterize uncorrelated and correlated (red) noise instead of classic chi-squared, yielding realistic posterior parameter uncertainties.

## When to use

Use when you have raw, non-phase-folded transit photometry (e.g. Kepler long-cadence or ground-based light curves) and need posterior transit parameters (P, a/Rs, Rp/Rs, i, Tmid, limb darkening) with robust uncertainty estimates.

## When NOT to use

- The light curves have already been phase-folded and rebinned — TAP is explicitly designed to analyze multiple transits with 'no need to phase fold and rebin'
- The data are not transit photometry (e.g. RV time series or non-transit flux calibrations) — the Mandel and Agol (2002) transit model would not apply
- Only a classic chi-squared uncertainty budget is required and red-noise handling is explicitly out of scope for the downstream comparison

## Inputs

- Normalized, unphased, non-rebinned transit light curves (e.g. long-cadence Kepler photometry from MAST; ground-based photometry of TrES-1b, WASP-4b, WASP-10b)
- Per-curve limb-darkening coefficients appropriate to each filter/instrument
- Parameter-locking matrix configuration specifying shared system parameters vs. free per-curve parameters
- Cadence setting (fine temporal grid for long-integration integration-to-cadence handling)

## Outputs

- MCMC posterior distributions for transit parameters (P, a/Rs, Rp/Rs, inclination, Tmid)
- Per-transit Tmid values with uncertainties
- Combined (locked) system parameter estimates with posterior uncertainties
- Uncertainty comparison between multicurve (locked) fits and single-curve fits

## How to apply

Retrieve and normalize each observed transit segment without phase folding or rebinning, since TAP is built for simultaneous analysis of multiple light curves of varying cadence, filter (limb darkening), and noise conditions. Build the parameter matrix and decide which parameters evolve together (lock shared system parameters such as P, a/Rs, Rp/Rs, i, and Tmid across N curves) and which remain free per curve (limb-darkening, wavelet noise, and trend parameters). For long-integration data such as Kepler light curves, enable the mode that computes the Mandel and Agol (2002) model (EXOFAST implementation) on a finer temporal scale and integrates it to the observed cadence. Run the MCMC with the Carter and Winn (2009) wavelet-based likelihood so white and red noise are both parameterized, then extract posterior distributions for each transit parameter. For a multicurve analysis, refit each light curve independently to verify that combined (locked) parameter uncertainties are no larger than the single-curve uncertainties.

## Related tools

- **Transit Analysis Package (TAP)** (Runs the MCMC transit fitting, wavelet likelihood, multicurve parameter locking, and long-integration handling) — zgazak/TAP
- **EXOFAST** (Provides the improved implementation of the Mandel and Agol (2002) analytic transit model used inside TAP's MCMC)
- **IDL** (Runtime environment hosting the TAP graphical user interface for the fitting workflow)
- **Kepler mission** (Source of long-cadence photometry tested with TAP; long-integration mode integrates the model to the observed Kepler cadence)

## Evaluation signals

- Per-transit Tmid values and combined system parameters are extracted with posterior uncertainties from the wavelet-likelihood MCMC chains
- Combined (locked) parameter uncertainties are no larger than the uncertainties from independent single-curve refits
- Derived parameters agree with prior publications (e.g. TrES-1b, WASP-4b, WASP-10b, and Kepler 4b-8b reanalyses show good agreement with TAP)
- Long-integration mode produces transit models computed on a finer temporal scale and integrated to the observed Kepler cadence without distorting transit shape
- Parameter posterior widths reflect the wavelet treatment of both uncorrelated and correlated noise rather than classic chi-squared uncertainties

## Limitations

- Derived transit parameters vary between fitting methods, and confidence quantification depends critically on the treatment of noise, stellar limb darkening, and cadence
- Long-integration observations (most Kepler light curves) require the finer-temporal-scale model integration; ignoring it distorts the fitted transit
- Alternate public packages applying transit models uniformly are rare (JKTEBOP being one), so cross-tool validation is limited

## Evidence

- [abstract] TAP fits light curves with MCMC and Mandel and Agol (2002) model: "The Transit Analysis Package (TAP) software uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002)."
- [abstract] Wavelet-based likelihood parameterizes white and red noise: "a wavelet-based likelihood function developed by Carter and Winn (2009), which allows the MCMC to assess parameter uncertainties more robustly than classic χ2 methods by parameterizing uncorrelated"
- [intro] Multicurve simultaneous analysis without phase folding: "TAP has been developed for the simultaneous analysis of multiple light curves of varying cadence, filter (limb darkening), and noise (observational) conditions with no need to phase fold and rebin"
- [intro] Long-integration handling for Kepler light curves: "For long integration observations (as is the case with most Kepler light curves) the software can be set to calculate transit models on a finer temporal scale"
