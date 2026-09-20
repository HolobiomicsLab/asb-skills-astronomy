---
name: pearson-spearman-matrix-computation
description: Use when a named-column catalog (e.g. galaxy semi-analytic mock data
  or SDSS supernova host properties) needs pairwise variable-similarity matrices to
  feed hierarchical clustering under different correlation choices.
when_to_use_negative:
- The input is already a correlation or distance matrix — recomputing Pearson/Spearman/MIC
  on it is meaningless
- The data contain many NA rows and MIC is required, since the current MIC implementation
  does not support NA and dropping rows would discard too much data
- The goal is a single fixed similarity measure — this skill deliberately produces
  multiple correlation matrices because AMADA clusters under different choices of
  correlation matrices
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Language in which the Pearson and Spearman correlation matrices are computed
    over the catalog's numeric columns and written to CSV
- name: minerva
  role: R package used to compute the Maximum Information Coefficient (MIC) matrix,
    restricted to rows without NA values
- name: AMADA
  role: Framework that consumes the correlation matrices, applying hierarchical clustering
    with correlation distance d(x,y)=1-|corr(x,y)| for different choices of correlation
    matrices
  repo: RafaelSdeSouza/AMADA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:945e740363d3c25850b515ad55ee14547a3764f0bd868067ad3b1a0267dd0988
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/RafaelSdeSouza/AMADA
  tools:
  - AMADA
  - R
  - minerva
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pearson-spearman-matrix-computation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes pairwise similarity structure of a high-dimensional astronomical catalog by building Pearson (linear), Spearman (rank-based/monotonic), and Maximum Information Coefficient (MIC) correlation matrices over its numeric columns, as done in the AMADA workflow so that downstream hierarchical clustering can be run under different correlation choices.

## When to use

Use when a named-column catalog (e.g. galaxy semi-analytic mock data or SDSS supernova host properties) needs pairwise variable-similarity matrices to feed hierarchical clustering under different correlation choices.

## When NOT to use

- The input is already a correlation or distance matrix — recomputing Pearson/Spearman/MIC on it is meaningless
- The data contain many NA rows and MIC is required, since the current MIC implementation does not support NA and dropping rows would discard too much data
- The goal is a single fixed similarity measure — this skill deliberately produces multiple correlation matrices because AMADA clusters under different choices of correlation matrices

## Inputs

- Space-separated CSV/TXT data table with named columns and NA-coded missing values (arbitrary number of rows/columns)
- Mock galaxy catalog (Guo et al. 2011) bundled with AMADA, or user-uploaded catalogs such as SDSS supernova host properties (Sako et al. 2014) or the ZENS catalog

## Outputs

- Pearson correlation matrix (square CSV, rows/columns = catalog variables)
- Spearman correlation matrix (square CSV)
- Maximum Information Coefficient (MIC) matrix computed via minerva (square CSV)

## How to apply

Load the catalog as a space-separated CSV/TXT table with named columns, marking missing entries as NA (AMADA ships a Guo et al. 2011 mock galaxy catalog formatted this way). Compute the Pearson correlation matrix for linear relationships and the Spearman matrix for monotonic (rank-based) relationships across all numeric columns of the same table. Compute the MIC matrix with the R minerva package, restricting to rows with no NA values because the current MIC implementation does not support NA. Write each of the three matrices to its own CSV file; the downstream AMADA hierarchical clustering uses the correlation distance d(x,y) = 1 - |corr(x,y)| on whichever matrix is selected, so multiple correlation options feed the clustering stage rather than a single fixed matrix.

## Related tools

- **R** (Language in which the Pearson and Spearman correlation matrices are computed over the catalog's numeric columns and written to CSV)
- **minerva** (R package used to compute the Maximum Information Coefficient (MIC) matrix, restricted to rows without NA values)
- **AMADA** (Framework that consumes the correlation matrices, applying hierarchical clustering with correlation distance d(x,y)=1-|corr(x,y)| for different choices of correlation matrices) — RafaelSdeSouza/AMADA

## Examples

```
m <- cor(dat, method="pearson", use="pairwise.complete.obs"); write.csv(m, "pearson.csv")  # repeat with method="spearman" and minerva::mine() for MIC
```

## Evaluation signals

- Each output CSV is a square matrix whose rows and columns correspond exactly to the catalog's variables
- Diagonal entries equal 1 for every correlation matrix
- All off-diagonal entries lie in [-1, 1]
- Spearman and Pearson matrices agree closely for variables related linearly/monotonically; discrepancies flag non-monotonic associations that MIC should capture
- CSV files reload cleanly (matching dimension names) before being passed to the AMADA hierarchical clustering step

## Limitations

- The current MIC implementation does not support NA values, so rows containing NA must be excluded before computing the MIC matrix
- The choice of correlation method (Pearson, Spearman, MIC) changes the downstream hierarchical clustering output, so no single matrix is canonical
- AMADA's similarity measure d(x,y)=1-|corr(x,y)| discards the sign of the correlation, so anti-correlated and correlated variable pairs are treated as equally similar

## Evidence
