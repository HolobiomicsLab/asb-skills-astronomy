---
name: sdss-sql-catalogue-querying
description: Use when a pipeline needs the SDSS DR10 galaxy catalogue (u'g'r'i'z'
  magnitudes plus spectroscopic redshifts) as raw input for photo-z estimation, PCA
  decomposition, or GLM training/testing splits.
when_to_use_negative:
- The analysis uses the PHAT0 simulated catalogue or PHAT1 observational catalogue,
  which are provided as files rather than queried from SDSS
- The magnitudes/photometry are already available locally as a feature table, so re-querying
  the SDSS database is unnecessary
- The target survey is not SDSS (e.g. a different photometric system than u'g'r'i'z'),
  since the Appendix C query and the PCA/PC interpretation are tied to DR10 five-band
  photometry
edam_operation: http://edamontology.org/operation_2422
tools:
- name: CosmoPhotoz
  role: Downstream R/Python package that consumes the retrieved SDSS DR10 sample for
    robust PCA and gamma-family GLM photo-z fitting
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: Python
  role: Environment for downloading/querying the catalogue and running the PCA + GLM
    pipeline (CosmoPhotoz Python implementation, statsmodels backend)
- name: R
  role: Alternative environment for the CosmoPhotoz implementation that fits the gamma
    GLM to the retrieved catalogue via iteratively reweighted least squares
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:ad0c708e25fde6f7686e850ad37821bc95baeaa013efe96634b20ce2d793869b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - CosmoPhotoz
  - Python
  - R
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# sdss-sql-catalogue-querying

## Summary

Retrieves the SDSS Data Release 10 galaxy sample (1,347,640 galaxies with u'g'r'i'z' magnitudes and spectroscopic redshifts, z = 0-1.0) via the SQL query published in Appendix C of the gamma-GLM photo-z paper, providing the real-data input for the robust PCA + gamma GLM photometric redshift pipeline.

## When to use

Use when a pipeline needs the SDSS DR10 galaxy catalogue (u'g'r'i'z' magnitudes plus spectroscopic redshifts) as raw input for photo-z estimation, PCA decomposition, or GLM training/testing splits.

## When NOT to use

- The analysis uses the PHAT0 simulated catalogue or PHAT1 observational catalogue, which are provided as files rather than queried from SDSS
- The magnitudes/photometry are already available locally as a feature table, so re-querying the SDSS database is unnecessary
- The target survey is not SDSS (e.g. a different photometric system than u'g'r'i'z'), since the Appendix C query and the PCA/PC interpretation are tied to DR10 five-band photometry

## Inputs

- SDSS DR10 database access (SQL endpoint) with the Appendix C query for galaxy photometry and spectroscopic redshifts
- Parameters: redshift window z = 0-1.0, five-band u'g'r'i'z' magnitudes (dereddened magnitudes for the O(b) outlier definition)

## Outputs

- Tabular catalogue (CSV / data frame) of 1,347,640 SDSS DR10 galaxies with u'g'r'i'z' magnitudes and spectroscopic redshifts
- Training/test split of the catalogue with the training sample holding at least 10% of galaxies (e.g. a 50,000-galaxy training subset used in the reported 5-PC run)

## How to apply

Issue the SQL query from Appendix C of Elliott et al. (2015, DOI 10.1016/j.ascom.2015.01.002) against the SDSS DR10 database to pull the galaxy sample of 1,347,640 objects covering z = 0-1.0 with u'g'r'i'z' magnitudes. Store the returned rows as a tabular data frame (e.g. CSV or pandas/R data frame) with one row per galaxy and magnitude plus redshift columns. The retrieved sample should be treated as a single complete data set: the article performs robust PCA on the full set (train + test) with a cumulative variance threshold of ~99.5%, then randomly splits it so the training sample holds at least 10% of galaxies. Validate the download by checking the object count, the redshift range, and that downstream PCA on the five magnitudes yields 4-5 components retaining 99.5% of variance before feeding the gamma-family GLM with log link.

## Related tools

- **CosmoPhotoz** (Downstream R/Python package that consumes the retrieved SDSS DR10 sample for robust PCA and gamma-family GLM photo-z fitting) — https://github.com/COINtoolbox/COSMOPhotoz/
- **Python** (Environment for downloading/querying the catalogue and running the PCA + GLM pipeline (CosmoPhotoz Python implementation, statsmodels backend))
- **R** (Alternative environment for the CosmoPhotoz implementation that fits the gamma GLM to the retrieved catalogue via iteratively reweighted least squares)

## Evaluation signals

- Returned row count matches the reported 1,347,640 galaxies
- Spectroscopic redshifts fall within the queried window z = 0-1.0
- All five u'g'r'i'z' magnitude columns are present and numeric, with dereddened magnitudes available for the O(b) metric
- Downstream robust PCA on the complete data set retains ~99.5% of variance with 4-5 components
- End-to-end reproduction yields catastrophic outlier rates O(a) = 8.087% (4 PCs) and 6.439% (5 PCs, 50,000-galaxy training set), with dereddened O(b) = 2.90 and 2.25 respectively

## Limitations

- Reported performance figures assume a specific low-end hardware/OS configuration (AMD Athlon X2 Dual-Core QL-64, 1.7 GB RAM, Ubuntu 10.04); timings on other systems will differ
- The R and Python implementations solve the SVD in the PCA step and minimise the GLM slightly differently, so results may not reconcile exactly between the two backends
- The dereddened-magnitude variant is required for the reported O(b) outlier fractions (2.90 / 2.25); using uncorrected magnitudes will not match them

## Evidence

- [other] Obtain the SDSS DR10 galaxy sample (1,347,640 galaxies, z = 0-1.0, u'g'r'i'z' magnitudes) via the SQL query given in Appendix C.: [evidence span withheld — anchor status: not_found]
- [results] The SDSS data set was randomly split into training and test sets: [evidence span withheld — anchor status: not_found]
