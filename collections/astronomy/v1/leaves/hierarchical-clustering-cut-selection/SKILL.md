---
name: hierarchical-clustering-cut-selection
description: Use when a dendrogram from AMADA's hierarchical clustering must be cut
  into a partition, and no prior number of clusters is specified — the Calinski-Harabasz
  index guides the cut height choice.
when_to_use_negative:
- The partition is produced by PCA-based or robust PCA analysis instead of the hierarchical
  clustering on correlation distance — the index was applied by AMADA to the clustering
  tree, not to PC scores
- The user already has a scientifically motivated cluster count and only needs the
  dendrogram visualized (Phylogram/Cladogram/Fan) without cut-point selection
- The data contain unhandled missing values that were not marked NA in the CSV/TXT
  input, corrupting the correlation distance matrix
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AMADA
  role: Provides the hierarchical clustering hierarchy and displays the optimal number
    of clusters computed via Calinski and Harabasz (1972)
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: fpc
  role: R package used to compute the Calinski-Harabasz (1972) index for each candidate
    number of clusters
- name: R
  role: Runtime environment in which the hierarchy is cut and the CH index is evaluated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:18e4433681dc800053180244979ca7991a441d6286a4e4e7ec224838326c44f1
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/RafaelSdeSouza/AMADA
  tools:
  - AMADA
  - R
  - fpc
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# hierarchical-clustering-cut-selection

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Selects an optimal number of clusters when cutting AMADA's agglomerative hierarchical clustering tree (built on the correlation distance d(x,y)=1-|corr(x,y)|) by maximizing the Calinski and Harabasz (1972) index computed with the R fpc package.

## When to use

Use when a dendrogram from AMADA's hierarchical clustering must be cut into a partition, and no prior number of clusters is specified — the Calinski-Harabasz index guides the cut height choice.

## When NOT to use

- The partition is produced by PCA-based or robust PCA analysis instead of the hierarchical clustering on correlation distance — the index was applied by AMADA to the clustering tree, not to PC scores
- The user already has a scientifically motivated cluster count and only needs the dendrogram visualized (Phylogram/Cladogram/Fan) without cut-point selection
- The data contain unhandled missing values that were not marked NA in the CSV/TXT input, corrupting the correlation distance matrix

## Inputs

- Hierarchical clustering hierarchy (hclust/agglomerative tree) built on correlation distance d(x,y)=1-|corr(x,y)| between catalog variables
- Catalog data table imported as CSV/TXT with named, space-separated columns (missing values marked NA), e.g. SDSS supernova host-galaxy properties, Guo et al. 2011 mock galaxy catalog, or ZENS catalog

## Outputs

- Single integer: the optimal number of clusters (candidate k with maximum Calinski-Harabasz index)
- calinski_harabasz_scores.csv with one row per candidate k and columns k, calinski_harabasz (ascending k)

## How to apply

Take the agglomerative hierarchy built on the correlation distance d(x,y)=1-|corr(x,y)| between the catalog variables (e.g. supernova host properties from Sako et al. 2014, the Guo et al. 2011 mock galaxy catalog, or the ZENS catalog). In R, cut the tree at each candidate k (from 2 up to a sensible fraction of the number of variables) and evaluate each partition with the Calinski and Harabasz (1972) index using the fpc package, as AMADA does to guide the user. Record the full CH score curve (one row per candidate k with columns k and calinski_harabasz) to calinski_harabasz_scores.csv and report the argmax k as the optimal cluster number. Rationale: hierarchical clustering does not require a prior specification of the number of clusters, so an internal validation index provides a data-driven cut point.

## Related tools

- **AMADA** (Provides the hierarchical clustering hierarchy and displays the optimal number of clusters computed via Calinski and Harabasz (1972)) — https://github.com/RafaelSdeSouza/AMADA
- **fpc** (R package used to compute the Calinski-Harabasz (1972) index for each candidate number of clusters)
- **R** (Runtime environment in which the hierarchy is cut and the CH index is evaluated)

## Examples

```
Rscript -e "library(fpc); d <- as.dist(1-abs(cor(dat,use='complete.obs'))); h <- hclust(d); ch <- sapply(2:10, function(k) calinhara(dat, cutree(h,k))); which.max(ch)+1"
```

## Evaluation signals

- calinski_harabasz_scores.csv contains exactly one row per candidate k, sorted in ascending order of k
- The reported optimal k equals the argmax of the calinski_harabasz column
- The k range starts at 2 and does not exceed a sensible fraction of the number of variables in the imported catalog
- The correlation distance used to build the tree is d(x,y)=1-|corr(x,y)|, consistent with AMADA's clustering step
- The optimal cluster count reproduces the value AMADA displays to the user in the Number of Clusters guidance

## Limitations

- The CH index only guides the user; it displays a recommended optimal number of clusters rather than enforcing one
- Hierarchical clustering implicitly assumes a similarity measure — here the correlation distance — so the CH scores are tied to that choice of correlation matrix (Pearson, Spearman, or MIC)
- Data must be imported as CSV/TXT with named, space-separated columns and missing values marked NA; malformed input corrupts the hierarchy and hence the CH curve

## Evidence
