---
name: sky-noise-correction-factor-application
description: Use when validating simulated survey light curves against measured uncertainties
  and residual scatter/means indicate the simulated sky-noise underestimates the observed
  per-band noise, as seen for skysurvey-simulated ZTF SN Ia DR2 data.
when_to_use_negative:
- The dataset is measured ZTF data only with no simulated counterpart — there is no
  sky-noise component to rescale.
- The survey/bands differ from ZTF g/r/i (e.g. DES, LSST opsim runs); the 1.23/1.17/1.20
  factors and 2.5%/3.5%/6% floors were deduced specifically from ZTF DR2 science magnitude
  limits.
- You are still generating raw simulated light curves (Target/Survey/DataSet construction)
  rather than calibrating their uncertainties.
edam_operation: http://edamontology.org/operation_3667
tools:
- name: skysurvey
  role: Provides the ZTF DataSet simulation whose light-curve uncertainties (sky-noise
    component) are corrected per band; loaded/inspected with pandas.
  repo: https://github.com/MickaelRigault/skysurvey
- name: sncosmo
  role: Backend skysurvey relies on for bandpass and light-curve generation feeding
    the simulated fluxes/fluxerrs.
- name: numpy
  role: Computes the quadrature sum of rescaled sky-noise and per-band error floors
    and residual statistics.
- name: scipy
  role: Used with numpy to compute residual statistics when binning simulated vs measured
    fluxes versus observing conditions.
- name: pandas
  role: Loads and manipulates the simulated light-curve dataset with flux, fluxerr,
    and band columns.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:8f59404c664be2e154293493c1848680f348e63ba137dc87a3819756e3e224ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MickaelRigault/skysurvey
  tools:
  - Python
  - numpy
  - pandas
  - scipy
  - skysurvey
  - sncosmo
  edam_operation: http://edamontology.org/operation_3667
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# sky-noise-correction-factor-application

## Summary

Calibrates simulated survey light-curve flux uncertainties by rescaling the sky-noise component per band (ZTF g/r/i factors 1.23/1.17/1.20) and adding per-band error floors (2.5% g, 3.5% r, 6% i of flux) in quadrature, so that skysurvey-simulated ZTF SN Ia DR2 uncertainties match the measured data.

## When to use

Use when validating simulated survey light curves against measured uncertainties and residual scatter/means indicate the simulated sky-noise underestimates the observed per-band noise, as seen for skysurvey-simulated ZTF SN Ia DR2 data.

## When NOT to use

- The dataset is measured ZTF data only with no simulated counterpart — there is no sky-noise component to rescale.
- The survey/bands differ from ZTF g/r/i (e.g. DES, LSST opsim runs); the 1.23/1.17/1.20 factors and 2.5%/3.5%/6% floors were deduced specifically from ZTF DR2 science magnitude limits.
- You are still generating raw simulated light curves (Target/Survey/DataSet construction) rather than calibrating their uncertainties.

## Inputs

- Simulated light-curve DataFrame from a skysurvey DataSet (flux, fluxerr, band columns) built via DataSet.from_targets_and_survey for the ZTF survey (ZTF.from_logs)
- Measured ZTF SN Ia DR2 flux uncertainties matched per observation to the simulated light curves
- Observed science magnitude limits per band (g, r, i) used to deduce the sky-noise correction factors
- Per-band error-floor fractions: 0.025 (g), 0.035 (r), 0.06 (i)

## Outputs

- Corrected simulated flux uncertainties per band (sky-noise rescaled and error floor added in quadrature)
- Per-band residual statistics table written to simulated_vs_measured_flux_residuals.csv (mean and scatter of simulated-minus-measured flux and uncertainty residuals)

## How to apply

Load the simulated light-curve dataset (flux, fluxerr, band, plus matched measured uncertainties) with pandas inside the skysurvey environment. Deduce per-band sky-noise corrections from the observed science magnitude limits — 1.23 for g, 1.17 for r, 1.20 for i — and multiply the simulated sky-noise component of fluxerr by the band-matched factor. Add the per-band error floors (2.5% of flux in g, 3.5% in r, 6% in i) in quadrature with the rescaled sky-noise term. After applying the ZTF SN Ia DR2 selection criteria, bin simulated and measured fluxes versus observing conditions and compute residual statistics (mean and scatter of simulated-minus-measured flux and uncertainty residuals) with numpy/scipy, writing them to simulated_vs_measured_flux_residuals.csv. The rationale is that the raw sky-noise inferred from magnitude limits systematically underestimates the true noise, so both a multiplicative correction and a fractional error floor are required for agreement in every band.

## Related tools

- **skysurvey** (Provides the ZTF DataSet simulation whose light-curve uncertainties (sky-noise component) are corrected per band; loaded/inspected with pandas.) — https://github.com/MickaelRigault/skysurvey
- **sncosmo** (Backend skysurvey relies on for bandpass and light-curve generation feeding the simulated fluxes/fluxerrs.)
- **numpy** (Computes the quadrature sum of rescaled sky-noise and per-band error floors and residual statistics.)
- **scipy** (Used with numpy to compute residual statistics when binning simulated vs measured fluxes versus observing conditions.)
- **pandas** (Loads and manipulates the simulated light-curve dataset with flux, fluxerr, and band columns.)

## Examples

```
corr={'g':1.23,'r':1.17,'i':1.20}; floor={'g':0.025,'r':0.035,'i':0.06}; dset.data['fluxerr_corr'] = np.sqrt((dset.data['fluxerr']*dset.data['band'].map(corr))**2 + (dset.data['band'].map(floor)*dset.data['flux'])**2)
```

## Evaluation signals

- Corrected simulated uncertainties agree well with measured uncertainties in each band (g, r, i): residual means consistent with zero.
- Residual scatters comparable to the measured scatter within the corrected noise model.
- Sky-noise correction factors exactly match the deduced values: 1.23 (g), 1.17 (r), 1.20 (i).
- Error floors applied are 2.5% (g), 3.5% (r), 6% (i) of the flux level, added in quadrature, not linearly.
- simulated_vs_measured_flux_residuals.csv exists and contains per-band residual means and scatters.

## Limitations

- The correction factors were deduced from observed science magnitude limits for ZTF DR2; they are band- and survey-specific and were validated only after applying the ZTF SN Ia DR2 selection criteria to the simulations.
- Agreement was demonstrated only for the g, r, i bands of the ZTF SN Ia DR2 sample; extrapolation to other surveys or filters is unsupported.
- No changelog was found for the skysurvey repository, so version-specific behaviour of the sky-noise model is not traceable.

## Evidence

- [abstract] Per-band sky-noise correction factors: "the sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band"
- [abstract] Per-band error floors: "an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands"
- [abstract] Agreement after selection criteria: "After the ZTF SN Ia DR2 selection criteria were applied, we found that the simulated fluxes and associated uncertainties agre well with the measured uncertainties"
- [intro] DataSet construction context: "You need to create a `Target` and a `Survey` to then simulate how your survey would observe your targets ; aka a `DataSet`."
- [intro] sncosmo dependency: "skysurvey relies on sncosmo for bandpass and lightcurve generations."
- [methods] ZTF survey loading from logs: "survey = skysurvey.ZTF.from_logs()"
