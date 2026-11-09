---
name: simultaneous-multi-transit-parameter-sharing
description: Use when you have two or more transit light curves of the same planet
  observed under different conditions (cadence, filter/limb darkening, weather) and
  want one shared system-parameter set with a free mid-transit time per transit.
when_to_use_negative:
- You have only a single transit light curve, so there is no second light curve with
  which to share system parameters or to jointly constrain Tmid versus system parameters
- Each transit needs an independent, unshared parameter set (e.g., testing for parameter
  variation between nights) rather than the single shared set TAP's simultaneous mode
  enforces
- You need an eccentric (e and omega free) fit; this skill's validated configuration
  locks eccentricity and periastron argument at zero
edam_operation: http://edamontology.org/operation_2945
tools:
- name: Transit Analysis Package (TAP)
  role: IDL GUI that drives MCMC simultaneous fitting of multiple light curves with
    shared parameters and per-transit Tmid
  repo: zgazak/TAP
- name: IDL
  role: Interactive Data Language runtime environment in which the TAP graphical user
    interface is developed and executed
- name: EXOFAST
  role: Improved implementation of the Mandel and Agol (2002) analytic transit model
    used inside TAP's MCMC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:1924e9b9b5f4195c2403f31801edf2f2a0be4b47604be949817c57021b16c0a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - Transit Analysis Package (TAP)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# simultaneous-multi-transit-parameter-sharing

## Summary

Fits multiple transit light curves of one exoplanet simultaneously in TAP (IDL) with a single shared parameter set (inclination, a/Rs, Rp/Rs, limb darkening) while freeing a per-transit Tmid, using the Mandel and Agol (2002) model and the Carter and Winn (2009) wavelet likelihood to capture red noise. Validated by reproducing Winn et al. (2007) TrES-1b values (e.g., inclination 89.01 +/- 0.61 deg, Rp/Rs 0.1375 +/- 0.0012).

## When to use

Use when you have two or more transit light curves of the same planet observed under different conditions (cadence, filter/limb darkening, weather) and want one shared system-parameter set with a free mid-transit time per transit.

## When NOT to use

- You have only a single transit light curve, so there is no second light curve with which to share system parameters or to jointly constrain Tmid versus system parameters
- Each transit needs an independent, unshared parameter set (e.g., testing for parameter variation between nights) rather than the single shared set TAP's simultaneous mode enforces
- You need an eccentric (e and omega free) fit; this skill's validated configuration locks eccentricity and periastron argument at zero

## Inputs

- Two or more ground-based photometric transit time series of the same planet (e.g., the three TrES-1b light curves from Winn et al. 2007)
- Per-transit time stamps allowing a free Tmid per transit
- Filter/limb-darkening and cadence metadata per light curve

## Outputs

- Posterior medians and uncertainties for inclination, a/Rs, Rp/Rs shared across all transits
- Per-transit Tmid values with uncertainties (TrES-1b targets: ~0.84298, 0.87341, 0.90371)
- Comparison table written to res_tres1b_wasp4b_agreement.csv (Table 2-style)

## How to apply

Load each photometric transit time series (e.g., the three TrES-1b light curves of Winn et al. 2007) into the TAP IDL GUI; TAP handles differing cadence, filter (limb darkening), and noise conditions simultaneously with no need to phase fold and rebin. Configure the fit to the Mandel and Agol (2002) analytic transit model (implemented via EXOFAST) with eccentricity and periastron argument fixed at zero, a single shared parameter set for all transits, and a free Tmid per transit. Run the TAP MCMC using the Carter and Winn (2009) wavelet-based likelihood, which parameterizes both uncorrelated (white) and correlated (red) noise instead of classic chi-squared, yielding more robust uncertainties. Extract posterior medians and uncertainties for inclination, a/Rs, Rp/Rs, and each Tmid, and record them as a comparison table against the published values (TrES-1b targets: i = 89.01 +/- 0.61 deg, a/Rs = 10.46 +0.13/-0.23, Rp/Rs = 0.1375 +/- 0.0012, Tmid ~ 0.84298/0.87341/0.90371 HJD-fraction). Write the comparison to res_tres1b_wasp4b_agreement.csv.

## Related tools

- **Transit Analysis Package (TAP)** (IDL GUI that drives MCMC simultaneous fitting of multiple light curves with shared parameters and per-transit Tmid) — zgazak/TAP
- **IDL** (Interactive Data Language runtime environment in which the TAP graphical user interface is developed and executed)
- **EXOFAST** (Improved implementation of the Mandel and Agol (2002) analytic transit model used inside TAP's MCMC)

## Evaluation signals

- Reproduced inclination, a/Rs, and Rp/Rs agree with the published Winn et al. (2007) TrES-1b values within their quoted 1-sigma uncertainties (89.01 +/- 0.61 deg, 10.46 +0.13/-0.23, 0.1375 +/- 0.0012)
- The three extracted Tmid values match ~0.84298, ~0.87341, and ~0.90371 per transit
- The MCMC used the Carter and Winn (2009) wavelet likelihood, so reported uncertainties account for both uncorrelated and correlated (red) noise rather than classic chi-squared
- The Table 2-style comparison was written to res_tres1b_wasp4b_agreement.csv
- Broader sanity check: TAP reanalyses of TrES-1b, WASP-4b, WASP-10b, and Kepler 4b-8b show good agreement with prior publications

## Limitations

- Different fitting methods lead to variations in derived quantities, and confidence quantification depends critically on the treatment of noise, stellar limb darkening, and cadence
- Classic chi-squared is avoided precisely because the wavelet-based likelihood gives more robust uncertainties; chi-squared-based fits may underestimate errors
- For long integration observations (e.g., Kepler light curves), transit models must be calculated on a finer temporal scale and integrated to the observed cadence, or long-integration distortions will bias the fit

## Evidence

- [abstract] simultaneous multi-transit fitting across conditions: "The software is able to simultaneously analyze multiple transits observed in different conditions (instrument, filter, weather, etc.)."
- [intro] no phase folding or rebinning required: "TAP has been developed for the simultaneous analysis of multiple light curves of varying cadence, filter (limb darkening), and noise (observational) conditions with no need to phase fold and rebin"
- [abstract] MCMC with Mandel and Agol model: "uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002)."
- [abstract] wavelet likelihood for robust uncertainties: "a wavelet-based likelihood function developed by Carter and Winn (2009), which allows the MCMC to assess parameter uncertainties more robustly than classic χ2 methods by parameterizing uncorrelated"
- [abstract] validated against prior publications: [evidence span withheld — anchor status: not_found]
