---
name: redshift-dependence-analysis
description: Use when a simulated or observed SN Ia survey sample needs a redshift
  limit defining its volume-limited (unbiased) regime, by checking whether fitted
  SALT2 stretch and colour show redshift trends within the DR2 redshift range.
when_to_use_negative:
- When the goal is validating simulated fluxes and uncertainties against measured
  ZTF DR2 values rather than determining a volume-limited redshift cut (that is the
  separate simulation-performance workflow).
- When input light curves are raw/unobserved target truth (snia.data) rather than
  a DataSet produced after survey simulation — SALT2 fits must use the observed simulated
  photometry.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: sncosmo
  role: Fits the SALT2 model to simulated g/r/i light curves to recover stretch and
    colour parameters per object; also underlies skysurvey's bandpass and lightcurve
    generation
  repo: https://sncosmo.readthedocs.io/en/stable/
- name: skysurvey
  role: Provides the simulated DataSet (targets + survey) whose selection-corrected
    light curves are input to the redshift-dependence analysis, and the ZTF survey
    framework for applying DR2 selection criteria
  repo: https://github.com/MickaelRigault/skysurvey
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:1ddd0b266cafe57bb0b15c6b68e5b3d094160b7c66a65bdbee07d15db2711047
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - skysurvey
  - sncosmo
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# redshift-dependence-analysis

## Summary

Reproduces the ZTF SN Ia DR2 volume-limited-sample determination by fitting SALT2 stretch (x1) and colour (c) to simulated light curves, binning these parameters versus redshift, and identifying the redshift above which selection effects bias the fits. The validated boundary is z ≤ 0.06, defining a sample of about 1000 SNe Ia.

## When to use

Use when a simulated or observed SN Ia survey sample needs a redshift limit defining its volume-limited (unbiased) regime, by checking whether fitted SALT2 stretch and colour show redshift trends within the DR2 redshift range.

## When NOT to use

- When the goal is validating simulated fluxes and uncertainties against measured ZTF DR2 values rather than determining a volume-limited redshift cut (that is the separate simulation-performance workflow).
- When input light curves are raw/unobserved target truth (snia.data) rather than a DataSet produced after survey simulation — SALT2 fits must use the observed simulated photometry.

## Inputs

- skysurvey DataSet (dset.data) of simulated ZTF SN Ia DR2 light curves with selection-corrected fluxes in g, r, i bands
- Redshift values per simulated SN Ia
- ZTF SN Ia DR2 selection criteria

## Outputs

- Fitted SALT2 parameter table (x1 stretch and c colour per object)
- Redshift-dependence plots of stretch and colour (binned versus redshift)
- Deduced volume-limited redshift limit (expected: z ≤ 0.06)

## How to apply

Load the simulated ZTF SN Ia DR2 DataSet (dset.data with selection-corrected fluxes) built by skysurvey's DataSet.from_targets_and_survey. Fit a SALT2 model to each simulated light curve in the g, r, and i bands with sncosmo to recover per-object stretch (x1) and colour (c). Bin the fitted parameters versus redshift across the DR2 redshift range and quantify their redshift dependence. Reapply the ZTF SN Ia DR2 selection criteria to the simulated sample and locate the redshift above which selection effects begin to bias x1 and c; the redshift below that threshold is the volume-limited boundary. In this study the procedure recovers z ≤ 0.06 (~1000 SNe Ia), and within that region the fitted stretch/colour distributions must show no redshift trend.

## Related tools

- **sncosmo** (Fits the SALT2 model to simulated g/r/i light curves to recover stretch and colour parameters per object; also underlies skysurvey's bandpass and lightcurve generation) — https://sncosmo.readthedocs.io/en/stable/
- **skysurvey** (Provides the simulated DataSet (targets + survey) whose selection-corrected light curves are input to the redshift-dependence analysis, and the ZTF survey framework for applying DR2 selection criteria) — https://github.com/MickaelRigault/skysurvey

## Examples

```
dset = skysurvey.DataSet.from_targets_and_survey(snia, survey); fits = sncosmo.fit_lc(dset.data, model='salt2', vparam_names=['x1','c'])
```

## Evaluation signals

- The deduced volume-limited redshift limit must equal the reported z ≤ 0.06 criterion.
- The resulting volume-limited sample must contain about 1000 SNe Ia, consistent with the DR2 finding.
- Fitted stretch (x1) and colour (c) distributions must show no redshift trend within the z ≤ 0.06 volume-limited region.
- Fits must be performed on the simulated DataSet after the ZTF SN Ia DR2 selection criteria were applied, so selection-biased objects at high z are retained and visible in the trend.

## Limitations

- The boundary z ≤ 0.06 is specific to ZTF SN Ia DR2 selection criteria and observing conditions; other surveys or selection functions require re-deriving the limit.
- Simulated sky-noise required band-dependent correction factors (1.23 in g, 1.17 in r, 1.20 in i) and error floors of 2.5%, 3.5%, and 6% of the flux level in g, r, i respectively; neglecting these biases the fits feeding the analysis.
- The analysis relies on skysurvey's simulation fidelity; simulated fluxes and uncertainties only 'agre well' with measured uncertainties after the sky-noise corrections were applied.

## Evidence

- [abstract] a redshift dependence of the SALT2 light-curve parameters (stretch and colour) was conducted to deduce the redshift limit that defines a volume-limited sample: "a redshift dependence of the SALT2 light-curve parameters (stretch and colour) was conducted to deduce the redshift limit that defines a volume-limited sample"
- [abstract] We found that the ZTF SN Ia DR2 volume-limited sample is characterized by z ≤ 0.06: "We found that the ZTF SN Ia DR2 volume-limited sample is characterized by z ≤ 0.06"
- [abstract] This volume-limited sample of about 1000 SNe Ia is unique: "This volume-limited sample of about 1000 SNe Ia is unique"
- [intro] skysurvey relies on sncosmo for bandpass and lightcurve generations.: "skysurvey relies on sncosmo for bandpass and lightcurve generations."
- [abstract] After the ZTF SN Ia DR2 selection criteria were applied: "After the ZTF SN Ia DR2 selection criteria were applied"
