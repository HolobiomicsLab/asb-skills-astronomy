---
name: correlation-matrix-eigendecomposition
description: Use when a named-column CSV/TXT table of high-dimensional astronomical
  data (e.g. galaxy catalog properties) needs dimensionality reduction into uncorrelated
  PCs, with PC scores and variable loadings extracted.
when_to_use_negative:
- The goal is clustering or correlation structure discovery rather than dimensionality
  reduction — hierarchical clustering over different correlation matrices (Pearson,
  Spearman, MIC) is the complementary AMADA capability
- Data cannot be formatted as a named-column, space-separated CSV/TXT table with NA-marked
  missing values
- The dataset requires preprocessing (e.g. imputation of missing entries) before eigendecomposition
  is valid; AMADA expects missing data marked as NA without transformation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3316
tools:
- name: AMADA
  role: R package providing the standard PCA (correlation-matrix diagonalization)
    and robust Projection-Pursuit PCA routines with scores/loadings extraction
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: R
  role: Runtime environment in which the AMADA package and its PCA routines are executed
  repo: http://www.r-project.org
- name: shiny
  role: Runs the AMADA Shiny app that exposes Standard vs Robust PCA method selection
    and number-of-PCs display as interactive control options
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny
- name: pcaPP
  role: R dependency installed to support the Projection-Pursuit robust PCA computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:32b6f2d40d9ad76fe9701d7060b5f4e6830f9fe170bb7bc5d3d1e7d934475b46
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/RafaelSdeSouza/AMADA
  tools:
  - AMADA
  - R
  - pcaPP
  - shiny
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# correlation-matrix-eigendecomposition

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Performs standard and robust principal components analysis on high-dimensional astronomical datasets by diagonalizing the data correlation matrix, so that eigenvectors give the principal components and eigenvalues the variance explained. Implemented in the AMADA R package, it also supports a Projection-Pursuit robust PCA variant.

## When to use

Use when a named-column CSV/TXT table of high-dimensional astronomical data (e.g. galaxy catalog properties) needs dimensionality reduction into uncorrelated PCs, with PC scores and variable loadings extracted.

## When NOT to use

- The goal is clustering or correlation structure discovery rather than dimensionality reduction — hierarchical clustering over different correlation matrices (Pearson, Spearman, MIC) is the complementary AMADA capability
- Data cannot be formatted as a named-column, space-separated CSV/TXT table with NA-marked missing values
- The dataset requires preprocessing (e.g. imputation of missing entries) before eigendecomposition is valid; AMADA expects missing data marked as NA without transformation

## Inputs

- Named-column CSV/TXT data table (e.g. Guo et al. 2011 mock galaxy catalog, Sako et al. 2014 SDSS supernova host properties, or ZENS catalog) with arbitrary numbers of rows and columns
- Missing entries encoded as NA
- User control parameters: PCA method (Standard or Robust), number of PCs to display

## Outputs

- PC scores table: one row per catalog object
- Loadings table: relative contribution of each input variable to each principal component, one row per variable per PC
- Eigenvalues consistent with the variance explained by each PC
- Nightingale charts of selected principal components (Shiny app visualization)

## How to apply

Import the dataset into R as a CSV/TXT table with named, space-separated columns, marking missing entries as NA. Run the standard PCA routine from the AMADA package, which diagonalizes the data correlation matrix so eigenvectors correspond to PCs and eigenvalues to the variance explained (Jolliffe 2002); PC1 is the eigenvector with the largest eigenvalue. Optionally run the robust PCA variant based on the Projection-Pursuit principle, projecting the data into a lower-dimensional space that maximizes a robust measure of variance of the projected data (Croux, Filzmoser and Oliveira 2007). Extract the PC scores for the objects and the loadings (relative contribution of each variable to each principal component) for both standard and robust solutions, and write them to tabular CSV output files. In the Shiny app, the number of PCs displayed as Nightingale charts and the PCA method (Standard vs Robust) are user-selectable control options.

## Related tools

- **AMADA** (R package providing the standard PCA (correlation-matrix diagonalization) and robust Projection-Pursuit PCA routines with scores/loadings extraction) — https://github.com/RafaelSdeSouza/AMADA
- **R** (Runtime environment in which the AMADA package and its PCA routines are executed) — http://www.r-project.org
- **shiny** (Runs the AMADA Shiny app that exposes Standard vs Robust PCA method selection and number-of-PCs display as interactive control options) — https://github.com/RafaelSdeSouza/AMADA_shiny
- **pcaPP** (R dependency installed to support the Projection-Pursuit robust PCA computation)

## Examples

```
install_github("RafaelSdeSouza/AMADA"); runUrl('https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip')
```

## Evaluation signals

- Scores output contains exactly one row per catalog object; loadings output contains one row per input variable per principal component
- Reported eigenvalues are consistent with the fraction of variance explained by each PC, with PC1 corresponding to the direction of greatest variance
- PCs are mutually uncorrelated (orthogonal transformation of the original variables)
- Both standard and robust PCA outputs exist as CSV files and agree on the ordering of dominant components where expected
- Input table validation: columns named and space-separated, missing entries marked NA before running the analysis

## Limitations

- Standard PCA diagonalizes the data correlation matrix and is sensitive to outliers; the robust Projection-Pursuit variant exists to mitigate this
- The number of PCs to display must be chosen by the user; AMADA only guides clustering via the optimal cluster number of Calinski and Harabasz (1972)
- Data must be imported as CSV/TXT with named columns separated by spaces; other formats require prior conversion
- Robust PCA projects onto a lower-dimensional space and may not reproduce the full eigenspectrum of the standard solution

## Evidence
