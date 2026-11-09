---
name: radial-velocity-uncertainty-statistics
description: Use when comparing the delivered RV uncertainty (not just scatter) of
  a template-matching pipeline against a CCF pipeline over a fixed set of spectrograph
  observations, and the RV files carry separate RV and drift-error columns.
when_to_use_negative:
- The analysis goal is RV scatter/precision of time-series signal detection (e.g.
  NZP or planet searches) rather than delivered per-observation uncertainty comparison
- The RV products are not S-BART outputs with RVc_ERR and DRIFT_ERR columns, or the
  comparison target is not a CCF-based ESO pipeline delivery
- Input spectra are HARPS observations validated against HARPS-TERRA/SERVAL rather
  than ESPRESSO M-type targets — use the Barnard's star HARPS validation workflow
  instead
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: S-BART
  role: Produces the template-matching RV time-series whose RVc_ERR [Km/s] uncertainties
    (combined in quadrature with DRIFT_ERR) are the primary input
  repo: https://github.com/iastro-pt/sBART
- name: ESO official pipeline
  role: Source of the CCF RV uncertainties used as the comparison baseline for the
    same ESPRESSO observations
- name: ESPRESSO
  role: Spectrograph whose GTO observations (309 observations, 16 M-type targets)
    supply the dataset
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:98782a8dcf014ed94c25fd8893218399a75a097a0673c3d916111d907db8ea20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - ESO official pipeline
  - ESPRESSO
  - S-BART
  - SBART
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# radial-velocity-uncertainty-statistics

## Summary

Quantifies the per-observation radial-velocity precision delivered by S-BART versus the ESO CCF pipeline by propagating instrumental drift uncertainties in quadrature and computing median RV uncertainties across the 309 ESPRESSO observations of 16 M-type targets, reproducing the paper's ~15 cm/s benchmark.

## When to use

Use when comparing the delivered RV uncertainty (not just scatter) of a template-matching pipeline against a CCF pipeline over a fixed set of spectrograph observations, and the RV files carry separate RV and drift-error columns.

## When NOT to use

- The analysis goal is RV scatter/precision of time-series signal detection (e.g. NZP or planet searches) rather than delivered per-observation uncertainty comparison
- The RV products are not S-BART outputs with RVc_ERR and DRIFT_ERR columns, or the comparison target is not a CCF-based ESO pipeline delivery
- Input spectra are HARPS observations validated against HARPS-TERRA/SERVAL rather than ESPRESSO M-type targets — use the Barnard's star HARPS validation workflow instead

## Inputs

- S-BART RV time-series .txt files under RV_step/individual_subInst/<instrument>/ with RVc_ERR [Km/s] and DRIFT_ERR columns
- CCF RV uncertainties from the ESO official pipeline for the same ESPRESSO observations
- Mapping of observations to the 16 M-type targets (out of 33 ESPRESSO GTO targets, 309 observations)

## Outputs

- rv_uncertainty_comparison.csv tabulating S-BART and CCF median RV uncertainties (cm/s) and their ratio
- Median RV uncertainty per pipeline over the 309 observations of the 16 M-type targets

## How to apply

Load the RV time-series .txt files written by the S-BART RV routine under RV_step/individual_subInst/<instrument>/, parsing the RVc_ERR [Km/s] column (the RV uncertainty in km/s) together with the DRIFT_ERR column. Where DRIFT_ERR is not 'nan' (i.e. the instrumental drift was not already corrected at the DRS level), add it in quadrature to the raw S-BART RV uncertainty; skip quadrature addition when the drift correction was already applied. Convert the propagated uncertainties from km/s to cm/s and compute the median over all 309 observations of the 16 M-type ESPRESSO GTO targets. Repeat the identical median computation on the CCF RV uncertainties delivered by the ESO official pipeline for the same observations, then tabulate both medians and their ratio (expected S-BART median ~15 cm/s and ~10% lower median scatter for M-type stars) and write the table to rv_uncertainty_comparison.csv.

## Related tools

- **S-BART** (Produces the template-matching RV time-series whose RVc_ERR [Km/s] uncertainties (combined in quadrature with DRIFT_ERR) are the primary input) — https://github.com/iastro-pt/sBART
- **ESO official pipeline** (Source of the CCF RV uncertainties used as the comparison baseline for the same ESPRESSO observations)
- **ESPRESSO** (Spectrograph whose GTO observations (309 observations, 16 M-type targets) supply the dataset)

## Examples

```
pip install SBART
```

## Evaluation signals

- S-BART median RV uncertainty over the 309 observations of 16 M-type targets reproduces the reported ~15 cm/s after km/s-to-cm/s conversion
- The S-BART-to-CCF ratio is consistent with the reported ~10% decrease in median RV scatter for M-type stars (and ~4% for K-type stars)
- Quadrature combination of DRIFT_ERR is applied only where DRIFT_ERR is not 'nan' (drift not already corrected at the DRS level)
- rv_uncertainty_comparison.csv contains side-by-side S-BART and CCF median uncertainties for the identical observation set (same 309 observations counted per pipeline)
- Count of observations aggregated equals 309; target subset restricted to the 16 M-type targets of the 33 ESPRESSO GTO targets

## Limitations

- DRIFT_ERR handling is conditional: when drift was already corrected at the DRS level the column is 'nan' and must not be added in quadrature, so misinterpretation inflates uncertainties
- The comparison of median RV scatter against CCF depends on template construction and telluric handling, which the authors attribute the performance differences to
- Uncertainties should be compared against the ESPRESSO NZP scatter of below ~0.7 m/s, which the authors caution should be taken as an upper limit on attainable RV precision

## Evidence

- [abstract] S-BART achieves a median RV uncertainty of ~15 cm/s over 309 observations of 16 M-type targets with ESPRESSO: "S-BART achieves a median uncertainty of ~15 cm s−1over 309 observations of 16 targets"
- [abstract] Median RV scatter decreased by ~10% for M-type and ~4% for K-type stars compared to the CCF method: "We find a decrease in the median RV scatter of ~10 and ~4% for M- and K-type stars, respectively."
- [abstract] NZP scatter as upper limit on attainable precision: [evidence span withheld — anchor status: not_found]
- [methods] RV column definition after drift correction: "RV measurement, in km/s, after being corrected by the instrumental drift (see DRIFT column description)"
- [abstract] Comparison target is the ESO CCF pipeline: [evidence span withheld — anchor status: not_found]
