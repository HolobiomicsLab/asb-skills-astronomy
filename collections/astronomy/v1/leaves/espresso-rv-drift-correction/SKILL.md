---
name: espresso-rv-drift-correction
description: Use when processing S-BART output RV .txt files (columns BJD, RVc, RVc_ERR,
  SA, DRIFT) from ESPRESSO observations and you need drift-corrected RVs in m/s or
  an instrument NZP estimate.
when_to_use_negative:
- Input RVs come from the ESO official CCF pipeline rather than S-BART output files
  with RVc/DRIFT/SA columns — the column semantics and correction convention differ
- The goal is stellar-template construction or telluric handling within S-BART; this
  skill only covers post-processing of already-extracted RVs
- The dataset has no multi-night coverage (e.g. a single ESPRESSO night), so a per-night
  NZP scatter cannot be derived from grouped BJD nights
edam_operation: http://edamontology.org/operation_2945
tools:
- name: S-BART
  role: Produces the RV time-series files (RVc, RVc_ERR, SA, DRIFT columns) whose
    drift correction this skill performs; installable via 'pip install SBART'
  repo: https://github.com/iastro-pt/sBART
- name: ESPRESSO
  role: VLT spectrograph whose GTO observations (33 targets) supply the RV data being
    drift-corrected; also the instrument whose NZP is estimated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:d36e0f472f9475481d619e240f7a5fdefda731dff98edd30b26ab5a7c2971785
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - ESPRESSO
  - S-BART
  - SBART
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# espresso-rv-drift-correction

## Summary

Corrects S-BART radial velocities of ESPRESSO GTO targets for instrumental drift (DRIFT column) and secular acceleration (SA column), producing drift-corrected RV time series in m/s from which the nightly zero point (NZP) can be estimated with a weighted scatter below ~0.7 m/s.

## When to use

Use when processing S-BART output RV .txt files (columns BJD, RVc, RVc_ERR, SA, DRIFT) from ESPRESSO observations and you need drift-corrected RVs in m/s or an instrument NZP estimate.

## When NOT to use

- Input RVs come from the ESO official CCF pipeline rather than S-BART output files with RVc/DRIFT/SA columns — the column semantics and correction convention differ
- The goal is stellar-template construction or telluric handling within S-BART; this skill only covers post-processing of already-extracted RVs
- The dataset has no multi-night coverage (e.g. a single ESPRESSO night), so a per-night NZP scatter cannot be derived from grouped BJD nights

## Inputs

- S-BART radial-velocity time-series .txt files (default SBART output format) for the 33 ESPRESSO GTO targets, with columns BJD, RVc, RVc_ERR, SA, DRIFT (and DRIFT_ERR where present)

## Outputs

- Drift- and secular-acceleration-corrected RV time series in m/s per target
- Per-night NZP table plus inverse-variance-weighted NZP scatter written to espresso_nzp.csv

## How to apply

Load each S-BART RV time-series .txt file with pandas, parsing the BJD, RVc, RVc_ERR, SA, and DRIFT columns of the default SBART output format; RVc is the RV measurement in km/s after instrumental-drift correction as defined by the DRIFT column, so first convert RVc from km/s to m/s and apply the drift correction encoded in DRIFT, adding DRIFT_ERR in quadrature where present. Remove the secular acceleration term by subtracting the SA column, which assumes the same time origin as DACE. Then group observations by night using BJD, compute a per-night uncertainty-weighted mean RV offset per target, derive the nightly zero point per night, and compute the inverse-variance-weighted scatter of the NZP across all nights. Write the per-night NZP table plus the scatter value to espresso_nzp.csv and check that the weighted scatter falls below ~0.7 m/s.

## Related tools

- **S-BART** (Produces the RV time-series files (RVc, RVc_ERR, SA, DRIFT columns) whose drift correction this skill performs; installable via 'pip install SBART') — https://github.com/iastro-pt/sBART
- **ESPRESSO** (VLT spectrograph whose GTO observations (33 targets) supply the RV data being drift-corrected; also the instrument whose NZP is estimated)

## Examples

```
import pandas as pd; df = pd.read_csv("SBART_rvs.txt", sep=r"\s+"); df["rv_ms"] = (df["RVc"] + df["DRIFT"] - df["SA"]) * 1e3; df.to_csv("drift_corrected.csv")
```

## Evaluation signals

- Weighted NZP scatter across all nights falls below ~0.7 m/s, the published upper-limit value
- RVc values are converted from km/s to m/s (factor of 1000) before the DRIFT and SA corrections are applied
- DRIFT_ERR is combined in quadrature with the RV uncertainty wherever a DRIFT_ERR column is present
- The SA column (DACE time-origin convention) has been subtracted from the drift-corrected RVs, leaving no residual secular-acceleration trend
- Observations are grouped strictly by night using the BJD column, and espresso_nzp.csv contains one NZP entry per night plus the final scatter value

## Limitations

- The ~0.7 m/s weighted NZP scatter includes stellar variability, photon noise and planetary signals, so it should be taken as an upper limit on the RV precision attainable with ESPRESSO data, not the pure instrumental floor
- The NZP estimate inherits assumptions of the S-BART pipeline (semi-Bayesian template matching); performance relative to the CCF method differs by spectral type (~10% improvement for M-type, ~4% for K-type stars)
- Full analysis of the planetary and activity signals present in the datasets was left for future work, so the NZP scatter cannot disentangle instrumental from astrophysical contributions

## Evidence

- [abstract] NZP estimated from same S-BART data: "with the same data we estimated the nightly zero point (NZP) of the instrument"
- [methods] DRIFT column semantics: "RV measurement, in km/s, after being corrected by the instrumental drift (see DRIFT column description)"
- [methods] SA column time-origin convention: "Value of the secular acceleration, assuming that same "time origin" as DACE."
- [abstract] ESPRESSO GTO application scope: "we applied it to 33 ESPRESSO GTO targets, evaluating its performance and comparing it to the CCF method"
