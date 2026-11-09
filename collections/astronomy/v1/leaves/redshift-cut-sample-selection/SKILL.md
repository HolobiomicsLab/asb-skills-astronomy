---
name: redshift-cut-sample-selection
description: Use when a catalog of classified SNe Ia must be restricted to a volume-limited
  subsample, and the redshift limit should be justified by the redshift dependence
  of SALT2 stretch and colour rather than chosen arbitrarily.
when_to_use_negative:
- The goal is flux/uncertainty simulation validation of the DR2 sample via skysurvey
  rather than defining a subsample by redshift
- The catalog redshifts have not been vetted to cosmological quality, since the z
  ≤ 0.06 limit is calibrated for the classified DR2 sample
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: skysurvey
  role: Framework used in the paper to simulate the ZTF SN Ia DR2 sample and study
    SALT2 parameter redshift dependence from which the z ≤ 0.06 limit is deduced
  repo: https://github.com/MickaelRigault/skysurvey
- name: sncosmo
  role: Backend used by skysurvey for bandpass and lightcurve generation underlying
    the SALT2 parameter study
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202452134
    title: 'ZTF SN Ia DR2: Simulations and volume-limited sample'
  source_content_hash: sha256:98331f02a64ce94b34f6dfad1ea4ceef42686ffcead9dafd83ec2c776c5ba358
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MickaelRigault/skysurvey
  tools:
  - Python
  - skysurvey
  - sncosmo
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# redshift-cut-sample-selection

## Summary

Defines a volume-limited SN Ia sample by applying a redshift cutoff (z ≤ 0.06) derived from studying the redshift dependence of SALT2 light-curve parameters (stretch and colour) in the ZTF SN Ia DR2 catalog of ~3000 cosmological-quality SNe Ia.

## When to use

Use when a catalog of classified SNe Ia must be restricted to a volume-limited subsample, and the redshift limit should be justified by the redshift dependence of SALT2 stretch and colour rather than chosen arbitrarily.

## When NOT to use

- The goal is flux/uncertainty simulation validation of the DR2 sample via skysurvey rather than defining a subsample by redshift
- The catalog redshifts have not been vetted to cosmological quality, since the z ≤ 0.06 limit is calibrated for the classified DR2 sample

## Inputs

- ZTF SN Ia DR2 catalog of classified SNe Ia of cosmological quality (redshifts and SALT2 stretch/colour per object)
- Redshift limit z ≤ 0.06 derived from the SALT2 stretch/colour redshift-dependence study

## Outputs

- Volume-limited sample of ~1000 SNe Ia (objects surviving the z ≤ 0.06 cut)
- sample_counts.json recording both the full-sample count (~3000) and the volume-limited count (~1000)

## How to apply

Load the ZTF SN Ia DR2 catalog of classified SNe Ia of cosmological quality with Python (pandas) and record the full-sample count (almost 3000). The redshift limit defining the volume-limited sample is deduced from a study of the redshift dependence of the SALT2 light-curve parameters (stretch and colour), which yields z ≤ 0.06 for ZTF SN Ia DR2. Apply this cut with pandas (df[df['z'] <= 0.06]) and count surviving objects to obtain the volume-limited sample size (about 1000 SNe Ia). Write both counts to sample_counts.json. Rationale: SALT2 parameter evolution with redshift signals Malmquist-like bias; the z ≤ 0.06 threshold marks the redshift beyond which such dependence appears, so cutting there yields a sample immune to that effect.

## Related tools

- **skysurvey** (Framework used in the paper to simulate the ZTF SN Ia DR2 sample and study SALT2 parameter redshift dependence from which the z ≤ 0.06 limit is deduced) — https://github.com/MickaelRigault/skysurvey
- **sncosmo** (Backend used by skysurvey for bandpass and lightcurve generation underlying the SALT2 parameter study)

## Examples

```
import pandas as pd; df = pd.read_csv('ztf_snia_dr2.csv'); vl = df[df['z'] <= 0.06]; json.dump({'full': len(df), 'volume_limited': len(vl)}, open('sample_counts.json','w'))
```

## Evaluation signals

- Full-sample count reproduces almost 3000 classified SNe Ia of cosmological quality
- Volume-limited count after the z ≤ 0.06 cut reproduces about 1000 SNe Ia, as reported in the paper's abstract
- Every retained object satisfies z ≤ 0.06 (no redshift above the cut in the filtered catalog)
- sample_counts.json contains exactly two counts (full sample and volume-limited sample) matching the abstract values

## Limitations

- The z ≤ 0.06 limit is specific to the ZTF SN Ia DR2 selection function and observing conditions; it should not be transplanted to other surveys without redoing the SALT2 stretch/colour redshift-dependence study
- The count is approximate ('almost 3000', 'about 1000'); exact values depend on the precise cosmological-quality classification criteria

## Evidence

- [abstract] volume-limited redshift cut definition: "We found that the ZTF SN Ia DR2 volume-limited sample is characterized by z ≤ 0.06"
- [abstract] volume-limited sample size: "This volume-limited sample of about 1000 SNe Ia is unique"
- [abstract] derivation of the redshift limit: "a redshift dependence of the SALT2 light-curve parameters (stretch and colour) was conducted to deduce the redshift limit that defines a volume-limited sample"
- [abstract] full DR2 sample size: "The ZTF SN Ia DR2 sample corresponds to almost 3000 classified SNe Ia of cosmological quality."
- [readme] sncosmo dependency: "skysurvey relies on sncosmo for bandpass and lightcurve generations."
