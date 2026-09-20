---
name: transient-lightcurve-simulation
description: Use when the goal is to reproduce observed survey light curves of transients
  (e.g. ~3000 ZTF SN Ia DR2 objects) by simulating photometry under real observing
  conditions (mjd, band, zeropoint, skynoise, gain, ra/dec) and instrument error calibrations.
when_to_use_negative:
- The analysis targets volume-limited sample definition from redshift dependence of
  SALT2 parameters (stretch, colour) rather than light-curve simulation itself.
- The survey observing history is unavailable or you only need generic template light
  curves from sncosmo without survey-specific observing conditions.
- Using a different simulation stack (e.g. SNANA) is mandated; skysurvey and SNANA
  are distinct frameworks and their outputs are not interchangeable.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: skysurvey
  role: 'Core framework: instantiate built-in SNeIa targets, draw truth data, build
    the Survey from observing logs (ZTF.from_logs), and combine both into a DataSet
    via DataSet.from_targets_and_survey.'
  repo: https://github.com/MickaelRigault/skysurvey
- name: sncosmo
  role: Underlying engine for bandpass and light-curve generation used by skysurvey
    targets.
- name: healpy
  role: Backend used by the Survey to match transient positions with the observing
    history during DataSet construction.
- name: shapely
  role: Used with geopandas by GridSurvey to handle survey footprints when building
    surveys from fields and pointings.
- name: geopandas
  role: Used with shapely by GridSurvey for footprint/field geometry handling.
- name: modeldag
  role: Backend for generating realistic targets through complex parametrisations
    (DAG-defined parameter models).
  repo: https://github.com/MickaelRigault/modeldag
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:7b42535e11e99b05f728fc6f2ce3f7329bc788f5c26e10f615add4512a4a6f54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MickaelRigault/skysurvey
  tools:
  - Python
  - geopandas
  - healpy
  - modeldag
  - shapely
  - skysurvey
  - sncosmo
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# transient-lightcurve-simulation

## Summary

Simulate individual transient light curves (e.g. SNe Ia) as they would be observed by a real survey, by combining drawn target truth data with the survey's actual observing history and instrument performance calibrations, as done for the ZTF SN Ia DR2 sample with the skysurvey framework.

## When to use

Use when the goal is to reproduce observed survey light curves of transients (e.g. ~3000 ZTF SN Ia DR2 objects) by simulating photometry under real observing conditions (mjd, band, zeropoint, skynoise, gain, ra/dec) and instrument error calibrations.

## When NOT to use

- The analysis targets volume-limited sample definition from redshift dependence of SALT2 parameters (stretch, colour) rather than light-curve simulation itself.
- The survey observing history is unavailable or you only need generic template light curves from sncosmo without survey-specific observing conditions.
- Using a different simulation stack (e.g. SNANA) is mandated; skysurvey and SNANA are distinct frameworks and their outputs are not interchangeable.

## Inputs

- Transient truth properties drawn via skysurvey (e.g. SNeIa draw, ~3000 objects for ZTF DR2)
- Survey observing logs (mjd, band, zeropoint, skynoise, gain, ra/dec) loaded via ZTF.from_logs()
- Band-dependent instrument calibration factors: sky-noise corrections 1.23/1.17/1.20 and error floors 2.5%/3.5%/6% (g/r/i)
- Survey selection criteria to apply post-simulation (e.g. ZTF SN Ia DR2 criteria)

## Outputs

- Simulated individual light curves stored in the DataSet (dset.data), exportable as e.g. simulated_lightcurves.csv
- Simulated fluxes with associated per-point uncertainties per band (g, r, i)

## How to apply

