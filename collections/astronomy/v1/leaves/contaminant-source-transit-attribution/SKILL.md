---
name: contaminant-source-transit-attribution
description: Use when a TESS target shows a transit and/or activity signal and you
  must decide, from TESS photometry alone, which nearby Gaia contaminant stars inside
  the aperture can or cannot be the origin of the signal.
when_to_use_negative:
- When the goal is only to quantify the flux contamination itself (flux fractions,
  CROWDSAP/FLFRCSAP metrics, heatmaps/pie charts) rather than attribute the signal
  — that is steps (1)-(2) of TESS-cont, not the dilution/filtering step
- 'When the measured transit depth is NOT corrected for dilution (e.g. raw SAP-like
  photometry) — running with dilution_corr: True would wrongly rescale the depth'
- When TESS photometry alone is insufficient and high-spatial-resolution follow-up
  (ground-based photometry/spectroscopy) is already available to directly confirm
  or discard contaminant origins
edam_operation: http://edamontology.org/operation_3695
tools:
- name: TESS-cont
  role: Python tool whose DILUTION feature performs the dilution correction, per-source
    depth computation, and 100%-threshold filtering of contaminant Gaia DR2/DR3 sources
  repo: https://github.com/castro-gzlz/TESS-cont
- name: ExoFOP
  role: Source of the measured transit depth (e.g. TOI-4479.01, 3471 ppm) fed as td
    into the DILUTION analysis
- name: TESS-PRF
  role: PRF modeling used to compute the flux distributions of nearby sources across
    the TESS TPFs/FFIs that feed the attribution step
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:3168a9e378124335c157d945590e278ddc44b20e0fe5a12273fffff889ac4cf5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castro-gzlz/TESS-cont
  tools:
  - ExoFOP
  - Python
  - TESS-PRF
  - TESS-cont
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# contaminant-source-transit-attribution

## Summary

Step (3) of TESS-cont: given the per-source flux fractions of Gaia DR2/DR3 contaminants inside the TESS photometric aperture, compute the transit depth each nearby star would need to produce the observed signal and discard sources whose implied depth is unphysical (>=100%).

## When to use

Use when a TESS target shows a transit and/or activity signal and you must decide, from TESS photometry alone, which nearby Gaia contaminant stars inside the aperture can or cannot be the origin of the signal.

## When NOT to use

- When the goal is only to quantify the flux contamination itself (flux fractions, CROWDSAP/FLFRCSAP metrics, heatmaps/pie charts) rather than attribute the signal — that is steps (1)-(2) of TESS-cont, not the dilution/filtering step
- When the measured transit depth is NOT corrected for dilution (e.g. raw SAP-like photometry) — running with dilution_corr: True would wrongly rescale the depth
- When TESS photometry alone is insufficient and high-spatial-resolution follow-up (ground-based photometry/spectroscopy) is already available to directly confirm or discard contaminant origins

## Inputs

- Per-source contaminant flux fractions inside the TESS photometric aperture (from TESS-cont PRF computation over TESS TPFs/FFIs, built on Gaia DR2/DR3 sources)
- Measured transit depth in the target's aperture (td) with its unit (td_unit: ppm, ppt, per, or frac), e.g. 3471 ppm from ExoFOP for TOI-4479.01
- dilution_corr flag indicating whether the measured depth is already corrected for dilution (e.g. SPOC)

## Outputs

- Per-contaminant implied transit depths in percent (columns Star, Gaia_ID, transit_depth(%) written to dilution_report.csv)
- Accept/reject verdict per contaminant source (discarded if implied depth >= 100%, otherwise cannot be discarded from TESS photometry alone)

## How to apply

