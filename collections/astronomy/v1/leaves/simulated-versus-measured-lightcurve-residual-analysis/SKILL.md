---
name: simulated-versus-measured-lightcurve-residual-analysis
description: Use when simulated light-curve flux uncertainties must be matched to
  measured survey uncertainties (e.g. ZTF DR2 g/r/i photometry) and you need per-band
  sky-noise correction factors and error floors validated via residual statistics.
when_to_use_negative:
- The simulated data have not yet had the ZTF SN Ia DR2 selection criteria applied,
  so the comparison samples are not matched in quality cuts.
- The survey/bands differ from ZTF g/r/i and no observed science magnitude limits
  are available to deduce band-dependent sky-noise correction factors.
- The goal is survey design or target simulation from scratch (Target + Survey + DataSet
  construction) rather than validating simulated photometry against measured data.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: skysurvey
  role: Provides the simulation framework and DataSet whose fluxes/uncertainties are
    compared to the measured ZTF SN Ia DR2 uncertainties; installed via pip or cloned
    from GitHub.
  repo: https://github.com/MickaelRigault/skysurvey
- name: sncosmo
  role: Backend used by skysurvey for bandpass and light-curve generation feeding
    the simulated fluxes being validated.
  repo: https://sncosmo.readthedocs.io/en/stable/
- name: Python
  role: 'Implementation language: pandas for loading the light-curve dataset, numpy/scipy
    for binning and residual statistics.'
- name: healpy
  role: Survey backend used to match transient positions with the observing history
    when building the simulated dataset.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:b5915993cacc35e2b3bbf541d5272a149f3109214d46debdf822d1ad45b691ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MickaelRigault/skysurvey
  tools:
  - Python
  - healpy
  - skysurvey
  - sncosmo
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# simulated-versus-measured-lightcurve-residual-analysis

## Summary

Validates survey-simulated supernova light-curve fluxes and uncertainties against measured data by rescaling the simulated sky-noise per band (1.23 g, 1.17 r, 1.20 i), adding flux-level error floors in quadrature (2.5% g, 3.5% r, 6% i), and computing per-band residual statistics after applying the ZTF SN Ia DR2 selection criteria.

## When to use

Use when simulated light-curve flux uncertainties must be matched to measured survey uncertainties (e.g. ZTF DR2 g/r/i photometry) and you need per-band sky-noise correction factors and error floors validated via residual statistics.

## When NOT to use

- The simulated data have not yet had the ZTF SN Ia DR2 selection criteria applied, so the comparison samples are not matched in quality cuts.
- The survey/bands differ from ZTF g/r/i and no observed science magnitude limits are available to deduce band-dependent sky-noise correction factors.
- The goal is survey design or target simulation from scratch (Target + Survey + DataSet construction) rather than validating simulated photometry against measured data.

## Inputs

- Simulated skysurvey DataSet light curves (flux, fluxerr, band) for the ZTF SN Ia DR2 sample
- Measured ZTF SN Ia DR2 light-curve fluxes and uncertainties (matched to simulations)
- Observed science magnitude limits per band (g, r, i) used to derive sky-noise correction factors
- ZTF SN Ia DR2 selection criteria

## Outputs

- Per-band corrected simulated uncertainties (sky-noise rescaled by 1.23/1.17/1.20, error floors 2.5%/3.5%/6% added in quadrature)
- Binned flux and uncertainty residuals versus observing conditions (mean and scatter)
- simulated_vs_measured_flux_residuals.csv (per-band residual statistics table)

## How to apply

Load the simulated light-curve DataSet (flux, fluxerr, band, and matched measured uncertainties) with pandas inside the skysurvey environment, then apply the ZTF SN Ia DR2 selection criteria so simulated and measured samples are comparable. Rescale the sky-noise component of the simulated flux uncertainties per band using the correction factors deduced from the observed science magnitude limits: 1.23 (g), 1.17 (r), 1.20 (i). Add the per-band error floors in quadrature as fractions of the flux level: 2.5% (g), 3.5% (r), 6% (i). Bin simulated and measured fluxes versus observing conditions and compute residual statistics (mean and scatter of simulated-minus-measured flux and uncertainty residuals) with numpy/scipy, then write the per-band residual table to simulated_vs_measured_flux_residuals.csv. Agreement is judged by residual means consistent with zero and residual scatters comparable to the measured scatter within the corrected noise model.

## Related tools

- **skysurvey** (Provides the simulation framework and DataSet whose fluxes/uncertainties are compared to the measured ZTF SN Ia DR2 uncertainties; installed via pip or cloned from GitHub.) — https://github.com/MickaelRigault/skysurvey
- **sncosmo** (Backend used by skysurvey for bandpass and light-curve generation feeding the simulated fluxes being validated.) — https://sncosmo.readthedocs.io/en/stable/
- **Python** (Implementation language: pandas for loading the light-curve dataset, numpy/scipy for binning and residual statistics.)
- **healpy** (Survey backend used to match transient positions with the observing history when building the simulated dataset.)

## Examples

```
import skysurvey; survey = skysurvey.ZTF.from_logs(); dset = skysurvey.DataSet.from_targets_and_survey(snia, survey)
```

## Evaluation signals

- Residual means of simulated-minus-measured flux and uncertainty residuals are consistent with zero in each band (g, r, i).
- Residual scatters are comparable to the measured scatter within the corrected noise model in each band.
- Simulated uncertainties reproduce the measured uncertainties only after applying the band-specific sky-noise corrections (1.23, 1.17, 1.20) and error floors (2.5%, 3.5%, 6%).
- simulated_vs_measured_flux_residuals.csv contains per-band residual statistics for all three bands.
- Comparison is made after applying the ZTF SN Ia DR2 selection criteria to the simulated sample.

## Limitations

- The sky-noise correction factors and error floors are band- and survey-specific (ZTF g/r/i, deduced from observed science magnitude limits) and do not transfer directly to other instruments.
- Agreement is conditional on taking observing conditions and instrument performances into account in the simulation; unmodeled conditions would break the comparison.
- The volume-limited regime is z <= 0.06 (~1000 SNe Ia), so conclusions are anchored to low-redshift, cosmological-quality SNe Ia.

## Evidence

- [abstract] the sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band: "the sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band"
- [abstract] error floors of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands: "an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands"
- [abstract] After the ZTF SN Ia DR2 selection criteria were applied, we found that the simulated fluxes and associated uncertainties agre well with the measured uncertainties: "After the ZTF SN Ia DR2 selection criteria were applied, we found that the simulated fluxes and associated uncertainties agre well with the measured uncertainties"
- [methods] skysurvey_ is a python package made to simulate astronomical targets as they would be observed by a survey.: "skysurvey_ is a python package made to simulate astronomical targets as they would be observed by a survey."
- [readme] skysurvey relies on sncosmo for bandpass and lightcurve generations.: "skysurvey relies on sncosmo for bandpass and lightcurve generations. (https://sncosmo.readthedocs.io/en/stable/)"
