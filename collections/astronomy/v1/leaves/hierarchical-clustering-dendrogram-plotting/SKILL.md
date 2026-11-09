---
name: hierarchical-clustering-dendrogram-plotting
description: Use when you need a tree representation of hidden structure among variables
  of a high-dimensional astronomical dataset (e.g. SDSS supernova hosts, Millennium
  mock catalogs, ZENS groups) and an optimal cluster count without pre-specifying
  it.
when_to_use_negative:
- When the analysis goal is PCA variance decomposition rather than partition hierarchy
  — use the Standard/Robust PCA tab (Projection-Pursuit per Croux et al. 2007) instead
- When pairwise relationship strength matters more than tree structure — the qgraph
  (Spring/Circular layout) or circlize chord diagram tabs represent correlation thickness
  directly
- When the number of clusters must be fixed a priori — hierarchical clustering here
  relies on the agglomerative merging plus Calinski-Harabasz guidance rather than
  a user-specified k
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ape
  role: R package used to plot the variable dendrogram from the agglomerative clustering
    output
- name: fpc
  role: provides the Calinski-Harabasz optimal-number-of-clusters index guiding dendrogram
    color-coding
- name: shiny
  role: hosts the AMADA web app whose left-panel Dendrogram type option (Phylogram/Cladogram/Fan)
    drives the plot
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip
- name: AMADA
  role: R package performing the hierarchical clustering on correlation matrices for
    different choices
  repo: https://github.com/RafaelSdeSouza/AMADA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:258d8f9c839d01be7766d4ea20f0370b06ecd5e8de5139c5ba6e9f2b8fea2d93
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
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hierarchical-clustering-dendrogram-plotting

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Renders the variable dendrogram from agglomerative hierarchical clustering of astronomical parameters, using the correlation distance d(x,y)=1-|corr(x,y)| within AMADA's Shiny interface. Trees are drawn with the ape R package in Phylogram, Cladogram, or Fan style, with clusters color-coded by the Calinski-Harabasz optimal partition.

## When to use

Use when you need a tree representation of hidden structure among variables of a high-dimensional astronomical dataset (e.g. SDSS supernova hosts, Millennium mock catalogs, ZENS groups) and an optimal cluster count without pre-specifying it.

## When NOT to use

- When the analysis goal is PCA variance decomposition rather than partition hierarchy — use the Standard/Robust PCA tab (Projection-Pursuit per Croux et al. 2007) instead
- When pairwise relationship strength matters more than tree structure — the qgraph (Spring/Circular layout) or circlize chord diagram tabs represent correlation thickness directly
- When the number of clusters must be fixed a priori — hierarchical clustering here relies on the agglomerative merging plus Calinski-Harabasz guidance rather than a user-specified k

## Inputs

- CSV/TXT data table with named columns separated by spaces; missing data marked NA
- Available catalogs: Supernova host properties (Sako et al. 2014, Table 1), Mock galaxy catalog (Guo et al. 2011), ZENS catalog (141 groups, 0.05 < z < 0.0585)
- Pairwise correlation matrix (Pearson, Spearman, or MIC) converted to correlation distance 1-|corr(x,y)|

## Outputs

- Dendrogram figure (Phylogram, Cladogram, or Fan style) with Calinski-Harabasz cluster groups color-coded
- Optimal number of clusters displayed to the user via Calinski and Harabasz (1972) index

## How to apply

Import the dataset as CSV/TXT (columns named, space-separated, missing values marked NA) into the AMADA Shiny app, then compute pairwise correlations (Pearson, Spearman, or MIC) and convert them to the correlation distance d(x,y)=1-|corr(x,y)|. Run agglomerative clustering, which starts with one cluster per object and progressively merges the two closest clusters; display the optimal number of clusters guided by the Calinski-Harabasz (1972) index. Plot the resulting hierarchy with ape, selecting the Dendrogram type option among Phylogram, Cladogram, or Fan, and color-code branches by the Calinski-Harabasz cluster groups. Use the left-panel controls, set the desired combination, and click 'Make it so' to update the plot. The dendrogram complements the heatmap, distogram, qgraph, chord diagram, and Nightingale-chart tabs, all switchable in the interface.

## Related tools

- **ape** (R package used to plot the variable dendrogram from the agglomerative clustering output)
- **fpc** (provides the Calinski-Harabasz optimal-number-of-clusters index guiding dendrogram color-coding)
- **shiny** (hosts the AMADA web app whose left-panel Dendrogram type option (Phylogram/Cladogram/Fan) drives the plot) — https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip
- **AMADA** (R package performing the hierarchical clustering on correlation matrices for different choices) — https://github.com/RafaelSdeSouza/AMADA

## Examples

```
require(shiny); runUrl('https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip')
```

## Evaluation signals

- Dendrogram figure file is produced, non-empty, and readable as an image matching the AMADA diagnostic
- Tree leaves correspond one-to-one to the imported dataset columns (arbitrary number of columns allowed)
- Branch color-coding partitions variables into the Calinski-Harabasz optimal number of clusters
- Selected tree style (Phylogram, Cladogram, or Fan) matches the left-panel option after clicking 'Make it so'
- Clustering is reproducibly derived from the correlation distance d(x,y)=1-|corr(x,y)| for the chosen correlation method

## Limitations

- Hierarchical clustering implicitly assumes a similarity measure between object pairs; here that is fixed to the correlation distance 1-|corr(x,y)|, so results depend on the chosen correlation method (Pearson/Spearman/MIC)
- No quantitative acceptance threshold for the dendrogram is reported; acceptance is successful rendering of the diagnostic
- The optimal number of clusters is only displayed as guidance (Calinski-Harabasz), not enforced or validated statistically

## Evidence
