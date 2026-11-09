---
name: multi-light-curve-simultaneous-transit-fitting
description: Use when you have N transit light curves of the same planet observed
  under different conditions (instrument, filter, weather, cadence) and need joint
  system-parameter estimates with per-transit Tmid values and robust uncertainty comparison
  versus single-curve fits.
when_to_use_negative:
- You only have a single transit light curve — use TAP's single-curve mode instead
  of the multicurve locked-parameter mechanism
- You need to combine light curves by phase folding and rebinning — TAP is explicitly
  designed to avoid this, and folding would discard per-transit Tmid information
- The light curves require a transit model other than the Mandel and Agol (2002)/EXOFAST
  analytic form that TAP implements
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Transit Analysis Package (TAP)
  role: Runs the multicurve MCMC with the parameter-locking matrix, wavelet-based
    likelihood, and long-integration integration of transit models to the observed
    cadence
  repo: zgazak/TAP
- name: EXOFAST
  role: Improved implementation of the Mandel and Agol (2002) analytic transit model
    that TAP calls for light-curve fitting
- name: IDL
  role: Language environment hosting TAP's graphical user interface for configuring
    and running the simultaneous analysis
- name: Kepler mission
  role: Source of long-cadence photometry (precision down to a few parts in 10^5)
    requiring the fine-temporal-scale model integration feature
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:fa5a76028d193364f22937d733d45891d3da3437addbecfdfb4cb3d3b403a9b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - Kepler mission
  - Transit Analysis Package (TAP)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# multi-light-curve-simultaneous-transit-fitting

## Summary

Simultaneously fit multiple transit light curves (e.g., long-cadence Kepler photometry) with TAP's MCMC engine, locking shared system parameters (P, a/Rs, Rp/Rs, i, Tmid) across curves while letting limb-darkening, wavelet-noise, and trend parameters evolve independently per curve, without phase folding or rebinning.

## When to use

Use when you have N transit light curves of the same planet observed under different conditions (instrument, filter, weather, cadence) and need joint system-parameter estimates with per-transit Tmid values and robust uncertainty comparison versus single-curve fits.

## When NOT to use

- You only have a single transit light curve — use TAP's single-curve mode instead of the multicurve locked-parameter mechanism
- You need to combine light curves by phase folding and rebinning — TAP is explicitly designed to avoid this, and folding would discard per-transit Tmid information
- The light curves require a transit model other than the Mandel and Agol (2002)/EXOFAST analytic form that TAP implements

## Inputs

- Multiple transit light curves of the same planet (ground-based and/or long-cadence Kepler photometry from MAST), each normalized per transit segment, not phase folded or rebinned
- TAP parameter configuration matrix specifying which parameters are shared (locked) versus free per curve
- Limb-darkening, wavelet-noise, and trend parameter priors per curve

## Outputs

- Per-transit Tmid values with posterior uncertainties
- Combined (locked) system parameters P, a/Rs, Rp/Rs, i with MCMC posterior distributions
- Uncertainty comparison between the multicurve combined fit and independent single-curve fits

## How to apply

Retrieve long-cadence Kepler photometry from the MAST archive and normalize each transit segment without phase folding or rebinning. Configure TAP's parameter-locking matrix so P, a/Rs, Rp/Rs, i, and Tmid evolve together across the N light curves (system parameters), while limb-darkening, Carter and Winn (2009) wavelet noise parameters, and trend parameters remain free per curve. Enable long-integration handling so the Mandel and Agol (2002) analytic transit model (via the EXOFAST implementation) is computed on a finer temporal scale and integrated to the observed Kepler cadence. Run the MCMC with the wavelet-based likelihood that parameterizes both uncorrelated (white) and correlated (red) photometric noise instead of classic chi-squared, then extract per-transit Tmid values and the combined locked system parameters with posteriors. Validate by refitting each light curve independently and confirming that combined-parameter uncertainties are no larger than the single-curve fit uncertainties.

## Related tools

- **Transit Analysis Package (TAP)** (Runs the multicurve MCMC with the parameter-locking matrix, wavelet-based likelihood, and long-integration integration of transit models to the observed cadence) — zgazak/TAP
- **EXOFAST** (Improved implementation of the Mandel and Agol (2002) analytic transit model that TAP calls for light-curve fitting)
- **IDL** (Language environment hosting TAP's graphical user interface for configuring and running the simultaneous analysis)
- **Kepler mission** (Source of long-cadence photometry (precision down to a few parts in 10^5) requiring the fine-temporal-scale model integration feature)

## Evaluation signals

- The combined (locked) system-parameter uncertainties from the multicurve MCMC are no larger than the uncertainties from refitting each light curve independently in single-curve mode
- Wavelet-based noise parameterization yields wider, more robust posteriors than a classic chi-squared treatment when correlated (red) noise is present
- Per-transit Tmid values are recovered for every curve in the matrix, confirming Tmid was not forced to a single shared value when per-transit timing is required
- Long-cadence Kepler light curves show no smearing artifacts, indicating the transit model was computed on a finer temporal scale and integrated to the observed cadence
- Results agree with prior publications when reanalyzing benchmark systems (e.g., TrES-1b, WASP-4b, WASP-10b, Kepler 4b-8b)

## Limitations

- Different fitting methods can lead to variations in derived quantities, and confidence quantification depends critically on the treatment of noise, stellar limb darkening, and cadence
- TAP fits only the Mandel and Agol (2002) analytic transit model (via EXOFAST); alternate packages like JKTEBOP use different models
- Long-integration (Kepler-style) observations require enabling the finer-temporal-scale model computation; without it, transit parameters can be distorted

## Evidence

- [abstract] Simultaneous analysis of multiple transits observed under different conditions: "The software is able to simultaneously analyze multiple transits observed in different conditions (instrument, filter, weather, etc.)."
- [intro] No phase folding or rebinning requirement: "TAP has been developed for the simultaneous analysis of multiple light curves of varying cadence, filter (limb darkening), and noise (observational) conditions with no need to phase fold and rebin"
- [intro] Wavelet-based likelihood for white and red noise: [evidence span withheld — anchor status: not_found]
- [intro] Long-integration handling for Kepler light curves: "For long integration observations (as is the case with most Kepler light curves) the software can be set to calculate transit models on a finer temporal scale"
- [intro] EXOFAST implementation of the analytic transit model: "The TAP software package uses EXOFAST [23], an improved implementation of the [15] analytic transit model"
