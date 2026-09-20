---
name: robust-pca-projection-pursuit-estimation
description: Use when a tabular astronomical catalog (e.g., CSV/TXT with named columns)
  may contain outliers or non-Gaussian structure that would distort standard PCA,
  and you need dimension reduction with loadings and scores robust to such contamination.
when_to_use_negative:
- Data are clean and approximately Gaussian, so standard PCA via correlation-matrix
  diagonalization suffices and is cheaper
- The dataset is not importable as a CSV/TXT table with named columns (e.g., binary
  or ragged formats)
- The goal is clustering structure rather than linear dimension reduction — AMADA's
  hierarchical clustering over correlation matrices (Pearson, Spearman, or MIC) would
  be the appropriate mode
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: AMADA
  role: Provides the Robust PCA analysis mode based on the Projection-Pursuit principle,
    selected from the Shiny app control panel
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: R
  role: Runtime environment in which the AMADA package (and its pcaPP dependency)
    executes the robust PCA
  repo: http://www.r-project.org
- name: RStudio
  role: IDE for installing AMADA from GitHub and running the Shiny app
  repo: http://www.rstudio.com
- name: shiny
  role: Runs the AMADA_shiny web app used to launch the Robust PCA analysis
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:4160ca85357bfa6a673986b8778a40b871e364c571a4deab759be90619d2049c
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/RafaelSdeSouza/AMADA
  tools:
  - AMADA
  - R
  - RStudio
  - shiny
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# robust-pca-projection-pursuit-estimation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Robust principal component analysis via the Projection-Pursuit principle, as implemented in the AMADA R package, which projects high-dimensional astronomical data onto a lower-dimensional space maximizing a robust measure of variance, offering an outlier-resistant alternative to standard correlation-matrix PCA.

## When to use

Use when a tabular astronomical catalog (e.g., CSV/TXT with named columns) may contain outliers or non-Gaussian structure that would distort standard PCA, and you need dimension reduction with loadings and scores robust to such contamination.

## When NOT to use

- Data are clean and approximately Gaussian, so standard PCA via correlation-matrix diagonalization suffices and is cheaper
- The dataset is not importable as a CSV/TXT table with named columns (e.g., binary or ragged formats)
- The goal is clustering structure rather than linear dimension reduction — AMADA's hierarchical clustering over correlation matrices (Pearson, Spearman, or MIC) would be the appropriate mode

## Inputs

- Tabular astronomical catalog in CSV/TXT format with named columns separated by spaces (e.g., the Guo et al. 2011 mock galaxy catalog built on the Millennium simulation)
- Missing values marked as NA in the input table

## Outputs

- PC scores table (one row per catalog object) written to CSV
- Loadings table (one row per input variable per principal component) written to CSV
- Eigenvalues / variance explained per principal component

## How to apply

Import the dataset into R as a CSV/TXT table with named, space-separated columns, marking missing entries as NA, using AMADA's 'Import Dataset' panel or the Shiny app. Select 'Robust PCA' as the PCA method (via the left-panel control options and the 'Make it so' button) instead of 'Standard PCA'. The robust routine applies the Projection-Pursuit principle (Croux, Filzmoser and Oliveira, 2007): it projects the data into a lower-dimensional space such that a robust measure of variance of the projected data is maximized, unlike standard PCA which diagonalizes the data correlation matrix (Jolliffe 2002). Extract the PC scores for the objects and the loadings (relative contribution of each variable to the i-th principal component), and write them to tabular CSV output files. Validate that both output tables exist — one row per catalog object for scores and one row per input variable per PC for loadings — and that reported eigenvalues are consistent with the variance explained by each PC.

## Related tools

- **AMADA** (Provides the Robust PCA analysis mode based on the Projection-Pursuit principle, selected from the Shiny app control panel) — https://github.com/RafaelSdeSouza/AMADA
- **R** (Runtime environment in which the AMADA package (and its pcaPP dependency) executes the robust PCA) — http://www.r-project.org
- **RStudio** (IDE for installing AMADA from GitHub and running the Shiny app) — http://www.rstudio.com
- **shiny** (Runs the AMADA_shiny web app used to launch the Robust PCA analysis) — https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip

## Examples

```
require(shiny); runUrl('https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip')
```

## Evaluation signals

- Scores CSV contains exactly one row per catalog object (per row of the input Guo et al. 2011 table)
- Loadings CSV contains one row per input variable per principal component
- Reported eigenvalues are consistent with the variance explained by each PC
- Both output tables exist and are well-formed after the run
- Robust PCA results remain stable when obvious outlier rows in the input catalog are perturbed, unlike standard correlation-matrix PCA

## Limitations

- Data must be imported as CSV/TXT with named columns separated by spaces; other formats require pre-conversion
- Missing data must be explicitly marked as NA in the input table
- Robust PCA is one of only two PCA methods offered (Standard or Robust) — no intermediate or sparse PCA variants are available
- The number of PCs displayed is user-limited via the 'Number of PCs' control (Nightingale charts)

## Evidence
