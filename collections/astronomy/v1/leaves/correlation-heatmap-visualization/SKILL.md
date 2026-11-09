---
name: correlation-heatmap-visualization
description: Use when you need a pairwise association overview (variable-by-variable
  correlation structure) of a high-dimensional dataset of named numeric columns, with
  clustering-based ordering and optional dendrograms.
when_to_use_negative:
- The goal is pairwise relationship geometry (edge-weighted graph or chord ribbons),
  for which AMADA's qgraph and circlize diagnostics are the intended views instead
  of the heatmap
- The input is raw, unaggregated time-series or image data rather than a table of
  per-object numeric parameters
- The data have not been imported/validated in the 'Import Dataset' panel (e.g., columns
  unnamed, separators not spaces, missing values not marked NA)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2269
tools:
- name: pheatmap
  role: R package that renders the clustered, color-coded correlation heatmap with
    edge dendrograms
- name: AMADA
  role: Shiny application providing the tabbed interface, dataset import, and correlation-method
    controls that drive the heatmap
  repo: github.com/RafaelSdeSouza/AMADA
- name: shiny
  role: Runs the AMADA web app in which the heatmap tab is displayed and controlled
- name: fpc
  role: Supports the Calinski-Harabasz optimal-number-of-clusters guidance that orders/colors
    the clustered heatmap view
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:8f680be0f32f057a40e7859993b35c26527604e305cc197d60386c16855f4fdf
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
  - pheatmap
  - shiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# correlation-heatmap-visualization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Render a color-coded correlation heatmap of a multidimensional astronomical dataset in AMADA, arranging rows and columns by hierarchical clustering with dendrograms at the matrix edges, using the pheatmap R package inside the Shiny interface.

## When to use

Use when you need a pairwise association overview (variable-by-variable correlation structure) of a high-dimensional dataset of named numeric columns, with clustering-based ordering and optional dendrograms.

## When NOT to use

- The goal is pairwise relationship geometry (edge-weighted graph or chord ribbons), for which AMADA's qgraph and circlize diagnostics are the intended views instead of the heatmap
- The input is raw, unaggregated time-series or image data rather than a table of per-object numeric parameters
- The data have not been imported/validated in the 'Import Dataset' panel (e.g., columns unnamed, separators not spaces, missing values not marked NA)

## Inputs

- CSV/TXT data table with named, space-separated numeric columns; NA marks missing data
- Choice of correlation method (Pearson, Spearman, or MIC)
- Bundled example datasets: Supernova host properties (Sako et al. 2014, SDSS Type Ia and II hosts), mock galaxy catalog (Guo et al. 2011), ZENS catalog (141 galaxy groups, 0.05 < z < 0.0585)

## Outputs

- Color-coded correlation heatmap image with rows/columns ordered by hierarchical clustering and edge dendrograms
- Optional in-cell correlation coefficient labels when 'Display numbers' is enabled
- Correlation distance matrix d(x,y)=1-|corr(x,y)| underlying the clustered ordering

## How to apply

Load a numeric table (CSV/TXT, columns named and space-separated, missing values marked NA) into the AMADA Shiny app via the 'Import Dataset' panel and verify the import at the bottom of that panel. Choose a correlation method from the Control Options — Pearson, Spearman, or Maximum Information Coefficient (MIC) — and click 'Make it so' to recompute the matrix. Render the heatmap with pheatmap, color-coding the correlation level of each variable pair and arranging rows and columns by the hierarchical clustering built on the correlation distance d(x,y)=1-|corr(x,y)|, so dendrograms are drawn at the matrix edges. Optionally toggle 'Display numbers' to print the correlation coefficients inside the heatmap cells. The clustering side uses the agglomerative approach with an optimal number of clusters suggested via Calinski and Harabasz (1972), which also colors the companion dendrogram diagnostic.

## Related tools

- **pheatmap** (R package that renders the clustered, color-coded correlation heatmap with edge dendrograms)
- **AMADA** (Shiny application providing the tabbed interface, dataset import, and correlation-method controls that drive the heatmap) — github.com/RafaelSdeSouza/AMADA
- **shiny** (Runs the AMADA web app in which the heatmap tab is displayed and controlled)
- **fpc** (Supports the Calinski-Harabasz optimal-number-of-clusters guidance that orders/colors the clustered heatmap view)

## Examples

```
install.packages('pheatmap',dependencies=TRUE); require(shiny); runUrl('https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip')
```

## Evaluation signals

- The 'Import Dataset' panel confirms the CSV/TXT table was properly imported and the 'Dataset' tab renders the same rows/columns
- The heatmap is rendered after clicking 'Make it so' and reflects the selected correlation method (Pearson, Spearman, or MIC)
- Rows and columns are reordered consistently with hierarchical clustering on d(x,y)=1-|corr(x,y)|, with dendrograms drawn at the matrix edges
- With 'Display numbers' on, the printed coefficients match the color coding of each cell
- The workflow's validation condition holds: the heatmap figure file exists and is a non-empty, readable image consistent with the described diagnostic

## Limitations

- The paper reports no quantitative acceptance threshold for the heatmap; acceptance is the successful rendering of the diagnostic from the shared input
- The dendrogram/cluster ordering assumes the correlation distance d(x,y)=1-|corr(x,y)| as the similarity measure; other distance choices are not exposed
- Data must be fully named numeric columns; missing entries must be marked NA, and arbitrarily structured files are not supported

## Evidence
