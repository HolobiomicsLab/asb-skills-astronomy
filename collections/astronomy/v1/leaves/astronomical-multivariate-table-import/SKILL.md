---
name: astronomical-multivariate-table-import
description: Use when a high-dimensional astronomical catalog must be loaded into
  AMADA for PCA/clustering exploration and the source table needs conversion to AMADA's
  CSV/TXT input format with named columns and NA-coded missing entries.
when_to_use_negative:
- The data are not tabular multivariate measurements (e.g., images, spectra, time
  series) that cannot be represented as named numeric columns
- Missing entries cannot be recoded as NA, so the file would fail AMADA's CSV/TXT
  import conventions
- The analysis goal is a single low-dimensional regression or classification task
  rather than exploratory dimensionality reduction and clustering of a high-dimensional
  table
edam_operation: http://edamontology.org/operation_1812
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: AMADA
  role: Shiny web application that ingests the CSV/TXT table (preloaded or uploaded)
    and exposes it for PCA, robust PCA, and hierarchical clustering exploration
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny
- name: R
  role: Runtime environment in which the AMADA package and Shiny app are installed
    and executed
- name: RStudio
  role: Recommended IDE for installing R packages and running the AMADA Shiny app
- name: shiny
  role: Required R library to launch the AMADA app via runUrl of the AMADA_shiny repository
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:1fc3649e7beabf6f2a0454d9cbefb16272a2541d36e15e682b518a893170c334
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/RafaelSdeSouza/AMADA_shiny
  tools:
  - AMADA
  - R
  - RStudio
  - shiny
  edam_operation: http://edamontology.org/operation_1812
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# Astronomical multivariate table import into AMADA

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Formats and imports multivariate astronomical data tables (e.g., the Guo et al. 2011 mock galaxy catalog, SDSS supernova host properties, or the ZENS catalog) into the AMADA Shiny/R workflow as CSV/TXT files with named, space-separated columns and NA markers for missing values, so downstream PCA, robust PCA, and hierarchical clustering can run.

## When to use

Use when a high-dimensional astronomical catalog must be loaded into AMADA for PCA/clustering exploration and the source table needs conversion to AMADA's CSV/TXT input format with named columns and NA-coded missing entries.

## When NOT to use

- The data are not tabular multivariate measurements (e.g., images, spectra, time series) that cannot be represented as named numeric columns
- Missing entries cannot be recoded as NA, so the file would fail AMADA's CSV/TXT import conventions
- The analysis goal is a single low-dimensional regression or classification task rather than exploratory dimensionality reduction and clustering of a high-dimensional table

## Inputs

- Multivariate astronomical catalog in CSV/TXT format with named columns separated by spaces (arbitrary number of columns and rows)
- Missing values encoded as NA
- Optional: preloaded datasets such as the Guo et al. 2011 mock galaxy catalog, Sako et al. 2014 supernova host properties (Table 1), or the ZENS catalog

## Outputs

- Properly parsed data table visible in the AMADA 'Dataset' tab, ready for PCA, robust PCA, and hierarchical clustering diagnostics

## How to apply

Export or reformat the catalog as a CSV/TXT table where columns are named and separated by spaces; the table may contain an arbitrary number of columns and rows, and any missing entries must be marked as NA. In AMADA, either select one of the preloaded datasets (Supernova host properties from Sako et al. 2014 Table 1, the Guo et al. 2011 mock galaxy catalog built on the Millennium/Millennium II simulations, or the ZENS catalog of 141 galaxy groups at 0.05 < z < 0.0585) or upload your own file via the 'Import Dataset' panel. After loading, verify the import by checking the bottom of the 'Import Dataset' panel and viewing the parsed table under the 'Dataset' tab before choosing analysis options such as the correlation method (Pearson, Spearman, or MIC) or the PCA method (Standard vs Robust). This import step is a prerequisite for AMADA's iterative exploration via hierarchical clustering over different correlation matrices and principal components analysis on the original data.

## Related tools

- **AMADA** (Shiny web application that ingests the CSV/TXT table (preloaded or uploaded) and exposes it for PCA, robust PCA, and hierarchical clustering exploration) — https://github.com/RafaelSdeSouza/AMADA_shiny
- **R** (Runtime environment in which the AMADA package and Shiny app are installed and executed)
- **RStudio** (Recommended IDE for installing R packages and running the AMADA Shiny app)
- **shiny** (Required R library to launch the AMADA app via runUrl of the AMADA_shiny repository) — https://github.com/RafaelSdeSouza/AMADA_shiny

## Examples

```
require(shiny); runUrl('https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip')
```

## Evaluation signals

- The bottom of the 'Import Dataset' panel reports the data were properly imported after upload
- Clicking the 'Dataset' tab shows the parsed table with all expected named columns and rows
- Missing entries in the source catalog appear as NA rather than breaking the parse or being silently coerced
- Column count and row count in the imported table match the arbitrary dimensions of the source CSV/TXT file
- Downstream AMADA controls (correlation method, PCA method, number of PCs) can be applied and plots update after clicking 'Make it so', confirming the data are usable

## Limitations

- Data must be imported as CSV/TXT with named, space-separated columns; other formats or delimiters are not accepted
- Missing data must be explicitly marked as NA, otherwise the import may fail or be mishandled
- Preloaded datasets follow the nomenclature of their source articles, so users are advised to consult the original papers (Sako et al. 2014; Guo et al. 2011; Carollo et al. 2012 / Cibinel et al. 2012, 2013) to interpret the columns

## Evidence
