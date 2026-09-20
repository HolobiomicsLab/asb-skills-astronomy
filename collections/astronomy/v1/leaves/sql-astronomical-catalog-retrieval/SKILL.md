---
name: sql-astronomical-catalog-retrieval
description: Use when a pipeline needs raw photometric source data from SDSS (optionally
  cross-matched with WISE) and the authors provide ready-made SQL queries (e.g. the
  'sql_query_casjobs' file) to fetch them via SkyServer CasJobs.
when_to_use_negative:
- Your input is already a processed feature table or catalogue extract — skip retrieval
  and go directly to 'create_data.py' feature engineering.
- Your data are non-tabular (e.g. images or spectra) — the README states the architecture
  'was built to deal with tabular data'.
- Your catalogue is a different survey (e.g. radio fluxes/magnitudes) — the SQL and
  tuned hyper-parameters are specific to SDSS DR15+WISE and would need re-derivation
  and re-optimisation.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: SDSS SkyServer CasJobs
  role: Web interface where the authors' SQL queries are run to retrieve the SDSS
    DR15+WISE photometric catalogue
  repo: https://skyserver.sdss.org/CasJobs/
- name: Python
  role: Language in which the retrieved table is handled during subsequent feature
    engineering ('create_data.py')
  repo: https://github.com/pedro-acunha/SHEEP
- name: SHEEP
  role: Downstream pipeline that consumes the retrieved photometry for photo-z estimation
    and galaxy/QSO/star classification
  repo: https://github.com/pedro-acunha/SHEEP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:d52182e56f58fd4c1be681183c6ad9caf37efa1a758e9d61993cb2ce43df9456
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CatBoost
  - Jupyter Notebook
  - LightGBM
  - Python
  - SDSS SkyServer CasJobs
  - SHEEP
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# sql-astronomical-catalog-retrieval

## Summary

Retrieve large astronomical photometric catalogues (e.g. SDSS DR15+WISE photometry of ~3.5 million sources) by executing the authors' SQL queries in the SDSS SkyServer CasJobs interface, as the data-acquisition step of the SHEEP classification pipeline.

## When to use

Use when a pipeline needs raw photometric source data from SDSS (optionally cross-matched with WISE) and the authors provide ready-made SQL queries (e.g. the 'sql_query_casjobs' file) to fetch them via SkyServer CasJobs.

## When NOT to use

- Your input is already a processed feature table or catalogue extract — skip retrieval and go directly to 'create_data.py' feature engineering.
- Your data are non-tabular (e.g. images or spectra) — the README states the architecture 'was built to deal with tabular data'.
- Your catalogue is a different survey (e.g. radio fluxes/magnitudes) — the SQL and tuned hyper-parameters are specific to SDSS DR15+WISE and would need re-derivation and re-optimisation.

## Inputs

- SQL query file 'sql_query_casjobs' from the SHEEP repository (pedro-acunha/SHEEP)
- SDSS SkyServer CasJobs account and target DR15 context

## Outputs

- Tabular photometric catalogue of ~3.5 million SDSS+WISE astronomical sources (magnitudes/fluxes for galaxies, quasars, and stars) for feature engineering

## How to apply

Open the SDSS SkyServer CasJobs interface at https://skyserver.sdss.org/CasJobs/ and execute the SQL queries supplied in the repository file 'sql_query_casjobs' to build the training table. For this paper the queries retrieve SDSS DR15 photometry cross-matched with WISE for ~3.5 million astronomical sources, which is then saved locally for downstream feature engineering in 'create_data.py'. Re-run the retrieval (or adapt the SQL) whenever the catalogue region, magnitude cuts, or cross-match requirements change, since downstream models are optimised specifically for SDSS DR15+WISE. The retrieved tabular photometry feeds the SHEEP photo-z estimation and multi-class/one-vs-all classifiers, so column completeness directly conditions classification quality.

## Related tools

- **SDSS SkyServer CasJobs** (Web interface where the authors' SQL queries are run to retrieve the SDSS DR15+WISE photometric catalogue) — https://skyserver.sdss.org/CasJobs/
- **Python** (Language in which the retrieved table is handled during subsequent feature engineering ('create_data.py')) — https://github.com/pedro-acunha/SHEEP
- **SHEEP** (Downstream pipeline that consumes the retrieved photometry for photo-z estimation and galaxy/QSO/star classification) — https://github.com/pedro-acunha/SHEEP

## Examples

```
python create_data.py  # after running sql_query_casjobs queries in https://skyserver.sdss.org/CasJobs/ to fetch the SDSS DR15+WISE catalogue
```

## Evaluation signals

- Retrieved row count is in the millions (paper benchmark: ~3.5 million sources with SDSS and WISE photometry).
- The table contains both SDSS and WISE photometric columns, since the classifiers are optimised for SDSS DR15+WISE.
- Downstream photo-z estimation (photo_z.py chain-regressor) and classification run without missing-column errors on the retrieved schema.
- Per-class F1-scores of the full pipeline land near the reported benchmarks: 0.992 (galaxies), 0.967 (quasars), 0.985 (stars).

## Limitations

- The models (and hence the tuned hyper-parameters) are optimised for SDSS DR15+WISE; using the retrieved data with a different survey or feature set requires re-running FLAML hyper-parameter optimisation.
- CasJobs queries are survey-specific — adapting to other tabular data (e.g. radio fluxes/magnitudes) requires modifying the SQL and feature engineering.

## Evidence

- [readme] README data retrieval: "To create the data you can use the SQL queries provided in the file "sql_query_casjobs". The data can be retrieved from https://skyserver.sdss.org/CasJobs/ ."
- [abstract] Abstract dataset description: "using a data set composed of SDSS and WISE photometry of 3.5 million astronomical sources"
- [methods] Methods hyper-parameter caveat: "the models are optimised for SDSS DR15+WISE."
- [intro] Intro scope statement: "It was built to deal with tabular data, while allowing the use of sparse data."
