---
name: galaxy-photometric-data-import
description: Use when a tabular catalog of galaxy or supernova host-galaxy photometric
  properties must be loaded into AMADA for clustering/PCA exploration, and the table
  needs formatting as named, space-separated CSV/TXT with NA for missing data.
when_to_use_negative:
- The input contains missing values but the downstream analysis requires the MIC correlation
  method, which does not support NA values
- The data are not tabular photometric properties (e.g., spectra, images) or lack
  named, space-separated columns, since AMADA requires CSV/TXT with named columns
  separated by spaces
edam_operation: http://edamontology.org/operation_1812
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0003
tools:
- name: AMADA
  role: Shiny application providing the 'Import Dataset' panel for loading user CSV/TXT
    tables or bundled galaxy datasets
  repo: https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip
- name: shiny
  role: R framework used to run the AMADA app via runUrl of the AMADA_shiny archive
- name: R
  role: Runtime environment in which AMADA and its Shiny app are installed and executed
  repo: http://www.r-project.org
- name: RStudio
  role: Recommended IDE for installing and running the AMADA R package and Shiny app
  repo: http://www.rstudio.com
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.006
    title: AMADA—Analysis of multidimensional astronomical datasets
  source_content_hash: sha256:66b91694b44f01bd012f6b77463c0028e23b35ec2dca847619907832eb589b71
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - AMADA
  - R
  - RStudio
  - ape
  - corrplot
  - fpc
  - shiny
  edam_operation: http://edamontology.org/operation_1812
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# galaxy-photometric-data-import

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Import galaxy/host-galaxy photometric property tables (e.g., SDSS Type Ia and II Supernova host-galaxy properties from Sako et al. 2014) into the AMADA Shiny workflow as named, space-separated CSV/TXT files with missing entries marked NA, so that hierarchical clustering and PCA can be run on high-dimensional astronomical datasets.

## When to use

Use when a tabular catalog of galaxy or supernova host-galaxy photometric properties must be loaded into AMADA for clustering/PCA exploration, and the table needs formatting as named, space-separated CSV/TXT with NA for missing data.

## When NOT to use

- The input contains missing values but the downstream analysis requires the MIC correlation method, which does not support NA values
- The data are not tabular photometric properties (e.g., spectra, images) or lack named, space-separated columns, since AMADA requires CSV/TXT with named columns separated by spaces

## Inputs

- CSV/TXT file with named, space-separated columns of galaxy/host-galaxy photometric properties (arbitrary number of rows and columns)
- Missing entries marked as NA
- Optionally, a bundled dataset: Supernova host properties (Sako et al. 2014), mock galaxy catalog (Guo et al. 2011), or ZENS catalog (Cibinel et al. 2012/2013)

## Outputs

- Validated in-memory data table visible under the AMADA 'Dataset' tab, ready for correlation, hierarchical clustering, and PCA analyses

## How to apply

Format the catalog as CSV/TXT with named columns separated by spaces; an arbitrary number of rows and columns is allowed, and missing entries must be marked as NA. In the AMADA Shiny app, use the 'Import Dataset' panel to upload the file or select a bundled dataset such as the Supernova host properties (Table 1, Sako et al. 2014, SDSS multi-band photometry), the Guo et al. 2011 mock galaxy catalog, or the ZENS catalog; verify the import via the check at the bottom of the 'Import Dataset' panel and inspect the data under the 'Dataset' tab. Consult the source articles/catalogs for column meanings, since datasets follow the nomenclature of their source articles. Note that the Maximum Information Coefficient (MIC) correlation option does not support NA values, so missing-data handling must be decided at import time. Once validated, the imported table feeds directly into the correlation-matrix, hierarchical clustering (correlation distance d(x,y)=1-|corr(x,y)|), and PCA analyses.

## Related tools

- **AMADA** (Shiny application providing the 'Import Dataset' panel for loading user CSV/TXT tables or bundled galaxy datasets) — https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip
- **shiny** (R framework used to run the AMADA app via runUrl of the AMADA_shiny archive)
- **R** (Runtime environment in which AMADA and its Shiny app are installed and executed) — http://www.r-project.org
- **RStudio** (Recommended IDE for installing and running the AMADA R package and Shiny app) — http://www.rstudio.com

## Examples

```
require(shiny); runUrl('https://github.com/RafaelSdeSouza/AMADA_shiny/archive/master.zip')
```

## Evaluation signals

- The check at the bottom of the 'Import Dataset' panel indicates the data were properly imported
- The imported table is visible under the 'Dataset' tab on the main page and matches the input file's rows/columns
- Columns are named and separated by spaces, with any missing entries represented as NA (required for non-MIC analyses)
- The loaded table proceeds without error into correlation-matrix computation, hierarchical clustering, and PCA steps

## Limitations

- MIC (Maximum Information Coefficient) correlation does not support NA values, so datasets with missing data cannot use that method
- Data must be imported as CSV/TXT with named, space-separated columns; other delimiters or unnamed columns are not supported
- Column meanings follow the nomenclature of the source articles; the user is advised to check the original articles/catalogs to understand the variables

## Evidence

- [methods] Supernova host dataset provenance: [evidence span withheld — anchor status: not_found]
