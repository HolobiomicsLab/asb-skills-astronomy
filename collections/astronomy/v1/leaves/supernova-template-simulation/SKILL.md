---
name: supernova-template-simulation
description: Use when a task requires generating synthetic transient (e.g. SN Ia)
  light curves as they would be observed by a survey such as ZTF, LSST, DES, SNLS,
  or Roman, including realistic fluxes and per-photopoint uncertainties.
when_to_use_negative:
- The task is to analyze real measured SN Ia light curves (e.g. the ~3000 SNe Ia of
  the ZTF SN Ia DR2 sample) rather than simulate them
- Noise-free light curves are wanted without intending to change the simulation semantics
  — only pass incl_error=False explicitly for that purpose
- A non-photometric, non-survey transient simulation (e.g. spectral modeling) is required;
  skysurvey is built around survey pointings and bandpass photometry via sncosmo
edam_operation: http://edamontology.org/operation_0244
edam_topics: []
tools:
- name: skysurvey
  role: 'Core framework: creates Target, Survey, and DataSet and simulates how the
    survey observes the targets'
  repo: https://github.com/MickaelRigault/skysurvey
- name: sncosmo
  role: Backend for bandpass and light-curve generation; provides TimeSerie Sources
    used as predefined targets
- name: modeldag
  role: Backend for building realistic targets through complex parametrization of
    the truth population
  repo: https://github.com/MickaelRigault/modeldag
- name: healpy
  role: Backend used by Survey to match target positions with observing history
- name: shapely
  role: Used by GridSurvey for footprint/geometry handling
- name: geopandas
  role: Used by GridSurvey for field-based geometry handling
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:38657bf2758c50151dd964f14a2af70be06c37b71b16fc15f51e750ae4a8490f
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
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# supernova-template-simulation

## Summary

Simulate how a sky survey observes transient populations using the skysurvey framework: create a Target (transient truth population) and a Survey (pointings/observing conditions), then build a DataSet of simulated light curves. The paper validates this against ZTF SN Ia DR2 observations.

## When to use

Use when a task requires generating synthetic transient (e.g. SN Ia) light curves as they would be observed by a survey such as ZTF, LSST, DES, SNLS, or Roman, including realistic fluxes and per-photopoint uncertainties.

## When NOT to use

- The task is to analyze real measured SN Ia light curves (e.g. the ~3000 SNe Ia of the ZTF SN Ia DR2 sample) rather than simulate them
- Noise-free light curves are wanted without intending to change the simulation semantics — only pass incl_error=False explicitly for that purpose
- A non-photometric, non-survey transient simulation (e.g. spectral modeling) is required; skysurvey is built around survey pointings and bandpass photometry via sncosmo

## Inputs

- Target object with drawn truth population (e.g. skysurvey.SNeIa with snia.draw(size=50_000))
- Observing logs: LSST opsim database file (baseline_v3.3_10yrs.db) or ZTF logs
- Survey footprint (e.g. shapely geometry box) and per-pointing observing conditions: ra, dec, gain, zp, skynoise, mjd, band (e.g. desg/desr/desi)
- sncosmo TimeSerie Source templates (e.g. 'v19-2005bf-corr' with magabs=[-18, 1])

## Outputs

- DataSet whose .data contains simulated light curves with fluxes and uncertainties per photopoint
- DataSet attributes .survey (input survey) and .targets (input targets)
- Exported table (e.g. dataset.csv) of simulated light curves

## How to apply

Install skysurvey via pip, then instantiate a predefined target (e.g. skysurvey.SNeIa) and draw the truth population with snia.draw(size=50_000, inplace=True), which populates snia.data. Build the survey from observing logs: use skysurvey.LSST.from_opsim(opsim_path) for an opsim database like baseline_v3.3_10yrs.db, skysurvey.ZTF.from_logs() for ZTF observing logs, or Survey.from_pointings(data, footprint=...) / GridSurvey.from_pointings for custom pointings (Survey uses healpy to match positions with observing history; GridSurvey uses shapely and geopandas). Then simulate how the survey observes the targets with skysurvey.DataSet.from_targets_and_survey(snia, survey), which includes observational noise by default (set incl_error=False only for noise-free light curves). The resulting dset.data holds the simulated light curves, dset.survey the input survey, and dset.targets the targets; export to dataset.csv. Validate by checking that simulated fluxes and uncertainties agree with measured uncertainties — the paper found sky-noise from observed science magnitude limits needs correction factors of 1.23 (g), 1.17 (r), 1.20 (i), plus error floors of 2.5%, 3.5%, and 6% in g, r, i.

