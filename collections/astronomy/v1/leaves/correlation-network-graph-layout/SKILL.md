---
name: correlation-network-graph-layout
description: Use when you have a parameter-by-parameter correlation matrix (Pearson,
  Spearman, or MIC) for a multidimensional dataset and want to inspect variable association
  structure as a weighted network graph.
when_to_use_negative:
- The goal is a clustered heatmap or dendrogram of the same correlations rather than
  a network representation — use the pheatmap/ape diagnostics instead
- Relationships between variables are best shown as ribbons around a circular layout
  (chord diagram) rather than a vertex-edge graph
- The data are not a parameter-by-variable table that supports pairwise correlation
  computation
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: AMADA
  role: Shiny application exposing the graph diagnostic; user selects graph layout
    (Spring or Circular) from the control panel and updates plots with 'Make it so'
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: qgraph
  role: R library used to render the clustered correlation graph with vertices as
    parameters and edge thickness weighted by pairwise correlation
- name: fpc
  role: Provides the Calinski-Harabasz optimal number of clusters that guides cluster
    structure in the network
- name: shiny
  role: Runs the AMADA Shiny app that hosts the interactive graph diagnostic
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:fa0b81a497ab6ab3a067d37992659b3e3b3cc10f1b69e1a544e51969a357f8a5
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/RafaelSdeSouza/AMADA
  tools:
  - AMADA
  - R
  - ape
  - circlize
  - corrplot
  - fpc
  - qgraph
  - shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Correlation network graph layout

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Builds a clustered correlation graph of dataset parameters where vertices are variables and edge thickness is weighted by the pairwise correlation, rendered with qgraph under a Spring or Circular layout as one of AMADA's visualization data-mining diagnostics.

## When to use

Use when you have a parameter-by-parameter correlation matrix (Pearson, Spearman, or MIC) for a multidimensional dataset and want to inspect variable association structure as a weighted network graph.

## When NOT to use

- The goal is a clustered heatmap or dendrogram of the same correlations rather than a network representation — use the pheatmap/ape diagnostics instead
- Relationships between variables are best shown as ribbons around a circular layout (chord diagram) rather than a vertex-edge graph
- The data are not a parameter-by-variable table that supports pairwise correlation computation

## Inputs

- Dataset as CSV/TXT file with named columns, space-separated, missing data marked NA (e.g. Supernova host properties from Sako et al. 2014, mock galaxy catalog from Guo et al. 2011, or ZENS catalog)
- Pairwise correlation matrix (Pearson, Spearman, or Maximum Information Coefficient/MIC)
- Hierarchical clustering output with the Calinski-Harabasz optimal number of clusters

## Outputs

- Correlation network graph image (vertices = parameters, edge thickness weighted by pairwise correlation, Spring or Circular layout)
- Interactive plot displayed in the Shiny app under the graph diagnostics tab

## How to apply

Compute the pairwise correlation matrix for the imported dataset (CSV/TXT, columns named and space-separated, missing values as NA) using the user-chosen correlation method. Build a graph whose vertices are the dataset parameters and whose edge thickness is weighted by the pairwise correlation strength. Render the graph with qgraph, choosing the layout via the AMADA control panel: Spring or Circular; the layout choice is applied after clicking 'Make it so' to update the plots. Optionally combine with the hierarchical clustering structure, where clusters are guided by the Calinski-Harabasz optimal number of clusters computed on the correlation distance d(x,y)=1-|corr(x,y)|.

## Related tools

- **AMADA** (Shiny application exposing the graph diagnostic; user selects graph layout (Spring or Circular) from the control panel and updates plots with 'Make it so') — https://github.com/RafaelSdeSouza/AMADA
- **qgraph** (R library used to render the clustered correlation graph with vertices as parameters and edge thickness weighted by pairwise correlation)
- **fpc** (Provides the Calinski-Harabasz optimal number of clusters that guides cluster structure in the network)
- **shiny** (Runs the AMADA Shiny app that hosts the interactive graph diagnostic) — https://github.com/RafaelSdeSouza/AMADA_shiny

## Examples

```
install.packages('qgraph',dependencies=TRUE); require(shiny); runUrl('https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip')
```

## Evaluation signals

- The rendered graph opens in the Shiny app after selecting a layout (Spring or Circular) and clicking 'Make it so'
- Each vertex in the graph corresponds to a column/parameter of the imported dataset
- Edge thickness visually varies with the magnitude of the pairwise correlation between the connected parameters
- Changing the layout option between Spring and Circular produces visibly different, valid arrangements of the same graph
- Changing the correlation method (Pearson, Spearman, MIC) in the control panel changes the resulting edge weights and the displayed graph

## Limitations

- The layout choice is limited to Spring or Circular; no other graph layouts are offered by the AMADA control panel
- No quantitative acceptance threshold for the graph diagnostic is reported in the paper — acceptance is the successful rendering of the diagnostic from the shared input
- Data must be imported as CSV/TXT with named, space-separated columns and NA for missing values; improperly formatted input must be checked in the 'Import Dataset' panel before rendering
- The paper offers no discussion-section caveats; nothing beyond these is documented

## Evidence
