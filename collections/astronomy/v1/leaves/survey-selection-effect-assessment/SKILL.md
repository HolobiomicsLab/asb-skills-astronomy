---
name: survey-selection-effect-assessment
description: Use when a flux-limited transient sample must be truncated to an unbiased
  volume-limited subsample, and you need to identify the redshift above which selection
  effects bias fitted light-curve parameters.
when_to_use_negative:
- The analysis goal is cosmological parameter fitting on the full flux-limited sample,
  where selection effects are handled via efficiency corrections rather than a hard
  redshift cut
- The input is an already volume-limited sample (e.g. z ≤ 0.06 already applied), so
  no further selection-effect assessment is needed
- The simulation framework cannot reproduce the survey's observing conditions or noise
  properties, making bias-versus-redshift comparisons unreliable
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: skysurvey
  role: Simulates individual light curves of the sample taking observing conditions
    and instrument performances into account; builds Target, Survey, and DataSet;
    applies survey selection criteria to the simulated sample
  repo: https://github.com/MickaelRigault/skysurvey
- name: sncosmo
  role: Fits the SALT2 model to each simulated light curve in the g, r, and i bands
    to recover stretch (x1) and colour (c) parameters used for the redshift-dependence
    study; skysurvey relies on sncosmo for bandpass and lightcurve generations
- name: modeldag
  role: Backend for building complex parametrisations of realistic targets in skysurvey
  repo: https://github.com/MickaelRigault/modeldag
- name: SNANA
  role: Alternative supernova analysis/simulation package referenced in the paper's
    methods context
  repo: https://github.com/RickKessler/SNANA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:17e2e69252e2d68d184802d275e9edfa071406b722f4235f6ffa9d6b1469594d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MickaelRigault/skysurvey
  tools:
  - Python
  - SNANA
  - modeldag
  - skysurvey
  - sncosmo
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# survey-selection-effect-assessment

## Summary

Assess survey selection effects by simulating a transient survey end-to-end (targets + observing conditions), applying the survey's selection criteria, and studying the redshift dependence of fitted light-curve parameters (SALT2 stretch x1 and colour c) to deduce the redshift limit defining a volume-limited sample.

## When to use

Use when a flux-limited transient sample must be truncated to an unbiased volume-limited subsample, and you need to identify the redshift above which selection effects bias fitted light-curve parameters.

## When NOT to use

- The analysis goal is cosmological parameter fitting on the full flux-limited sample, where selection effects are handled via efficiency corrections rather than a hard redshift cut
- The input is an already volume-limited sample (e.g. z ≤ 0.06 already applied), so no further selection-effect assessment is needed
- The simulation framework cannot reproduce the survey's observing conditions or noise properties, making bias-versus-redshift comparisons unreliable

## Inputs

- Simulated DataSet of survey-observed transient light curves (dset.data with selection-corrected fluxes from skysurvey DataSet.from_targets_and_survey)
- Survey observing logs / pointings (e.g. skysurvey.ZTF.from_logs()) with footprint, gain, zp, skynoise, mjd, band
- Target truth data (e.g. snia.draw(size=50_000) SNe Ia properties)
- Survey selection criteria (e.g. ZTF SN Ia DR2 cuts)

## Outputs

- Fitted SALT2 parameter table (x1, c per object with uncertainties)
- Binned stretch and colour versus redshift plots quantifying redshift dependence
- Deduced volume-limiting redshift (z ≤ 0.06 for ZTF SN Ia DR2, ~1000 SNe Ia)

## How to apply

