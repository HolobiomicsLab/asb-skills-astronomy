---
name: rv-output-file-parsing
description: Use when you have S-BART output .txt files from ESPRESSO (or HARPS) observations
  and need to load, correct (SA subtraction, quadrature drift-error propagation),
  and unit-convert the RV time series before computing median uncertainties or running
  a GLS periodogram.
when_to_use_negative:
- The input is already an SA-corrected, drift-corrected RV table in m/s — re-applying
  SA subtraction or drift-error propagation would double-correct
- You need raw spectra-level products (e.g. telluric templates or normalized spectra)
  rather than the per-observation RV text outputs
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SBART
  role: Produces the RV time-series text files that this skill parses; installable
    via pip install SBART
  repo: https://github.com/iastro-pt/sBART
- name: pandas
  role: Used to load the S-BART RV output text file and read the BJD, RVc [Km/s],
    RVc_ERR, SA, and DRIFT columns
- name: ESPRESSO
  role: Instrument whose observations generate the RV files (309 observations of 16
    M-type GTO targets); DRS-level drift handling determines whether DRIFT is 'nan'
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:e22b24a330e2a8eb6a839efb88604d4fb3769919e775e33b49fd8da348e49f2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - ESO official pipeline
  - ESPRESSO
  - S-BART
  - SBART
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# RV output file parsing

## Summary

Parse the plain-text RV time-series files produced by the S-BART pipeline (columns BJD, RVc [Km/s], RVc_ERR, SA, DRIFT, DRIFT_ERR), apply the documented SA and drift corrections, and prepare clean per-target RV tables (e.g. under RV_step/individual_subInst/<instrument>/) for uncertainty statistics or periodogram analysis.

## When to use

Use when you have S-BART output .txt files from ESPRESSO (or HARPS) observations and need to load, correct (SA subtraction, quadrature drift-error propagation), and unit-convert the RV time series before computing median uncertainties or running a GLS periodogram.

## When NOT to use

- The input is already an SA-corrected, drift-corrected RV table in m/s — re-applying SA subtraction or drift-error propagation would double-correct
- You need raw spectra-level products (e.g. telluric templates or normalized spectra) rather than the per-observation RV text outputs

## Inputs

- S-BART RV time-series .txt files (e.g. under RV_step/individual_subInst/<instrument>/) with columns BJD, RVc [Km/s], RVc_ERR, SA, DRIFT, DRIFT_ERR
- CCF RV uncertainties from the ESO official pipeline for the same observations (for comparison workflows)

## Outputs

- Corrected RV time series (SA-corrected, drift-uncertainty-propagated, converted from km/s to m/s or cm/s)
- Tables such as rv_uncertainty_comparison.csv or inputs to significant_periodicities.csv (period, semi-amplitude, FAP, target name)

## How to apply

Load the RV time-series .txt files with pandas, reading the BJD, RVc [Km/s], RVc_ERR, SA, and DRIFT columns. Apply the secular-acceleration correction by subtracting the SA column from RVc (keeping the DACE time origin as documented for the SA column). Check the DRIFT column: where drift was already applied at the DRS level it is 'nan'; where DRIFT_ERR is not 'nan', propagate the instrumental drift uncertainty by adding DRIFT_ERR in quadrature to the raw S-BART RV uncertainties. Finally convert RVs and uncertainties from km/s to m/s (or cm/s when comparing against the ~15 cm/s median-uncertainty benchmark) before downstream analysis.

## Related tools

- **SBART** (Produces the RV time-series text files that this skill parses; installable via pip install SBART) — https://github.com/iastro-pt/sBART
- **pandas** (Used to load the S-BART RV output text file and read the BJD, RVc [Km/s], RVc_ERR, SA, and DRIFT columns)
- **ESPRESSO** (Instrument whose observations generate the RV files (309 observations of 16 M-type GTO targets); DRS-level drift handling determines whether DRIFT is 'nan')

## Examples

```
import pandas as pd; df = pd.read_csv('RV_step/individual_subInst/ESPRESSO/target.txt', sep='\s+'); rv = df['RVc [Km/s]'] - df['SA']
```

## Evaluation signals

- Parsed uncertainties reproduce the reported ~15 cm/s median RV uncertainty over the 309 ESPRESSO observations of the 16 M-type targets (after km/s→cm/s conversion)
- Quadrature combination of DRIFT_ERR with RVc_ERR only applied where the drift value is not 'nan' (i.e. drift not already corrected at DRS level)
- SA correction keeps the DACE time origin: corrected RV = RVc − SA
- Downstream CCF comparison is consistent with the reported ~10% decrease in median RV scatter for M-type stars
- Any derived periodogram peaks carry FAP values within [0,1] and below the 0.1% significance threshold

## Limitations

- Drift correction status is file-dependent: the DRIFT column may be 'nan' when drift was already applied at the DRS level, so blind quadrature addition of DRIFT_ERR would double-count the uncertainty
- RV values are delivered in km/s and must be converted before comparing against m/s-scale benchmarks such as the ~0.7 m/s ESPRESSO NZP noise floor
- S-BART's RV uncertainty behaviour is tied to its template construction and telluric handling; differences versus other pipelines (HARPS-TERRA, SERVAL, CCF) may not transfer to other instruments

## Evidence

- [methods] SA and drift column semantics: "Value of the secular acceleration, assuming that same "time origin" as DACE."
- [methods] DRIFT column semantics: "RV measurement, in km/s, after being corrected by the instrumental drift (see DRIFT column description)"
- [abstract] Median uncertainty benchmark: "S-BART achieves a median uncertainty of ~15 cm s−1over 309 observations of 16 targets"
- [abstract] CCF scatter comparison: "We find a decrease in the median RV scatter of ~10 and ~4% for M- and K-type stars, respectively."
- [methods] Tool installation: "SBART can be pip-installed:

    pip install SBART"
- [abstract] Pipeline scope: "This methodology, named the Semi-Bayesian Approach for RVs with Template matching, S-BART, can currently be applied to HARPS and ESPRESSO data."
