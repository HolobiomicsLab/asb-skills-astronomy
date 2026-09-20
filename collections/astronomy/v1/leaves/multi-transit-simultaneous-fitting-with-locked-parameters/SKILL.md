---
name: multi-transit-simultaneous-fitting-with-locked-parameters
description: Use when several transit light curves observed under different conditions
  (instrument, filter, weather) must be fit at once with shared system parameters
  but free per-transit mid-transit times.
when_to_use_negative:
- The orbit is genuinely eccentric and must be fitted — this configuration locks e=0
  and omega=0, so eccentric solutions cannot be recovered
- You plan to phase fold or rebin the light curves — TAP was developed for simultaneous
  analysis with no need to phase fold and rebin, and this skill follows that configuration
- Only one light curve with no parameters to share is available — the skill's rationale
  is uniform simultaneous fitting across multiple transits observed under different
  conditions
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Transit Analysis Package (TAP)
  role: IDL graphical user interface that drives the simultaneous MCMC fit of multiple
    light curves with locked/shared parameters
  repo: zgazak/TAP
- name: IDL
  role: Host language and runtime for the TAP graphical user interface
- name: EXOFAST
  role: Provides the improved implementation of the Mandel and Agol (2002) analytic
    transit model fitted by TAP
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:4c3cd494575e265b13233b5a1d448aa04fa967cc52dc0ee60b3c1b5041e0d3b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - Transit Analysis Package (TAP)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# multi-transit-simultaneous-fitting-with-locked-parameters

## Summary

Fit multiple transit light curves simultaneously in TAP with a shared set of system parameters (e=0, omega=0 locked) while leaving per-transit mid-transit times, wavelet noise parameters, and airmass extinction trends free, validated against published parameter sets such as Winn et al. (2009) WASP-4b.

## When to use

Use when several transit light curves observed under different conditions (instrument, filter, weather) must be fit at once with shared system parameters but free per-transit mid-transit times.

## When NOT to use

- The orbit is genuinely eccentric and must be fitted — this configuration locks e=0 and omega=0, so eccentric solutions cannot be recovered
- You plan to phase fold or rebin the light curves — TAP was developed for simultaneous analysis with no need to phase fold and rebin, and this skill follows that configuration
- Only one light curve with no parameters to share is available — the skill's rationale is uniform simultaneous fitting across multiple transits observed under different conditions

## Inputs

- Ground-based transit light curves of a single target observed in multiple transits (e.g., the two Winn et al. 2009 WASP-4b transits)
- Per-filter limb-darkening specification for each light curve
- TAP configuration: e=0, omega=0 locks; shared system parameters; per-transit free Tmid, noise parameters, and airmass extinction trend

## Outputs

- Posterior median and 1-sigma uncertainties for inclination (deg), a/Rs, and Rp/Rs from the simultaneous MCMC
- Per-transit mid-transit times Tmid (e.g., 0.797484 +/- 0.000054 and 0.650490 +/- 0.000047 days for WASP-4b)
- Wavelet-based white and red noise parameters for each light curve
- A Table-3-equivalent row comparing TAP values against the published column (Winn et al. 2009)

## How to apply

Load the ground-based light curves (e.g., the two Winn et al. 2009 WASP-4b transits) into the TAP IDL graphical interface and configure it to fit the Mandel and Agol (2002) analytic transit model via EXOFAST under the Carter and Winn (2009) wavelet-based likelihood that parameterizes both uncorrelated (white) and correlated (red) photometric noise. Lock orbital eccentricity and argument of periastron (e=0, omega=0) and share the global system parameters across all transits, while leaving per-transit mid-transit times, noise parameters, and the airmass extinction trend free. Run the TAP MCMC simultaneously on all light curves without phase folding or rebinning, following the same configuration used for the paper's other ground-based reanalyses. Extract posterior medians and 1-sigma uncertainties for inclination, a/Rs, Rp/Rs, and each transit's Tmid, assembling them into a Table-3-equivalent row. The rationale is that different fitting methods lead to variations in derived quantities and that confidence depends critically on noise treatment, limb darkening, and cadence, so uniform locked-parameter simultaneous MCMC fitting enables direct comparison against prior publications.

## Related tools

- **Transit Analysis Package (TAP)** (IDL graphical user interface that drives the simultaneous MCMC fit of multiple light curves with locked/shared parameters) — zgazak/TAP
- **IDL** (Host language and runtime for the TAP graphical user interface)
- **EXOFAST** (Provides the improved implementation of the Mandel and Agol (2002) analytic transit model fitted by TAP)

## Evaluation signals

- Each fitted parameter and its uncertainty must be consistent with the published Winn et al. (2009) value within the reported 1-sigma errors (inclination 88.97 +/- 0.65 deg, a/Rs 5.476 +0.023/-0.030, Rp/Rs 0.15398 +/- 0.00055)
- Per-transit Tmid posteriors match the published values (0.797484 +/- 0.000054 and 0.650490 +/- 0.000047 days) within 1 sigma
- e and omega are confirmed locked at 0 and all system parameters shared across transits in the fitting configuration
- Light curves are analyzed without phase folding or rebinning, at native cadence
- Uncertainties come from the Carter and Winn (2009) wavelet likelihood, not a classic chi-squared treatment

## Limitations

- Derived transit parameters vary between fitting methods, and confidence quantification depends critically on noise treatment, stellar limb darkening, and cadence
- The locked-parameter configuration assumes a circular orbit (e=0, omega=0); eccentric orbital solutions are outside this skill's scope
- For long integration observations, transit models must be calculated on a finer temporal scale and integrated to the observed cadence to avoid distortion

## Evidence

- [abstract] Simultaneous multi-transit MCMC fit: "The software is able to simultaneously analyze multiple transits observed in different conditions (instrument, filter, weather, etc.)."
- [intro] No phase folding or rebinning: "TAP has been developed for the simultaneous analysis of multiple light curves of varying cadence, filter (limb darkening), and noise (observational) conditions with no need to phase fold and rebin"
- [abstract] Wavelet likelihood rationale: "a wavelet-based likelihood function developed by Carter and Winn (2009), which allows the MCMC to assess parameter uncertainties more robustly than classic χ2 methods by parameterizing uncorrelated"
- [abstract] Agreement with prior publications: [evidence span withheld — anchor status: not_found]
- [intro] Transit model implementation: "The TAP software package uses EXOFAST [23], an improved implementation of the [15] analytic transit model"
