---
name: cluster-validity-diagnostic-reproduction
description: Use when a hierarchical clustering of catalog variables (correlation
  distance d(x,y)=1-|corr(x,y)|) must be cut and an objective, data-driven number
  of clusters is needed.
when_to_use_negative:
- The data are already partitioned or the number of clusters is fixed a priori — the
  CH sweep is a diagnostic for choosing k, not a clustering method itself
- The similarity measure between variables is not correlation-based, so the hierarchy
  being cut differs from AMADA's d(x,y)=1-|corr(x,y)| construction
- The dataset contains missing values not encoded as NA, since AMADA requires NA marking
  for missing data before analysis
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: AMADA
  role: Provides the Shiny-app 'Number of Clusters' control that displays the optimal
    cluster count via the Calinski-Harabasz index
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: fpc
  role: R library used to compute the Calinski and Harabasz (1972) cluster validity
    index
- name: R
  role: Runtime environment in which the index sweep and CSV export are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:35f73b66b9dd4e4d471bd3dd9da9002be95cdea768e5adce74a663559745319a
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# cluster-validity-diagnostic-reproduction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reproduces AMADA's mechanism for guiding the choice of cluster count: the Calinski and Harabasz (1972) index is computed (via the R fpc package) for each candidate partition obtained by cutting the hierarchical clustering tree, and the argmax k is reported to the user as the optimal number of clusters.

## When to use

Use when a hierarchical clustering of catalog variables (correlation distance d(x,y)=1-|corr(x,y)|) must be cut and an objective, data-driven number of clusters is needed.

## When NOT to use

- The data are already partitioned or the number of clusters is fixed a priori — the CH sweep is a diagnostic for choosing k, not a clustering method itself
- The similarity measure between variables is not correlation-based, so the hierarchy being cut differs from AMADA's d(x,y)=1-|corr(x,y)| construction
- The dataset contains missing values not encoded as NA, since AMADA requires NA marking for missing data before analysis

## Inputs

- Hierarchical clustering hierarchy (e.g. R hclust object) built on correlation distance d(x,y)=1-|corr(x,y)|
- Catalog variable matrix / dataset imported as CSV or TXT with named, space-separated columns (missing values marked NA)

## Outputs

- Optimal number of clusters as a single integer (argmax of the Calinski-Harabasz index)
- calinski_harabasz_scores.csv with one row per candidate k and columns k, calinski_harabasz

## How to apply

Start from the agglomerative clustering hierarchy built on the correlation distance d(x,y)=1-|corr(x,y)| between the catalog variables (e.g. Supernova host properties, Guo et al. 2011 mock catalog, or ZENS group properties loaded as CSV/TXT with space-separated named columns). In R, use the fpc package to evaluate the Calinski-Harabasz (1972) index for each candidate number of clusters obtainable by cutting the tree, sweeping k from 2 up to a sensible fraction of the number of variables. Record the full CH score curve, one row per candidate k with columns k and calinski_harabasz, and select the optimal k as the argmax of the index. This mirrors AMADA's Shiny-app diagnostic under the 'Number of Clusters' control option, which displays an optimal number of clusters via Calinski and Harabasz (1972) purely to guide the user.

## Related tools

- **AMADA** (Provides the Shiny-app 'Number of Clusters' control that displays the optimal cluster count via the Calinski-Harabasz index) — https://github.com/RafaelSdeSouza/AMADA
- **fpc** (R library used to compute the Calinski and Harabasz (1972) cluster validity index)
- **R** (Runtime environment in which the index sweep and CSV export are executed)

## Examples

```
Rscript -e "library(fpc); ch <- sapply(2:10, function(k) calinhara(X, cutree(hc, k))); best <- which.max(ch)+1; write.csv(data.frame(k=2:10, calinski_harabasz=ch), 'calinski_harabasz_scores.csv', row.names=FALSE)"
```

## Evaluation signals

- calinski_harabasz_scores.csv contains exactly one row per candidate k, in ascending order of k
- The reported optimal k equals the argmax of the calinski_harabasz column in the CSV
- The candidate-k range starts at 2 and does not exceed a sensible fraction of the number of variables
- The CH scores are recomputable from the same hierarchy cut (deterministic given the hclust object)

## Limitations

- The CH optimum only guides the user — AMADA still lets the user choose the number of clusters, so the argmax is a recommendation, not a prescription
- The index is computed on partitions of a hierarchy built with correlation distance 1-|corr(x,y)|; a different similarity measure changes the candidate partitions and hence the optimum
- Missing data must be marked NA on import; the hierarchy is agglomerative (single objects merged progressively), so the index inherits the greedy merging structure

## Evidence
