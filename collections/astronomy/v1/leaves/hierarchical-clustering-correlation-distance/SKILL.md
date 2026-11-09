---
name: hierarchical-clustering-correlation-distance
description: Use when you need to reveal hidden multi-level structure among variables
  (or objects) in a high-dimensional galaxy-catalog or photometric table without pre-specifying
  the number of clusters.
when_to_use_negative:
- The dataset contains missing values AND MIC (Maximum Information Coefficient) was
  selected as the correlation method, since MIC does not support NA values.
- A prior partitioning of the data is already known or required — hierarchical clustering
  builds a hierarchy of partitions rather than a single clustering.
- The input is raw multi-band imaging rather than a reduced, tabular (CSV/TXT) catalog
  of named numeric columns.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_0092
tools:
- name: AMADA
  role: Shiny app / R package that performs the hierarchical clustering on correlation
    distances and displays the dendrogram diagnostics
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: fpc
  role: Provides the Calinski-Harabasz index used to display the optimal number of
    clusters
- name: ape
  role: Renders the hierarchical clustering outcome as dendrograms (Phylogram, Cladogram,
    Fan) with color-coded groups
- name: corrplot
  role: Displays the correlation matrix heatmap feeding the clustering distance
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:8f3c78234ad36450130ecdfb2b91373ef1e2c5ca0a2146cb489bdf38c390f89d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# hierarchical-clustering-correlation-distance

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Agglomerative hierarchical clustering of variables in high-dimensional astronomical datasets (e.g., SDSS supernova host-galaxy properties) using the correlation distance d(x,y) = 1 - |corr(x,y)|, with the optimal number of clusters guided by the Calinski-Harabasz index via the fpc package in AMADA.

## When to use

Use when you need to reveal hidden multi-level structure among variables (or objects) in a high-dimensional galaxy-catalog or photometric table without pre-specifying the number of clusters.

## When NOT to use

- The dataset contains missing values AND MIC (Maximum Information Coefficient) was selected as the correlation method, since MIC does not support NA values.
- A prior partitioning of the data is already known or required — hierarchical clustering builds a hierarchy of partitions rather than a single clustering.
- The input is raw multi-band imaging rather than a reduced, tabular (CSV/TXT) catalog of named numeric columns.

## Inputs

- CSV/TXT data table with named, space-separated columns and arbitrary rows (e.g., SDSS Type Ia and II supernova host-galaxy properties, Table 1 of Sako et al. 2014)
- Missing-data marker: NA
- User-selected correlation method (Pearson, Spearman, or MIC)

## Outputs

- Optimal number of clusters derived from the Calinski-Harabasz index
- Per-object cluster assignments
- Hierarchical dendrogram visualization (Phylogram, Cladogram, or Fan) with color-coded clusters (ape package)

## How to apply

Load the dataset as CSV/TXT with named, space-separated columns, marking missing entries as NA. Compute the similarity matrix among variables with Pearson correlation (Spearman or MIC are selectable alternatives; MIC does not support NA values). Convert correlations to the distance d(x,y) = 1 - |corr(x,y)| and run agglomerative clustering that starts with a single cluster per object and progressively merges the two closest clusters until one remains — no prior number of clusters is needed. Guide the user to an optimal cluster count via the Calinski-Harabasz index (fpc package) and color-code groups in the dendrogram rendered by the ape package; in the AMADA Shiny app the dendrogram can be drawn as a Phylogram, Cladogram, or Fan. Write the optimal cluster count and per-object cluster assignments so downstream tab-based diagnostics (correlation heatmap, Nightingale PC charts) can be iterated with the 'Make it so' button.

## Related tools

- **AMADA** (Shiny app / R package that performs the hierarchical clustering on correlation distances and displays the dendrogram diagnostics) — https://github.com/RafaelSdeSouza/AMADA
- **fpc** (Provides the Calinski-Harabasz index used to display the optimal number of clusters)
- **ape** (Renders the hierarchical clustering outcome as dendrograms (Phylogram, Cladogram, Fan) with color-coded groups)
- **corrplot** (Displays the correlation matrix heatmap feeding the clustering distance)

## Examples

```
install_github("RafaelSdeSouza/AMADA"); require(shiny); runUrl('https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip')
```

## Evaluation signals

- The chosen correlation method (Pearson/Spearman/MIC) is recorded and the distance used is exactly d(x,y) = 1 - |corr(x,y)|.
- Every variable/object in the input table receives a cluster assignment (no NA-driven drops in the assignment vector).
- The reported optimal number of clusters is reproducible from the Calinski-Harabasz index computation (fpc) on the same distance matrix.
- The dendrogram is renderable in all three types (Phylogram, Cladogram, Fan) without errors and color-codes the Calinski-Harabasz-derived groups.
- Merges start from single-object clusters and end at a single cluster, forming a complete hierarchy of partitions.

## Limitations

- The method implicitly assumes a measure of similarity between pairs of objects, here the correlation distance; results depend on the chosen correlation method.
- MIC cannot be used when the dataset contains NA missing entries.
- The optimal cluster count is only a guide 'to guide the user'; hierarchical clustering does not itself require or enforce a single best partition.

## Evidence