## Related tools

- **skysurvey** (Core framework: creates Target, Survey, and DataSet and simulates how the survey observes the targets) — https://github.com/MickaelRigault/skysurvey
- **sncosmo** (Backend for bandpass and light-curve generation; provides TimeSerie Sources used as predefined targets)
- **modeldag** (Backend for building realistic targets through complex parametrization of the truth population) — https://github.com/MickaelRigault/modeldag
- **healpy** (Backend used by Survey to match target positions with observing history)
- **shapely** (Used by GridSurvey for footprint/geometry handling)
- **geopandas** (Used by GridSurvey for field-based geometry handling)

## Examples

```
import skysurvey; snia = skysurvey.SNeIa(); snia.draw(size=50_000, inplace=True); survey = skysurvey.LSST.from_opsim('baseline_v3.3_10yrs.db'); dset = skysurvey.DataSet.from_targets_and_survey(snia, survey); dset.data.to_csv('dataset.csv')
```

## Evaluation signals

- dataset.csv exists and is a non-empty table of simulated light curves
- Simulated fluxes and associated uncertainties are consistent with the survey observing conditions recorded in dset.survey
- Simulated fluxes and uncertainties agree with measured uncertainties once sky-noise correction factors (1.23 g, 1.17 r, 1.20 i) and error floors (2.5% g, 3.5% r, 6% i) are applied
- After applying survey selection criteria, the simulated sample reproduces the observed sample statistics (as done against ZTF SN Ia DR2)
- Truth population is present as snia.data / target.from_draw output with the requested draw size

## Limitations

- Loading large opsim files (e.g. baseline_v3.3_10yrs.db) may take a few minutes
- Sky-noise deduced from observed science magnitude limits requires band-dependent correction factors (1.23 g, 1.17 r, 1.20 i) and band-dependent error floors (2.5%, 3.5%, 6%) to match measured uncertainties
- Predefined targets/surveys are limited to those implemented (SNe Ia, SNII, any sncosmo TimeSerie Source; ZTF, DES shallow/deep, SNLS, LSST, Roman) — other cases require custom Target/Survey construction
- No changelog available for the skysurvey repository

## Evidence

- [methods] skysurvey_ is a python package made to simulate astronomical targets as they would be observed by a survey.: "skysurvey_ is a python package made to simulate astronomical targets as they would be observed by a survey."
- [intro] skysurvey relies on sncosmo for bandpass and lightcurve generations.: "skysurvey relies on sncosmo for bandpass and lightcurve generations."
- [methods] `Survey` uses healpy_ as backend to match position with observing history: "`Survey` uses healpy_ as backend to match position with observing history"
- [methods] ``GridSurvey`` uses shapely_ and geopandas_: "``GridSurvey`` uses shapely_ and geopandas_"
- [intro] You need to create a `Target` and a `Survey` to then simulate how your survey would observe your targets ; aka a `DataSet`.: "You need to create a `Target` and a `Survey` to then simulate how your survey would observe your targets ; aka a `DataSet`."
- [methods] dset = skysurvey.DataSet.from_targets_and_survey(snia, survey): "dset = skysurvey.DataSet.from_targets_and_survey(snia, survey)"
- [methods] survey = skysurvey.LSST.from_opsim(opsim_path): "survey = skysurvey.LSST.from_opsim(opsim_path)"
- [abstract] the sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band: "the sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band"
- [abstract] an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands: "an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands"
- [readme] pip install skysurvey: "bash
pip install skysurvey
"
- [readme] the simulated lightcurves are in dset.data, the input survey is stored in dset.survey, the input targets is stored in dset.targets: "The simulated lightcurves are in
dset.data, the input survey is stored in dset.survey, the input
targets is stored in dset.targets"