Simulate individual light curves of the survey sample (e.g. ZTF SN Ia DR2 in g, r, i) with skysurvey, taking observing conditions and instrument performances into account — the paper corrects sky-noise deduced from science magnitude limits by factors 1.23 (g), 1.17 (r), and 1.20 (i), and adds error floors of 2.5%, 3.5%, and 6% of the flux level in g, r, and i. Build the simulated DataSet from targets and survey pointings (DataSet.from_targets_and_survey), then apply the survey's published selection criteria (here the ZTF SN Ia DR2 cuts) to the simulated sample. Fit SALT2 to each simulated light curve with sncosmo to recover stretch (x1) and colour (c). Bin the fitted x1 and c versus redshift and quantify their redshift dependence; the redshift above which these parameters become biased marks where selection effects set in, and the sample below it is volume-limited. For ZTF SN Ia DR2 this procedure yields a volume-limited sample defined by z ≤ 0.06 containing about 1000 SNe Ia.

## Related tools

- **skysurvey** (Simulates individual light curves of the sample taking observing conditions and instrument performances into account; builds Target, Survey, and DataSet; applies survey selection criteria to the simulated sample) — https://github.com/MickaelRigault/skysurvey
- **sncosmo** (Fits the SALT2 model to each simulated light curve in the g, r, and i bands to recover stretch (x1) and colour (c) parameters used for the redshift-dependence study; skysurvey relies on sncosmo for bandpass and lightcurve generations)
- **modeldag** (Backend for building complex parametrisations of realistic targets in skysurvey) — https://github.com/MickaelRigault/modeldag
- **SNANA** (Alternative supernova analysis/simulation package referenced in the paper's methods context) — https://github.com/RickKessler/SNANA

## Examples

```
survey = skysurvey.ZTF.from_logs(); dset = skysurvey.DataSet.from_targets_and_survey(snia, survey)
```

## Evaluation signals

- The deduced volume-limited redshift limit equals the reported z ≤ 0.06 criterion for ZTF SN Ia DR2
- The volume-limited sample contains about 1000 SNe Ia after applying the limit
- Fitted SALT2 stretch (x1) and colour (c) distributions show no redshift trend within the volume-limited region
- Simulated fluxes and associated uncertainties agree with measured uncertainties once the sky-noise correction factors (1.23 g, 1.17 r, 1.20 i) and error floors (2.5%, 3.5%, 6%) are applied
- Applying the survey's selection criteria to the simulated sample recovers flux/uncertainty agreement with the real DR2 measurements

## Limitations

- The sky-noise deduced from observed science magnitude limits requires empirical band-dependent correction factors (1.23, 1.17, 1.20); without these the simulated uncertainties disagree with measured ones
- Error floors of 2.5% (g), 3.5% (r), and 6% (i) of the flux level must be explicitly accounted for; omitting them biases simulated flux uncertainties
- Selection-effect biases only manifest above the deduced redshift limit; below it the sample is assumed unbiased but the redshift-dependence test is the sole diagnostic
- The skysurvey repository has no changelog available, so version-specific behaviour changes cannot be traced

## Evidence

- [abstract] a redshift dependence of the SALT2 light-curve parameters (stretch and colour) was conducted to deduce the redshift limit that defines a volume-limited sample: "a redshift dependence of the SALT2 light-curve parameters (stretch and colour) was conducted to deduce the redshift limit that defines a volume-limited sample"
- [abstract] We found that the ZTF SN Ia DR2 volume-limited sample is characterized by z ≤ 0.06: "We found that the ZTF SN Ia DR2 volume-limited sample is characterized by z ≤ 0.06"
- [abstract] the sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band: "the sky-noise deduced from the observed science magnitude limits is corrected for by a factor 1.23 for the g band, 1.17 for the r band, and 1.20 for the i band"
- [readme] You need to create a `Target` and a `Survey` to then simulate how your survey would observe your targets ; aka a `DataSet`.: "You need to create a `Target` and a `Survey` to then
simulate how your survey would observe your targets ; 
aka a `DataSet`."
- [abstract] After the ZTF SN Ia DR2 selection criteria were applied, we found that the simulated fluxes and associated uncertainties agre well with the measured uncertainties: "After the ZTF SN Ia DR2 selection criteria were applied, we found that the simulated fluxes and associated uncertainties agre well with the measured uncertainties"
