---
name: point-source-extended-source-discrimination
description: Use when preprocessing an HSC catalog extract for transient host-galaxy
  association and you need to retain only extended (galaxy) sources, discarding point-like
  (star) sources via the extended flag.
when_to_use_negative:
- Input catalog is Pan-STARRS — stars must be discarded using colour criteria rather
  than an extended flag
- Input catalog is AllWISE or GLADE, which follow different galaxy-identification
  strategies than HSC's extended flag
- Point-like transient sources (not galaxies) are the objects of interest, e.g. pure
  star-catalog work
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: galclaim
  role: Software that consumes the galaxy-only catalog before computing chance-alignment
    probabilities between transient localisations and nearby galaxies
  repo: https://gitlab.in2p3.fr/ducoin/galclaim.git
- name: astropy
  role: Loads the raw HSC ECSV extract, applies the filter, and writes the galaxy-only
    ECSV output preserving all columns
- name: numpy
  role: Boolean masking of table rows to retain only sources whose extended flag indicates
    a galaxy
- name: Association_pval.py
  role: galclaim entry-point script invoked with --catalogs HSC to run the downstream
    association computation on the filtered catalog
  repo: https://gitlab.in2p3.fr/ducoin/galclaim.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.softx.2023.101582
    title: 'Galclaim: A tool to identify host galaxy of astrophysical transient sources'
  source_content_hash: sha256:e4dd6b4f9426e165c263269f130ab54948e93d1d09390d2dea3ef29b36d114b0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Association_pval.py
  - Galclaim
  - Python
  - astropy
  - galclaim
  - numpy
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# point-source-extended-source-discrimination

## Summary

A star/galaxy separation step used by galclaim to keep only galaxies from an input catalog before chance-alignment probability computation; for HSC this is done by filtering rows on the catalog-provided extended flag.

## When to use

Use when preprocessing an HSC catalog extract for transient host-galaxy association and you need to retain only extended (galaxy) sources, discarding point-like (star) sources via the extended flag.

## When NOT to use

- Input catalog is Pan-STARRS — stars must be discarded using colour criteria rather than an extended flag
- Input catalog is AllWISE or GLADE, which follow different galaxy-identification strategies than HSC's extended flag
- Point-like transient sources (not galaxies) are the objects of interest, e.g. pure star-catalog work

## Inputs

- Raw HSC catalog extract as an astropy-readable ECSV table containing a per-source extended-flag (morphological classification) column

## Outputs

- Galaxy-only ECSV file (e.g. 'hsc_galaxies.ecsv') preserving all original columns, containing only rows whose extended flag is set

## How to apply

Load the raw HSC catalog extract into an astropy Table and read the per-source morphological classification column (extended flag). Apply a numpy boolean mask that keeps only rows whose extended flag indicates an extended (galaxy) source, discarding point-like sources. Write the galaxy-only table with astropy to an ECSV file (e.g. 'hsc_galaxies.ecsv') in the same ECSV format as the input, preserving all original columns. Rationale: the galclaim paper states that for HSC, 'we simply used the extended flag provided' with no additional star/galaxy separation criteria — unlike Pan-STARRS, which requires colour criteria. Note that Pan-STARRS, AllWISE, and GLADE use different star/galaxy discrimination strategies, so this flag-based approach is HSC-specific.

## Related tools

- **galclaim** (Software that consumes the galaxy-only catalog before computing chance-alignment probabilities between transient localisations and nearby galaxies) — https://gitlab.in2p3.fr/ducoin/galclaim.git
- **astropy** (Loads the raw HSC ECSV extract, applies the filter, and writes the galaxy-only ECSV output preserving all columns)
- **numpy** (Boolean masking of table rows to retain only sources whose extended flag indicates a galaxy)
- **Association_pval.py** (galclaim entry-point script invoked with --catalogs HSC to run the downstream association computation on the filtered catalog) — https://gitlab.in2p3.fr/ducoin/galclaim.git

## Examples

```
python ./src/Association_pval.py --transient_file ./Transient_sources.ecsv --catalogs HSC
```

## Evaluation signals

- Output file 'hsc_galaxies.ecsv' exists and is readable by astropy
- The filtered table retains the original extended-flag column
- Every row in the output has the extended flag set (galaxy sources); no point-like rows remain
- Output preserves all original columns from the input catalog in the same ECSV format
- Row count of output equals the number of rows in the input whose extended flag is set

## Limitations

- No additional star/galaxy separation criteria are applied for HSC — quality relies entirely on the catalog's extended flag
- The extended-flag approach is catalog-specific: Pan-STARRS instead requires applying colour criteria to discard stars
- This step only prepares the catalog; it does not itself compute chance-alignment probabilities or redshift compatibility

## Evidence

- [abstract] In the HSC we simply used the extended flag provided.: "In the HSC we simply used the extended flag provided."
- [abstract] before any computation, As we are interested on galaxies only, we should identify galaxies in the used catalogs.: "before any computation, As we are interested on galaxies only, we should identify galaxies in the used catalogs."
- [abstract] In the Pan-STARRS catalog, we discard stars from galaxies applying the colour criteria: "In the Pan-STARRS catalog, we discard stars from galaxies applying the colour criteria"
- [methods] python ./src/Association_pval.py --transient_file ./Transient_sources.ecsv --catalogs AllWISE Pan-STARRS HSC GLADE: "python ./src/Association_pval.py --transient_file ./Transient_sources.ecsv --catalogs AllWISE Pan-STARRS HSC GLADE"
