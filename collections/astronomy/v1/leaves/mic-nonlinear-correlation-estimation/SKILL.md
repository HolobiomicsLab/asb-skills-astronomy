---
name: mic-nonlinear-correlation-estimation
description: Use when a multivariate table (e.g. galaxy catalog or supernova host
  properties) may contain non-linear pairwise dependencies and you need a model-free
  dependence measure to feed clustering or correlation heatmaps.
when_to_use_negative:
- Variables are expected to be linearly related and only Pearson correlation is needed
  — MIC adds computational cost without extra information
- The dataset contains many rows with NA values, since the current MIC implementation
  does not support NA and complete-case filtering can substantially shrink the sample
- The goal is PCA on the original data — AMADA computes PCs by diagonalization of
  the data correlation matrix via Standard or Robust PCA, not via MIC
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3360
tools:
- name: minerva
  role: R package that computes the Maximum Information Coefficient (MIC) matrix;
    installed with dependencies alongside the other AMADA libraries
- name: AMADA
  role: Exposes MIC as a selectable correlation method whose resulting matrix feeds
    hierarchical clustering; installed from GitHub via devtools
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: AMADA_shiny
  role: Shiny app UI where the user selects MIC among the Correlation method options
    and clicks 'Make it so' to update the plots
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny
- name: R
  role: Runtime environment in which minerva and AMADA are installed and executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:879e913c8eb289f02abe4db5674e788118d88b0419cb5a17054688da62387de8
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/RafaelSdeSouza/AMADA
  tools:
  - AMADA
  - AMADA_shiny
  - R
  - minerva
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mic-nonlinear-correlation-estimation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes the Maximum Information Coefficient (MIC) matrix as one of three correlation options (alongside Pearson and Spearman) that AMADA feeds into its hierarchical clustering of high-dimensional astronomical datasets. MIC captures non-linear and non-monotonic dependence that Pearson (linear) and Spearman (rank/monotonic) miss.

## When to use

Use when a multivariate table (e.g. galaxy catalog or supernova host properties) may contain non-linear pairwise dependencies and you need a model-free dependence measure to feed clustering or correlation heatmaps.

## When NOT to use

- Variables are expected to be linearly related and only Pearson correlation is needed — MIC adds computational cost without extra information
- The dataset contains many rows with NA values, since the current MIC implementation does not support NA and complete-case filtering can substantially shrink the sample
- The goal is PCA on the original data — AMADA computes PCs by diagonalization of the data correlation matrix via Standard or Robust PCA, not via MIC

## Inputs

- Space-separated CSV/TXT table with named columns (e.g. AMADA's mock galaxy catalog from Guo et al. 2011, supernova host properties from Sako et al. 2014, or ZENS catalog)
- Numeric columns with missing values coded as NA

## Outputs

- Square MIC matrix (rows/columns = catalog variables) written to CSV
- Correlation heatmap / dendrogram of the MIC-based similarity within the AMADA Shiny app

## How to apply

Load the catalog (e.g. the Guo et al. 2011 mock galaxy catalog shipped with AMADA) as a space-separated CSV/TXT with named columns and NA-coded missing values. Compute MIC for all numeric variable pairs using the minerva R package, first restricting to rows with no NA values because the current MIC implementation does not support NA. Select MIC via the AMADA Shiny 'Correlation method' control, choosing among Pearson, Spearman, or Maximum Information Coefficient (MIC). Write the resulting MIC matrix to CSV; it enters the same downstream path as the other correlation matrices — hierarchical clustering with correlation distance d(x,y)=1-|corr(x,y)| — so multiple correlation-matrix options feed the clustering stage rather than a single fixed matrix.

## Related tools

- **minerva** (R package that computes the Maximum Information Coefficient (MIC) matrix; installed with dependencies alongside the other AMADA libraries)
- **AMADA** (Exposes MIC as a selectable correlation method whose resulting matrix feeds hierarchical clustering; installed from GitHub via devtools) — https://github.com/RafaelSdeSouza/AMADA
- **AMADA_shiny** (Shiny app UI where the user selects MIC among the Correlation method options and clicks 'Make it so' to update the plots) — https://github.com/RafaelSdeSouza/AMADA_shiny
- **R** (Runtime environment in which minerva and AMADA are installed and executed)

## Examples

```
install.packages('minerva',dependencies=TRUE); library(minerva); MIC <- mine(dat[complete.cases(dat), ])$MIC
```

## Evaluation signals

- Output MIC matrix is square, with rows and columns corresponding to the catalog variables
- Diagonal values equal 1 and off-diagonal MIC entries fall in the valid dependence range (0–1 for MIC, bounded non-negative; contrast with Pearson/Spearman in [-1, 1])
- No NA values present in the matrix rows used — confirm complete-case filtering was applied before calling minerva
- Written CSV parses correctly and column/row names match the input table's variable names
- Switching the Correlation method control to MIC and clicking 'Make it so' regenerates the heatmap/dendrogram without errors

## Limitations

- The current MIC implementation (minerva) does not support NA values, so rows with missing data must be removed beforehand
- MIC is offered only as a pairwise correlation-method choice within AMADA; it does not replace the PCA branch of the workflow
- MIC computation on large catalogs can be expensive relative to Pearson/Spearman

## Evidence
