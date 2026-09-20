---
name: phylogenetic-style-dendrogram-visualization
description: Use when a correlation matrix (Pearson, Spearman, or MIC) has been converted
  to correlation distance d(x,y)=1-|corr(x,y)| and you must visualize the hierarchy
  of merged clusters as a color-coded dendrogram in one of the AMADA styles.
when_to_use_negative:
- The input is a raw CSV/TXT dataset that has not yet been converted to a correlation
  matrix — first run the correlation stage (Pearson, Spearman, or MIC), then cluster.
- The analysis goal is dimensionality reduction or projection rather than hierarchy
  discovery — use the PCA tab (Standard or Robust Projection–Pursuit PCA) instead
  of dendrogram visualization.
- A prior specification of the number of clusters is required by the downstream analysis
  — hierarchical clustering's stated advantage is the needless of a prior number-of-clusters
  specification.
edam_operation: http://edamontology.org/operation_0315
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: ape
  role: R package used to draw the hierarchical-merge structure as a Phylogram, Cladogram,
    or Fan dendrogram
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: fpc
  role: Provides the Calinski–Harabasz (1972) index used to display the optimal number
    of clusters guiding the color-coding
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: AMADA
  role: Shiny application whose left-panel 'Dendrogram type' option (Phylogram, Cladogram,
    Fan) and 'Make it so' button drive the visualization
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: shiny
  role: Runtime for the AMADA Shiny App launched via runUrl from the AMADA_shiny repository
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:610558e3478a7644312eaf0cf829c7ece2c7b5f1cf0b7dc00e83f7390e324d30
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/RafaelSdeSouza/AMADA
  tools:
  - AMADA
  - R
  - ape
  - fpc
  - shiny
  edam_operation: http://edamontology.org/operation_0315
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# phylogenetic-style-dendrogram-visualization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Render the results of agglomerative hierarchical clustering over correlation-distance matrices as phylogenetic-style dendrograms (Phylogram, Cladogram, or Fan) using the ape package in AMADA, color-coding clusters at the Calinski–Harabasz-optimal number of clusters.

## When to use

Use when a correlation matrix (Pearson, Spearman, or MIC) has been converted to correlation distance d(x,y)=1-|corr(x,y)| and you must visualize the hierarchy of merged clusters as a color-coded dendrogram in one of the AMADA styles.

## When NOT to use

- The input is a raw CSV/TXT dataset that has not yet been converted to a correlation matrix — first run the correlation stage (Pearson, Spearman, or MIC), then cluster.
- The analysis goal is dimensionality reduction or projection rather than hierarchy discovery — use the PCA tab (Standard or Robust Projection–Pursuit PCA) instead of dendrogram visualization.
- A prior specification of the number of clusters is required by the downstream analysis — hierarchical clustering's stated advantage is the needless of a prior number-of-clusters specification.

## Inputs

- Correlation matrix computed with the selected method (Pearson, Spearman, or MIC)
- Distance matrix derived as d(x,y) = 1 - |corr(x,y)|
- Dendrogram type option: Phylogram, Cladogram, or Fan
- Optimal number of clusters from the Calinski–Harabasz index (fpc)

## Outputs

- Dendrogram plot (Phylogram, Cladogram, or Fan style) rendered with the ape package
- Cluster assignments per variable color-coded on the dendrogram at the Calinski–Harabasz-optimal k

## How to apply

Take the correlation matrix produced by the correlation stage (Pearson, Spearman, or Maximum Information Coefficient, selectable in the AMADA Shiny 'Correlation method' option) and convert it to a distance matrix using the correlation distance d(x,y)=1-|corr(x,y)|, which is the similarity measure AMADA assumes for clustering. Run agglomerative hierarchical clustering: start with a single cluster per object and progressively merge the two closest clusters until one cluster remains. Guide the user by displaying an optimal number of clusters via the Calinski and Harabasz (1972) criterion implemented in the fpc package. Plot the dendrogram with the ape package in the chosen 'Dendrogram type' — Phylogram, Cladogram, or Fan — color-coding the variables by their cluster assignment, and update the plot after clicking the 'Make it so' button.

## Related tools

- **ape** (R package used to draw the hierarchical-merge structure as a Phylogram, Cladogram, or Fan dendrogram) — https://github.com/RafaelSdeSouza/AMADA
- **fpc** (Provides the Calinski–Harabasz (1972) index used to display the optimal number of clusters guiding the color-coding) — https://github.com/RafaelSdeSouza/AMADA
- **AMADA** (Shiny application whose left-panel 'Dendrogram type' option (Phylogram, Cladogram, Fan) and 'Make it so' button drive the visualization) — https://github.com/RafaelSdeSouza/AMADA
- **shiny** (Runtime for the AMADA Shiny App launched via runUrl from the AMADA_shiny repository) — https://github.com/RafaelSdeSouza/AMADA_shiny

## Examples

```
install.packages('ape',dependencies=TRUE); install.packages('fpc',dependencies=TRUE); require(shiny); runUrl('https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip')
```

## Evaluation signals

- The dendrogram renders a hierarchy of partitions in which clusters at each level are created by merging clusters at the next lower level (agglomerative merge structure is visible).
- The displayed number of clusters matches the optimum reported by the Calinski–Harabasz index from fpc.
- Variables in the dendrogram are color-coded consistently with their cluster membership under the chosen correlation method (Pearson, Spearman, or MIC).
- Switching the 'Dendrogram type' option among Phylogram, Cladogram, and Fan re-renders the same hierarchical structure in the requested style after 'Make it so'.
- The clustering outcome is reproducible from the same correlation-distance matrix d(x,y)=1-|corr(x,y)|, independent of the visualization style chosen.

## Limitations

- The method implicitly assumes a measure of similarity between pairs of objects; results depend on the choice of correlation method (Pearson, Spearman, MIC) selected in the control panel.
- The number of clusters shown is only a guide via Calinski–Harabasz; the user is not locked to that partitioning.
- The AMADA_shiny README is truncated in the source material, so exact plotting function signatures are not documented here.

## Evidence
