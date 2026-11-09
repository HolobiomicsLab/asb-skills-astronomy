---
name: correlation-distance-matrix-analysis
description: Use when a tabular astronomical catalog (CSV/TXT with named columns)
  must be explored by clustering its variables, and similarity should be the AMADA
  correlation distance d(x,y)=1-|corr(x,y)| feeding an agglomerative hierarchy with
  a Calinski-Harabasz-guided cluster count.
when_to_use_negative:
- 'The analysis must distinguish anti-correlated from correlated variables: d(x,y)=1-|corr(x,y)|
  maps both corr=-1 and corr=+1 to distance 0, discarding the sign'
- A single flat partition with a known number of clusters is required; AMADA's hierarchical
  clustering instead builds a hierarchy of partitions that reveals structure at multiple
  levels of association
- The columns are not numeric/continuous catalog properties, so Pearson/Spearman/MIC
  correlations are not meaningful inputs to the distance conversion
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2269
tools:
- name: AMADA
  role: R package whose clustering stage consumes the selected correlation matrix,
    converts it via d(x,y)=1-|corr(x,y)|, and runs the agglomerative analysis; installed
    with install_github('RafaelSdeSouza/AMADA')
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: R
  role: Runtime in which the correlation-distance conversion, agglomerative clustering,
    and Calinski-Harabasz computation are executed (install from http://www.r-project.org)
  repo: http://www.r-project.org
- name: fpc
  role: Computes the Calinski and Harabasz (1972) index for each candidate k obtainable
    by cutting the agglomerative tree, to select the optimal number of clusters
- name: ape
  role: Plots the variable dendrogram in the chosen style (Phylogram, Cladogram, or
    Fan) with clusters color-coded
- name: pheatmap
  role: Renders the correlation heatmap with rows/columns arranged by this hierarchical
    clustering and dendrograms drawn at the matrix edges
- name: minerva
  role: Provides the Maximum Information Coefficient (MIC) correlation option whose
    matrix can feed the correlation-distance conversion
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:b6192c16160bdf62768c1b6424c6f66057e08be0b538f8b6821db69960d65315
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/RafaelSdeSouza/AMADA
  tools:
  - AMADA
  - R
  - ape
  - circlize
  - corrplot
  - fpc
  - minerva
  - pheatmap
  - shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# correlation-distance-matrix-analysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

AMADA's correlation-distance clustering stage: convert the chosen correlation matrix (Pearson, Spearman, or MIC) into the distance d(x,y)=1-|corr(x,y)|, build an agglomerative hierarchy over the catalog variables, and display the Calinski-Harabasz-optimal number of clusters to guide the user.

## When to use

Use when a tabular astronomical catalog (CSV/TXT with named columns) must be explored by clustering its variables, and similarity should be the AMADA correlation distance d(x,y)=1-|corr(x,y)| feeding an agglomerative hierarchy with a Calinski-Harabasz-guided cluster count.

## When NOT to use

- The analysis must distinguish anti-correlated from correlated variables: d(x,y)=1-|corr(x,y)| maps both corr=-1 and corr=+1 to distance 0, discarding the sign
- A single flat partition with a known number of clusters is required; AMADA's hierarchical clustering instead builds a hierarchy of partitions that reveals structure at multiple levels of association
- The columns are not numeric/continuous catalog properties, so Pearson/Spearman/MIC correlations are not meaningful inputs to the distance conversion

## Inputs

- CSV/TXT dataset with named columns separated by spaces, missing data marked NA (e.g., Sako et al. 2014 supernova host properties, Guo et al. 2011 mock galaxy catalog, ZENS catalog)
- Correlation matrix over the catalog variables computed with the selected method (Pearson, Spearman, or MIC)
- Chosen dendrogram style: Phylogram, Cladogram, or Fan

## Outputs

- Agglomerative hierarchy (merge structure) over the catalog variables
- Optimal number of clusters (integer argmax of the Calinski-Harabasz index)
- cluster_assignments.csv — one cluster assignment per variable
- dendrogram.png — dendrogram with Calinski-Harabasz-optimal clusters color-coded
- calinski_harabasz_scores.csv — CH score curve, one row per candidate k (columns k, calinski_harabasz)

## How to apply

Import the catalog as CSV/TXT with named, space-separated columns (missing values marked NA) and compute the selected correlation matrix — Pearson, Spearman, or Maximum Information Coefficient (MIC). Convert it to a distance matrix with d(x,y)=1-|corr(x,y)|, the similarity measure AMADA's hierarchical clustering implicitly assumes; note the absolute value makes strongly anti-correlated variables maximally similar. Run the agglomerative approach starting with a single cluster per object and progressively merging the two closest clusters until a single cluster remains. Evaluate each candidate cut of the tree with the Calinski and Harabasz (1972) index from R's fpc package and take the argmax k as the optimal number of clusters displayed to the user. Plot the dendrogram with ape in the chosen style (Phylogram, Cladogram, or Fan), color-coding the CH-optimal groups, and write per-variable assignments to cluster_assignments.csv plus the full CH score curve to calinski_harabasz_scores.csv. Repeat the clustering for each correlation-matrix choice, since AMADA re-runs this stage for every selected matrix.

## Related tools

- **AMADA** (R package whose clustering stage consumes the selected correlation matrix, converts it via d(x,y)=1-|corr(x,y)|, and runs the agglomerative analysis; installed with install_github('RafaelSdeSouza/AMADA')) — https://github.com/RafaelSdeSouza/AMADA
- **R** (Runtime in which the correlation-distance conversion, agglomerative clustering, and Calinski-Harabasz computation are executed (install from http://www.r-project.org)) — http://www.r-project.org
- **fpc** (Computes the Calinski and Harabasz (1972) index for each candidate k obtainable by cutting the agglomerative tree, to select the optimal number of clusters)
- **ape** (Plots the variable dendrogram in the chosen style (Phylogram, Cladogram, or Fan) with clusters color-coded)
- **pheatmap** (Renders the correlation heatmap with rows/columns arranged by this hierarchical clustering and dendrograms drawn at the matrix edges)
- **minerva** (Provides the Maximum Information Coefficient (MIC) correlation option whose matrix can feed the correlation-distance conversion)

## Examples

```
dat <- read.csv("catalog.csv"); hc <- hclust(as.dist(1 - abs(cor(dat)))); plot(as.phylo(hc), type = "phylogram")
```

## Evaluation signals

- calinski_harabasz_scores.csv contains one row per candidate k in ascending order and the reported optimal k equals the argmax of the calinski_harabasz column
- The derived distance matrix is symmetric with a zero diagonal, equals 1-|corr(x,y)| elementwise, and is bounded in [0,1]
- cluster_assignments.csv covers every catalog variable exactly once and the dendrogram renders the full agglomerative merge structure (singletons progressively merged to one cluster)
- Dendrogram color-coding matches the Calinski-Harabasz-optimal cluster grouping as reported in the paper's visualization diagnostics
- Switching the correlation method (Pearson/Spearman/MIC) in the Shiny control panel and clicking 'Make it so' regenerates the hierarchy, since clustering is repeated per matrix choice

## Limitations

- d(x,y)=1-|corr(x,y)| treats fully anti-correlated variables (corr=-1) as maximally similar (distance 0), so sign-sensitive questions need a different distance
- The hierarchy depends on the chosen correlation method; AMADA performs the clustering analysis separately for different choices of correlation matrices, so results differ per matrix
- No quantitative acceptance threshold is reported in the paper for the dendrogram/diagnostics — acceptance is the successful rendering of the structure from the shared input
- Data must be imported as CSV/TXT with named columns separated by spaces and missing data marked NA; other formats require conversion first

## Evidence
