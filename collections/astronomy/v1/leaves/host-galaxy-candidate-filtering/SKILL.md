---
name: host-galaxy-candidate-filtering
description: Use when preparing galaxy-only catalog samples around transient sky localisations
  (e.g. from a Transient_sources.ecsv file) prior to computing chance-alignment probabilities,
  and stars must be removed from Pan-STARRS or HSC catalog data.
when_to_use_negative:
- The catalog is HSC and the provided extended flag is already sufficient — apply
  the flag directly instead of colour criteria.
- The analysis targets star candidates rather than host galaxies; this skill discards
  stars by design.
- The catalog is AllWISE or GLADE, for which this Pan-STARRS colour-criteria separation
  is not the described identification step.
tools:
- name: galclaim
  role: Host-galaxy association software whose pre-computation step identifies galaxies
    in the used catalogs, applying Pan-STARRS colour criteria to discard stars
  repo: github.com/jgducoin/galclaim
- name: astroquery
  role: Dependency used to query the Pan-STARRS catalog around each transient source
    position
- name: astropy
  role: Loads Transient_sources.ecsv inputs and writes the galaxy-only panstarrs_galaxies.ecsv
    output
- name: numpy
  role: Computes the Pan-STARRS colour indices (g-r, r-i, i-z) from retrieved point-source
    magnitudes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.softx.2023.101582
    title: 'Galclaim: A tool to identify host galaxy of astrophysical transient sources'
  source_content_hash: sha256:019fd15d3277a2e62b4536a1a4a397c73b4781e60461c9d0a7e0b9edcf017007
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - astropy
  - astroquery
  - galclaim
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# host-galaxy-candidate-filtering

## Summary

Filters galaxy candidates from astrophysical catalog data (Pan-STARRS, HSC, AllWISE, GLADE) before chance-alignment computation in galclaim, discarding stellar contamination. In Pan-STARRS this is done by applying colour criteria that separate the stellar locus from extended sources in colour-colour space.

## When to use

Use when preparing galaxy-only catalog samples around transient sky localisations (e.g. from a Transient_sources.ecsv file) prior to computing chance-alignment probabilities, and stars must be removed from Pan-STARRS or HSC catalog data.

## When NOT to use

- The catalog is HSC and the provided extended flag is already sufficient — apply the flag directly instead of colour criteria.
- The analysis targets star candidates rather than host galaxies; this skill discards stars by design.
- The catalog is AllWISE or GLADE, for which this Pan-STARRS colour-criteria separation is not the described identification step.

## Inputs

- Transient source positions in ECSV format (Transient_sources.ecsv)
- Pan-STARRS catalog query results with point-source magnitudes (g, r, i, z bands)
- HSC catalog records with extended flag (alternative galaxy-identification route)

## Outputs

- panstarrs_galaxies.ecsv — galaxy-only records in ECSV format preserving original catalog columns
- Dedicated output file of nearby bright galaxy properties from the RC3 pre-check

## How to apply

Load transient source positions from Transient_sources.ecsv with astropy and query the Pan-STARRS catalog around each position via astroquery. Compute Pan-STARRS colour indices (g-r, r-i, i-z) from the retrieved point-source magnitudes using numpy, then classify each object as star or galaxy by applying the paper's colour criteria that separate the stellar locus from extended sources in colour-colour space. Discard all star-classified objects and write surviving galaxy-only records to panstarrs_galaxies.ecsv with astropy, preserving the original catalog columns. Rationale: galclaim performs this identification of galaxies in the used catalogs before any chance-alignment computation, because only galaxies are of interest as host candidates. For HSC data the alternative approach is simpler — rely on the catalog's provided extended flag — while the code also pre-checks nearby bright galaxies via the RC3 catalog and saves their properties to a dedicated output file.

## Related tools

- **galclaim** (Host-galaxy association software whose pre-computation step identifies galaxies in the used catalogs, applying Pan-STARRS colour criteria to discard stars) — github.com/jgducoin/galclaim
- **astroquery** (Dependency used to query the Pan-STARRS catalog around each transient source position)
- **astropy** (Loads Transient_sources.ecsv inputs and writes the galaxy-only panstarrs_galaxies.ecsv output)
- **numpy** (Computes the Pan-STARRS colour indices (g-r, r-i, i-z) from retrieved point-source magnitudes)

## Examples

```
python ./src/Association_pval.py --transient_file ./Transient_sources.ecsv --catalogs AllWISE Pan-STARRS HSC GLADE
```

## Evaluation signals

- panstarrs_galaxies.ecsv exists and is readable with astropy
- No retained row fails the galaxy colour criteria — all rows satisfy the galaxy-side colour classification
- Original Pan-STARRS catalog columns are preserved in the output ECSV records
- No star-classified object survives in the galaxy-only output file
- HSC-based filtering retains only objects flagged as extended

## Limitations

- Colour criteria are specific to the Pan-STARRS catalog; other catalogs (HSC, AllWISE, GLADE) need different identification steps.
- The code does not plot or retain objects with pval>0.95, so candidate lists may be empty and 'the output ecsv file is not empty when at least one compatible object is found'.
- No changelog found for source repository github.com/jgducoin/galclaim, so filter-criteria revisions across versions cannot be tracked from context.

## Evidence

- [abstract] In the Pan-STARRS catalog, we discard stars from galaxies applying the colour criteria: "In the Pan-STARRS catalog, we discard stars from galaxies applying the colour criteria"
- [abstract] we should identify galaxies in the used catalogs: "before any computation, As we are interested on galaxies only, we should identify galaxies in the used catalogs."
- [intro] The current version of the code allows to use Pan-STARRS, HSC, AllWISE and GLADE catalogs.: "The current version of the code allows to use Pan-STARRS, HSC, AllWISE and GLADE catalogs."
- [abstract] In the HSC we simply used the extended flag provided.: "In the HSC we simply used the extended flag provided."
- [readme] the output ecsv file is not empty when at least one compatible object is found: "Please check the results in the result directory, the output ecsv file is not empty when at least one compatible object is found."
