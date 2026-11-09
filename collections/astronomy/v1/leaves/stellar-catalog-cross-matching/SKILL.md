---
name: stellar-catalog-cross-matching
description: Use when you have a survey's observed target list and an external reference
  catalog of known objects of the same class (e.g. M dwarfs within 10 pc) and need
  the fraction of reference objects actually included in the survey, restricted to
  those observable from a given site.
when_to_use_negative:
- The task is computing radial velocities or activity indicators from spectra — that
  is the CCF workflow handled by tools like raccoon, not catalog completeness analysis
- The reference catalog is not an independent census (e.g. you want the planet occurrence
  rate per star, 1.44 ± 0.20 planets/star, which uses different methodology)
- The survey sample cannot be expressed as a coordinate-bearing target list, so angular
  cross-matching is not possible
edam_operation: http://edamontology.org/operation_2238
tools:
- name: Python
  role: pandas is used to load both catalogs, perform the coordinate-based cross-match
    with a small angular tolerance, apply the declination accessibility filter, and
    compute the fraction
- name: CARMENES
  role: 'Source of the survey target sample being tested for completeness: the CARMENES
    instrument at the 3.5 m Calar Alto telescope provided the DR1 GTO sample of 362
    targets'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202244879
    title: The CARMENES search for exoplanets around M dwarfs
  source_content_hash: sha256:81032d6404c8449b6e325ef3325c1ca0eae742c01d0301b0d3da4902deb27abe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CARMENES
  - Python
  edam_operation: http://edamontology.org/operation_2238
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# stellar-catalog-cross-matching

## Summary

Quantifies how complete a spectroscopic survey sample (e.g. the 362-target CARMENES GTO sample) is relative to a reference census of nearby stars, by angular cross-matching two catalogs and computing an inclusion fraction after applying a site-accessibility (declination) filter.

## When to use

Use when you have a survey's observed target list and an external reference catalog of known objects of the same class (e.g. M dwarfs within 10 pc) and need the fraction of reference objects actually included in the survey, restricted to those observable from a given site.

## When NOT to use

- The task is computing radial velocities or activity indicators from spectra — that is the CCF workflow handled by tools like raccoon, not catalog completeness analysis
- The reference catalog is not an independent census (e.g. you want the planet occurrence rate per star, 1.44 ± 0.20 planets/star, which uses different methodology)
- The survey sample cannot be expressed as a coordinate-bearing target list, so angular cross-matching is not possible

## Inputs

- Survey target catalog (CARMENES DR1 stellar sample, 362 GTO targets, with stellar coordinates)
- Reference catalog of known M dwarfs within 10 pc (with coordinates)
- Site accessibility definition (observable declination range from Calar Alto, Almería, Spain)

## Outputs

- Inclusion fraction of accessible known M dwarfs present in the survey sample (~0.70 for CARMENES)
- Matched star list (known M dwarfs observed by the survey)
- Unmatched star list (known accessible M dwarfs not observed)
- Combined results file, e.g. survey_mdwarf_completeness.csv

## How to apply

Load the survey catalog (here, CARMENES DR1 with its 362 GTO targets from the 2016–2020 survey) and the reference catalog of known M dwarfs within 10 pc into pandas. Cross-match the two catalogs by stellar coordinates using a small angular tolerance to identify which known nearby M dwarfs were observed. Filter the reference catalog to objects accessible from the observing site — for Calar Alto Observatory in Almería, Spain, this means restricting to the observable declination range — to define the denominator. Compute the inclusion fraction as matched accessible stars divided by total accessible known M dwarfs; in the CARMENES case this reproduces the reported ~70%. Save the fraction together with the matched and unmatched star lists so the match decisions remain auditable.

## Related tools

- **Python** (pandas is used to load both catalogs, perform the coordinate-based cross-match with a small angular tolerance, apply the declination accessibility filter, and compute the fraction)
- **CARMENES** (Source of the survey target sample being tested for completeness: the CARMENES instrument at the 3.5 m Calar Alto telescope provided the DR1 GTO sample of 362 targets)

## Examples

```
frac = len(matched) / len(accessible); matched.to_csv('survey_mdwarf_completeness.csv', index=False)
```

## Evaluation signals

- The computed inclusion fraction reproduces the published value of about 70% of known M dwarfs within 10 pc accessible from Calar Alto
- The denominator equals the number of reference-catalog stars within 10 pc AND inside the Calar Alto observable declination range
- Every matched star appears in both catalogs within the chosen angular tolerance, and the survey side is capped at the 362 GTO targets
- matched + unmatched lists partition the accessible reference sample with no duplicates or dropped stars
- Output CSV contains the fraction plus the matched and unmatched star lists as specified

## Limitations

- The result depends on the reference census of 'known' M dwarfs within 10 pc; new discoveries change the denominator and hence the fraction
- The accessibility filter is site-specific (Calar Alto declination range) and not transferable to other observatories without redefining the cut
- The angular tolerance used in the cross-match is a decision point; too large or too small a tolerance can introduce false matches or miss true counterparts

## Evidence

- [abstract] About 70% of all known M dwarfs within 10 pc accessible from Calar Alto were included in the survey: "about 70% of all known M dwarfs within 10 pc and accessible from Calar Alto were included"
- [abstract] CARMENES DR1 defines the survey sample: "We present the CARMENES Data Release 1 (DR1), which makes public all observations obtained during the GTO of the CARMENES survey."
- [abstract] Survey sample size and timeframe: "spanning from 2016 to 2020, during which 19 633 spectra for a sample of 362 targets were collected"
- [other] Cross-match and fraction workflow from the source task card: [evidence span withheld — anchor status: not_found]
- [other] Calar Alto accessibility filter from the source task card: [evidence span withheld — anchor status: not_found]
