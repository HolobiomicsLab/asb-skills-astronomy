---
name: survey-instrument-calibration
description: Use when simulating survey photometry of transients under real observing
  conditions and you must match simulated flux uncertainties to measured survey uncertainties
  (e.g. ZTF g/r/i sky-noise and error floors).
when_to_use_negative:
- Input is already observed (measured) photometry rather than simulated light curves
  — no sky-noise correction or error floor needs to be inferred.
- The target survey is not ZTF and you lack the measured uncertainties needed to derive
  band-dependent correction factors and error floors; the g/r/i factors 1.23/1.17/1.20
  and floors 2.5%/3.5%/6% are ZTF DR2-specific.
edam_operation: http://edamontology.org/operation_3799
tools:
- name: skysurvey
  role: 'Simulation framework: draws targets, loads the ZTF survey from observing
    logs (ZTF.from_logs), and builds the DataSet with simulated photometry and per-point
    errors'
  repo: https://github.com/MickaelRigault/skysurvey
- name: sncosmo
  role: Backend used by skysurvey for bandpass and light-curve generation of the transient
    sources
- name: healpy
  role: Backend skysurvey's Survey uses to match transient positions with the observing
    history
- name: modeldag
  role: Backend for building complex, realistic parametrisations of target properties
    (e.g. magnitude, error model with floor)
  repo: https://github.com/MickaelRigault/modeldag
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:e77a2e558a8bbdcb449b03f3f83451bc4b716aa287793f1ac6b97d88617f3dcd
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# survey-instrument-calibration

## Summary

Calibrate a survey simulation's per-point flux uncertainties against real observing conditions by applying band-dependent sky-noise correction factors and fractional error floors, as validated on the ZTF SN Ia DR2 sample with the skysurvey framework.

## When to use

Use when simulating survey photometry of transients under real observing conditions and you must match simulated flux uncertainties to measured survey uncertainties (e.g. ZTF g/r/i sky-noise and error floors).

## When NOT to use

- Input is already observed (measured) photometry rather than simulated light curves — no sky-noise correction or error floor needs to be inferred.
- The target survey is not ZTF and you lack the measured uncertainties needed to derive band-dependent correction factors and error floors; the g/r/i factors 1.23/1.17/1.20 and floors 2.5%/3.5%/6% are ZTF DR2-specific.

## Inputs

- Survey observing logs (mjd, band, zeropoint, skynoise, gain, ra/dec), e.g. via skysurvey.ZTF.from_logs()
- Drawn target truth data (e.g. skysurvey SNeIa draw with ~3000 objects matching the DR2 sample size)
- Band-dependent sky-noise correction factors (1.23 g, 1.17 r, 1.20 i)
- Fractional error floors (2.5% g, 3.5% r, 6% i of flux level)

## Outputs

- A skysurvey DataSet (dset.data) of simulated light curves with calibrated per-point flux uncertainties under real observing conditions
- Calibrated simulated fluxes and uncertainties comparable to measured survey (ZTF SN Ia DR2) uncertainties

## How to apply

Build the dataset by combining drawn targets with the survey via skysurvey.DataSet.from_targets_and_survey(snia, survey), where the survey is loaded from actual observing logs (mjd, band, zeropoint, skynoise, gain, ra/dec) using skysurvey.ZTF.from_logs(). Generate photometry with per-point flux uncertainties (incl_error=True). Then correct the sky-noise deduced from the observed science magnitude limits by the empirically derived band-dependent factors: 1.23 in g, 1.17 in r, and 1.20 in i. Add fractional error floors of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands respectively. These calibrations were derived by requiring that simulated fluxes and uncertainties agree with the measured uncertainties of the ZTF SN Ia DR2 sample (~3000 classified SNe Ia) after applying its selection criteria.

## Related tools

- **skysurvey** (Simulation framework: draws targets, loads the ZTF survey from observing logs (ZTF.from_logs), and builds the DataSet with simulated photometry and per-point errors) — https://github.com/MickaelRigault/skysurvey
- **sncosmo** (Backend used by skysurvey for bandpass and light-curve generation of the transient sources)
- **healpy** (Backend skysurvey's Survey uses to match transient positions with the observing history)
- **modeldag** (Backend for building complex, realistic parametrisations of target properties (e.g. magnitude, error model with floor)) — https://github.com/MickaelRigault/modeldag

## Examples

```
import skysurvey; snia = skysurvey.SNeIa(); snia.draw(size=3000, inplace=True); survey = skysurvey.ZTF.from_logs(); dset = skysurvey.DataSet.from_targets_and_survey(snia, survey, incl_error=True)
```

## Evaluation signals

- After applying the ZTF SN Ia DR2 selection criteria, simulated fluxes and associated uncertainties should agree well with the measured DR2 uncertainties.
- The applied sky-noise correction must be 1.23 in g, 1.17 in r, and 1.20 in i (factors relative to sky-noise deduced from observed science magnitude limits).
- The applied error floors must equal 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands respectively.
- The dataset size should correspond to roughly the DR2 sample size (~3000 classified SNe Ia) before selection.
- dset.data should contain simulated photometry with per-point uncertainties (incl_error=True) for every matched pointing.

## Limitations

- The sky-noise correction factors and error floors are empirically calibrated for ZTF g, r, i bands and the ZTF SN Ia DR2 sample; they are not transferable to other instruments or surveys without re-derivation.
- Agreement holds only after the ZTF SN Ia DR2 selection criteria are applied to the simulated sample; raw simulated uncertainties may not match.
- skysurvey requires a Target and a Survey to produce a DataSet; positions are matched via healpy, so survey footprints/pointing coverage affect which targets get observations.

## Evidence

- [abstract] sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band: "the sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band"
- [abstract] error floors 2.5/3.5/6 percent: "an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands"
- [methods] ZTF.from_logs usage: "survey = skysurvey.ZTF.from_logs()"
- [methods] DataSet construction: "dset = skysurvey.DataSet.from_targets_and_survey(snia, survey)"
- [methods] healpy position matching: "`Survey` uses healpy_ as backend to match position with observing history"
- [abstract] DR2 sample size: "The ZTF SN Ia DR2 sample corresponds to almost 3000 classified SNe Ia of cosmological quality."
