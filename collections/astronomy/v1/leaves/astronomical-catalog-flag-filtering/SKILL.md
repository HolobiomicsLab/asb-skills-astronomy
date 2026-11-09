---
name: astronomical-catalog-flag-filtering
description: Use when a source catalog already ships a per-source morphological classification
  column (e.g. HSC's extended flag) and only extended (galaxy) sources should be retained
  before downstream computations such as chance-alignment probability estimates.
when_to_use_negative:
- The catalog lacks a pre-computed morphological flag and requires star/galaxy separation
  from scratch — e.g. Pan-STARRS, where galclaim instead discards stars by applying
  colour criteria
- The downstream analysis needs point-like (stellar) sources retained rather than
  discarded
- The flag column semantics are unknown or unreliable, since this skill trusts the
  catalog's own extended flag without cross-validation
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3077
tools:
- name: astropy
  role: loads and writes the catalog as an astropy Table in ECSV format for filtering
    and output
- name: numpy
  role: boolean masking of the extended-flag column to retain galaxy rows
- name: Galclaim
  role: host-galaxy association tool whose HSC galaxy identification relies on this
    extended-flag filtering before chance-alignment computation
  repo: https://gitlab.in2p3.fr/ducoin/galclaim.git
- name: Association_pval.py
  role: galclaim entry-point script that consumes the filtered catalog when run with
    --catalogs HSC
  repo: github.com/jgducoin/galclaim
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.softx.2023.101582
    title: 'Galclaim: A tool to identify host galaxy of astrophysical transient sources'
  source_content_hash: sha256:02575ddd25d25a716cbe1ae622a0f57cc50d718d8775edd84ff76e32ff43e426
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Association_pval.py
  - Galclaim
  - Python
  - astropy
  - numpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# astronomical-catalog-flag-filtering

## Summary

Identifies galaxies in an astronomical catalog by filtering on a pre-computed morphological classification flag, as galclaim does for the HSC catalog where galaxies are selected simply via the catalog-provided extended flag with no additional star/galaxy separation criteria.

## When to use

Use when a source catalog already ships a per-source morphological classification column (e.g. HSC's extended flag) and only extended (galaxy) sources should be retained before downstream computations such as chance-alignment probability estimates.

## When NOT to use

- The catalog lacks a pre-computed morphological flag and requires star/galaxy separation from scratch — e.g. Pan-STARRS, where galclaim instead discards stars by applying colour criteria
- The downstream analysis needs point-like (stellar) sources retained rather than discarded
- The flag column semantics are unknown or unreliable, since this skill trusts the catalog's own extended flag without cross-validation

## Inputs

- Raw HSC catalog extract as an astropy-readable ECSV table containing a per-source extended (morphological classification) flag column

## Outputs

- Galaxy-only ECSV file (e.g. hsc_galaxies.ecsv) retaining all original columns with only rows whose extended flag is set

## How to apply

Load the raw catalog extract (e.g. an HSC cutout) into an astropy Table and locate the per-source morphological classification column — for HSC this is the provided extended flag. Apply a numpy boolean mask over that column, keeping only rows where the flag indicates an extended (galaxy) source and discarding point-like sources; per the paper, for HSC no additional star/galaxy separation criteria (such as the colour criteria used for Pan-STARRS) are applied. Write the galaxy-only table to disk in the same ECSV format as the input (e.g. 'hsc_galaxies.ecsv') using astropy, preserving all original columns so downstream galclaim steps see an unchanged schema.

## Related tools

- **astropy** (loads and writes the catalog as an astropy Table in ECSV format for filtering and output)
- **numpy** (boolean masking of the extended-flag column to retain galaxy rows)
- **Galclaim** (host-galaxy association tool whose HSC galaxy identification relies on this extended-flag filtering before chance-alignment computation) — https://gitlab.in2p3.fr/ducoin/galclaim.git
- **Association_pval.py** (galclaim entry-point script that consumes the filtered catalog when run with --catalogs HSC) — github.com/jgducoin/galclaim

## Examples

```
python ./src/Association_pval.py --transient_file ./Transient_sources.ecsv --catalogs HSC
```

## Evaluation signals

- hsc_galaxies.ecsv exists and is readable by astropy without errors
- The output retains the extended-flag column with its original metadata/schema
- Every row in hsc_galaxies.ecsv has the extended flag set (galaxy sources); no point-like rows remain
- Column set of the output matches the input catalog (all original columns preserved), only row count reduced
- Consistency with the paper's statement that for HSC galaxies are identified simply via the provided extended flag

## Limitations

- Selection relies entirely on the catalog's own extended flag quality; no additional star/galaxy separation criteria are applied, so misclassified sources propagate
- The filter is catalog-specific: Pan-STARRS uses colour criteria instead, so the same flag-based step cannot be reused across catalogs
- Filtering discards point-like sources permanently; if stars are later needed, the raw catalog must be reloaded

## Evidence

- [abstract] HSC extended flag selection: "In the HSC we simply used the extended flag provided."
- [abstract] Galaxy identification requirement: "before any computation, As we are interested on galaxies only, we should identify galaxies in the used catalogs."
- [abstract] Pan-STARRS contrast: "In the Pan-STARRS catalog, we discard stars from galaxies applying the colour criteria"
- [intro] Supported catalogs: "The current version of the code allows to use Pan-STARRS, HSC, AllWISE and GLADE catalogs."
- [readme] Run command: "python ./src/Association_pval.py --transient_file ./Transient_sources.ecsv --catalogs AllWISE Pan-STARRS HSC GLADE"
