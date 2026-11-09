---
name: astronomical-parameter-correlation-matrix-export
description: Use when a high-dimensional astronomical dataset (e.g., a galaxy catalog
  or supernova host-galaxy property table in CSV/TXT with named, space-separated columns)
  needs pairwise variable correlation matrices under multiple measures for clustering
  or visualization.
when_to_use_negative:
- The goal is dimensionality reduction itself (PCA / Robust PCA) rather than exporting
  correlation matrices — AMADA performs PCA on the original data separately
- The input contains missing values and the MIC measure is required, but rows cannot
  be filtered to complete cases without losing the sample
- The input is not a tabular numeric catalog (e.g., spectra or images) for which pairwise
  variable correlations across named columns are undefined
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Compute Pearson and Spearman correlation matrices and write them to CSV
  repo: http://www.r-project.org
- name: minerva
  role: R package providing the Maximum Information Coefficient (MIC) computation
    for the MIC matrix
- name: AMADA
  role: R package whose workflow consumes the correlation matrices for hierarchical
    clustering of high-dimensional astronomical datasets
  repo: https://github.com/RafaelSdeSouza/AMADA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:053b76ad6787ca9c20d77fa7efe1db3a9fda4c2bee3b97df5f908bc9cd1ec0a8
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - AMADA
  - R
  - minerva
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# astronomical-parameter-correlation-matrix-export

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Compute and export pairwise correlation matrices (Pearson, Spearman, and Maximum Information Coefficient) from a tabular astronomical catalog, as done in AMADA's iterative exploration of high-dimensional datasets, so that multiple correlation choices can feed downstream hierarchical clustering.

## When to use

Use when a high-dimensional astronomical dataset (e.g., a galaxy catalog or supernova host-galaxy property table in CSV/TXT with named, space-separated columns) needs pairwise variable correlation matrices under multiple measures for clustering or visualization.

## When NOT to use

- The goal is dimensionality reduction itself (PCA / Robust PCA) rather than exporting correlation matrices — AMADA performs PCA on the original data separately
- The input contains missing values and the MIC measure is required, but rows cannot be filtered to complete cases without losing the sample
- The input is not a tabular numeric catalog (e.g., spectra or images) for which pairwise variable correlations across named columns are undefined

## Inputs

- Astronomical catalog in CSV/TXT format, columns named and separated by spaces, missing values marked as NA (e.g., the mock galaxy catalog of Guo et al. 2011 bundled with AMADA)
- Choice of correlation method: Pearson, Spearman, or Maximum Information Coefficient (MIC)

## Outputs

- Square Pearson correlation matrix (rows/columns = catalog variables) written to CSV
- Square Spearman correlation matrix written to CSV
- MIC matrix (computed on complete-case rows via minerva) written to CSV

## How to apply

Load the catalog as a space-separated CSV/TXT table with named columns and NA-coded missing values, as required by AMADA's data input. Compute the Pearson correlation matrix (linear correlation) and the Spearman correlation matrix (rank-based/monotonic correlation) across the numeric columns with R's cor(). Compute the Maximum Information Coefficient (MIC) matrix with the minerva R package; because the current MIC implementation does not support NA, restrict the input to rows with no NA values (complete cases) before running mine(). Write the three matrices to separate CSV files. Note that AMADA's hierarchical clustering uses the correlation distance d(x,y) = 1 - |corr(x,y)|, so each exported matrix is a valid similarity basis for that clustering stage.

## Related tools

- **R** (Compute Pearson and Spearman correlation matrices and write them to CSV) — http://www.r-project.org
- **minerva** (R package providing the Maximum Information Coefficient (MIC) computation for the MIC matrix)
- **AMADA** (R package whose workflow consumes the correlation matrices for hierarchical clustering of high-dimensional astronomical datasets) — https://github.com/RafaelSdeSouza/AMADA

## Examples

```
M <- cor(data, method="spearman", use="complete.obs"); write.csv(M, "spearman_matrix.csv")
```

## Evaluation signals

- Each output CSV is a square matrix whose rows and columns correspond to the catalog variables
- Diagonal entries equal 1 for the Pearson and Spearman matrices
- All off-diagonal correlation entries fall within [-1, 1]
- The MIC matrix is computed only on rows with no NA values, and its dimensions match the complete-case variable set
- One CSV file is produced per correlation method (Pearson, Spearman, MIC), matching the three choices offered in AMADA

## Limitations

- The current MIC implementation (minerva) does not support NA values, so rows containing NA must be excluded before computing the MIC matrix
- Data must be imported as CSV/TXT with named, space-separated columns; other formats require reformatting
- MIC is more computationally expensive than Pearson/Spearman, so it may be slow on catalogs with many variables

## Evidence
