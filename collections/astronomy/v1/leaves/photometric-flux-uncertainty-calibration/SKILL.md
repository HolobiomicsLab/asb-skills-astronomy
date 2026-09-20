---
name: photometric-flux-uncertainty-calibration
description: Use when simulated light-curve flux uncertainties disagree with measured
  uncertainties of a survey dataset (e.g. ZTF DR2 g/r/i) and you need per-band sky-noise
  correction factors plus percentage error floors.
when_to_use_negative:
- The simulated uncertainties already agree with measured data — recalibrating would
  double-correct the noise model
- The survey lacks per-band science magnitude limits from which to deduce sky-noise
  correction factors (e.g. a different instrument whose noise structure is not sky-noise-dominated)
- The task is simulating targets/surveys themselves (Target.draw, Survey.from_pointings)
  rather than calibrating the resulting uncertainties
edam_operation: http://edamontology.org/operation_3435
tools:
- name: skysurvey
  role: Simulation framework producing the ZTF DataSet whose flux uncertainties are
    calibrated; provides the ZTF survey via ZTF.from_logs()
  repo: https://github.com/MickaelRigault/skysurvey
- name: sncosmo
  role: Backend used by skysurvey for bandpass and light-curve generations feeding
    the simulated fluxes
- name: Python
  role: pandas/numpy/scipy used to load the dataset, rescale sky-noise, add error
    floors in quadrature, and compute residual statistics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:5b891e2e4baaee27f9af1cb3ce82b7d6cfc7a64e3f6a67d3288c166a43f71d80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MickaelRigault/skysurvey
  tools:
  - Python
  - skysurvey
  - sncosmo
  edam_operation: http://edamontology.org/operation_3435
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# photometric-flux-uncertainty-calibration

## Summary

Calibrates simulated photometric flux uncertainties against measured survey data by rescaling the sky-noise component per band and adding band-dependent error floors in quadrature, as done for the ZTF SN Ia DR2 simulation validation.

## When to use

Use when simulated light-curve flux uncertainties disagree with measured uncertainties of a survey dataset (e.g. ZTF DR2 g/r/i) and you need per-band sky-noise correction factors plus percentage error floors.

## When NOT to use

- The simulated uncertainties already agree with measured data — recalibrating would double-correct the noise model
- The survey lacks per-band science magnitude limits from which to deduce sky-noise correction factors (e.g. a different instrument whose noise structure is not sky-noise-dominated)
- The task is simulating targets/surveys themselves (Target.draw, Survey.from_pointings) rather than calibrating the resulting uncertainties

## Inputs

- Simulated light-curve DataSet from skysurvey (flux, fluxerr, band columns; e.g. dset.data from DataSet.from_targets_and_survey)
- Measured ZTF SN Ia DR2 light-curve uncertainties matched per observation
- Observed science magnitude limits per band (g, r, i) used to deduce sky-noise correction factors

## Outputs

- Corrected simulated flux uncertainties (sky-noise rescaled and floor-augmented) per band
- Per-band residual statistics table (mean and scatter of simulated-minus-measured flux and uncertainty residuals), written to simulated_vs_measured_flux_residuals.csv

## How to apply

Load the simulated DataSet (flux, fluxerr, band) alongside matched measured uncertainties using pandas within the skysurvey environment. Rescale the simulated sky-noise component of the flux uncertainties per band by the correction factors deduced from the observed science magnitude limits: 1.23 (g), 1.17 (r), and 1.20 (i). Add the per-band error floors in quadrature as a percentage of the flux level: 2.5% (g), 3.5% (r), and 6% (i). Then apply the ZTF SN Ia DR2 selection criteria to the simulated sample, bin simulated and measured fluxes versus observing conditions, and compute residual statistics (mean and scatter of simulated-minus-measured residuals) with numpy/scipy to quantify agreement per band.

## Related tools

- **skysurvey** (Simulation framework producing the ZTF DataSet whose flux uncertainties are calibrated; provides the ZTF survey via ZTF.from_logs()) — https://github.com/MickaelRigault/skysurvey
- **sncosmo** (Backend used by skysurvey for bandpass and light-curve generations feeding the simulated fluxes)
- **Python** (pandas/numpy/scipy used to load the dataset, rescale sky-noise, add error floors in quadrature, and compute residual statistics)

## Examples

```
import skysurvey; survey = skysurvey.ZTF.from_logs(); dset = skysurvey.DataSet.from_targets_and_survey(snia, survey); fluxerr_corrected = np.sqrt((fluxerr_skynoise*band_factor)**2 + (band_floor*flux)**2)
```

## Evaluation signals

- Corrected simulated uncertainties agree with measured uncertainties per band (g, r, i): residual means consistent with zero
- Residual scatters comparable to the measured scatter within the corrected noise model
- Sky-noise correction factors applied exactly match 1.23 (g), 1.17 (r), 1.20 (i)
- Error floors added in quadrature equal 2.5% (g), 3.5% (r), 6% (i) of the flux level
- simulated_vs_measured_flux_residuals.csv contains per-band residual statistics for all three bands

## Limitations

- Correction factors and error floors are specific to ZTF DR2 observing conditions and instrument performances; they must be re-deduced from science magnitude limits for other surveys
- Agreement is only validated after applying the ZTF SN Ia DR2 selection criteria to the simulated sample — raw, uncut simulations may not show the same agreement
- The i-band error floor is large (6% of flux level), so uncertainty agreement in i is intrinsically looser than in g

## Evidence

- [abstract] the sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band: "the sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band"
- [abstract] an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands: "an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands"
- [abstract] After the ZTF SN Ia DR2 selection criteria were applied, we found that the simulated fluxes and associated uncertainties agre well with the measured uncertainties: "After the ZTF SN Ia DR2 selection criteria were applied, we found that the simulated fluxes and associated uncertainties agre well with the measured uncertainties"
- [readme] You need to create a `Target` and a `Survey` to then simulate how your survey would observe your targets ; aka a `DataSet`.: "You need to create a `Target` and a `Survey` to then simulate how your survey would observe your targets ; aka a `DataSet`."
- [abstract] We simulated individual light curves of the ZTF SN Ia DR2 sample to confirm the validity of the framework while taking the observing conditions and instrument performances into account.: "We simulated individual light curves of the ZTF SN Ia DR2 sample to confirm the validity of the framework while taking the observing conditions and instrument performances into account."
