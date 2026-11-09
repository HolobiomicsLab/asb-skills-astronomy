---
name: nightly-zero-point-estimation
description: Use when you have per-spectrum radial velocities (with uncertainties,
  drift and secular-acceleration columns) from a spectrograph's pipeline and need
  to quantify the instrument's night-to-night zero-point stability as an upper limit
  on RV precision.
when_to_use_negative:
- When the goal is to detect or characterize planetary or stellar activity signals
  in the RV time series — the paper explicitly leaves a full analysis of the planetary
  and activity signals present in the considered datasets for future work, and the
  NZP scatter is only an upper-limit diagnostic.
- When RVs come from a pipeline whose output lacks the DRIFT and SA columns (e.g.,
  non-SBART CCF or SERVAL outputs), since drift and secular-acceleration corrections
  are essential to the per-night offset computation.
tools:
- name: S-BART
  role: Produces the semi-Bayesian template-matching RV time series (with RVc, RVc_ERR,
    DRIFT, SA columns) whose nightly offsets are used to estimate the ESPRESSO NZP
  repo: https://github.com/iastro-pt/sBART
- name: ESPRESSO
  role: Target spectrograph whose nightly zero point is estimated from the S-BART
    RVs of its 33 GTO targets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:59943ded17cd2eb06db4778a8efe2009d7c1cdb358f3d311c29ac88a258d8756
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - ESPRESSO
  - S-BART
  - SBART
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# nightly-zero-point-estimation

## Summary

Estimates the nightly zero point (NZP) of an echelle spectrograph (ESPRESSO) from S-BART radial-velocity time series of 33 GTO targets by grouping drift-corrected, secular-acceleration-subtracted RVs per night and computing the inverse-variance-weighted scatter of the per-night offsets. The resulting weighted NZP scatter (~0.7 m/s here) serves as an upper limit on the instrument's attainable RV precision.

## When to use

Use when you have per-spectrum radial velocities (with uncertainties, drift and secular-acceleration columns) from a spectrograph's pipeline and need to quantify the instrument's night-to-night zero-point stability as an upper limit on RV precision.

## When NOT to use

- When the goal is to detect or characterize planetary or stellar activity signals in the RV time series — the paper explicitly leaves a full analysis of the planetary and activity signals present in the considered datasets for future work, and the NZP scatter is only an upper-limit diagnostic.
- When RVs come from a pipeline whose output lacks the DRIFT and SA columns (e.g., non-SBART CCF or SERVAL outputs), since drift and secular-acceleration corrections are essential to the per-night offset computation.

## Inputs

- S-BART output RV time-series .txt files for the 33 ESPRESSO GTO targets, with BJD, RVc, RVc_ERR, SA, and DRIFT columns
- DRIFT_ERR column where present (added in quadrature)

## Outputs

- Per-night NZP table (espresso_nzp.csv)
- Inverse-variance-weighted NZP scatter value (expected below ~0.7 m/s)

## How to apply

Load the S-BART RV time-series .txt files (BJD, RVc, RVc_ERR, SA, DRIFT columns of the default SBART output format) with pandas for the 33 ESPRESSO GTO targets. Convert RVc from km/s to m/s, apply the instrumental drift correction encoded in the DRIFT column (adding DRIFT_ERR in quadrature where present), then subtract the SA column, which assumes the same 'time origin' as DACE, to remove secular acceleration. Group observations by night using the BJD column and compute a per-night, uncertainty-weighted mean RV offset per target; combine these to derive the nightly zero point per night. Finally compute the inverse-variance-weighted scatter of the NZP across all nights; for S-BART ESPRESSO data the published value falls below ~0.7 m/s and should be treated as an upper limit since it includes stellar variability, photon noise and planetary signals.

## Related tools

- **S-BART** (Produces the semi-Bayesian template-matching RV time series (with RVc, RVc_ERR, DRIFT, SA columns) whose nightly offsets are used to estimate the ESPRESSO NZP) — https://github.com/iastro-pt/sBART
- **ESPRESSO** (Target spectrograph whose nightly zero point is estimated from the S-BART RVs of its 33 GTO targets)

## Examples

```
pip install SBART
```

## Evaluation signals

- Weighted NZP scatter falls below ~0.7 m/s, matching the published upper-limit value that includes stellar variability, photon noise and planetary signals
- Per-night NZP table is written to espresso_nzp.csv with one row per observation night across the 33 GTO targets
- RV values are consistently in m/s (RVc converted from km/s) and drift-corrected with DRIFT, with DRIFT_ERR propagated in quadrature
- Secular acceleration removed by subtracting the SA column, consistent with the DACE time-origin convention
- Per-night offsets are inverse-variance-weighted (RVc_ERR-based), not simple means

## Limitations

- The ~0.7 m/s weighted NZP scatter should be taken as an upper limit on the RV precision attainable with ESPRESSO data because it includes stellar variability, photon noise and planetary signals.
- A full analysis of the planetary and activity signals present in the considered datasets was deferred to future work, so the NZP scatter does not isolate instrumental noise alone.

## Evidence

- [abstract] with the same data we estimated the nightly zero point (NZP) of the instrument: "with the same data we estimated the nightly zero point (NZP) of the instrument"
- [abstract] it should be taken as an upper limit on the RV precision attainable with ESPRESSO data: "it should be taken as an upper limit on the RV precision attainable with ESPRESSO data."
- [methods] RV measurement, in km/s, after being corrected by the instrumental drift (see DRIFT column description): "RV measurement, in km/s, after being corrected by the instrumental drift (see DRIFT column description)"
- [methods] Value of the secular acceleration, assuming that same "time origin" as DACE: "Value of the secular acceleration, assuming that same "time origin" as DACE."
- [readme] s-BART: A semi-Bayesian implementation of template matching for precise Radial Velocities: "s-BART: A semi-Bayesian implementation of template matching for precise Radial Velocities"
