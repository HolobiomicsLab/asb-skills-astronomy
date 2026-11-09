---
name: pca-loading-nightingale-chart-rendering
description: Use when a user has a multivariate dataset (e.g. astronomical catalogs
  like SDSS supernova host properties, mock galaxy catalogs, or ZENS) and needs a
  visual read-out of which variables drive each principal component.
when_to_use_negative:
- The analysis goal is clustering/partition structure rather than PC loading interpretation
  — use the dendrogram or number-of-clusters (Calinski-Harabasz) controls instead
- The goal is pairwise correlation structure between variables — use the heatmap,
  distogram, chord diagram, or correlation graph diagnostics
- Data are not a rectangular numeric table (CSV/TXT with named columns); AMADA cannot
  import them directly
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: AMADA
  role: Shiny web application whose tabs expose the nightingale chart rendering of
    PCA loadings
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip
- name: shiny
  role: Runs the AMADA app in which the nightingale charts are displayed and controlled
    via the left-panel options
  repo: http://www.r-project.org
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:edabad87c485820324fcdd0890e88dfcbb20a818511424e79b565d125e4c684e
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - AMADA
  - R
  - ape
  - circlize
  - corrplot
  - fpc
  - shiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# pca-loading-nightingale-chart-rendering

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Render AMADA's nightingale (polar barplot) charts of PCA loadings, where each strip length represents the relative contribution of each variable to the i-th principal component, as one of the tabbed visualization data-mining diagnostics of the AMADA Shiny app.

## When to use

Use when a user has a multivariate dataset (e.g. astronomical catalogs like SDSS supernova host properties, mock galaxy catalogs, or ZENS) and needs a visual read-out of which variables drive each principal component.

## When NOT to use

- The analysis goal is clustering/partition structure rather than PC loading interpretation — use the dendrogram or number-of-clusters (Calinski-Harabasz) controls instead
- The goal is pairwise correlation structure between variables — use the heatmap, distogram, chord diagram, or correlation graph diagnostics
- Data are not a rectangular numeric table (CSV/TXT with named columns); AMADA cannot import them directly

## Inputs

- CSV/TXT data table with named columns separated by spaces, missing data marked NA
- Choice of PCA method (Standard PCA or Robust PCA / Projection–Pursuit)
- Number of principal components to display as Nightingale charts

## Outputs

- Nightingale (polar barplot) charts of variable loadings per selected principal component
- Non-empty readable image files of the nightingale diagnostics rendered in the Shiny app

## How to apply

Import the dataset as CSV/TXT (named columns, space-separated, missing values marked NA) into the AMADA Shiny app. Choose the PCA method — Standard PCA (eigenvector/eigenvalue diagonalization of the correlation matrix, as in Jolliffe 2002) or Robust PCA using the Projection–Pursuit principle (Croux, Filzmoser and Oliveira, 2007) — and set 'Number of PCs' to select how many principal components are displayed as Nightingale charts. Apply any other control-panel options (fraction of data to display, correlation method among Pearson, Spearman or MIC) and click the 'Make it so' button to update the plots. The nightingale chart renders each selected PC as a polar barplot in which strip length encodes each variable's relative loading contribution, letting the user interpret high-dimensional structure alongside the heatmap, distogram, dendrogram, graph, and chord-diagram tabs.

## Related tools

- **AMADA** (Shiny web application whose tabs expose the nightingale chart rendering of PCA loadings) — https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip
- **shiny** (Runs the AMADA app in which the nightingale charts are displayed and controlled via the left-panel options) — http://www.r-project.org

## Examples

```
require(shiny); runUrl('https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip')
```

## Evaluation signals

- All six figure files (heatmap, distogram, dendrogram, graph, chord diagram, nightingale chart) exist and are non-empty, readable image files
- Nightingale chart content matches the described diagnostic: strip length per variable represents its relative contribution to the i-th PC
- Number of rendered nightingale charts equals the 'Number of PCs' selected in the control panel
- Charts update only after the 'Make it so' button is clicked, reflecting the chosen Standard vs Robust PCA method
- Input data were verified as properly imported via the 'Import Dataset' panel / 'Dataset' tab before rendering

## Limitations

- No quantitative acceptance threshold for the rendering is reported in the paper — acceptance is the successful rendering of all six diagnostics
- Robust PCA relies on the Projection–Pursuit principle (Croux, Filzmoser and Oliveira, 2007), which differs from standard eigen-decomposition and may give different loadings
- Missing data must be marked NA before import; improperly imported data will corrupt the diagnostics
- No discussion-section content was present in the document, so further caveats are undocumented

## Evidence
