---
name: mcmc-transit-light-curve-fitting
description: Use when you have one or more photometric transit time series (ground-based
  or Kepler 29.4-min long-cadence from MAST) and need MCMC-derived system parameters
  with uncertainties that properly treat correlated (red) noise.
when_to_use_negative:
- The input is not a transit light curve (e.g., radial-velocity time series or imaging
  data) — TAP fits photometric transit light curves only.
- A free-eccentricity orbit fit is required — the reanalysis configuration locks e
  = 0 and omega = 0, so eccentric orbits are outside this setup.
- You need a quick chi-squared point estimate without noise-model uncertainty — although
  TAP can emulate chi-squared by locking the red-noise amplitude at zero, the skill's
  purpose is the wavelet-likelihood MCMC.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Transit Analysis Package (TAP)
  role: IDL GUI that drives the MCMC transit fitting, wavelet likelihood, and simultaneous
    multi-light-curve analysis
  repo: https://github.com/zgazak/TAP
- name: EXOFAST
  role: Provides the Mandel & Agol (2002) analytic transit model used inside TAP's
    MCMC
- name: IDL (Interactive Data Language)
  role: Runtime environment in which the TAP graphical user interface is executed
provenance:
  source_task_ids:
  - task_003
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:ebb3cc89d25ee9b239215e097eadfb288d86facc7d9b02dcae7ca28e2b09d027
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# mcmc-transit-light-curve-fitting

## Summary

Fit exoplanet transit light curves with the Transit Analysis Package (TAP), an IDL GUI that runs MCMC against the Mandel & Agol (2002) analytic model (via EXOFAST) using the Carter & Winn (2009) wavelet likelihood that parameterizes both white and red noise, yielding robust posterior parameters (inclination, a/Rs, Rp/Rs, Tmid) for one or many simultaneous transits.

## When to use

Use when you have one or more photometric transit time series (ground-based or Kepler 29.4-min long-cadence from MAST) and need MCMC-derived system parameters with uncertainties that properly treat correlated (red) noise.

## When NOT to use

- The input is not a transit light curve (e.g., radial-velocity time series or imaging data) — TAP fits photometric transit light curves only.
- A free-eccentricity orbit fit is required — the reanalysis configuration locks e = 0 and omega = 0, so eccentric orbits are outside this setup.
- You need a quick chi-squared point estimate without noise-model uncertainty — although TAP can emulate chi-squared by locking the red-noise amplitude at zero, the skill's purpose is the wavelet-likelihood MCMC.

## Inputs

- Photometric transit light curve time series (ground-based, e.g., the three TrES-1b transits of Winn et al. 2007, two WASP-4b transits of Winn et al. 2009, one WASP-10b curve of Johnson et al. 2009)
- Kepler long-cadence photometry (29.4-minute cadence) for Kepler-4b through -8b retrieved from the MAST archive
- Fixed orbital elements for the target (e.g., orbital period, e = 0 and omega = 0 locked) and limb-darkening configuration (free or per-filter)

## Outputs

- Posterior medians and 1-sigma uncertainties for inclination, a/Rs, Rp/Rs, and per-transit mid-transit times (Tmid), e.g., inclination 89.01 +/- 0.61 deg, a/Rs 10.46 +0.13/-0.23, Rp/Rs 0.1375 +/- 0.0012 for TrES-1b
- Comparison table of reproduced vs. published parameters (e.g., res_tres1b_wasp4b_agreement.csv, tap_kepler_4_8_parameters.csv)
- Fitted red-noise amplitude indicating whether a light curve contains significant correlated noise

## How to apply

Load the light curve(s) into the TAP IDL GUI and fit the Mandel & Agol (2002) analytic transit model implemented through EXOFAST. Use the Carter & Winn (2009) wavelet-based likelihood instead of classic chi-squared so both uncorrelated and correlated (red) photometric noise are parameterized in the MCMC. For the reanalyses in this paper, lock eccentricity and argument of periastron at zero (e = 0, omega = 0), share a single set of system parameters across all transits, and leave per-transit Tmid (plus per-transit noise and any airmass extinction trend) free; for Kepler long-cadence data compute the model on a 10x finer temporal scale and rebin it to the 29.4-minute sampling to correct long-integration smearing. Run the chains simultaneously on all light curves with no phase folding or rebinning, then extract posterior medians and 1-sigma uncertainties for Period, inclination, a/Rs, Rp/Rs, and Tmid into a comparison table (e.g., CSV).

## Related tools

- **Transit Analysis Package (TAP)** (IDL GUI that drives the MCMC transit fitting, wavelet likelihood, and simultaneous multi-light-curve analysis) — https://github.com/zgazak/TAP
- **EXOFAST** (Provides the Mandel & Agol (2002) analytic transit model used inside TAP's MCMC)
- **IDL (Interactive Data Language)** (Runtime environment in which the TAP graphical user interface is executed)

## Evaluation signals

- Reproduced inclination, a/Rs, Rp/Rs, and per-transit Tmid values must fall within the 1-sigma uncertainties of the published values (e.g., Winn et al. 2007 TrES-1b: 89.01 +/- 0.61 deg, 10.46 +0.13/-0.23, 0.1375 +/- 0.0012).
- For the Kepler reanalyses, the Eq. (5) fractional difference versus Kipping & Bakos (2010) must satisfy |FD| < 1 for Kepler-4b through Kepler-7b.
- In the red-noise-free run, the fitted correlated-noise amplitude should be negligible when the light curve is not red-noise contaminated (as found for WASP-10b).
- Both the red-noise-locked (chi-squared emulation) and red-noise-free chains must converge to consistent parameters (e.g., WASP-10b inclination 88.44 vs 88.50 deg).
- Per-transit Tmid posteriors should match the published mid-transit times within 1-sigma (e.g., TrES-1b Tmid ~0.84298/0.87341/0.90371).

## Limitations

- Different fitting methods can lead to variations in derived transit quantities, and confidence depends critically on the treatment of noise, stellar limb darkening, and cadence.
- Significant discrepancies can remain even when most parameters agree — e.g., the Kepler-8b period difference versus Kipping & Bakos (2010) was flagged as significant.
- Long-integration (long-cadence) observations distort transit shapes unless the model is computed on a finer temporal scale and integrated/rebinned to the observed cadence.

## Evidence

- [abstract] The Transit Analysis Package (TAP) software uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002).: "The Transit Analysis Package (TAP) software uses Markov Chain Monte Carlo (MCMC) techniques to fit light curves using the analytic model of Mandal and Agol (2002)."
- [abstract] wavelet likelihood parameterizes white and red noise: "a wavelet-based likelihood function developed by Carter and Winn (2009), which allows the MCMC to assess parameter uncertainties more robustly than classic χ2 methods by parameterizing uncorrelated"
- [intro] simultaneous multi-light-curve analysis without phase folding: "TAP has been developed for the simultaneous analysis of multiple light curves of varying cadence, filter (limb darkening), and noise (observational) conditions with no need to phase fold and rebin"
- [intro] fine temporal scale for long integration: "For long integration observations (as is the case with most Kepler light curves) the software can be set to calculate transit models on a finer temporal scale"