Load the per-source contaminant flux fractions inside the TESS aperture (TESS-cont PRF output) together with the measured transit depth (e.g. TOI-4479.01 at 3471 ppm from ExoFOP), setting td_unit among ppm/ppt/per/frac. Apply dilution_corr: True when the depths come from SPOC PDCSAP photometry, since those depths are already corrected for dilution, which rescales the measured depth by each contaminant's flux fraction inside the aperture. Compute the depth each of the N most contaminant stars (n_sources, default 5) would need to have to produce the observed signal, and compare each inferred depth against the unphysical threshold of 100%: sources yielding depths at or above 100% are discarded as possible signal origins based on TESS photometry alone. Write the attribution table (Star, Gaia_ID, transit_depth(%)) with an accept/reject verdict per source, and interpret depths well below 100% (e.g. the 2-28% range for the six TOI-4479 contaminants) as 'cannot discard' — flagging the need for higher-spatial-resolution follow-up (ground-based photometry or spectroscopy) to confidently reject those sources.

## Related tools

- **TESS-cont** (Python tool whose DILUTION feature performs the dilution correction, per-source depth computation, and 100%-threshold filtering of contaminant Gaia DR2/DR3 sources) — https://github.com/castro-gzlz/TESS-cont
- **ExoFOP** (Source of the measured transit depth (e.g. TOI-4479.01, 3471 ppm) fed as td into the DILUTION analysis)
- **TESS-PRF** (PRF modeling used to compute the flux distributions of nearby sources across the TESS TPFs/FFIs that feed the attribution step)

## Examples

```
python TESS-cont.py TOI-4479_S41.ini  # config sets td: 3471, td_unit: ppm, dilution_corr: True
```

## Evaluation signals

- dilution_report.csv contains the columns Star, Gaia_ID, transit_depth(%) with one row per studied contaminant and an accept/reject verdict per source
- For the TOI-4479 (Sector 41) benchmark with td = 3471 ppm and dilution_corr: True, the report must list six contaminants with implied depths of 1.74%, 6.10%, 8.256%, 11.30%, 24.35% and 27.69%
- All reported implied depths for the benchmark fall in the 2-28% range, i.e. below the 100% unphysical threshold, so no contaminant is discarded from TESS data alone
- Sources whose implied depth reaches or exceeds 100% are classified as discarded (unphysical: a star cannot dim by more than its own flux fraction allows)
- Number of individually studied sources matches the configured n_sources (default 5) parameter in the configuration file

## Limitations

- Depths below 100% mean only that the contaminant cannot be DISCARDED from TESS photometry alone — it is not evidence that the contaminant is the actual signal origin; higher-spatial-resolution observations (ground-based photometry or spectroscopy) are needed to confidently discard those sources
- When analyzing SPOC PDCSAP photometry, the DR2 Gaia catalog must be selected (gaia_catalog: DR2) because TIC/SPOC is stacked to Gaia DR2; discrepancies between DR2 and DR3 can affect the contamination metrics
- The approximate PRF method (method_prf: approximate) is only a first hint; the accurate method should be used for final analyses/publications, since implied depths depend directly on the computed flux fractions

## Evidence

- [intro] identifies the main contaminant Gaia DR2/DR3 sources and computes their contributions, then determines whether any could explain the signals: "The package **(1)** identifies the main contaminant *Gaia* DR2 / DR3 sources, **(2)** computes their contributions to the aperture, and **(3)** determines whether any of these sources could be"
- [readme] By introducing the measured transit depth, and selecting dilution_corr: True ... the observed transit signal could have originated in the six most contaminant stars: "By introducing the measured transit depth, and selecting ```dilution_corr: True``` (since the SPOC transit depths are already corrected for dilution), we find that the observed transit signal"
- [readme] Since those values (2-28%) are lower than the unphysical threshold of 100%, we cannot discard that they could be the origin of the transit signal found based on the TESS data alone: "Since those values (**2-28%**) are lower than the unphysical threshold of 100%, we cannot discard that they could be the origin of the transit signal found based on the TESS data alone"
- [readme] DILUTION configuration parameters: td, td_unit (ppm/ppt/per/frac), dilution_corr default True: "| td | Any number| Measured transit depth in the target's aperture |
| td_unit | ppm, ppt, per, frac | Unit of the measured transit depth |"
- [readme] necessity of higher-spatial resolution observations to confidently discard contaminant sources: "This example reveals the necessity of **higher-spatial resolution observations** (e.g. ground-based photometry or spectroscopy) to confidently **discard those contaminant sources**."
