---
name: astronomical-catalog-parameter-clustering
description: Use when a tabular astronomical catalog (CSV/TXT, columns = variables,
  missing values marked NA) needs unsupervised structure discovery and no number of
  clusters is known a priori.
when_to_use_negative:
- The data have already been reduced to a small set of uncorrelated variables (AMADA
  handles that case with its Standard PCA or Projection-Pursuit Robust PCA path instead
  of clustering the correlation matrix)
- A single predefined partition with a known cluster count is required — the method
  builds a hierarchy of partitions and only reports an optimal count as guidance via
  Calinski-Harabasz
- The dataset cannot be expressed as a rectangular CSV/TXT variable-by-object table
  (e.g., contains non-numeric or unstructured fields that cannot feed a correlation
  matrix)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: AMADA
  role: R package implementing the hierarchical clustering over different choices
    of correlation matrices
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: ape
  role: renders the hierarchical merge structure as a dendrogram (Phylogram, Cladogram,
    or Fan)
- name: fpc
  role: supplies the Calinski-Harabasz index used to display the optimal number of
    clusters
- name: shiny
  role: drives the AMADA Shiny app whose tabs and 'Make it so' button expose the clustering
    controls
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:3bc1a7bd38a402374f43ff9192cb7e9bd6e07a24e978bdf27d8b80fa6de963bb
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# astronomical-catalog-parameter-clustering

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Agglomerative hierarchical clustering over astronomical catalog parameters as implemented in AMADA: a correlation matrix (Pearson, Spearman, or MIC) is converted to a correlation distance d(x,y) = 1 - |corr(x,y)| and merged bottom-up, with the optimal cluster count guided by the Calinski-Harabasz index.

## When to use

Use when a tabular astronomical catalog (CSV/TXT, columns = variables, missing values marked NA) needs unsupervised structure discovery and no number of clusters is known a priori.

## When NOT to use

- The data have already been reduced to a small set of uncorrelated variables (AMADA handles that case with its Standard PCA or Projection-Pursuit Robust PCA path instead of clustering the correlation matrix)
- A single predefined partition with a known cluster count is required — the method builds a hierarchy of partitions and only reports an optimal count as guidance via Calinski-Harabasz
- The dataset cannot be expressed as a rectangular CSV/TXT variable-by-object table (e.g., contains non-numeric or unstructured fields that cannot feed a correlation matrix)

## Inputs

- Astronomical catalog as CSV/TXT file with named, space-separated columns (e.g., supernova host properties from Sako et al. 2014, mock galaxy catalog from Guo et al. 2011, or ZENS catalog); missing values marked NA
- Correlation matrix computed from the catalog (Pearson, Spearman, or MIC per the selected correlation method)

## Outputs

- Dendrogram plot (ape package) in Phylogram, Cladogram, or Fan style, with clusters color-coded (e.g., dendrogram.png)
- Per-variable cluster assignment table (e.g., cluster_assignments.csv)
- Optimal number of clusters reported via the Calinski-Harabasz index (fpc)

## How to apply

Import the catalog as CSV/TXT (named, space-separated columns; missing data as NA) and compute a correlation matrix choosing among Pearson, Spearman, or Maximum Information Coefficient (MIC). Convert the correlation matrix to a distance using the correlation distance d(x,y) = 1 - |corr(x,y)|, which AMADA assumes as the similarity measure between pairs of objects. Run agglomerative hierarchical clustering: start with one cluster per object and progressively merge the two closest clusters until a single cluster remains, building a hierarchy of partitions rather than a single partitioning. Because agglomerative clustering does not require a prior specification of the number of clusters, display an optimal number of clusters via the Calinski-Harabasz criterion (fpc package) to guide the user. Render the hierarchy as a dendrogram with the ape package in the chosen style (Phylogram, Cladogram, or Fan), color-coding the Calinski-Harabasz-optimal clusters, and repeat the whole analysis for each correlation-matrix choice.

## Related tools

- **AMADA** (R package implementing the hierarchical clustering over different choices of correlation matrices) — https://github.com/RafaelSdeSouza/AMADA
- **ape** (renders the hierarchical merge structure as a dendrogram (Phylogram, Cladogram, or Fan))
- **fpc** (supplies the Calinski-Harabasz index used to display the optimal number of clusters)
- **shiny** (drives the AMADA Shiny app whose tabs and 'Make it so' button expose the clustering controls) — https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip

## Examples

```
require(shiny); runUrl('https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip')
```

## Evaluation signals

- The dendrogram file exists and renders the full agglomerative merge structure (one cluster per object at the bottom, a single cluster at the root)
- The number of displayed clusters equals the Calinski-Harabasz-optimal count from fpc, and clusters are color-coded in the plot
- Re-running with each correlation method (Pearson, Spearman, MIC) produces a clustering analysis for each matrix choice, as the workflow prescribes
- Cluster assignments cover every variable/column of the imported catalog, and rows with NA in the source CSV/TXT were handled rather than dropped silently
- The distance used to merge clusters equals 1 - |corr(x,y)| between the corresponding catalog variables

## Limitations

- The method implicitly assumes a pairwise similarity measure; the fixed choice d(x,y) = 1 - |corr(x,y)| means highly anti-correlated variables (corr near -1) are treated as maximally similar
- The optimal cluster count via Calinski-Harabasz is only guidance for the user, not a definitive partition
- Data must be imported as CSV/TXT with columns named and separated by spaces, and missing data must be marked as NA; other encodings of missingness break the import

## Evidence

- [readme] Agglomerative merge procedure: "We employ an agglomerative approach, which starts with a single cluster assigned for each object and then progressively merge the two closest clusters until a single cluster remains."
- [readme] Correlation distance definition: "the method implicitly  assumes a measure of similarity between pairs of objects. Which in our case is given by the correlation distance d(x,y)= 1-|corr(x,y)|"
- [intro] Clustering over correlation matrices: "This is done by performing a hierarchical clustering analysis for different choices of correlation matrices"
- [readme] Dendrogram rendering options: "**Dendrogram type**: choose among *Phylogram*, *Cladogram* or *Fan*."
- [readme] Optimal cluster count guidance: "**Number of Clusters**: To guide the user, we  display an optimal"
