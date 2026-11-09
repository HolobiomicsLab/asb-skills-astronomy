---
name: hsc-extended-flag-galaxy-classification
description: Use when preparing an HSC galaxy catalog for transient host-galaxy association
  (e.g. galclaim / Association_pval.py) and you need to keep only extended (galaxy)
  sources, since HSC requires no extra star/galaxy separation beyond its provided
  extended flag.
when_to_use_negative:
- The input catalog is Pan-STARRS, AllWISE, or GLADE — those catalogs require different
  star/galaxy separation (e.g. Pan-STARRS needs colour criteria to discard stars),
  not the HSC extended flag.
- You need finer star/galaxy classification than the catalog-provided flag provides
  — the paper explicitly applies no additional star/galaxy separation criteria for
  HSC.
- The catalog extract is not HSC and lacks an extended flag column.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Galclaim
  role: Host pipeline that consumes the galaxy-only HSC catalog before computing chance-alignment
    probabilities between transient localisations and nearby galaxies
  repo: github.com/jgducoin/galclaim
- name: astropy
  role: Reads the HSC ECSV extract into a Table and writes the filtered galaxy-only
    table back to ECSV preserving all columns
- name: numpy
  role: Provides the boolean masking used to retain only rows whose extended flag
    marks an extended source
- name: Association_pval.py
  role: galclaim entry script run with --catalogs HSC that performs the association
    computation on the filtered galaxy table
  repo: github.com/jgducoin/galclaim
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.softx.2023.101582
    title: 'Galclaim: A tool to identify host galaxy of astrophysical transient sources'
  source_content_hash: sha256:f2603a59fcb24489534b5f56450156ac676b1e1137c8163f82d65f1d62a0eb21
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Association_pval.py
  - Galclaim
  - Python
  - astropy
  - numpy
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# hsc-extended-flag-galaxy-classification

## Summary

Identifies galaxies in the Hyper Suprime-Cam (HSC) catalog by filtering on the per-source extended (morphological) flag provided with the catalog, discarding point-like sources before galclaim computes chance-alignment probabilities with transient localisations.

## When to use

Use when preparing an HSC galaxy catalog for transient host-galaxy association (e.g. galclaim / Association_pval.py) and you need to keep only extended (galaxy) sources, since HSC requires no extra star/galaxy separation beyond its provided extended flag.

## When NOT to use

- The input catalog is Pan-STARRS, AllWISE, or GLADE — those catalogs require different star/galaxy separation (e.g. Pan-STARRS needs colour criteria to discard stars), not the HSC extended flag.
- You need finer star/galaxy classification than the catalog-provided flag provides — the paper explicitly applies no additional star/galaxy separation criteria for HSC.
- The catalog extract is not HSC and lacks an extended flag column.

## Inputs

- Raw HSC catalog extract in ECSV format (astropy-readable), containing the extended flag column per source
- Transient source list (Transient_sources.ecsv) for the subsequent galclaim association step

## Outputs

- Galaxy-only HSC table (e.g. hsc_galaxies.ecsv) in the same ECSV format as the input, retaining all original columns including the extended flag

## How to apply

Load the raw HSC catalog extract into an astropy Table in ECSV format, reading the per-source morphological classification column (the extended flag). Build a numpy boolean mask that retains only rows whose extended flag indicates an extended source, thereby discarding point-like (star-like) detections — the paper states that for HSC 'we simply used the extended flag provided' with no additional colour or morphology criteria, unlike Pan-STARRS which needs colour criteria. Write the filtered galaxy-only table back to ECSV (e.g. 'hsc_galaxies.ecsv') with astropy, preserving all original columns so downstream galclaim computations see the full schema. This filtering happens before any chance-alignment probability computation, since galclaim is 'interested on galaxies only'.

## Related tools

- **Galclaim** (Host pipeline that consumes the galaxy-only HSC catalog before computing chance-alignment probabilities between transient localisations and nearby galaxies) — github.com/jgducoin/galclaim
- **astropy** (Reads the HSC ECSV extract into a Table and writes the filtered galaxy-only table back to ECSV preserving all columns)
- **numpy** (Provides the boolean masking used to retain only rows whose extended flag marks an extended source)
- **Association_pval.py** (galclaim entry script run with --catalogs HSC that performs the association computation on the filtered galaxy table) — github.com/jgducoin/galclaim

## Examples

```
python ./src/Association_pval.py --transient_file ./Transient_sources.ecsv --catalogs AllWISE Pan-STARRS HSC GLADE
```

## Evaluation signals

- hsc_galaxies.ecsv exists and is readable by astropy.Table.read as ECSV
- The output retains the extended-flag column and all original input columns
- Every row in the output has the extended flag set (galaxy sources); no point-like sources remain
- Output row count equals the number of input rows whose extended flag is set — a lossless, no-new-columns filter
- Downstream Association_pval.py runs on the filtered table produce results consistent with the paper's statement that HSC galaxies are identified simply via the provided extended flag

## Limitations

- Accuracy is bounded by the quality of the HSC extended flag itself — no additional star/galaxy separation criteria are applied, so misclassified sources pass through.
- This filter is catalog-specific: Pan-STARRS requires colour criteria instead, so the extended-flag approach cannot be reused on other galclaim-supported catalogs.
- No plot is produced by galclaim if no candidate is found (objects with pval>0.95 are not plotted), so an empty result does not itself indicate a filtering error.

## Evidence

- [abstract] For the HSC catalog, galclaim identifies galaxies by simply using the extended flag provided with the catalog, with no additional star/galaxy separation criteria applied.: [evidence span withheld — anchor status: not_found]
- [abstract] In the HSC we simply used the extended flag provided.: "In the HSC we simply used the extended flag provided."
- [abstract] Before any computation, only galaxies are of interest.: "before any computation, As we are interested on galaxies only, we should identify galaxies in the used catalogs."
- [readme] README: catalogs supported and their identification strategies.: "The current version of the code allows to use Pan-STARRS, HSC, AllWISE and GLADE catalogs."
