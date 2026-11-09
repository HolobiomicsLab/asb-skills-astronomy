---
name: ztf-survey-simulation-validation
description: Use when validating that a transient survey simulation reproduces measured
  light-curve flux uncertainties as a function of observing conditions, or when calibrating
  simulated sky-noise and error floors per photometric band against real survey data.
when_to_use_negative:
- The goal is forward simulation of a survey without comparison to measured uncertainties
  (e.g. LSST opsim forecasting via LSST.from_opsim) — no per-band noise calibration
  against data is then required
- The dataset is not flux-calibrated light curves with per-band uncertainties (e.g.
  already-aggregated statistics or spectra)
- The survey is not ZTF-like; the correction factors 1.23/1.17/1.20 and error floors
  2.5%/3.5%/6% are empirical to ZTF DR2 g/r/i and must be re-derived for other instruments
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: skysurvey
  role: Simulates transients as observed by a survey (Target, Survey, DataSet) and
    provides the ZTF survey implementation; sky-noise from observing logs is the component
    being corrected
  repo: https://github.com/MickaelRigault/skysurvey
- name: sncosmo
  role: Backend used by skysurvey for bandpass and light-curve generation of the SNe
    Ia targets
- name: modeldag
  role: Backend for building complex target parametrisations when drawing transient
    truth data
  repo: https://github.com/MickaelRigault/modeldag
- name: Python
  role: pandas/numpy/scipy used to load the DataSet, apply per-band corrections, and
    compute residual statistics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:86c144d58a555ba10cc11662b82124f3e4637e186c33a7364ee0024246136bd5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MickaelRigault/skysurvey
  tools:
  - Python
  - modeldag
  - skysurvey
  - sncosmo
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ztf-survey-simulation-validation

## Summary

Simulate ZTF SN Ia DR2 light curves with skysurvey, apply the DR2 selection criteria, then calibrate the simulated noise model per band (sky-noise corrections of 1.23/1.17/1.20 and error floors of 2.5%/3.5%/6% in g/r/i) until simulated fluxes and uncertainties agree with the measured data.

## When to use

Use when validating that a transient survey simulation reproduces measured light-curve flux uncertainties as a function of observing conditions, or when calibrating simulated sky-noise and error floors per photometric band against real survey data.

## When NOT to use

- The goal is forward simulation of a survey without comparison to measured uncertainties (e.g. LSST opsim forecasting via LSST.from_opsim) — no per-band noise calibration against data is then required
- The dataset is not flux-calibrated light curves with per-band uncertainties (e.g. already-aggregated statistics or spectra)
- The survey is not ZTF-like; the correction factors 1.23/1.17/1.20 and error floors 2.5%/3.5%/6% are empirical to ZTF DR2 g/r/i and must be re-derived for other instruments

## Inputs

- ZTF SN Ia DR2 simulated light-curve dataset (flux, fluxerr, band, observing conditions) as a pandas DataFrame from a skysurvey DataSet
- Measured ZTF DR2 light-curve flux uncertainties matched to the simulated points
- ZTF observing logs (for Survey.from_pointings / ZTF.from_logs)
- Observed science magnitude limits per band (source of the sky-noise correction factors)

## Outputs

- Per-band corrected simulated flux uncertainties (sky-noise scaled by 1.23/1.17/1.20 with 2.5%/3.5%/6% error floors in g/r/i)
- Residual statistics (mean and scatter of simulated-minus-measured flux and uncertainty residuals) per band
- simulated_vs_measured_flux_residuals.csv

## How to apply

Build the simulation with skysurvey: draw ZTF SN Ia targets (e.g. snia.draw with sncosmo-based light curves), load the ZTF survey from observing logs via ZTF.from_logs(), and generate the DataSet with DataSet.from_targets_and_survey. Apply the ZTF SN Ia DR2 selection criteria to the simulated sample, then rescale the simulated sky-noise component of the flux uncertainties per band by the factors deduced from observed science magnitude limits: 1.23 (g), 1.17 (r), 1.20 (i). Add per-band error floors in quadrature: 2.5% (g), 3.5% (r), 6% (i) of the flux level. Bin simulated and measured fluxes versus observing conditions and compute residual statistics (mean and scatter of simulated-minus-measured flux and uncertainty residuals) with numpy/scipy, comparing corrected simulated uncertainties to measured uncertainties per band. The rationale is that raw skysurvey sky-noise deduced from magnitude limits underestimates the measured ZTF uncertainties, so a per-band empirical correction plus flux-proportional error floor is needed before the simulation can be considered valid.

## Related tools

- **skysurvey** (Simulates transients as observed by a survey (Target, Survey, DataSet) and provides the ZTF survey implementation; sky-noise from observing logs is the component being corrected) — https://github.com/MickaelRigault/skysurvey
- **sncosmo** (Backend used by skysurvey for bandpass and light-curve generation of the SNe Ia targets)
- **modeldag** (Backend for building complex target parametrisations when drawing transient truth data) — https://github.com/MickaelRigault/modeldag
- **Python** (pandas/numpy/scipy used to load the DataSet, apply per-band corrections, and compute residual statistics)

## Examples

```
import skysurvey; survey = skysurvey.ZTF.from_logs(); dset = skysurvey.DataSet.from_targets_and_survey(snia, survey)
```

## Evaluation signals

- Corrected simulated uncertainties agree with measured uncertainties in each band (g, r, i): residual means consistent with zero
- Residual scatters comparable to the measured scatter within the corrected noise model
- Sky-noise rescaling factors match the empirically deduced values 1.23 (g), 1.17 (r), 1.20 (i)
- Error floors present in quadrature at 2.5% (g), 3.5% (r), 6% (i) of the flux level
- simulated_vs_measured_flux_residuals.csv contains the per-band residual statistics table

## Limitations

- Correction factors and error floors are empirical to ZTF DR2 (g/r/i) and derived from observed science magnitude limits; they do not transfer to other surveys or instruments
- The framework's agreement holds only after applying the ZTF SN Ia DR2 selection criteria to the simulated sample; raw uncut simulated data need not agree
- No changelog is available for the skysurvey repository, so version-dependent behaviour of the noise model cannot be tracked from the repo alone

## Evidence

- [abstract] Per-band sky-noise correction factors: "the sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band"
- [abstract] Per-band error floors: "an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands"
- [abstract] Agreement conditional on selection criteria: "After the ZTF SN Ia DR2 selection criteria were applied, we found that the simulated fluxes and associated uncertainties agre well with the measured uncertainties"
- [readme] skysurvey relies on sncosmo: "skysurvey relies on sncosmo for bandpass and lightcurve generations."
- [methods] Dataset construction: "dset = skysurvey.DataSet.from_targets_and_survey(snia, survey)"
- [methods] Survey construction from logs: "survey = skysurvey.Survey.from_pointings(data, footprint=footprint)"
