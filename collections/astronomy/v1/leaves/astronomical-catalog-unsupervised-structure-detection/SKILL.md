---
name: astronomical-catalog-unsupervised-structure-detection
description: Use when a hierarchical clustering tree built on correlation distance
  d(x,y)=1-|corr(x,y)| between catalog variables needs a data-driven choice of how
  many clusters to cut it into.
when_to_use_negative:
- When a partitioning method with a fixed, user-specified cluster count is already
  being used and no tree-cutting decision is needed
- When the hierarchy was built on a metric other than the correlation distance used
  by AMADA's agglomerative clustering, unless the CH evaluation is re-justified for
  that input space
- When the dataset contains unhandled missing values that are not marked NA per AMADA's
  import format, since the index computation requires a complete numeric table
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: AMADA
  role: Provides the hierarchical clustering output and displays the optimal number
    of clusters via the Calinski-Harabasz index to guide the user
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: fpc
  role: R package used to compute the Calinski-Harabasz (1972) index for candidate
    cluster counts
- name: R
  role: Runtime environment in which the clustering, index computation, and CSV export
    are performed
  repo: http://www.r-project.org
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:f26898468db51ee9289089ae9dda61e025fc623718248e384a27e1febcd375d5
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# astronomical-catalog-unsupervised-structure-detection

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Guides selection of the optimal number of clusters in hierarchical clustering of astronomical catalog variables using the Calinski-Harabasz (1972) index, as implemented in AMADA's 'Number of Clusters' diagnostic.

## When to use

Use when a hierarchical clustering tree built on correlation distance d(x,y)=1-|corr(x,y)| between catalog variables needs a data-driven choice of how many clusters to cut it into.

## When NOT to use

- When a partitioning method with a fixed, user-specified cluster count is already being used and no tree-cutting decision is needed
- When the hierarchy was built on a metric other than the correlation distance used by AMADA's agglomerative clustering, unless the CH evaluation is re-justified for that input space
- When the dataset contains unhandled missing values that are not marked NA per AMADA's import format, since the index computation requires a complete numeric table

## Inputs

- Hierarchical clustering hierarchy (hclust object) built from correlation distance d(x,y)=1-|corr(x,y)| on catalog variables
- CSV/TXT data table with named columns, space-separated, missing values marked NA (AMADA 'Import Dataset' format)

## Outputs

- Single integer: optimal number of clusters (argmax of the Calinski-Harabasz index)
- calinski_harabasz_scores.csv with one row per candidate k and columns k, calinski_harabasz (ascending k, optimal k equals argmax)

## How to apply

Cut the agglomerative hierarchy (built on AMADA's correlation distance d(x,y)=1-|corr(x,y)|) at candidate k values from 2 up to a sensible fraction of the number of catalog variables. For each candidate k, compute the Calinski-Harabasz index in R via the fpc package (e.g. calinhara on the clustered data and cutree labels). Select the k that maximizes the CH index and report it to the user as the optimal cluster count, mirroring AMADA's 'Number of Clusters' display. Rationale: hierarchical clustering needs no a priori cluster number, but an internal validation index like CH balances between-cluster dispersion against within-cluster dispersion to pick the best partition level. Record the full CH curve so the argmax choice is auditable.

## Related tools

- **AMADA** (Provides the hierarchical clustering output and displays the optimal number of clusters via the Calinski-Harabasz index to guide the user) — https://github.com/RafaelSdeSouza/AMADA
- **fpc** (R package used to compute the Calinski-Harabasz (1972) index for candidate cluster counts)
- **R** (Runtime environment in which the clustering, index computation, and CSV export are performed) — http://www.r-project.org

## Examples

```
library(fpc); ks <- 2:20; ch <- sapply(ks, function(k) calinhara(X, cutree(hc, k=k))); which.max(ch)
```

## Evaluation signals

- calinski_harabasz_scores.csv contains one row per candidate k, in ascending order of k
- The reported optimal k equals the argmax of the calinski_harabasz column
- Candidate k values start at 2 and do not exceed a sensible fraction of the number of variables in the input catalog
- The optimal k is a single integer suitable for cutting the agglomerative tree with cutree
- The input table passed AMADA import conventions: named columns, space-separated, NA for missing data

## Limitations

- The CH index is computed per candidate k on cuts of a single agglomerative tree; it cannot recover partitions the tree never proposes
- The guidance is diagnostic only — AMADA 'displays' the optimal number to guide the user, who retains the choice of cluster count
- Missing data must be explicitly marked NA; arbitrary handling of gaps in the catalog will corrupt the index

## Evidence

- [methods] To guide the user, we display an optimal number of clusters via [Calinski and Harabasz, 1972]: "**Number of Clusters**: To guide the user, we display an optimal number of clusters via [Calinski and Harabasz, 1972]"
- [readme] the method implicitly assumes a measure of similarity between pairs of objects. Which in our case is given by the correlation distance d(x,y)= 1-|corr(x,y)|.: "the method implicitly  assumes a measure of similarity between pairs of objects. Which in our case is given by the correlation distance d(x,y)= 1-|corr(x,y)|."