First draw the 'truth' data by instantiating the built-in SNeIa transient class in skysurvey and calling draw() on roughly the target sample size (~3000 objects), populating the target data table with nature-given transient properties. Second, load the survey observing history with skysurvey.ZTF.from_logs(), which exposes the actual observing logs (mjd, band, zeropoint, skynoise, gain, ra/dec). Third, combine targets and survey via skysurvey.DataSet.from_targets_and_survey(snia, survey), which uses a healpy backend to match each transient's position to its observing history and generate simulated photometry. Fourth, include per-point flux uncertainties (incl_error=True) while applying the band-dependent instrument calibrations reported for ZTF: sky-noise corrections of 1.23 (g), 1.17 (r), 1.20 (i) on the skynoise deduced from observed science magnitude limits, plus error floors of 2.5%, 3.5%, and 6% of the flux level in g, r, i. Finally, extract the simulated light curves from the resulting DataSet (dset.data) and apply the survey's selection criteria before comparing simulated fluxes and uncertainties to the measured data. Validation rests on the agreement of simulated fluxes and uncertainties with measured uncertainties once these calibrations are applied.

## Related tools

- **skysurvey** (Core framework: instantiate built-in SNeIa targets, draw truth data, build the Survey from observing logs (ZTF.from_logs), and combine both into a DataSet via DataSet.from_targets_and_survey.) — https://github.com/MickaelRigault/skysurvey
- **sncosmo** (Underlying engine for bandpass and light-curve generation used by skysurvey targets.)
- **healpy** (Backend used by the Survey to match transient positions with the observing history during DataSet construction.)
- **shapely** (Used with geopandas by GridSurvey to handle survey footprints when building surveys from fields and pointings.)
- **geopandas** (Used with shapely by GridSurvey for footprint/field geometry handling.)
- **modeldag** (Backend for generating realistic targets through complex parametrisations (DAG-defined parameter models).) — https://github.com/MickaelRigault/modeldag

## Examples

```
import skysurvey; snia = skysurvey.SNeIa(); snia.draw(size=3000, inplace=True); survey = skysurvey.ZTF.from_logs(); dset = skysurvey.DataSet.from_targets_and_survey(snia, survey)
```

## Evaluation signals

- After applying the ZTF SN Ia DR2 selection criteria to the simulated sample, simulated fluxes and their uncertainties should agree well with the measured DR2 uncertainties.
- The applied sky-noise corrections must be band-dependent: 1.23 in g, 1.17 in r, 1.20 in i, relative to skynoise deduced from observed science magnitude limits.
- Error floors of 2.5% (g), 3.5% (r), and 6% (i) of the flux level must be included in the per-point uncertainties.
- Each simulated transient should be matched to the correct observing history (healpy position matching), so every simulated photometry point maps to a real ZTF pointing (mjd, band, zeropoint, skynoise, gain).
- The drawn sample size should be roughly the DR2 sample size (~3000 classified SNe Ia of cosmological quality).

## Limitations

- The skynoise deduced from observed science magnitude limits must be corrected by band-dependent factors (1.23 g, 1.17 r, 1.20 i); raw log skynoise values alone do not reproduce measured uncertainties.
- Simulated uncertainties only match observations after adding band-dependent error floors (2.5%/3.5%/6% in g/r/i); omitting them degrades agreement.
- Validation of flux/uncertainty agreement only holds after applying the ZTF SN Ia DR2 selection criteria to the simulated sample.
- No changelog is available for the skysurvey repository, so version-dependent behaviour changes may be hard to track.

## Evidence

- [abstract] sky-noise corrections and error floors: "the sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band"
- [abstract] error floors: "an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands"
- [methods] survey from logs: "survey = skysurvey.ZTF.from_logs()"
- [methods] dataset construction: "dset = skysurvey.DataSet.from_targets_and_survey(snia, survey)"
- [methods] healpy matching: "`Survey` uses healpy_ as backend to match position with observing history"
- [methods] skysurvey purpose: "skysurvey_ is a python package made to simulate astronomical targets as they would be observed by a survey."
- [readme] quick start workflow: "You need to create a `Target` and a `Survey` to then simulate how your survey would observe your targets ; aka a `DataSet`."
- [readme] target drawing: "snia = skysurvey.SNeIa()
data = snia.draw(size=50_000, tstart=56_000, tstop=56_100, inplace=True)"
