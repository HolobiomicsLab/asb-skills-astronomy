---
name: calinski-harabasz-index-computation
description: Use when a hierarchical clustering hierarchy (e.g. built on correlation
  distance d(x,y)=1-|corr(x,y)|) must be cut at a data-driven number of clusters rather
  than a user-specified k.
when_to_use_negative:
- The partitioning method already requires a pre-specified k and no hierarchy exists
  to cut (the index needs candidate partitions from the agglomerative tree)
- The goal is PCA dimensionality reduction (Number of PCs / Nightingale charts) rather
  than cluster-count selection
- The clustering similarity measure is not correlation-based and the candidate k range
  has not been defined by cutting the dendrogram
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: fpc
  role: R package used to compute the Calinski-Harabasz (1972) index for each candidate
    number of clusters
- name: AMADA
  role: R package / Shiny app whose 'Number of Clusters' control displays the optimal
    cluster count computed via Calinski and Harabasz (1972)
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: AMADA_shiny
  role: Shiny app front end where the optimal cluster count is surfaced to guide the
    user
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny
- name: R
  role: Runtime environment in which the fpc-based CH index computation is executed
  repo: http://www.r-project.org
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:fb200fea76990cc6d8b45b81ef954d1837bb4269fb689434d719ce5e4d49dbef
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/RafaelSdeSouza/AMADA
  tools:
  - AMADA
  - AMADA_shiny
  - R
  - fpc
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# calinski-harabasz-index-computation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes the Calinski and Harabasz (1972) index over candidate cuts of an agglomerative hierarchical clustering tree to identify the optimal number of clusters, as implemented in AMADA to guide users of its 'Number of Clusters' diagnostic.

## When to use

Use when a hierarchical clustering hierarchy (e.g. built on correlation distance d(x,y)=1-|corr(x,y)|) must be cut at a data-driven number of clusters rather than a user-specified k.

## When NOT to use

- The partitioning method already requires a pre-specified k and no hierarchy exists to cut (the index needs candidate partitions from the agglomerative tree)
- The goal is PCA dimensionality reduction (Number of PCs / Nightingale charts) rather than cluster-count selection
- The clustering similarity measure is not correlation-based and the candidate k range has not been defined by cutting the dendrogram

## Inputs

- Hierarchical clustering (agglomerative tree) built on catalog variables using correlation distance d(x,y)=1-|corr(x,y)|
- Catalog/table of variables (e.g. CSV/TXT with named, space-separated columns; NA marks missing data), such as the SDSS supernova host properties, Guo et al. 2011 mock galaxy catalog, or ZENS catalog

## Outputs

- Single integer: optimal number of clusters (k with maximum Calinski-Harabasz index)
- calinski_harabasz_scores.csv: one row per candidate k with columns k and calinski_harabasz, in ascending k order

## How to apply

Load the agglomerative hierarchy produced from the correlation distance d(x,y)=1-|corr(x,y)| between catalog variables, then in R use the fpc package to compute the Calinski-Harabasz (1972) index for each candidate k obtainable by cutting the tree (from 2 up to a sensible fraction of the number of variables). Select the k with the maximum CH score as the optimal cluster count, matching AMADA's behavior of displaying an optimal number of clusters to guide the user. Write the full score curve (columns k and calinski_harabasz, one row per candidate in ascending order) so the choice is auditable. Validation: the reported optimal k must equal the argmax of the calinski_harabasz column.

## Related tools

- **fpc** (R package used to compute the Calinski-Harabasz (1972) index for each candidate number of clusters)
- **AMADA** (R package / Shiny app whose 'Number of Clusters' control displays the optimal cluster count computed via Calinski and Harabasz (1972)) — https://github.com/RafaelSdeSouza/AMADA
- **AMADA_shiny** (Shiny app front end where the optimal cluster count is surfaced to guide the user) — https://github.com/RafaelSdeSouza/AMADA_shiny
- **R** (Runtime environment in which the fpc-based CH index computation is executed) — http://www.r-project.org

## Examples

```
library(fpc); ch <- sapply(2:kmax, function(k) calinhara(dist_matrix, cutree(hclust_out, k=k))); which.max(ch) + 1
```

## Evaluation signals

- calinski_harabasz_scores.csv contains exactly one row per candidate k, sorted in ascending order of k
- The reported optimal k equals the argmax of the calinski_harabasz column
- Candidate k values start at 2 and do not exceed a sensible fraction of the number of variables
- The single-integer optimal k is consistent with the row achieving the maximum CH score in the written CSV
- The clustering input used correlation distance d(x,y)=1-|corr(x,y)| on the catalog variables

## Limitations

- The index only guides cluster-count choice for cuts of the agglomerative tree; the hierarchy itself is built on correlation distance d(x,y)=1-|corr(x,y)|, so results depend on the chosen correlation matrix/method
- The upper bound of candidate k is only 'a sensible fraction of the number of variables' — the article gives no fixed cutoff
- Datasets containing missing values must encode them as NA on import; the README gives no guidance for CH computation under heavy missingness

## Evidence
