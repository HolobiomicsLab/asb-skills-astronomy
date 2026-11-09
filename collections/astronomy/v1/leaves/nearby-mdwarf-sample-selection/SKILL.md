---
name: nearby-mdwarf-sample-selection
description: Use when auditing the completeness of a spectroscopic survey's target
  list against a reference catalog of known nearby stars — i.
when_to_use_negative:
- The goal is measuring RV precision or deriving radial velocities from raw spectra
  — use the CCF pipeline (raccoon) instead; this skill works only at the catalog/target-list
  level
- The survey sample and the reference census are the same catalog, so the fraction
  is trivially 1 and no cross-match information is gained
- The reference catalog is not distance-limited or not site-filtered — an inclusion
  fraction without the declination/accessibility cut is not comparable to the published
  ~70% value
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Python (pandas)
  role: Load both stellar catalogs and compute the coordinate cross-match, the Calar
    Alto declination accessibility filter, and the inclusion fraction
- name: CARMENES DR1
  role: Public release of all GTO observations (19,633 spectra of 362 targets) providing
    the survey target sample to cross-match against the 10 pc M dwarf census
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202244879
    title: The CARMENES search for exoplanets around M dwarfs
  source_content_hash: sha256:eb304fd387d4daab970920821178fdb6f04a6badff7b297d17d32353637f265f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CARMENES DR1
  - Python
  - Python (pandas)
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# nearby-mdwarf-sample-selection

## Summary

Quantifies how completely a radial-velocity survey's target sample covers the census of known nearby stars: cross-match the survey catalog (e.g. the 362-target CARMENES DR1 GTO sample) against a reference catalog of known M dwarfs within 10 pc, restrict to stars accessible from the observatory site, and compute the inclusion fraction (~70% for CARMENES from Calar Alto).

## When to use

Use when auditing the completeness of a spectroscopic survey's target list against a reference catalog of known nearby stars — i.e. inputs are two stellar catalogs (survey sample + distance-limited census) and the goal is a matched/unmatched breakdown and an inclusion fraction for site-accessible stars.

## When NOT to use

- The goal is measuring RV precision or deriving radial velocities from raw spectra — use the CCF pipeline (raccoon) instead; this skill works only at the catalog/target-list level
- The survey sample and the reference census are the same catalog, so the fraction is trivially 1 and no cross-match information is gained
- The reference catalog is not distance-limited or not site-filtered — an inclusion fraction without the declination/accessibility cut is not comparable to the published ~70% value

## Inputs

- Survey stellar sample catalog (e.g. CARMENES DR1 GTO target list, 362 targets) with stellar coordinates
- Reference catalog of known M dwarfs within 10 pc with coordinates and declinations
- Observatory site definition (latitude/declination accessibility limits, e.g. Calar Alto, Almería, Spain)

## Outputs

- Inclusion fraction (matched accessible known M dwarfs / total accessible known M dwarfs within 10 pc), ~70% for CARMENES
- CSV of matched and unmatched star lists (survey_mdwarf_completeness.csv)

## How to apply

Load the survey's stellar sample catalog (for CARMENES DR1, the 362 GTO targets collected 2016–2020) and the reference catalog of known M dwarfs within 10 pc into pandas. Cross-match the catalogs by stellar coordinates using a small angular tolerance to identify which known nearby M dwarfs were observed. Filter the reference catalog to stars accessible from the observatory site — for Calar Alto this means the observable declination range as seen from Almería, Spain — to define the accessible denominator. Compute the inclusion fraction as matched accessible stars divided by total accessible known M dwarfs, and save the fraction together with the matched and unmatched star lists (e.g. survey_mdwarf_completeness.csv). Validate by checking the fraction reproduces the published value (~70%).

## Related tools

- **Python (pandas)** (Load both stellar catalogs and compute the coordinate cross-match, the Calar Alto declination accessibility filter, and the inclusion fraction)
- **CARMENES DR1** (Public release of all GTO observations (19,633 spectra of 362 targets) providing the survey target sample to cross-match against the 10 pc M dwarf census)

## Examples

```
import pandas as pd; cat = pd.read_csv('carmenes_dr1_targets.csv'); ref = pd.read_csv('mdwarfs_10pc.csv'); frac = cat[cat['accessible']]['matched'].sum() / ref[ref['dec_min'] <= ref['dec']].shape[0]
```

## Evaluation signals

- The computed inclusion fraction reproduces the published ~70% of known M dwarfs within 10 pc accessible from Calar Alto
- The matched list contains at most 362 stars (the size of the CARMENES DR1 GTO sample)
- Every star in the matched list satisfies both the 10 pc distance limit and the Calar Alto declination accessibility cut
- matched_count + unmatched_accessible_count equals the total accessible known-M-dwarf denominator
- Output CSV contains the fraction plus both matched and unmatched star lists as specified in the workflow

## Limitations

- The result is conditioned on the completeness of the reference census: newly discovered M dwarfs within 10 pc after the survey target list was frozen will appear as false 'unmatched' stars
- Accessibility is defined only by the observatory's declination range; other observability constraints (airmass, brightness, binarity) are not captured by this catalog-level fraction
- The fraction is site-specific — it is valid for Calar Alto (Almería, Spain) and cannot be transferred to observatories at other latitudes without recomputing the denominator

## Evidence

- [abstract] About 70% of all known M dwarfs within 10 pc accessible from Calar Alto were included in the survey: "about 70% of all known M dwarfs within 10 pc and accessible from Calar Alto were included"
- [abstract] CARMENES DR1 sample size (362 GTO targets): "19 633 spectra for a sample of 362 targets were collected"
