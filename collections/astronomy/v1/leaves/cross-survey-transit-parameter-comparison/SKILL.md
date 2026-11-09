---
name: cross-survey-transit-parameter-comparison
description: Use when a previously published exoplanet transit light curve (e.g. MAST
  Kepler long-cadence photometry or ground-based transits) needs an independent MCMC
  reanalysis whose system parameters are compared against the published values.
when_to_use_negative:
- The goal is producing new photometry rather than reanalyzing published light curves
  — this skill is a reanalysis-and-comparison workflow, not an observation/reduction
  pipeline
- The analysis requires phase folding and rebinning of many transits under differing
  conditions, rather than a targeted comparison of a specific published transit analysis
- The goal is fitting non-transit systematics alone (e.g. airmass extinction trends
  by themselves) without the transit model comparison to a published parameter set
edam_operation: http://edamontology.org/operation_2422
tools:
- name: Transit Analysis Package (TAP)
  role: IDL GUI that drives the MCMC reanalysis and simultaneous multi-light-curve
    fitting used to obtain the comparison posteriors
  repo: zgazak/TAP
- name: IDL
  role: Interactive Data Language environment in which TAP runs and the comparison
    tables are assembled
- name: EXOFAST
  role: Implementation of the Mandel & Agol (2002) analytic transit model called by
    TAP to generate the fitted light-curve model
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1155/2012/697967
    title: 'Transit Analysis Package: An IDL Graphical User Interface for Exoplanet
      Transit Photometry'
  source_content_hash: sha256:114855678085b85914e2cefc05a7e9ba4c7c32ab18ac2307f02cce7fd966309a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zgazak/TAP
  tools:
  - EXOFAST
  - IDL
  - Transit Analysis Package (TAP)
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cross-survey-transit-parameter-comparison

## Summary

Reanalyze one or more published transit light curves with TAP's MCMC (Mandel & Agol 2002 model via EXOFAST, Carter & Winn wavelet likelihood) and quantitatively compare the fitted Period, Inclination, a/Rs, and Rp/Rs against the original publication's values to verify statistical agreement across independent analysis methods.

## When to use

Use when a previously published exoplanet transit light curve (e.g. MAST Kepler long-cadence photometry or ground-based transits) needs an independent MCMC reanalysis whose system parameters are compared against the published values.

## When NOT to use

- The goal is producing new photometry rather than reanalyzing published light curves — this skill is a reanalysis-and-comparison workflow, not an observation/reduction pipeline
- The analysis requires phase folding and rebinning of many transits under differing conditions, rather than a targeted comparison of a specific published transit analysis
- The goal is fitting non-transit systematics alone (e.g. airmass extinction trends by themselves) without the transit model comparison to a published parameter set

## Inputs

- Published transit light curves, e.g. Kepler long-cadence photometry (29.4-minute cadence) for Kepler-4b through Kepler-8b from MAST
- Ground-based transit light curves from prior publications (e.g. the two WASP-4b transits of Winn et al. 2009, ref [42])
- Published parameter values from the comparison paper (e.g. KB10 Table 5 values; Winn et al. Table 3 targets such as inclination 88.97 +/- 0.65 deg, a/Rs 5.476 +0.023/-0.030, Rp/Rs 0.15398 +/- 0.00055)

## Outputs

- Posterior summaries (median and 1-sigma) of Period, Inclination, a/Rs, and Rp/Rs per planet, assembled into a parameter table (e.g. tap_kepler_4_8_parameters.csv)
- Per-parameter comparison against the published values within quoted 1-sigma, including Eq. (5) fractional-difference (FD) values versus KB10
- An agreement verdict per target, e.g. |FD| < 1 for Kepler-4b through -7b, with the Kepler-8b period difference flagged as significant, and agreement with Winn et al. (2009) for WASP-4b

## How to apply

Retrieve the published photometry (e.g. Kepler 29.4-minute long-cadence data from MAST, or ground-based light curves such as the two WASP-4b transits of Winn et al. 2009) and load it into TAP in IDL. Fit the Mandel & Agol (2002) analytic transit model via EXOFAST using the Carter & Winn (2009) wavelet-based likelihood that parameterizes uncorrelated (white) and correlated (red) noise; for Kepler long-cadence data, compute the model on a 10x finer temporal scale and rebin to the observed 29.4-minute sampling to correct long-integration smearing. Choose the appropriate configuration per comparison: for Kepler-4b through -8b lock circular orbits (e=0) and leave limb darkening free, while for shared multi-transit fits (e.g. WASP-4b) lock e=0, omega=0, share global system parameters across transits, and free only per-transit Tmid, noise parameters, and airmass extinction trend. Run the MCMC chains without phase folding or rebinning and extract posterior medians with 1-sigma uncertainties for Period, Inclination, a/Rs, and Rp/Rs. Judge agreement by checking each fitted value against the published column within its quoted 1-sigma (e.g. Table 3/5 targets) and, for the KB10 comparison, computing the Eq. (5) fractional difference FD and requiring |FD| < 1.

## Related tools

- **Transit Analysis Package (TAP)** (IDL GUI that drives the MCMC reanalysis and simultaneous multi-light-curve fitting used to obtain the comparison posteriors) — zgazak/TAP
- **IDL** (Interactive Data Language environment in which TAP runs and the comparison tables are assembled)
- **EXOFAST** (Implementation of the Mandel & Agol (2002) analytic transit model called by TAP to generate the fitted light-curve model)

## Evaluation signals

- Each fitted parameter matches the published value within the quoted 1-sigma (e.g. WASP-4b inclination 88.97 +/- 0.65 deg, a/Rs 5.476 +0.023/-0.030, Rp/Rs 0.15398 +/- 0.00055)
- Equation (5) fractional difference versus the KB10 values satisfies |FD| < 1 for Kepler-4b through Kepler-7b, reproducing the res_kepler_4_7_agreement result
- The Kepler-8b period difference is correctly flagged as significant, reproducing the res_kepler8_discrepancy result rather than masking it
- Posterior uncertainties are robust because the wavelet likelihood parameterizes both white and red noise instead of classic chi-squared
- Per-transit Tmid posteriors (e.g. 0.797484 +/- 0.000054 and 0.650490 +/- 0.000047 days for WASP-4b) are consistent with the published mid-transit times

## Limitations

- Agreement is method-dependent: derived transit parameters vary between fitting methods, so quantification of confidence depends critically on the treatment of noise, stellar limb darkening, and cadence
- Different comparison configurations are needed per target: circular orbits locked and limb darkening free for Kepler-4b–8b versus shared global parameters with free per-transit Tmid for multi-transit ground-based fits
- A known disagreement can be a genuine result: the Kepler-8b period difference versus KB10 is flagged as significant rather than reproduced as agreement

## Evidence

- [abstract] Good agreement between TAP reanalysis and prior publications: [evidence span withheld — anchor status: not_found]
- [intro] Wavelet likelihood replaces classic chi-squared for robust uncertainties: [evidence span withheld — anchor status: not_found]
- [intro] Finer temporal scale for long-integration Kepler light curves: "For long integration observations (as is the case with most Kepler light curves) the software can be set to calculate transit models on a finer temporal scale"
- [abstract] Simultaneous multi-light-curve analysis without phase folding or rebinning: "The software is able to simultaneously analyze multiple transits observed in different conditions (instrument, filter, weather, etc.)."
- [intro] Method-dependence caveat on derived parameters: [evidence span withheld — anchor status: not_found]
