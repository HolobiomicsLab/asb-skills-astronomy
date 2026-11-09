---
name: lightcurve-dataset-generation
description: Use when you need simulated, noise-inclusive light curves of transients
  (e.g. SNe Ia) as they would be observed by a survey such as ZTF or LSST, starting
  from a target population and pointing/obsim observing history.
when_to_use_negative:
- You only need the transient truth parameters without survey observation effects
  — use snia.draw alone instead of building a full DataSet
- You require noise-free light curves — pass incl_error=False rather than relying
  on the default noisy dataset, or this skill's default output (with noise) is the
  wrong product
- The task is to fit or classify real observed light curves rather than simulate how
  a survey would observe a target population
tools:
- name: skysurvey
  role: 'Core framework: provides Target, Survey, and DataSet classes; DataSet.from_targets_and_survey
    builds the simulated light curves'
  repo: https://github.com/MickaelRigault/skysurvey
- name: sncosmo
  role: Backend skysurvey relies on for bandpass and light-curve generations
- name: modeldag
  role: Backend for generating realistic targets with complex parametrisation when
    building the target model
  repo: https://github.com/MickaelRigault/modeldag
- name: healpy
  role: Backend used by Survey to match positions with observing history
- name: shapely
  role: Used by GridSurvey to define the footprint (e.g. geometry.box(-1, -1, +1,
    +1))
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:dab63cac2e95c2e865389db58473f1ef7305645ad2c2b7e907dc8b633224b029
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MickaelRigault/skysurvey
  tools:
  - Python
  - healpy
  - modeldag
  - shapely
  - skysurvey
  - sncosmo
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# lightcurve-dataset-generation

## Summary

Simulate survey-observed transient light curves by pairing a skysurvey Target (e.g. SNeIa truth population) with a Survey built from observing logs or an opsim database, producing a DataSet of simulated fluxes and uncertainties that reproduce survey observing conditions.

## When to use

Use when you need simulated, noise-inclusive light curves of transients (e.g. SNe Ia) as they would be observed by a survey such as ZTF or LSST, starting from a target population and pointing/obsim observing history.

## When NOT to use

- You only need the transient truth parameters without survey observation effects — use snia.draw alone instead of building a full DataSet
- You require noise-free light curves — pass incl_error=False rather than relying on the default noisy dataset, or this skill's default output (with noise) is the wrong product
- The task is to fit or classify real observed light curves rather than simulate how a survey would observe a target population

## Inputs

- Target object with drawn 'truth' population (e.g. skysurvey.SNeIa with snia.data set by snia.draw(size=50_000, inplace=True))
- Survey built from an opsim database file (e.g. baseline_v3.3_10yrs.db via LSST.from_opsim) or from pointing data (ra, dec, gain, zp, skynoise, mjd, band) with a footprint (e.g. shapely geometry box)
- Optional custom sncosmo TimeSerie Source name and magabs range (e.g. TSTransient('v19-2005bf-corr', magabs=[-18, 1]))

## Outputs

- DataSet object: dset.data (simulated light curves with fluxes and uncertainties), dset.survey, dset.targets
- dataset.csv: exported non-empty table of simulated light curves

## How to apply

Instantiate the target (e.g. skysurvey.SNeIa) and draw the transient 'truth' population with snia.draw(size=50_000, inplace=True), which populates snia.data with parameters like z, t0, x0, x1, c. Build the survey either from an opsim database (skysurvey.LSST.from_opsim(opsim_path), where opsim_path is the full path to baseline_v3.3_10yrs.db and loading may take minutes) or from observing logs (Survey.from_pointings / ZTF.from_logs), supplying ra, dec, gain, zp, skynoise, mjd, and band columns plus a footprint. Then create the dataset with skysurvey.DataSet.from_targets_and_survey(snia, survey); by default observational noise is included — pass incl_error=False only for noise-free light curves. The simulated light curves land in dset.data with the input survey in dset.survey and targets in dset.targets; export to dataset.csv and validate that simulated fluxes and uncertainties are consistent with the survey observing conditions in dset.survey.

## Related tools

- **skysurvey** (Core framework: provides Target, Survey, and DataSet classes; DataSet.from_targets_and_survey builds the simulated light curves) — https://github.com/MickaelRigault/skysurvey
- **sncosmo** (Backend skysurvey relies on for bandpass and light-curve generations)
- **modeldag** (Backend for generating realistic targets with complex parametrisation when building the target model) — https://github.com/MickaelRigault/modeldag
- **healpy** (Backend used by Survey to match positions with observing history)
- **shapely** (Used by GridSurvey to define the footprint (e.g. geometry.box(-1, -1, +1, +1)))

## Examples

```
import skysurvey; snia = skysurvey.SNeIa(); snia.draw(size=50_000, inplace=True); survey = skysurvey.LSST.from_opsim(opsim_path); dset = skysurvey.DataSet.from_targets_and_survey(snia, survey); dset.data.to_csv('dataset.csv')
```

## Evaluation signals

- dataset.csv exists and is a non-empty table of simulated light curves
- Simulated fluxes and associated uncertainties are consistent with the survey observing conditions recorded in dset.survey (the paper's check that simulated uncertainties agree with measured uncertainties)
- dset.survey and dset.targets contain the exact input survey and targets passed to from_targets_and_survey
- Truth population present in snia.data after the inplace draw (size=50_000 rows in the quick-start configuration)
- Running with incl_error=False yields noise-free light curves, confirming noise was injected by default

## Limitations

- Loading large opsim files such as baseline_v3.3_10yrs.db may take a few minutes
- Reproducing a real survey's flux uncertainties may require band-dependent corrections to the sky-noise deduced from observed magnitude limits (1.23 for g, 1.17 for r, 1.20 for i in the ZTF DR2 validation) and error floors (2.5% g, 3.5% r, 6% i)
- Only surveys already implemented in skysurvey (ZTF, DES shallow and deep fields, SNLS, LSST, Roman) can be loaded by their predefined classes; custom surveys must be built from pointings with a footprint

## Evidence

- [readme] You need to create a `Target` and a `Survey` to then simulate how your survey would observe your targets ; aka a `DataSet`.: "You need to create a `Target` and a `Survey` to then
simulate how your survey would observe your targets ; 
aka a `DataSet`."
- [methods] dset = skysurvey.DataSet.from_targets_and_survey(snia, survey): "dset = skysurvey.DataSet.from_targets_and_survey(snia, survey)"
- [abstract] the simulated fluxes and associated uncertainties agre well with the measured uncertainties: "the simulated fluxes and associated uncertainties agre well with the measured uncertainties"
- [intro] skysurvey relies on sncosmo for bandpass and lightcurve generations.: "skysurvey relies on sncosmo for bandpass and lightcurve generations."
- [methods] `Survey` uses healpy_ as backend to match position with observing history: "`Survey` uses healpy_ as backend to match position with observing history"
