---
name: snia-stretch-colour-parameterisation
description: Use when you have simulated or observed SN Ia multi-band light curves
  (e.g. skysurvey dset.data with selection-corrected fluxes) and need SALT2 stretch/colour
  fits plus a redshift-bias assessment to define a volume-limited sample.
when_to_use_negative:
- The light curves have not been simulated/observed yet — build the Target/Survey/DataSet
  chain (snia.draw, Survey.from_pointings, DataSet.from_targets_and_survey) first
- The goal is simulation of survey observations rather than cosmology-sample definition
  (use the skysurvey simulation skill instead)
- Only a single band or non-SALT2 parameterisation is available, so stretch and colour
  cannot both be recovered per object
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: sncosmo
  role: Fits the SALT2 model to each simulated g/r/i light curve to recover stretch
    (x1) and colour (c); skysurvey relies on it for bandpass and light-curve generation
  repo: https://github.com/MickaelRigault/skysurvey
- name: skysurvey
  role: Produces the simulated DataSet (dset.data) whose light curves are fit, and
    provides the ZTF survey/selection context
  repo: https://github.com/MickaelRigault/skysurvey
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:1faf2da2dcc3ea9b6153fa24bf9bf22ff196e80b30314a22c640182506071439
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MickaelRigault/skysurvey
  tools:
  - Python
  - skysurvey
  - sncosmo
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# snia-stretch-colour-parameterisation

## Summary

Fit SALT2 light-curve parameters (stretch x1 and colour c) to simulated ZTF SN Ia DR2 light curves in the g, r, and i bands with sncosmo, then study their redshift dependence to deduce the redshift limit (z ≤ 0.06) defining the volume-limited sample.

## When to use

Use when you have simulated or observed SN Ia multi-band light curves (e.g. skysurvey dset.data with selection-corrected fluxes) and need SALT2 stretch/colour fits plus a redshift-bias assessment to define a volume-limited sample.

## When NOT to use

- The light curves have not been simulated/observed yet — build the Target/Survey/DataSet chain (snia.draw, Survey.from_pointings, DataSet.from_targets_and_survey) first
- The goal is simulation of survey observations rather than cosmology-sample definition (use the skysurvey simulation skill instead)
- Only a single band or non-SALT2 parameterisation is available, so stretch and colour cannot both be recovered per object

## Inputs

- Simulated ZTF SN Ia DR2 DataSet (dset.data) with selection-corrected fluxes in g, r, i bands from skysurvey
- ZTF SN Ia DR2 selection criteria
- SALT2 model (via sncosmo TimeSeries Source)

## Outputs

- Per-object SALT2 parameter table (x1, c, redshift)
- Redshift-dependence plots of stretch and colour
- Deduced volume-limited redshift limit (expected z ≤ 0.06, ~1000 SNe Ia)

## How to apply

Load the simulated ZTF SN Ia DR2 DataSet (dset.data produced by DataSet.from_targets_and_survey) and fit a SALT2 model to each object's light curve in the g, r, and i bands using sncosmo, recovering the stretch (x1) and colour (c) per object. Bin the fitted x1 and c versus redshift and quantify any redshift trend across the DR2 redshift range. Apply the ZTF SN Ia DR2 selection criteria to the simulated sample first, since selection effects bias the fitted parameters at high redshift. Identify the redshift above which the stretch/colour distributions deviate from their low-z behaviour; the article's simulations show this boundary is z ≤ 0.06, yielding a volume-limited sample of about 1000 SNe Ia. Save the fitted SALT2 parameter table, redshift-dependence plots, and the deduced redshift limit. Validation: within the volume-limited region the fitted stretch and colour distributions must show no redshift trend.

## Related tools

- **sncosmo** (Fits the SALT2 model to each simulated g/r/i light curve to recover stretch (x1) and colour (c); skysurvey relies on it for bandpass and light-curve generation) — https://github.com/MickaelRigault/skysurvey
- **skysurvey** (Produces the simulated DataSet (dset.data) whose light curves are fit, and provides the ZTF survey/selection context) — https://github.com/MickaelRigault/skysurvey

## Examples

```
import sncosmo; model = sncosmo.Model(source='salt2'); result, fitted_model = sncosmo.fit_lc(lc_data, model, ['x1', 'c'], bounds={'x1': (-3, 3), 'c': (-0.3, 0.3)})
```

## Evaluation signals

- Deduced volume-limited redshift limit equals the reported z ≤ 0.06 criterion
- Fitted stretch (x1) and colour (c) distributions show no redshift trend within z ≤ 0.06
- Fitted SALT2 parameters exist for every simulated object passing the ZTF DR2 selection criteria
- Volume-limited sample size is consistent with the reported ~1000 SNe Ia
- Fitted parameter table and redshift-dependence plots are saved to disk

## Limitations

- The z ≤ 0.06 boundary is specific to the ZTF SN Ia DR2 selection criteria and g/r/i observing conditions; other surveys or cadences will shift the limit
- Redshift-dependence inference depends on the realism of the simulated fluxes and uncertainties (sky-noise correction factors 1.23 g, 1.17 r, 1.20 i; error floors 2.5%/3.5%/6% in g/r/i)
- Selection effects above the limit bias the fitted parameters, so the high-z side of the trend cannot be interpreted as astrophysical

## Evidence

- [abstract] a redshift dependence of the SALT2 light-curve parameters (stretch and colour) was conducted to deduce the redshift limit that defines a volume-limited sample: "a redshift dependence of the SALT2 light-curve parameters (stretch and colour) was conducted to deduce the redshift limit that defines a volume-limited sample"
- [abstract] We found that the ZTF SN Ia DR2 volume-limited sample is characterized by z ≤ 0.06: "We found that the ZTF SN Ia DR2 volume-limited sample is characterized by z ≤ 0.06"
- [intro] skysurvey relies on sncosmo for bandpass and lightcurve generations.: "skysurvey relies on sncosmo for bandpass and lightcurve generations."
- [readme] You need to create a `Target` and a `Survey` to then simulate how your survey would observe your targets ; aka a `DataSet`.: "You need to create a `Target` and a `Survey` to then simulate how your survey would observe your targets ; aka a `DataSet`."
