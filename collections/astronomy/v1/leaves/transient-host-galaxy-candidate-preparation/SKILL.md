---
name: transient-host-galaxy-candidate-preparation
description: Use when a transient localization analysis needs galaxy candidates extracted
  from a survey catalog (e.g., HSC, Pan-STARRS, AllWISE, GLADE) before computing chance-alignment
  probabilities with galclaim.
when_to_use_negative:
- The catalog already provides a trusted galaxy-only subset, so re-filtering would
  discard valid sources or duplicate work.
- The target catalog is Pan-STARRS but photometric colours are missing, making the
  colour-criteria star discard inapplicable as described.
- The task is the chance-alignment probability computation itself (Association_pval.py)
  rather than the upstream galaxy candidate preparation.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: Galclaim
  role: The parent software whose chance-alignment workflow requires galaxy-only catalog
    inputs; galaxy identification happens before any computation in galclaim
  repo: https://gitlab.in2p3.fr/ducoin/galclaim.git
- name: astropy
  role: Loads the raw catalog extract as an astropy Table and writes the filtered
    galaxy-only table to ECSV format
- name: numpy
  role: Provides boolean masking to retain only rows whose extended flag indicates
    an extended (galaxy) source
- name: Association_pval.py
  role: galclaim entry-point script that consumes the prepared transient file and
    catalogs downstream of the galaxy filtering step
  repo: https://gitlab.in2p3.fr/ducoin/galclaim.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.softx.2023.101582
    title: 'Galclaim: A tool to identify host galaxy of astrophysical transient sources'
  source_content_hash: sha256:0fec5c17fa4d3e1332f4b454e717c97c5daf995e213d84683131b7efb66f3b9e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Association_pval.py
  - Galclaim
  - Python
  - astropy
  - numpy
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# transient-host-galaxy-candidate-preparation

## Summary

Prepares galaxy-only candidate lists from astronomical catalogs (HSC, Pan-STARRS, AllWISE, GLADE) before galclaim computes chance-alignment probabilities, since only galaxies can be host galaxies of astrophysical transients.

## When to use

Use when a transient localization analysis needs galaxy candidates extracted from a survey catalog (e.g., HSC, Pan-STARRS, AllWISE, GLADE) before computing chance-alignment probabilities with galclaim.

## When NOT to use

- The catalog already provides a trusted galaxy-only subset, so re-filtering would discard valid sources or duplicate work.
- The target catalog is Pan-STARRS but photometric colours are missing, making the colour-criteria star discard inapplicable as described.
- The task is the chance-alignment probability computation itself (Association_pval.py) rather than the upstream galaxy candidate preparation.

## Inputs

- Raw HSC catalog extract as an ECSV astropy Table with a per-source morphological classification column (extended flag)
- Pan-STARRS catalog rows with photometric colour information for star/galaxy colour-criteria separation
- Transient source list (Transient_sources.ecsv format)

## Outputs

- Galaxy-only catalog table, e.g. 'hsc_galaxies.ecsv' in ECSV format preserving all original columns
- Filtered table consumed by galclaim's Association_pval.py for chance-alignment pval computation

## How to apply

Load the raw catalog extract into an astropy Table and apply the catalog-specific star/galaxy separation before running Association_pval.py. For HSC, filter with a numpy boolean mask on the per-source extended flag, retaining only extended (galaxy) rows — the paper states no additional separation criteria are applied for HSC. For Pan-STARRS, discard stars from galaxies by applying the catalog's colour criteria instead. Write the filtered galaxy-only table back to ECSV (e.g., 'hsc_galaxies.ecsv') preserving all original columns, then pass it to the galclaim workflow. The rationale is that chance-alignment host identification is meaningless for point-like stellar sources, so galaxies must be identified in the used catalogs before any computation.

## Related tools

- **Galclaim** (The parent software whose chance-alignment workflow requires galaxy-only catalog inputs; galaxy identification happens before any computation in galclaim) — https://gitlab.in2p3.fr/ducoin/galclaim.git
- **astropy** (Loads the raw catalog extract as an astropy Table and writes the filtered galaxy-only table to ECSV format)
- **numpy** (Provides boolean masking to retain only rows whose extended flag indicates an extended (galaxy) source)
- **Association_pval.py** (galclaim entry-point script that consumes the prepared transient file and catalogs downstream of the galaxy filtering step) — https://gitlab.in2p3.fr/ducoin/galclaim.git

## Examples

```
python ./src/Association_pval.py --transient_file ./Transient_sources.ecsv --catalogs AllWISE Pan-STARRS HSC GLADE
```

## Evaluation signals

- hsc_galaxies.ecsv exists and is readable by astropy
- The filtered table retains the extended-flag column from the original catalog
- Every row in hsc_galaxies.ecsv has the extended flag set (galaxy sources); no point-like sources remain
- The output is written in the same ECSV format as the input and preserves all original columns
- The behavior matches the paper's statement that for HSC, galaxies are identified simply via the provided extended flag with no additional star/galaxy separation criteria

## Limitations

- The method is catalog-specific: HSC uses only the provided extended flag, whereas Pan-STARRS requires colour criteria to discard stars — applying the HSC flag-only approach to Pan-STARRS is not what the paper describes.
- For HSC, no additional star/galaxy separation criteria are applied, so misclassified sources in the extended flag pass through unfiltered.
- galclaim's current version supports only Pan-STARRS, HSC, AllWISE and GLADE catalogs, so the preparation step cannot be extended to other catalogs without code changes.

## Evidence

- [results] For the HSC catalog, galclaim identifies galaxies by simply using the extended flag provided with the catalog, with no additional star/galaxy separation criteria applied.: [evidence span withheld — anchor status: not_found]
- [abstract] In the HSC we simply used the extended flag provided.: "In the HSC we simply used the extended flag provided."
- [abstract] before any computation, As we are interested on galaxies only, we should identify galaxies in the used catalogs.: "before any computation, As we are interested on galaxies only, we should identify galaxies in the used catalogs."
- [abstract] In the Pan-STARRS catalog, we discard stars from galaxies applying the colour criteria: "In the Pan-STARRS catalog, we discard stars from galaxies applying the colour criteria"
- [readme] The current version of the code allows to use Pan-STARRS, HSC, AllWISE and GLADE catalogs.: "The current version of the code allows to use Pan-STARRS, HSC, AllWISE and GLADE catalogs."
