---
name: volume-limited-sample-definition
description: Use when constructing a magnitude-limited SN survey sample (e.g. ZTF
  SN Ia DR2) and you need the redshift above which selection effects bias fitted SALT2
  stretch and colour, to impose a volume-limited z cut.
when_to_use_negative:
- Your sample is already volume-limited or flux-limited such that selection effects
  on stretch/colour are absent — no redshift-dependence study is needed.
- You lack simulated light curves with the survey's selection function applied, so
  selection-induced parameter bias cannot be isolated from intrinsic trends.
- The analysis targets photometric classification or light-curve fitting quality itself
  rather than defining a sample redshift boundary.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: skysurvey
  role: Provides the simulated DataSet (targets + survey + selection effects) whose
    fitted SALT2 parameters are binned versus redshift
  repo: https://github.com/MickaelRigault/skysurvey
- name: sncosmo
  role: Fits the SALT2 model to g, r, i light curves to recover stretch (x1) and colour
    (c) parameters
  repo: https://sncosmo.readthedocs.io/en/stable/
- name: Python
  role: Binning stretch/colour versus redshift, applying the z ≤ 0.06 cut, and writing
    sample_counts.json with pandas
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:40425f5f24cf4d70f2356f9ea26ba4f457a20f9481469b14fff7502db3328377
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MickaelRigault/skysurvey
  tools:
  - Python
  - skysurvey
  - sncosmo
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# volume-limited-sample-definition

## Summary

Defines the redshift limit of a volume-limited (selection-unbiased) supernova sample by studying the redshift dependence of SALT2 light-curve parameters (stretch x1 and colour c) in a simulated survey dataset. Applied to ZTF SN Ia DR2, this yields a volume-limited cut of z ≤ 0.06 containing about 1000 SNe Ia.

## When to use

Use when constructing a magnitude-limited SN survey sample (e.g. ZTF SN Ia DR2) and you need the redshift above which selection effects bias fitted SALT2 stretch and colour, to impose a volume-limited z cut.

## When NOT to use

- Your sample is already volume-limited or flux-limited such that selection effects on stretch/colour are absent — no redshift-dependence study is needed.
- You lack simulated light curves with the survey's selection function applied, so selection-induced parameter bias cannot be isolated from intrinsic trends.
- The analysis targets photometric classification or light-curve fitting quality itself rather than defining a sample redshift boundary.

## Inputs

- Simulated ZTF SN Ia DR2 DataSet (dset.data with fluxes and uncertainties in g, r, i) from skysurvey DataSet.from_targets_and_survey
- ZTF SN Ia DR2 catalog of classified SNe Ia of cosmological quality (pandas-readable table)

## Outputs

- Fitted SALT2 parameter table (x1, c per object with redshift)
- Redshift-dependence plots of SALT2 stretch and colour
- Deduced volume-limited redshift limit (z ≤ 0.06)
- sample_counts.json with full-sample (~3000) and volume-limited (~1000) counts

## How to apply

Load the simulated dataset (dset.data with selection-corrected fluxes) produced by a skysurvey DataSet built from targets and survey pointings. Fit the SALT2 model to each simulated light curve in the g, r, and i bands with sncosmo, recovering the stretch (x1) and colour (c) per object. Bin fitted stretch and colour versus redshift across the DR2 redshift range and quantify their redshift dependence. Apply the ZTF SN Ia DR2 selection criteria to the simulated sample and identify the redshift above which selection effects bias the fitted parameters; the redshift below which stretch/colour show no trend defines the volume-limited limit. For ZTF SN Ia DR2 this limit must equal z ≤ 0.06, and applying it to the ~3000 classified SNe Ia of cosmological quality should retain about 1000 SNe Ia.

## Related tools

- **skysurvey** (Provides the simulated DataSet (targets + survey + selection effects) whose fitted SALT2 parameters are binned versus redshift) — https://github.com/MickaelRigault/skysurvey
- **sncosmo** (Fits the SALT2 model to g, r, i light curves to recover stretch (x1) and colour (c) parameters) — https://sncosmo.readthedocs.io/en/stable/
- **Python** (Binning stretch/colour versus redshift, applying the z ≤ 0.06 cut, and writing sample_counts.json with pandas)

## Examples

```
dset = skysurvey.DataSet.from_targets_and_survey(snia, survey); params = sncosmo.fit_lc(lc, 'salt2', ['x1','c']); cut = params[params.z <= 0.06]
```

## Evaluation signals

- The deduced volume-limited redshift limit equals the reported z ≤ 0.06 criterion.
- Within z ≤ 0.06, binned SALT2 stretch (x1) and colour (c) show no significant redshift trend.
- Applying the z ≤ 0.06 cut to the full DR2 sample (~3000 classified SNe Ia) yields about 1000 SNe Ia.
- Above the deduced limit, fitted stretch/colour distributions deviate systematically from low-redshift values, indicating selection bias.
- The z cut count written to sample_counts.json matches the paper's abstract figures (~3000 full, ~1000 volume-limited).

## Limitations

- The limit is specific to the ZTF survey's observing conditions, instrument performance, and DR2 selection criteria; other surveys require their own simulation and redshift-dependence study.
- The simulation relies on skysurvey's noise model: sky-noise from science magnitude limits needed correction factors of 1.23 (g), 1.17 (r), 1.20 (i), and error floors of 2.5%/3.5%/6% were applied in g/r/i — mis-set noise properties shift the inferred limit.
- Result depends on the quality of the simulated SALT2 fits; photometric quality in the reddest band (i-band, 6% error floor) constrains colour recovery at the faint end.

## Evidence

- [abstract] a redshift dependence of the SALT2 light-curve parameters (stretch and colour) was conducted to deduce the redshift limit that defines a volume-limited sample: "a redshift dependence of the SALT2 light-curve parameters (stretch and colour) was conducted to deduce the redshift limit that defines a volume-limited sample"
- [abstract] We found that the ZTF SN Ia DR2 volume-limited sample is characterized by z ≤ 0.06: "We found that the ZTF SN Ia DR2 volume-limited sample is characterized by z ≤ 0.06"
- [abstract] an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands: "an error floor of 2.5%, 3.5%, and 6% of the flux level in the g, r, and i bands"
- [readme] You need to create a `Target` and a `Survey` to then simulate how your survey would observe your targets ; aka a `DataSet`.: "You need to create a `Target` and a `Survey` to then simulate how your survey would observe your targets ; aka a `DataSet`."
