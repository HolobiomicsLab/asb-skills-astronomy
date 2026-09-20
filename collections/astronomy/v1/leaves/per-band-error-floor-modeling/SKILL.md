---
name: per-band-error-floor-modeling
description: Use when simulated light-curve flux uncertainties systematically disagree
  with measured uncertainties and need band-dependent sky-noise corrections plus percentage-level
  error floors.
when_to_use_negative:
- The data are real measured uncertainties rather than simulated light curves — there
  is no sky-noise component to rescale
- The survey or photometric system is not ZTF-like in g/r/i; the 1.23/1.17/1.20 factors
  and 2.5%/3.5%/6% floors are ZTF DR2-specific calibrations and must be re-derived
  for other instruments
- The goal is simulating transients from scratch (Target/Survey/DataSet construction)
  rather than calibrating the noise model of an existing simulated dataset
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3995
tools:
- name: skysurvey
  role: Framework used to simulate the ZTF SN Ia DR2 light curves whose uncertainties
    are then calibrated; provides the DataSet of simulated fluxes and fluxerr per
    band
  repo: https://github.com/MickaelRigault/skysurvey
- name: sncosmo
  role: Backend skysurvey relies on for bandpass and light-curve generation, defining
    the per-band (g, r, i) flux and fluxerr values being corrected
- name: numpy
  role: Used to rescale sky-noise per band and add error floors in quadrature, and
    to compute residual means and scatters
- name: pandas
  role: Loads the simulated light-curve dataset (ztf DataSet with flux, fluxerr, band,
    and matched measured uncertainties)
- name: scipy
  role: Used alongside numpy for residual statistics computation in the binned comparison
    of simulated vs measured uncertainties
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:a1dfe895c9260dc2f3f40b4168ee59740df726e48d1f6460b4ec62514fec05a0
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3995
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# per-band-error-floor-modeling

## Summary

Calibrates simulated photometric flux uncertainties against measured survey data by applying per-band sky-noise rescaling factors and additive error floors, as calibrated for ZTF g/r/i in the ZTF SN Ia DR2 skysurvey simulations.

## When to use

Use when simulated light-curve flux uncertainties systematically disagree with measured uncertainties and need band-dependent sky-noise corrections plus percentage-level error floors.

## When NOT to use

- The data are real measured uncertainties rather than simulated light curves — there is no sky-noise component to rescale
- The survey or photometric system is not ZTF-like in g/r/i; the 1.23/1.17/1.20 factors and 2.5%/3.5%/6% floors are ZTF DR2-specific calibrations and must be re-derived for other instruments
- The goal is simulating transients from scratch (Target/Survey/DataSet construction) rather than calibrating the noise model of an existing simulated dataset

## Inputs

- Simulated ZTF SN Ia DR2 light-curve DataSet (flux, fluxerr, band) from skysurvey
- Matched measured ZTF DR2 flux uncertainties per SN and observation
- Observed science magnitude limits per band (used to deduce sky-noise correction factors)

## Outputs

- Corrected simulated flux uncertainties per band (g, r, i)
- Per-band residual statistics table (mean and scatter of simulated-minus-measured flux and uncertainty residuals) written to simulated_vs_measured_flux_residuals.csv

## How to apply

Load the simulated ZTF DataSet (flux, fluxerr, band) with measured uncertainties using pandas inside the skysurvey environment, and separate the sky-noise component of the simulated fluxerr per band. Rescale the sky-noise component by the empirically deduced per-band factors 1.23 (g), 1.17 (r), and 1.20 (i), derived from the observed science magnitude limits. Then add the per-band error floors in quadrature as percentages of the flux level: 2.5% (g), 3.5% (r), and 6% (i), i.e. fluxerr_corr = sqrt((factor*skynoise)^2 + (floor_frac*flux)^2). Binned comparisons of simulated vs measured fluxes and uncertainties versus observing conditions should then show residual means consistent with zero and residual scatters comparable to the measured scatter within the corrected noise model; write the per-band residual statistics table to simulated_vs_measured_flux_residuals.csv.

## Related tools

- **skysurvey** (Framework used to simulate the ZTF SN Ia DR2 light curves whose uncertainties are then calibrated; provides the DataSet of simulated fluxes and fluxerr per band) — https://github.com/MickaelRigault/skysurvey
- **sncosmo** (Backend skysurvey relies on for bandpass and light-curve generation, defining the per-band (g, r, i) flux and fluxerr values being corrected)
- **numpy** (Used to rescale sky-noise per band and add error floors in quadrature, and to compute residual means and scatters)
- **pandas** (Loads the simulated light-curve dataset (ztf DataSet with flux, fluxerr, band, and matched measured uncertainties))
- **scipy** (Used alongside numpy for residual statistics computation in the binned comparison of simulated vs measured uncertainties)

## Examples

```
fluxerr_corr = np.sqrt((sky_corr[band]*fluxerr_sky)**2 + (floor[band]*flux)**2)  # g: 1.23/0.025, r: 1.17/0.035, i: 1.20/0.06
```

## Evaluation signals

- After applying the ZTF SN Ia DR2 selection criteria, residual means of simulated-minus-measured flux and uncertainty residuals are consistent with zero in each band (g, r, i)
- Residual scatters of corrected simulated uncertainties are comparable to the measured scatter within the corrected noise model
- The sky-noise component of fluxerr was rescaled by exactly 1.23 (g), 1.17 (r), 1.20 (i) — not the total fluxerr
- Error floors added in quadrature equal 2.5%, 3.5%, and 6% of the flux level in g, r, i respectively
- The per-band residual statistics table is written to simulated_vs_measured_flux_residuals.csv with mean and scatter per band

## Limitations

- The correction factors and error floors are empirically deduced from observed ZTF science magnitude limits and are specific to ZTF DR2 bands; they do not transfer to other surveys without re-derivation
- Agreement is only validated after applying the ZTF SN Ia DR2 selection criteria to the simulated sample; uncorrected/uncut simulations may still disagree
- Sky-noise corrections are band-dependent — a single global rescaling of fluxerr does not reproduce the measured uncertainties

## Evidence

- [abstract] the sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band: "the sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band"
- [abstract] an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands: "an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands"
- [intro] skysurvey relies on sncosmo for bandpass and lightcurve generations.: "skysurvey relies on sncosmo for bandpass and lightcurve generations."
- [readme] You need to create a `Target` and a `Survey` to then simulate how your survey would observe your targets ; aka a `DataSet`.: "You need to create a `Target` and a `Survey` to then simulate how your survey would observe your targets ; aka a `DataSet`."
- [abstract] After the ZTF SN Ia DR2 selection criteria were applied, we found that the simulated fluxes and associated uncertainties agre well with the measured uncertainties: "After the ZTF SN Ia DR2 selection criteria were applied, we found that the simulated fluxes and associated uncertainties agre well with the measured uncertainties"
