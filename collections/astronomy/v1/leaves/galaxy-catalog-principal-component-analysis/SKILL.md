---
name: galaxy-catalog-principal-component-analysis
description: Use when a high-dimensional galaxy/astronomical catalog in CSV/TXT format
  with named columns needs dimensionality reduction, and you want PC scores and loadings
  from either standard correlation-matrix PCA or robust Projection-Pursuit PCA.
when_to_use_negative:
- The analysis goal is finding hidden structure/partitions (hierarchical clustering
  over different correlation matrices) rather than PC scores and loadings — use AMADA's
  hierarchical clustering analysis instead
- The data are not a named-column CSV/TXT table with space-separated fields and require
  extensive reshaping before they can be imported into AMADA
- Missing values cannot be represented as NA in the input table
tools:
- name: AMADA
  role: R package providing both Standard PCA (correlation-matrix diagonalization)
    and Robust PCA (Projection-Pursuit) on the imported catalog
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: AMADA_shiny
  role: Shiny web app exposing PCA method selection (Standard vs Robust) and number-of-PCs
    control for the catalog
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip
- name: R
  role: Runtime environment in which AMADA is installed and executed
  repo: http://www.r-project.org
- name: RStudio
  role: IDE used to run the AMADA R workflow
  repo: http://www.rstudio.com
- name: pcaPP
  role: Required R library supporting the Projection-Pursuit robust PCA variant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:3e0b6a8acadb47ac88b64728cfd51c4ed43b6f2f6ef38997dbf53fc960718758
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/RafaelSdeSouza/AMADA
  tools:
  - AMADA
  - AMADA_shiny
  - R
  - RStudio
  - pcaPP
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# galaxy-catalog-principal-component-analysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Run standard and robust (Projection-Pursuit) principal components analysis on high-dimensional astronomical catalogs (e.g., the Guo et al. 2011 mock galaxy catalog) using the AMADA R package, extracting PC scores for objects and loadings per variable.

## When to use

Use when a high-dimensional galaxy/astronomical catalog in CSV/TXT format with named columns needs dimensionality reduction, and you want PC scores and loadings from either standard correlation-matrix PCA or robust Projection-Pursuit PCA.

## When NOT to use

- The analysis goal is finding hidden structure/partitions (hierarchical clustering over different correlation matrices) rather than PC scores and loadings — use AMADA's hierarchical clustering analysis instead
- The data are not a named-column CSV/TXT table with space-separated fields and require extensive reshaping before they can be imported into AMADA
- Missing values cannot be represented as NA in the input table

## Inputs

- High-dimensional astronomical catalog in CSV/TXT format with named, space-separated columns (e.g., Guo et al. 2011 mock galaxy catalog; SDSS supernova host properties from Sako et al. 2014 Table 1; ZENS catalog)
- Missing-value marker: NA

## Outputs

- PC scores table: one row per catalog object
- Loadings table: one row per input variable per principal component (relative contribution of each variable to the i-th PC)
- Eigenvalues quantifying variance explained by each PC (standard PCA)

## How to apply

Import the dataset as a CSV/TXT table with named columns (space-separated); missing entries must be marked NA. From R, install AMADA from GitHub and choose the PCA method in the Shiny interface: Standard PCA diagonalizes the data correlation matrix so eigenvectors give the PCs and eigenvalues the variance explained (Jolliffe 2002), while Robust PCA applies the Projection-Pursuit principle, projecting the data into a lower-dimensional space that maximizes a robust measure of variance (Croux, Filzmoser and Oliveira, 2007). Extract PC scores for the catalog objects and loadings (relative contribution of each variable to each PC) for both standard and robust solutions. Write the scores and loadings to tabular CSV outputs and check that eigenvalues are consistent with the variance explained per PC.

## Related tools

- **AMADA** (R package providing both Standard PCA (correlation-matrix diagonalization) and Robust PCA (Projection-Pursuit) on the imported catalog) — https://github.com/RafaelSdeSouza/AMADA
- **AMADA_shiny** (Shiny web app exposing PCA method selection (Standard vs Robust) and number-of-PCs control for the catalog) — https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip
- **R** (Runtime environment in which AMADA is installed and executed) — http://www.r-project.org
- **RStudio** (IDE used to run the AMADA R workflow) — http://www.rstudio.com
- **pcaPP** (Required R library supporting the Projection-Pursuit robust PCA variant)

## Examples

```
require(devtools); install_github("RafaelSdeSouza/AMADA"); require(shiny); runUrl('https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip')
```

## Evaluation signals

- Scores output table contains exactly one row per catalog object (one row per entry of the input CSV/TXT table)
- Loadings output table contains one row per input variable per principal component
- Standard-PCA eigenvalues are consistent with the variance explained by each PC, with PC1's eigenvector giving the direction of greatest variance
- The input table was imported correctly: named columns, space-separated, missing values marked as NA (check the 'Import Dataset' panel)
- Both standard and robust variants were run when requested, with robust PCA maximizing a robust variance measure of the projected (lower-dimensional) data

## Limitations

- Data must be imported as CSV/TXT with named, space-separated columns; missing data must be marked as NA
- Robust PCA is restricted to the Projection-Pursuit formulation (Croux, Filzmoser and Oliveira, 2007) — it projects onto a lower-dimensional space but the number of PCs displayed must be chosen by the user
- The workflow assumes an arbitrary but tabular number of rows/columns; non-tabular astronomical data are not supported by the import path

## Evidence
