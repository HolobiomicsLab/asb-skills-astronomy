---
name: weighted-scatter-computation
description: Use when you have S-BART RV time-series .txt files (columns BJD, RVc,
  RVc_ERR, SA, DRIFT) for multiple targets and need to quantify instrument stability
  via the nightly zero point's weighted scatter.
when_to_use_negative:
- The goal is to analyze planetary or activity signals in the RV time series — the
  source article explicitly leaves 'a full analysis of the planetary and activity
  signals present in the considered datasets for future work'
- The RVs come from the ESO CCF pipeline or other template-matching tools (HARPS-TERRA,
  SERVAL) rather than the SBART output format, so the BJD/RVc/RVc_ERR/SA/DRIFT column
  semantics do not apply
- The input is a single-epoch spectrum or per-order RVs rather than a time series
  of drift-corrected nightly RVs grouped by observation night
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: S-BART
  role: Produces the template-matching RV time series (with BJD, RVc, RVc_ERR, SA,
    DRIFT columns) whose nightly zero point scatter is computed; installable via pip
    install SBART
  repo: https://github.com/iastro-pt/sBART
- name: ESPRESSO
  role: Instrument whose NZP is estimated from the 33 GTO targets' S-BART RVs; the
    weighted NZP scatter upper-bounds ESPRESSO's attainable RV precision
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:ae172cf1dd2bc281ec448fd8024e02733d9f8b5699139c7a8cb2114dee8b7b59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - ESPRESSO
  - S-BART
  - SBART
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# weighted-scatter-computation

## Summary

Computation of the inverse-variance-weighted scatter of ESPRESSO's nightly zero point (NZP) from S-BART radial-velocity time series, used to place an upper limit (~0.7 m/s) on the instrument's attainable RV precision.

## When to use

Use when you have S-BART RV time-series .txt files (columns BJD, RVc, RVc_ERR, SA, DRIFT) for multiple targets and need to quantify instrument stability via the nightly zero point's weighted scatter.

## When NOT to use

- The goal is to analyze planetary or activity signals in the RV time series — the source article explicitly leaves 'a full analysis of the planetary and activity signals present in the considered datasets for future work'
- The RVs come from the ESO CCF pipeline or other template-matching tools (HARPS-TERRA, SERVAL) rather than the SBART output format, so the BJD/RVc/RVc_ERR/SA/DRIFT column semantics do not apply
- The input is a single-epoch spectrum or per-order RVs rather than a time series of drift-corrected nightly RVs grouped by observation night

## Inputs

- S-BART RV time-series .txt files for the 33 ESPRESSO GTO targets (default SBART output format with BJD, RVc, RVc_ERR, SA, DRIFT, DRIFT_ERR columns)
- Instrumental drift corrections (DRIFT, DRIFT_ERR) from SBART output
- Secular acceleration values (SA column, DACE time-origin convention)

## Outputs

- Per-night NZP table (one weighted mean RV offset per observation night per target)
- Weighted NZP scatter value in m/s
- espresso_nzp.csv containing the per-night NZP table and the scatter value

## How to apply

Load the S-BART output RV files with pandas, parsing the BJD, RVc, RVc_ERR, SA, and DRIFT columns of the default SBART output format. Convert RVc from km/s to m/s (the DRIFT column is defined as 'RV measurement, in km/s, after being corrected by the instrumental drift'), apply the instrumental drift correction adding DRIFT_ERR in quadrature, and subtract the SA (secular acceleration) column, which follows the DACE time-origin convention. Group observations by night using BJD, compute a per-night uncertainty-weighted mean RV offset per target, derive the NZP per night, and compute the inverse-variance-weighted scatter of the NZP across nights. Validate that the resulting weighted NZP scatter falls below ~0.7 m/s — the published upper-limit value that includes stellar variability, photon noise and planetary signals — and write the per-night NZP table plus the scatter to espresso_nzp.csv.

## Related tools

- **S-BART** (Produces the template-matching RV time series (with BJD, RVc, RVc_ERR, SA, DRIFT columns) whose nightly zero point scatter is computed; installable via pip install SBART) — https://github.com/iastro-pt/sBART
- **ESPRESSO** (Instrument whose NZP is estimated from the 33 GTO targets' S-BART RVs; the weighted NZP scatter upper-bounds ESPRESSO's attainable RV precision)

## Examples

```
import pandas as pd; df = pd.read_csv('sbart_rv.txt', delim_whitespace=True); df['rv'] = df['RVc']*1000 - df['DRIFT'] - df['SA']; df.groupby('BJD')
```

## Evaluation signals

- The inverse-variance-weighted NZP scatter across all nights falls below ~0.7 m/s, the published upper-limit value
- The per-night NZP table and the scatter value are written to espresso_nzp.csv
- RVc values were converted from km/s to m/s before weighting (1000x scale factor verified)
- Drift correction applied with DRIFT_ERR propagated in quadrature, and the SA column (DACE time-origin convention) subtracted from drift-corrected RVs
- Observations are grouped by night via the BJD column with one weighted mean RV offset per night per target

## Limitations

- The ~0.7 m/s weighted NZP scatter should be taken as an upper limit on the RV precision attainable with ESPRESSO data because it includes stellar variability, photon noise and planetary signals
- A full analysis of the planetary and activity signals present in the datasets was left for future work, so the NZP scatter cannot disentangle these contributors
- S-BART was validated on HARPS data (e.g. Barnard's star archival observations) before being applied to ESPRESSO; cross-instrument reuse of the NZP workflow should account for instrument-specific drift behavior

## Evidence

- [abstract] with the same data we estimated the nightly zero point (NZP) of the instrument: "with the same data we estimated the nightly zero point (NZP) of the instrument"
- [abstract] NZP scatter upper limit: "finding a weighted NZP scatter of below ~0.7 m s−1"
- [methods] DRIFT column definition: "RV measurement, in km/s, after being corrected by the instrumental drift (see DRIFT column description)"
- [methods] SA column convention: "Value of the secular acceleration, assuming that same "time origin" as DACE."
