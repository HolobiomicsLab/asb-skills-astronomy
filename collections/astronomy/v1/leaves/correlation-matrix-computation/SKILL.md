---
name: correlation-matrix-computation
description: Use when exploring pairwise relationships among numeric columns of a
  tabular astronomical catalog (CSV/TXT with named, space-separated columns) before
  clustering variables or running PCA, especially with possible NA entries.
when_to_use_negative:
- The input is not a tabular set of numeric variables (e.g., raw spectra, images,
  or free-text metadata) rather than a named-column catalog.
- MIC correlation is desired but the data still contain NA entries, since MIC does
  not support NA values.
- The goal is supervised prediction or classification of objects rather than unsupervised
  exploration of variable structure.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: AMADA
  role: Shiny web app exposing the correlation-method selector (Pearson/Spearman/MIC)
    and rendering the correlation heatmap as one of its data-mining diagnostics
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: corrplot
  role: R library used to visualize the computed correlation matrix/heatmap with optional
    numeric coefficients
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: fpc
  role: R library providing the Calinski-Harabasz index used to pick the optimal number
    of clusters on the correlation-distance dendrogram
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: ape
  role: R library used to draw the dendrogram (Phylogram, Cladogram, or Fan) built
    from the correlation distance
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: shiny
  role: Runs the AMADA Shiny app via runUrl of the AMADA_shiny repository
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:0d3b9b8596e3dcc2a1562478e00b3e4776228d84a7bd7bb33074b1a751d16f49
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
  - shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# correlation-matrix-computation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes a pairwise correlation matrix among the numeric variables of a high-dimensional astronomical dataset (e.g., SDSS Type Ia and II supernova host-galaxy properties), selectable among Pearson, Spearman, or Maximum Information Coefficient (MIC), as the basis for AMADA's hierarchical clustering of variables and correlation-matrix-diagonalization PCA.

## When to use

Use when exploring pairwise relationships among numeric columns of a tabular astronomical catalog (CSV/TXT with named, space-separated columns) before clustering variables or running PCA, especially with possible NA entries.

## When NOT to use

- The input is not a tabular set of numeric variables (e.g., raw spectra, images, or free-text metadata) rather than a named-column catalog.
- MIC correlation is desired but the data still contain NA entries, since MIC does not support NA values.
- The goal is supervised prediction or classification of objects rather than unsupervised exploration of variable structure.

## Inputs

- CSV/TXT dataset with named, space-separated columns and arbitrary rows (e.g., SDSS supernova host-galaxy properties from Table 1, Sako et al. 2014)
- Missing values encoded as NA
- User-selected correlation method (Pearson, Spearman, or MIC) and display options

## Outputs

- Pairwise correlation matrix among host-galaxy variables
- Correlation heatmap with optional displayed coefficients (corrplot)
- Correlation-distance matrix d(x,y)=1-|corr(x,y)| feeding hierarchical clustering and the Calinski-Harabasz optimal cluster count
- Correlation-matrix eigenvectors (PCs) and eigenvalues (variance explained) for standard PCA

## How to apply

Import the dataset in CSV/TXT format with named, space-separated columns, marking any missing entries as NA. In the AMADA Shiny control panel choose the correlation method among Pearson (default), Spearman, or Maximum Information Coefficient (MIC); note that MIC does not support NA values, so it requires complete columns or prior NA handling. Optionally toggle 'Display numbers' so the correlation coefficients are shown in the heatmap rendered via corrplot. The resulting matrix feeds two downstream analyses: hierarchical agglomerative clustering of variables using the correlation distance d(x,y)=1-|corr(x,y)|, with the optimal number of clusters guided by the Calinski-Harabasz index (fpc package) and dendrograms drawn with ape; and standard PCA computed by diagonalization of the data correlation matrix, whose eigenvectors are the PCs and eigenvalues the variance explained. Once the combination of correlation method and PCA method (Standard or Robust PCA via Projection-Pursuit) is chosen, click 'Make it so' to update the plots.

## Related tools

- **AMADA** (Shiny web app exposing the correlation-method selector (Pearson/Spearman/MIC) and rendering the correlation heatmap as one of its data-mining diagnostics) — https://github.com/RafaelSdeSouza/AMADA
- **corrplot** (R library used to visualize the computed correlation matrix/heatmap with optional numeric coefficients) — https://github.com/RafaelSdeSouza/AMADA
- **fpc** (R library providing the Calinski-Harabasz index used to pick the optimal number of clusters on the correlation-distance dendrogram) — https://github.com/RafaelSdeSouza/AMADA
- **ape** (R library used to draw the dendrogram (Phylogram, Cladogram, or Fan) built from the correlation distance) — https://github.com/RafaelSdeSouza/AMADA
- **shiny** (Runs the AMADA Shiny app via runUrl of the AMADA_shiny repository) — https://github.com/RafaelSdeSouza/AMADA_shiny

## Examples

```
install.packages('corrplot',dependencies=TRUE); install.packages('fpc',dependencies=TRUE); require(shiny); runUrl('https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip')
```

## Evaluation signals

- The correlation matrix is symmetric with unit diagonal, and its dimensions match the number of named numeric columns in the input CSV/TXT file.
- Rows with missing entries are handled as NA, and MIC is only used when the selected columns contain no NA values.
- The variable clustering uses the correlation distance d(x,y)=1-|corr(x,y)|, so the dendrogram merges reflect absolute correlations, and an optimal cluster count from the Calinski-Harabasz index is displayed.
- The heatmap optionally shows the numeric correlation coefficients when the 'Display numbers' option is enabled.
- Standard PCA eigenvectors/eigenvalues derive from diagonalizing the same data correlation matrix, and PC1 corresponds to the largest eigenvalue.

## Limitations

- MIC does not support NA values, so datasets with missing data must use Pearson or Spearman, or the missing entries must be resolved first.
- Input must be CSV/TXT with named, space-separated columns and missing values marked NA; other formats are not accepted by the Import Dataset panel.
- Pearson/Spearman/MIC choices change both the heatmap and the downstream clustering distance, so results are method-dependent; users should verify data were properly imported via the 'Import Dataset' panel and the 'Dataset' tab.

## Evidence
