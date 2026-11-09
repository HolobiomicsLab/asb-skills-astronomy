---
name: principal-component-analysis-eigendecomposition
description: Use when a dataset of astronomical objects (e.g., SDSS supernova host-galaxy
  properties in CSV/TXT with named, space-separated columns) must be reduced to uncorrelated
  variables to explore its dominant variance directions.
when_to_use_negative:
- The data contain NA values and the Maximum Information Coefficient (MIC) correlation
  method is required, since MIC does not support NA values
- The analysis goal is clustering/grouping of variables rather than variance decomposition
  — hierarchical clustering with correlation distance d(x,y)=1-|corr(x,y)| is the
  appropriate AMADA mode
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0092
tools:
- name: AMADA
  role: Shiny-based R package that runs the PCA (standard or robust) on the imported
    astronomical dataset and visualizes PCs as Nightingale charts
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: R
  role: Runtime environment in which the AMADA package and its Shiny app execute
- name: shiny
  role: Runs the AMADA Shiny App that exposes the PCA controls and plots
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny
- name: pcaPP
  role: Required R library underpinning robust PCA via the Projection-Pursuit principle
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:2b18db749ee008df1307c894adbc66f30128298bd3b5efb54602c45bbcb64c40
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/RafaelSdeSouza/AMADA
  tools:
  - AMADA
  - R
  - ape
  - corrplot
  - fpc
  - pcaPP
  - shiny
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# principal-component-analysis-eigendecomposition

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Performs principal components analysis on high-dimensional astronomical datasets by diagonalizing the data correlation matrix, yielding eigenvectors as principal components and eigenvalues as variance explained; AMADA also offers a Robust PCA alternative based on the Projection-Pursuit principle.

## When to use

Use when a dataset of astronomical objects (e.g., SDSS supernova host-galaxy properties in CSV/TXT with named, space-separated columns) must be reduced to uncorrelated variables to explore its dominant variance directions.

## When NOT to use

- The data contain NA values and the Maximum Information Coefficient (MIC) correlation method is required, since MIC does not support NA values
- The analysis goal is clustering/grouping of variables rather than variance decomposition — hierarchical clustering with correlation distance d(x,y)=1-|corr(x,y)| is the appropriate AMADA mode

## Inputs

- CSV/TXT dataset with named columns separated by spaces and an arbitrary number of rows/columns; missing data marked as NA (e.g., SDSS Type Ia and II supernova host-galaxy properties from Sako et al. 2014, Table 1)
- User-selected number of principal components to display

## Outputs

- Eigenvectors (principal components) of the data correlation matrix
- Eigenvalues giving the variance explained per PC
- PC scores for the objects, displayed as Nightingale charts

## How to apply

Import the data into AMADA (R/Shiny) as CSV/TXT with named columns separated by spaces, marking missing entries as NA. Compute the data correlation matrix (Pearson is default; Spearman or MIC are selectable, though MIC does not support NA values) and diagonalize it: the resulting eigenvectors are the principal components and the eigenvalues give the variance explained by each PC, with PC1 the direction of greatest variance, PC2 the next, and so on. Select the number of PCs to display as Nightingale charts via the 'Number of PCs' control option. If outliers or non-Gaussian structure are a concern, switch the 'PCA method' control to Robust PCA, which projects the data onto a lower-dimensional space maximizing a robust measure of projected variance (Croux, Filzmoser and Oliveira, 2007). Once the combination of options is chosen, click 'Make it so' to update the plots and extract the PC scores for the objects.

## Related tools

- **AMADA** (Shiny-based R package that runs the PCA (standard or robust) on the imported astronomical dataset and visualizes PCs as Nightingale charts) — https://github.com/RafaelSdeSouza/AMADA
- **R** (Runtime environment in which the AMADA package and its Shiny app execute)
- **shiny** (Runs the AMADA Shiny App that exposes the PCA controls and plots) — https://github.com/RafaelSdeSouza/AMADA_shiny
- **pcaPP** (Required R library underpinning robust PCA via the Projection-Pursuit principle)

## Examples

```
require(shiny); runUrl('https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip')
```

## Evaluation signals

- Confirm the dataset imported correctly via the 'Import Dataset' panel and the 'Dataset' tab before running PCA
- Check that the number of PCs computed matches the correlation-matrix dimensionality and that eigenvalues are non-negative and sorted in descending order (PC1 largest)
- Verify the displayed Nightingale charts correspond to the user-selected 'Number of PCs'
- When Robust PCA is selected, confirm the projection maximizes a robust measure of variance of the projected data (Projection-Pursuit)
- Confirm PC scores exist for every object in the input table (respecting the 'Fraction of data to display' setting)

## Limitations

- Standard PCA is sensitive to outliers and non-Gaussian structure; Robust PCA (Projection-Pursuit) is offered as the alternative
- MIC correlation method cannot handle NA values in the data
- PCA is computed on the data correlation matrix, so variable scaling choices affect the decomposition

## Evidence
