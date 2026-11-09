---
name: agglomerative-hierarchical-clustering-of-parameters
description: Use when a correlation matrix (Pearson, Spearman, or MIC) of parameters
  has been computed and you need to reveal multi-level structure among variables via
  agglomerative hierarchical clustering without pre-specifying the cluster count.
when_to_use_negative:
- The goal is a partitioning method that requires spherical, flat clusters with a
  pre-specified k rather than a hierarchy of partitions revealing structure at multiple
  levels of association
- The input is the raw data table rather than a correlation matrix — the clustering
  stage consumes the correlation matrix computed upstream, not the original parameter
  columns
- The data contain missing entries that have not been encoded as NA per the AMADA
  import format (CSV/TXT, named, space-separated columns)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0092
tools:
- name: AMADA
  role: Shiny-based pipeline that performs the hierarchical clustering analysis for
    different choices of correlation matrices and renders the dendrograms
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: fpc
  role: Provides the Calinski-Harabasz index used to display the optimal number of
    clusters
- name: ape
  role: Used to plot the dendrogram in Phylogram, Cladogram, or Fan style with color-coded
    clusters
- name: shiny
  role: Runs the AMADA_shiny app in which the clustering tab and dendrogram options
    are exposed to the user
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:c29fc9c79efd2452d0b9d561a801d5068a49a3b5762a15c5b6940b43001ba7ae
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# agglomerative-hierarchical-clustering-of-parameters

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Agglomerative hierarchical clustering of dataset variables using correlation distance d(x,y) = 1 - |corr(x,y)|, repeated for each chosen correlation matrix (Pearson, Spearman, or MIC), with the optimal number of clusters guided by the Calinski-Harabasz index and dendrograms rendered with the ape package.

## When to use

Use when a correlation matrix (Pearson, Spearman, or MIC) of parameters has been computed and you need to reveal multi-level structure among variables via agglomerative hierarchical clustering without pre-specifying the cluster count.

## When NOT to use

- The goal is a partitioning method that requires spherical, flat clusters with a pre-specified k rather than a hierarchy of partitions revealing structure at multiple levels of association
- The input is the raw data table rather than a correlation matrix — the clustering stage consumes the correlation matrix computed upstream, not the original parameter columns
- The data contain missing entries that have not been encoded as NA per the AMADA import format (CSV/TXT, named, space-separated columns)

## Inputs

- Correlation matrix of dataset parameters (Pearson, Spearman, or MIC) produced by the upstream correlation step
- Dataset imported as CSV/TXT with named, space-separated columns and missing values marked as NA (e.g. Supernova host properties from Sako et al. 2014, mock galaxy catalog from Guo et al. 2011, or ZENS catalog)

## Outputs

- Dendrogram plot (dendrogram.png) rendered in Phylogram, Cladogram, or Fan style with clusters color-coded
- Cluster assignment per variable (cluster_assignments.csv)
- Optimal number of clusters as scored by the Calinski-Harabasz index

## How to apply

Start from the correlation matrix produced by the correlation stage (Pearson, Spearman, or Maximum Information Coefficient depending on the selected method). Convert it to a distance matrix using the correlation distance d(x,y) = 1 - |corr(x,y)|, which is the similarity measure AMADA assumes for clustering. Run agglomerative clustering in R: begin with a single cluster per object and progressively merge the two closest clusters until a single cluster remains. Because hierarchical clustering does not require the number of clusters a priori, guide the user by computing the optimal number of clusters via the Calinski-Harabasz (1972) index using the fpc package. Plot the dendrogram with the ape package in the chosen style — Phylogram, Cladogram, or Fan — color-coding the Calinski-Harabasz-optimal cluster grouping. Repeat the analysis for each choice of correlation matrix so structure can be compared across correlation methods.

## Related tools

- **AMADA** (Shiny-based pipeline that performs the hierarchical clustering analysis for different choices of correlation matrices and renders the dendrograms) — https://github.com/RafaelSdeSouza/AMADA
- **fpc** (Provides the Calinski-Harabasz index used to display the optimal number of clusters)
- **ape** (Used to plot the dendrogram in Phylogram, Cladogram, or Fan style with color-coded clusters)
- **shiny** (Runs the AMADA_shiny app in which the clustering tab and dendrogram options are exposed to the user) — https://github.com/RafaelSdeSouza/AMADA_shiny

## Examples

```
R> hc <- hclust(as.dist(1 - abs(cor(dat, method="pearson")))); plot(as.phylo(hc), type="phylogram")
```

## Evaluation signals

- The dendrogram file (dendrogram.png) exists and renders the full hierarchical merge structure, starting from one cluster per object and ending at a single cluster
- The number of color-coded groups in the dendrogram matches the optimal number of clusters reported by the Calinski-Harabasz index from the fpc package
- The dendrogram topology changes consistently when switching the correlation method among Pearson, Spearman, and MIC, since the analysis is repeated for each correlation matrix
- The cluster_assignments.csv contains one row per variable/column of the input correlation matrix, with the assigned cluster label
- The distance matrix used to build the tree equals 1 - |corr(x,y)|, so tree merge heights are consistent with the absolute-correlation dissimilarity

## Limitations

- The method implicitly assumes a similarity measure between pairs of objects — here fixed to correlation distance d(x,y) = 1 - |corr(x,y)| — so variables with anti-correlated behavior may be grouped as identical
- Hierarchical clustering does not produce a single partitioning; the number of clusters must be guided by a diagnostic (Calinski-Harabasz), which may not reflect biophysically meaningful groupings
- Missing data must be explicitly marked NA in the imported CSV/TXT; malformed or improperly imported data (check the 'Import Dataset' panel) will corrupt the correlation matrix feeding the clustering

## Evidence

- [readme] correlation distance defines clustering input: "the method implicitly  assumes a measure of similarity between pairs of objects. Which in our case is given by the correlation distance d(x,y)= 1-|corr(x,y)|."
- [readme] agglomerative merge procedure: "We employ an agglomerative approach, which starts with a single cluster assigned for each object and then progressively merge the two closest clusters until a single cluster remains."
- [intro] clustering repeated per correlation matrix: "This is done by performing a hierarchical clustering analysis for different choices of correlation matrices"
- [methods] Calinski-Harabasz optimal cluster count: "**Number of Clusters**: To guide the user, we display an optimal number of clusters via [Calinski and Harabasz, 1972]"
- [readme] dendrogram style options: "**Dendrogram type**: choose among *Phylogram*, *Cladogram* or *Fan*."
