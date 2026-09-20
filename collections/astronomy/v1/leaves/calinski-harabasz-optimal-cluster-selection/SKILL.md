---
name: calinski-harabasz-optimal-cluster-selection
description: Use when a hierarchical clustering of variables has been built from a
  correlation distance and you need a data-driven, a-priori-free choice of how many
  clusters to report and color-code.
when_to_use_negative:
- The data contain NaNs and the Maximum Information Coefficient (MIC) correlation
  method is selected, since MIC does not support NA values — fall back to Pearson
  or Spearman before clustering
- The analysis goal is partitioning objects rather than building a hierarchy of variable
  partitions; AMADA's hierarchical approach targets multi-level structure revealed
  across the dendrogram, not a single fixed-k partition
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: fpc
  role: Provides the Calinski-Harabasz index used to determine the optimal number
    of clusters from the dendrogram cuts
- name: ape
  role: Renders the dendrogram (Phylogram, Cladogram, or Fan) with the Calinski-Harabasz-optimal
    clusters color-coded
- name: AMADA
  role: Shiny app that runs the hierarchical clustering pipeline and displays the
    Calinski-Harabasz-optimal number of clusters to guide the user
  repo: https://github.com/RafaelSdeSouza/AMADA
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:a17032fcff7144cf8834a278a31dd8fea6c2bcf6652218b1674d55c06d0b4a37
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# calinski-harabasz-optimal-cluster-selection

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Determines the optimal number of clusters for AMADA's agglomerative hierarchical clustering of astronomical variables by maximizing the Calinski-Harabasz index (fpc package), computed over partitions produced with the correlation distance d(x,y) = 1 - |corr(x,y)|.

## When to use

Use when a hierarchical clustering of variables has been built from a correlation distance and you need a data-driven, a-priori-free choice of how many clusters to report and color-code.

## When NOT to use

- The data contain NaNs and the Maximum Information Coefficient (MIC) correlation method is selected, since MIC does not support NA values — fall back to Pearson or Spearman before clustering
- The analysis goal is partitioning objects rather than building a hierarchy of variable partitions; AMADA's hierarchical approach targets multi-level structure revealed across the dendrogram, not a single fixed-k partition

## Inputs

- Correlation matrix among host-galaxy variables (Pearson, Spearman, or MIC) from an AMADA-imported CSV/TXT dataset with space-separated, named columns and NA-marked missing entries
- Agglomerative hierarchical clustering tree (hclust object) built from the correlation distance d(x,y) = 1 - |corr(x,y)|

## Outputs

- Optimal number of clusters (integer) selected by maximum Calinski-Harabasz index
- Cluster assignment for every variable/object in the input table (cluster_assignments.csv)
- Dendrogram plot with Calinski-Harabasz-optimal cluster groups color-coded (dendrogram.png)

## How to apply

After converting the Pearson (or Spearman/MIC) correlation matrix to the distance d(x,y) = 1 - |corr(x,y)|, run agglomerative clustering starting with one cluster per object and progressively merging the two closest clusters. Rather than fixing k a priori, evaluate candidate partitions of the dendrogram and compute the Calinski-Harabasz index with the fpc package, selecting the k that maximizes it. AMADA uses this index specifically 'to guide the user' by displaying an optimal number of clusters alongside the dendrogram. The selected k is then used to color-code cluster groups in the ape-rendered dendrogram (Phylogram, Cladogram, or Fan style) and to emit per-variable cluster assignments. Rationale: hierarchical clustering 'implicitly assumes a measure of similarity between pairs of objects,' and the CH index provides an internal validation of the resulting hierarchy without requiring prior specification of the number of clusters.

## Related tools

- **fpc** (Provides the Calinski-Harabasz index used to determine the optimal number of clusters from the dendrogram cuts)
- **ape** (Renders the dendrogram (Phylogram, Cladogram, or Fan) with the Calinski-Harabasz-optimal clusters color-coded)
- **AMADA** (Shiny app that runs the hierarchical clustering pipeline and displays the Calinski-Harabasz-optimal number of clusters to guide the user) — https://github.com/RafaelSdeSouza/AMADA

## Examples

```
d <- as.dist(1-abs(cor(X, use="pairwise"))); hc <- hclust(d); library(fpc); ks <- sapply(2:10, function(k) calinhara(X, cutree(hc, k))); k_opt <- which.max(ks) + 1
```

## Evaluation signals

- The reported optimal k is the argmax of the Calinski-Harabasz index computed with fpc over candidate dendrogram cuts (2..n), not a user-supplied constant
- Every variable in the input correlation matrix receives exactly one cluster assignment, and the assignment count equals the reported optimal k
- The dendrogram file exists, renders the full hierarchical merge structure (from single-object clusters up to one cluster), and displays the optimal grouping color-coded
- The distance matrix used for clustering reproduces d(x,y) = 1 - |corr(x,y)| from the selected correlation method (Pearson, Spearman, or MIC)
- The clustering pipeline starts from one cluster per object and merges the two closest clusters iteratively, consistent with an agglomerative (bottom-up) linkage

## Limitations

- The choice among Pearson, Spearman, or MIC correlations changes the input distance matrix and hence the optimal k; MIC cannot be used when the dataset contains NA values
- The Calinski-Harabasz number of clusters is displayed 'to guide the user' — it is an internal-validation heuristic, not a guarantee that the hierarchy's structure is scientifically meaningful
- The method implicitly assumes the correlation distance 1 - |corr(x,y)| is an appropriate similarity measure between pairs of objects; datasets where correlation is not meaningful (e.g., very few rows, heavy missingness marked NA) are poor inputs

## Evidence
