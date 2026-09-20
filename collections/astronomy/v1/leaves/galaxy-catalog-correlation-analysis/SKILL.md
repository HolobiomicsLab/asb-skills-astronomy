---
name: galaxy-catalog-correlation-analysis
description: Use when a named-column CSV/TXT catalog of galaxy or supernova-host properties
  must be explored for pairwise variable relationships before clustering or PCA, and
  linear, rank-based, and information-theoretic measures are all wanted.
when_to_use_negative:
- The data are not a feature/variable table (e.g. raw images or spectra) — AMADA expects
  a CSV/TXT table of named columns and spaces as separators
- Only one correlation measure is needed for a final publication figure rather than
  comparative exploration feeding clustering under different matrix choices
- The dataset has extensive missing values and MIC is required — rows with NA must
  be dropped first, which may leave too few complete cases
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AMADA
  role: R package / Shiny app that offers Pearson, Spearman, or MIC as the correlation
    method and applies hierarchical clustering to the resulting matrices
  repo: https://github.com/RafaelSdeSouza/AMADA
- name: minerva
  role: R package used to compute the Maximum Information Coefficient (MIC) correlation
    matrix; requires complete cases (no NA)
- name: R
  role: Language in which the three correlation matrices are computed and exported
    to CSV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:8184923835faacacf4e7c7936fcdf72bf69a95e12754f533b6109a8e6b5183b9
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# galaxy-catalog-correlation-analysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes Pearson, Spearman, and Maximum Information Coefficient (MIC) correlation matrices across variables of an astronomical catalog (e.g., the Guo et al. 2011 mock galaxy catalog bundled with AMADA) to feed downstream hierarchical clustering and PCA diagnostics.

## When to use

Use when a named-column CSV/TXT catalog of galaxy or supernova-host properties must be explored for pairwise variable relationships before clustering or PCA, and linear, rank-based, and information-theoretic measures are all wanted.

## When NOT to use

- The data are not a feature/variable table (e.g. raw images or spectra) — AMADA expects a CSV/TXT table of named columns and spaces as separators
- Only one correlation measure is needed for a final publication figure rather than comparative exploration feeding clustering under different matrix choices
- The dataset has extensive missing values and MIC is required — rows with NA must be dropped first, which may leave too few complete cases

## Inputs

- Space-separated CSV/TXT catalog with named columns (e.g. mock galaxy catalog from Guo et al. 2011, or SDSS supernova host properties from Sako et al. 2014 Table 1)
- Missing values encoded as NA
- An arbitrary number of columns (variables) and rows (objects)

## Outputs

- Pearson correlation matrix (square, variables x variables)
- Spearman correlation matrix (square, variables x variables)
- MIC correlation matrix computed via the minerva package (square, variables x variables)
- Three CSV files, one per correlation matrix

## How to apply

Import the catalog as a space-separated CSV/TXT table with named columns, marking missing values as NA (the AMADA convention, e.g. the Guo et al. 2011 mock galaxy catalog distributed with the package). Compute the Pearson correlation matrix for the linear relationships between variable pairs, then the Spearman matrix for monotonic rank-based relationships, on the same numeric columns. Compute the MIC matrix with the R minerva package, first restricting rows to complete cases because the current MIC implementation does not support NA. Write each of the three matrices to CSV; AMADA then feeds these different correlation-matrix choices into its hierarchical clustering analysis, using the correlation distance d(x,y) = 1-|corr(x,y)| as the pairwise similarity measure.

## Related tools

- **AMADA** (R package / Shiny app that offers Pearson, Spearman, or MIC as the correlation method and applies hierarchical clustering to the resulting matrices) — https://github.com/RafaelSdeSouza/AMADA
- **minerva** (R package used to compute the Maximum Information Coefficient (MIC) correlation matrix; requires complete cases (no NA))
- **R** (Language in which the three correlation matrices are computed and exported to CSV)

## Examples

```
install.packages('minerva',dependencies=TRUE); mic <- mine(dat[complete.cases(dat),]); write.csv(mic, 'mic_matrix.csv')
```

## Evaluation signals

- Each output CSV is a square matrix whose rows and columns correspond to the catalog variables
- Diagonal entries of every correlation matrix equal 1
- All off-diagonal Pearson/Spearman entries lie in [-1, 1]
- MIC matrix was computed only on rows with no NA values (complete cases), since minerva's MIC implementation does not support NA
- Import sanity check: verify at the bottom of the 'Import Dataset' panel that the data were properly imported before computing correlations

## Limitations

- The current MIC implementation (minerva) does not support NA values, so rows with missing data must be excluded before computing the MIC matrix
- MIC is computationally heavier than Pearson/Spearman, so restricting to complete cases can materially shrink the sample
- Different correlation choices yield different clusterings; AMADA surfaces this by performing hierarchical clustering for different choices of correlation matrices rather than fixing one

## Evidence
