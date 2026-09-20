---
name: transit-parameter-comparison-across-publications
description: Use when a published transit system parameter set (e.
when_to_use_negative:
- 'The light curve contains demonstrable correlated (red) noise: locking red noise
  at zero emulates a classic chi-squared analysis and will understate parameter uncertainties,
  so the locked-vs-free comparison should not be relied on for uncertainty claims'
- The goal is parameter derivation from scratch rather than verification of an existing
  published fit — a single well-motivated TAP run suffices; the dual-run comparison
  is a reproducibility protocol
- The original analysis used a different transit model or fixed-parameter set that
  cannot be matched (TAP requires setting period and eccentricity to the published
  fixed values for a faithful comparison)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Transit Analysis Package (TAP)
  role: IDL GUI that runs the MCMC transit fits (Mandel & Agol 2002 model) with the
    wavelet-based likelihood, with red noise locked or free
  repo: https://github.com/zgazak/TAP
- name: IDL
  role: Interactive Data Language environment in which the TAP graphical user interface
    is executed
- name: EXOFAST
  role: Implementation of the analytic transit model used inside TAP for the light-curve
    fitting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:51af22dea64abaad15c9d3e1acb3ce2a5948d11340e7ef1716e3005c27efaa82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - IDL (Interactive Data Language)
  - Transit Analysis Package (TAP)
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# transit-parameter-comparison-across-publications

## Summary

Reanalysis of a published exoplanet transit light curve with an independent MCMC fitting pipeline (TAP: Mandel & Agol 2002 model, wavelet likelihood) to verify whether the originally published system parameters (inclination, a/Rs, Rp/Rs, Tmid) are recovered, and to compile a side-by-side parameter comparison table. Serves as a reproducibility check on literature transit results and diagnoses whether red noise was the source of any discrepancy.

## When to use

Use when a published transit system parameter set (e.g. inclination, a/Rs, Rp/Rs, Tmid) needs independent verification: load the original light curve, refit with both a classic chi-squared setup (red noise locked at zero) and a free-red-noise wavelet-likelihood MCMC, and compare recovered values to the published ones within uncertainties.

## When NOT to use

- The light curve contains demonstrable correlated (red) noise: locking red noise at zero emulates a classic chi-squared analysis and will understate parameter uncertainties, so the locked-vs-free comparison should not be relied on for uncertainty claims
- The goal is parameter derivation from scratch rather than verification of an existing published fit — a single well-motivated TAP run suffices; the dual-run comparison is a reproducibility protocol
- The original analysis used a different transit model or fixed-parameter set that cannot be matched (TAP requires setting period and eccentricity to the published fixed values for a faithful comparison)

## Inputs

- Ground-based transit light curve from the original publication (flux vs. time, no phase folding)
- Fixed orbital period and eccentricity values as used in the original publication
- Published system parameter set (inclination, a/Rs, Rp/Rs, Tmid) with uncertainties for comparison

## Outputs

- Side-by-side comparison table of fitted inclination, a/Rs, Rp/Rs, and Tmid (red-noise-locked run vs. red-noise-free run vs. published values), e.g. inclination 88.44 vs 88.50 deg, a/Rs 11.60 vs 11.64, Rp/Rs 0.1593 vs 0.1588
- Fitted red-noise amplitude from the free run, indicating whether the light curve contains significant correlated noise
- MCMC posterior chains for each fitted transit parameter

## How to apply

Load the published ground-based transit light curve into Transit Analysis Package (TAP) in IDL with the original work's fixed orbital period and eccentricity. Run TAP's MCMC twice: once with the Carter & Winn (2009) wavelet-based likelihood and the correlated (red) noise amplitude locked at zero to emulate a classic chi-squared analysis, and once with the red-noise amplitude left free to fit. TAP fits light curves with MCMC using the Mandel & Agol (2002) analytic transit model via EXOFAST, without phase folding or rebinning. Extract inclination, a/Rs, Rp/Rs, and Tmid from both chains and compile them side-by-side with the published values into a comparison table. Accept the reproduction when both runs agree with the published parameters within uncertainties and the free-red-noise run returns a negligible fitted red-noise amplitude (e.g. the WASP-10b case: inclination 88.44 vs 88.50 deg, a/Rs 11.60 vs 11.64, Rp/Rs 0.1593 vs 0.1588).

## Related tools

- **Transit Analysis Package (TAP)** (IDL GUI that runs the MCMC transit fits (Mandel & Agol 2002 model) with the wavelet-based likelihood, with red noise locked or free) — https://github.com/zgazak/TAP
- **IDL** (Interactive Data Language environment in which the TAP graphical user interface is executed)
- **EXOFAST** (Implementation of the analytic transit model used inside TAP for the light-curve fitting)

## Evaluation signals

- Fitted inclination, a/Rs, and Rp/Rs from both MCMC runs agree with the published values within their quoted uncertainties (e.g. WASP-10b: inclination 88.44 vs 88.50 deg, a/Rs 11.60 vs 11.64, Rp/Rs 0.1593 vs 0.1588)
- The fitted red-noise parameter in the free run is negligible, consistent with the original paper's noise assessment
- The red-noise-locked and red-noise-free runs yield mutually consistent parameters, indicating results are not driven by noise-model assumptions
- MCMC chains have converged (parameter posteriors are stationary and Tmid values match, e.g. 0.03731 vs 0.037312)

## Limitations

- Parameter uncertainties depend critically on the treatment of noise, stellar limb darkening, and cadence, so agreement or disagreement may reflect methodological choices rather than data quality
- Different fitting methods can lead to variations in derived transit parameters; the locked-red-noise run is only a valid approximation for light curves with negligible correlated noise
- Fixed period and eccentricity must be taken from the original work, so the comparison cannot validate those inputs themselves

## Evidence

- [abstract] TAP fits with wavelet likelihood; red noise lockable: "a wavelet-based likelihood function developed by Carter and Winn (2009), which allows the MCMC to assess parameter uncertainties more robustly than classic χ2 methods"
- [abstract] Good agreement of TAP reanalyses with prior publications including WASP-10b: [evidence span withheld — anchor status: not_found]
- [intro] TAP model and likelihood motivation: [evidence span withheld — anchor status: not_found]
- [intro] TAP built on EXOFAST implementing the Mandel & Agol model: "The TAP software package uses EXOFAST [23], an improved implementation of the [15] analytic transit model"
- [intro] No phase folding or rebinning required: "simultaneous analysis of multiple light curves of varying cadence, filter (limb darkening), and noise (observational) conditions with no need to phase fold and rebin"
