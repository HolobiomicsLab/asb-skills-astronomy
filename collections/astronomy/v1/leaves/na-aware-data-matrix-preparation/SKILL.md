---
name: na-aware-data-matrix-preparation
description: Use when importing a galaxy/host-galaxy catalog (CSV/TXT, space-separated
  named columns) that contains missing entries and you intend to compute Pearson,
  Spearman, or MIC correlation matrices downstream.
when_to_use_negative:
- The input is already a clean, correlation-ready numeric matrix with no missing values
  — no NA filtering step is needed
- The analysis is principal components analysis (standard or Robust/Projection-Pursuit
  PCA) rather than correlation-matrix-based hierarchical clustering, where NA handling
  requirements differ
- Data are not tabular catalog measurements (e.g., images or time series), which AMADA's
  CSV/TXT import convention does not support
edam_operation: http://edamontology.org/operation_0278
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0091
tools:
- name: AMADA
  role: R package providing the data-import convention (CSV/TXT, NA-coded missing
    values) and the correlation-method selection (Pearson, Spearman, MIC) used downstream
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: minerva
  role: R library used to compute the Maximum Information Coefficient (MIC) matrix;
    requires the input to contain no NA values
- name: R
  role: Runtime environment in which the catalog is imported, NA rows are filtered,
    and the three correlation matrices are computed
  repo: http://www.r-project.org
- name: RStudio
  role: IDE for running the R installation and AMADA workflow
  repo: http://www.rstudio.com
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:b2f998102bf1bf674e33bf4c6d49ac3056cdb498d5ad90bd4a96784c331f7743
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/RafaelSdeSouza/AMADA
  tools:
  - AMADA
  - R
  - RStudio
  - minerva
  edam_operation: http://edamontology.org/operation_0278
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# na-aware-data-matrix-preparation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Prepare a high-dimensional astronomical catalog (e.g., CSV/TXT with space-separated, named columns) for downstream correlation-matrix computation in AMADA by correctly encoding missing values as NA and removing NA-containing rows before methods such as the MIC (minerva) matrix that cannot handle missing data.

## When to use

Use when importing a galaxy/host-galaxy catalog (CSV/TXT, space-separated named columns) that contains missing entries and you intend to compute Pearson, Spearman, or MIC correlation matrices downstream.

## When NOT to use

- The input is already a clean, correlation-ready numeric matrix with no missing values — no NA filtering step is needed
- The analysis is principal components analysis (standard or Robust/Projection-Pursuit PCA) rather than correlation-matrix-based hierarchical clustering, where NA handling requirements differ
- Data are not tabular catalog measurements (e.g., images or time series), which AMADA's CSV/TXT import convention does not support

## Inputs

- CSV/TXT dataset with named, space-separated columns and missing values marked as NA (e.g., mock galaxy catalog from Guo et al. 2011, supernova host properties from Sako et al. 2014, or ZENS catalog)
- User-uploaded catalog with an arbitrary number of columns and rows

## Outputs

- Pearson correlation matrix (CSV, square, diagonal = 1)
- Spearman correlation matrix (CSV, square, diagonal = 1)
- MIC correlation matrix computed on the NA-free subset (CSV, square, diagonal = 1)
- Row-filtered dataset with all NA-containing rows removed (for MIC computation)

## How to apply

Import the catalog as a space-separated CSV/TXT table with named columns, marking missing entries as NA per AMADA's data-input convention (e.g., the bundled Guo et al. 2011 mock galaxy catalog). Verify the import via the 'Import Dataset' panel before analysis. For linear (Pearson) and rank-based (Spearman) correlation matrices, R's cor() tolerates NA, but the Maximum Information Coefficient implementation in the minerva package does not, so restrict to rows with no NA values before computing the MIC matrix. This is a decision point: dropping NA rows shrinks the sample, so check how many rows survive before relying on MIC-based clustering. Write the resulting square matrices to CSV, with rows/columns corresponding to the catalog variables.

## Related tools

- **AMADA** (R package providing the data-import convention (CSV/TXT, NA-coded missing values) and the correlation-method selection (Pearson, Spearman, MIC) used downstream) — https://github.com/RafaelSdeSouza/AMADA
- **minerva** (R library used to compute the Maximum Information Coefficient (MIC) matrix; requires the input to contain no NA values)
- **R** (Runtime environment in which the catalog is imported, NA rows are filtered, and the three correlation matrices are computed) — http://www.r-project.org
- **RStudio** (IDE for running the R installation and AMADA workflow) — http://www.rstudio.com

## Examples

```
require(minerva); X <- read.table('guo2011_catalog.txt', header=TRUE); Xc <- X[complete.cases(X), ]; mic_mat <- mine(Xc)$MIC; write.csv(mic_mat, 'mic_matrix.csv')
```

## Evaluation signals

- Each output correlation CSV is a square matrix whose rows and columns correspond to the catalog variables
- Diagonal entries of each correlation matrix equal 1
- All off-diagonal Pearson/Spearman entries fall within [-1, 1]; MIC entries fall within [0, 1]
- The MIC matrix is computed only on the row subset with zero NA values — verify no NA error from minerva and record how many rows were dropped
- Imported dataset matches the declared format: named columns, space-separated values, missing entries coded as NA (checkable in the 'Import Dataset' panel / 'Dataset' tab)

## Limitations

- The current MIC implementation (minerva) does not support NA values, so rows containing NA must be dropped, potentially reducing the sample size and biasing MIC-based results
- Data must be imported as CSV/TXT with named, space-separated columns; other formats require prior conversion
- Missing data must be explicitly marked as NA; silently coded missing values (e.g., -99, blank fields) will not be handled correctly

## Evidence
