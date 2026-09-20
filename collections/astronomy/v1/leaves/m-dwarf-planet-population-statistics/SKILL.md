---
name: m-dwarf-planet-population-statistics
description: Use when you have RV time series (e.g. CARMENES DR1, 238-target subsample)
  and need a per-star planet occurrence rate over a mass–period grid, with completeness
  correction for minimum masses M pl sin i.
when_to_use_negative:
- The analysis goal is deriving radial velocities themselves from spectra — use the
  raccoon CCF pipeline (raccoonccf) instead of the occurrence-rate statistics step.
- Input data are transit or astrometric detections where masses are true masses, not
  M pl sin i — the inclination-completeness treatment of this RV-based method does
  not apply.
- The sample is too small or the completeness correction cannot be characterized per
  target, making grid normalization by star count unreliable.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CARMENES
  role: Instrument supplying the high-accuracy radial-velocity measurements and DR1
    data release (19,633 spectra of 362 targets) on which the occurrence-rate computation
    is based
- name: raccoon
  role: Python package implementing the CCF method to derive the radial velocities
    and activity indicators used as inputs to the population statistics
  repo: https://github.com/mlafarga/raccoon
- name: Python
  role: Language used to compute per-cell occurrence rates with detection-completeness
    correction and normalize by the number of stars
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202244879
    title: The CARMENES search for exoplanets around M dwarfs
  source_content_hash: sha256:a1a3290de37fcb9afe06b4b14fa0d1a3bc2f863ae389789e31459890622e0e65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mlafarga/raccoon
  tools:
  - CARMENES
  - Python
  - raccoon
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# M-dwarf planet population statistics

## Summary

Derives detection-completeness-corrected planet occurrence rates for M dwarfs from radial-velocity survey data, exemplified by the CARMENES DR1 result of 1.44 ± 0.20 planets per star for 1 M⊕ < M pl sin i < 1000 M⊕ and 1 day < P orb < 1000 days.

## When to use

Use when you have RV time series (e.g. CARMENES DR1, 238-target subsample) and need a per-star planet occurrence rate over a mass–period grid, with completeness correction for minimum masses M pl sin i.

## When NOT to use

- The analysis goal is deriving radial velocities themselves from spectra — use the raccoon CCF pipeline (raccoonccf) instead of the occurrence-rate statistics step.
- Input data are transit or astrometric detections where masses are true masses, not M pl sin i — the inclination-completeness treatment of this RV-based method does not apply.
- The sample is too small or the completeness correction cannot be characterized per target, making grid normalization by star count unreliable.

## Inputs

- CARMENES DR1 RV time series (FITS spectra or derived RV products) for the 238-target subsample
- Per-target planet detections with minimum masses (M pl sin i) and orbital periods
- Per-target survey completeness maps in M pl sin i vs P orb space

## Outputs

- Per-cell occurrence rates over the 1–1000 M⊕ × 1–1000 day grid
- Overall average occurrence rate in planets per star with uncertainty (target: 1.44 ± 0.20), written to occurrence_rate.json

## How to apply

Load the CARMENES DR1 radial-velocity time series for the targets of interest (the occurrence-rate derivation used the 238-target subsample from the 345 targets with 18,642 precise RVs). Characterize each target's planet detections and survey completeness in minimum-mass (M pl sin i) versus orbital-period space, because RV masses are minimum masses and the M pl sin i distribution must be corrected for unknown inclinations. Compute the occurrence rate per cell across the 1–1000 M⊕ by 1–1000 day grid with detection-completeness correction in Python, then sum the per-cell rates and normalize by the number of stars to obtain planets per star. Validation: the recomputed score should be consistent with the reported overall average of 1.44 ± 0.20 planets per star, indicating nearly every M dwarf hosts at least one planet.

## Related tools

- **CARMENES** (Instrument supplying the high-accuracy radial-velocity measurements and DR1 data release (19,633 spectra of 362 targets) on which the occurrence-rate computation is based)
- **raccoon** (Python package implementing the CCF method to derive the radial velocities and activity indicators used as inputs to the population statistics) — https://github.com/mlafarga/raccoon
- **Python** (Language used to compute per-cell occurrence rates with detection-completeness correction and normalize by the number of stars)

## Examples

```
python compute_occurrence.py --rv-series carmenes_dr1_rvs.fits --targets 238 --mass-range 1 1000 --period-range 1 1000 --output occurrence_rate.json
```

## Evaluation signals

- Recomputed overall average is consistent with the reported 1.44 ± 0.20 planets per star over 1–1000 M⊕ and 1–1000 days.
- The occurrence-rate grid covers exactly 1 M⊕ < M pl sin i < 1000 M⊕ and 1 day < P orb < 1000 days with per-cell completeness corrections applied.
- Per-cell rates summed and normalized by 238 stars yields a single numeric score comparable to the reported value.
- Output file occurrence_rate.json contains a single numeric score within the reported uncertainty range.

## Limitations

- RV-derived masses are minimum masses (M pl sin i), so occurrence rates depend on the statistical correction for unknown orbital inclinations.
- The rate applies to the CARMENES GTO sample — about 70% of all known M dwarfs within 10 pc accessible from Calar Alto — and may not generalize to other M-dwarf populations.
- Occurrence rates are only as good as the per-target survey completeness characterization; targets with sparse RV coverage in specific mass–period cells can bias per-cell rates.

## Evidence

- [abstract] Overall occurrence rate of 1.44 ± 0.20 planets per star in the 1–1000 M⊕, 1–1000 day range: "yielding an overall average of 1.44 ± 0.20 planets with 1 M ⊕ < M pl sin i < 1000 M ⊕ and 1 day < P orb < 1000 days per star, and indicating that nearly every M dwarf hosts at least one planet"
- [abstract] DR1 provides 18,642 precise RVs for 345 targets from pipeline-processed spectra: "The data were pipeline-processed, and high-level data products, including 18 642 precise RVs for 345 targets, were derived."
- [readme] raccoon derives the RVs feeding the population statistics: "It is able to
- build weighted binary masks from a stellar spectrum template
- compute the CCF of stellar spectra with a mask
- derive radial velocities (RVs) and activity indicators from the CCF"
- [abstract] Survey sample coverage of nearby M dwarfs constrains generalizability: "about 70% of all known M dwarfs within 10 pc and accessible from Calar Alto were included"
