---
name: photometric-noise-modelling
description: Use when simulated photometric flux uncertainties from a survey simulation
  framework (e.g. skysurvey) need to be validated against — or tuned to match — the
  measured uncertainties of an observed sample such as ZTF SN Ia DR2, requiring band-dependent
  sky-noise corrections and error floors.
when_to_use_negative:
- When only simulated fluxes (not uncertainties) are needed and no comparison to measured
  uncertainties is intended — the sky-noise rescaling and floors are specific to uncertainty
  validation.
- When working with a survey or instrument other than the calibrated bands (g/r/i
  of ZTF-like photometry); the correction factors 1.23/1.17/1.20 and floors 2.5%/3.5%/6%
  are band- and instrument-specific and would need re-derivation.
- When the observed sample has not yet had its selection criteria applied — agreement
  was established only after applying the ZTF SN Ia DR2 selection criteria to the
  simulated sample.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: skysurvey
  role: Simulation framework that generates the survey-observed photometry whose flux
    uncertainties are calibrated; includes error floors in the uncertainty model
  repo: https://github.com/MickaelRigault/skysurvey
- name: sncosmo
  role: Backend used by skysurvey for bandpass and light-curve generation underlying
    the simulated fluxes and errors
- name: modeldag
  role: Backend for building complex parametrisation of realistic targets feeding
    the simulated noise model
  repo: https://github.com/MickaelRigault/modeldag
- name: healpy
  role: Backend used by Survey to match transient positions with observing history
    so each simulated point inherits the correct sky-noise conditions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:c5c1fb740cb594e92ae1f1509cea1e3b6ac112dfc266d725279c21fec97a5234
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
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# photometric-noise-modelling

## Summary

Calibrates the per-point photometric flux uncertainty model in simulated survey data so that simulated flux errors match measured uncertainties, using band-dependent sky-noise scaling factors and multiplicative error floors.

## When to use

Use when simulated photometric flux uncertainties from a survey simulation framework (e.g. skysurvey) need to be validated against — or tuned to match — the measured uncertainties of an observed sample such as ZTF SN Ia DR2, requiring band-dependent sky-noise corrections and error floors.

## When NOT to use

- When only simulated fluxes (not uncertainties) are needed and no comparison to measured uncertainties is intended — the sky-noise rescaling and floors are specific to uncertainty validation.
- When working with a survey or instrument other than the calibrated bands (g/r/i of ZTF-like photometry); the correction factors 1.23/1.17/1.20 and floors 2.5%/3.5%/6% are band- and instrument-specific and would need re-derivation.
- When the observed sample has not yet had its selection criteria applied — agreement was established only after applying the ZTF SN Ia DR2 selection criteria to the simulated sample.

## Inputs

- Simulated light curves from skysurvey DataSet with per-point flux uncertainties (incl_error=True)
- Survey observing logs with mjd, band, zeropoint, skynoise, gain, ra/dec (via skysurvey.ZTF.from_logs())
- Measured photometric uncertainties from the ZTF SN Ia DR2 sample

## Outputs

- Calibrated per-point flux uncertainty model with band-dependent sky-noise corrections (1.23 g, 1.17 r, 1.20 i)
- Band-dependent error floors (2.5% g, 3.5% r, 6% i of flux level)
- Simulated light curves whose uncertainties agree with measured DR2 uncertainties

## How to apply

Build the simulated dataset with skysurvey (DataSet.from_targets_and_survey on drawn SNeIa targets and the ZTF survey loaded via ZTF.from_logs()), generating photometry with per-point flux uncertainties (incl_error=True). Compare the simulated uncertainties to the measured uncertainties of the observed sample after applying the sample's selection criteria. When the raw sky-noise deduced from the observed science magnitude limits is insufficient, rescale it by band-dependent factors — 1.23 in g, 1.17 in r, and 1.20 in i — and add error floors of 2.5%, 3.5%, and 6% of the flux level in g, r, and i respectively. The rationale is that the observed science magnitude limits underestimate the true sky noise, so a calibrated correction reconciles simulated flux errors with real instrument performance. Iterate until simulated fluxes and associated uncertainties agree with the measured uncertainties band by band.

## Related tools

- **skysurvey** (Simulation framework that generates the survey-observed photometry whose flux uncertainties are calibrated; includes error floors in the uncertainty model) — https://github.com/MickaelRigault/skysurvey
- **sncosmo** (Backend used by skysurvey for bandpass and light-curve generation underlying the simulated fluxes and errors)
- **modeldag** (Backend for building complex parametrisation of realistic targets feeding the simulated noise model) — https://github.com/MickaelRigault/modeldag
- **healpy** (Backend used by Survey to match transient positions with observing history so each simulated point inherits the correct sky-noise conditions)

## Examples

```
survey = skysurvey.ZTF.from_logs(); dset = skysurvey.DataSet.from_targets_and_survey(snia, survey, incl_error=True)
```

## Evaluation signals

- After applying the corrections, simulated fluxes and associated uncertainties agree well with the measured uncertainties of the ZTF SN Ia DR2 sample.
- Sky-noise in each band is rescaled by exactly the calibrated factors: 1.23 (g), 1.17 (r), 1.20 (i).
- The error model includes floors of 2.5%, 3.5%, and 6% of the flux level in g, r, and i respectively, so no simulated uncertainty falls below these fractions of flux.
- Comparison is done after applying the ZTF SN Ia DR2 selection criteria to the simulated sample, matching the observed sample definition.

## Limitations

- The sky-noise deduced from observed science magnitude limits systematically underestimates the true sky noise, requiring band-dependent correction factors; these values are calibrated specifically for ZTF g/r/i.
- Error floors are multiplicative fractions of flux level (2.5%/3.5%/6%), so they do not capture additive noise regimes outside the DR2 flux range.
- The corrections were validated against the ZTF SN Ia DR2 sample (~3000 classified SNe Ia of cosmological quality) and may not transfer to other surveys or transient classes.

## Evidence

- [abstract] sky-noise deduced from observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band: "the sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band"
- [abstract] error floors of 2.5%, 3.5%, 6% of flux level in g, r, i: "an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands"
- [methods] survey uses healpy as backend to match position with observing history: "`Survey` uses healpy_ as backend to match position with observing history"
- [readme] Build dataset via DataSet.from_targets_and_survey: [evidence span withheld — anchor status: external_doc]
- [methods] ZTF survey loaded from observing logs: "survey = skysurvey.ZTF.from_logs()"
