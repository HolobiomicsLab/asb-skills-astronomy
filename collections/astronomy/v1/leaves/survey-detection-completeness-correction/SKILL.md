---
name: survey-detection-completeness-correction
description: 'Use when estimating planet occurrence rates from RV survey data where
  detections are completeness-limited: you have RV time series per star, planet detections
  in M pl sin i / P orb space, and need detection-completeness-corrected rates.'
when_to_use_negative:
- The analysis goal is RV extraction or CCF computation itself rather than occurrence-rate
  inference — use raccoon's CCF workflow directly instead
- Planet radii/transit (S/N-based) completeness is required; this skill uses RV minimum
  masses (M pl sin i), which differ from true masses due to unknown inclination
- The sample lacks per-target detection-threshold characterization; completeness correction
  without measured sensitivity will bias rates
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: CARMENES
  role: High-accuracy radial velocity spectrograph whose DR1 observations (19,633
    spectra, 362 targets) supply the RV time series for the 238-target occurrence-rate
    subsample
- name: raccoon
  role: Open source Python package implementing the CCF method to build weighted binary
    masks, compute CCFs, and derive RVs and activity indicators used in the completeness
    analysis
  repo: https://github.com/mlafarga/raccoon
- name: Python
  role: Implementation language for the occurrence-rate computation, per-cell grid
    integration, and completeness correction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202244879
    title: The CARMENES search for exoplanets around M dwarfs
  source_content_hash: sha256:b96f96d327fe15204c69c3b6d6e02b464e2f95c8f8d3161f4de48b5c6108e021
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
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# survey-detection-completeness-correction

## Summary

Derives survey detection completeness in minimum-mass (M pl sin i) versus orbital-period space from radial-velocity time series, then corrects per-cell planet counts to estimate occurrence rates in planets per star. Applied in the CARMENES DR1 M-dwarf survey to obtain an overall average of 1.44 ± 0.20 planets per star.

## When to use

Use when estimating planet occurrence rates from RV survey data where detections are completeness-limited: you have RV time series per star, planet detections in M pl sin i / P orb space, and need detection-completeness-corrected rates.

## When NOT to use

- The analysis goal is RV extraction or CCF computation itself rather than occurrence-rate inference — use raccoon's CCF workflow directly instead
- Planet radii/transit (S/N-based) completeness is required; this skill uses RV minimum masses (M pl sin i), which differ from true masses due to unknown inclination
- The sample lacks per-target detection-threshold characterization; completeness correction without measured sensitivity will bias rates

## Inputs

- CARMENES DR1 RV time series for the 238-target subsample (1 M⊕ < M pl sin i < 1000 M⊕, 1 day < P orb < 1000 days)
- Per-target planet detections and detection thresholds in minimum-mass versus orbital-period space
- Optional: reduced FITS spectra (e.g. CARMENES VIS/NIR car*vis_A.fits) for re-deriving RVs via CCF

## Outputs

- Per-cell occurrence rates over the 1–1000 M⊕ × 1–1000 day grid after completeness correction
- Overall average planet occurrence rate in planets per star (target: 1.44 ± 0.20), written to occurrence_rate.json
- Completeness maps for each target in M pl sin i versus P orb space

## How to apply

Load the CARMENES DR1 RV time series for the 238 targets used in the occurrence-rate derivation, with high-level products such as the 18,642 precise RVs for 345 targets available from the release. Characterize each target's planet detections and survey completeness in minimum-mass (M pl sin i) versus orbital-period space, because RV masses are minimum masses and the detectability floor depends on per-target measurement precision and cadence. Compute the occurrence rate per cell across the 1–1000 M⊕ by 1–1000 day grid with detection-completeness correction, then sum the per-cell rates and normalize by the number of stars to obtain the overall average in planets per star. RVs and activity indicators can be re-derived from the raw spectra with the raccoon package's cross-correlation function (CCF) method (e.g. via raccoonccf with CARM_VIS/CARM_NIR instrument IDs and weighted binary masks). Validation: the recomputed rate should be consistent with the reported 1.44 ± 0.20 planets per star over the same mass and period ranges.

## Related tools

- **CARMENES** (High-accuracy radial velocity spectrograph whose DR1 observations (19,633 spectra, 362 targets) supply the RV time series for the 238-target occurrence-rate subsample)
- **raccoon** (Open source Python package implementing the CCF method to build weighted binary masks, compute CCFs, and derive RVs and activity indicators used in the completeness analysis) — https://github.com/mlafarga/raccoon
- **Python** (Implementation language for the occurrence-rate computation, per-cell grid integration, and completeness correction)

## Examples

```
raccoonccf PATH/TO/OBS/car*vis_A.fits CARM_VIS J07274+052default --obj OBJ --filtell default --rvshift header --fcorrorders obshighsnr --ords_use 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 --plot_sv --verbose
```

## Evaluation signals

- Recomputed overall occurrence rate is consistent with 1.44 ± 0.20 planets per star for 1 M⊕ < M pl sin i < 1000 M⊕ and 1 day < P orb < 1000 days
- Per-cell rates, when summed and normalized by the number of stars, reproduce the single numeric score in occurrence_rate.json
- Completeness maps cover the full 1–1000 M⊕ × 1–1000 day grid for each of the 238 targets
- All RV masses are reported as minimum masses (M pl sin i), not true masses
- Re-derived RVs with raccoon match the CARMENES DR1 pipeline high-level products (18,642 precise RVs for 345 targets)

## Limitations

- RV-derived masses are minimum masses (M pl sin i); the inclination distribution is not measured, so occurrence rates per true-mass cell carry an inclination-weighting assumption
- The rate applies to the 238-target subsample and the stated mass/period window only (1–1000 M⊕, 1–1000 days); extrapolation outside this range is unsupported
- raccoon's installation and usage documentation was incomplete in the extracted repository sections (no changelog found; pip install path commented out in README)

## Evidence

- [abstract] Overall planet occurrence rate of 1.44 ± 0.20 planets per star (1-1000 M_Earth, 1-1000 days), suggesting nearly every M dwarf hosts at least one planet: "yielding an overall average of 1.44 ± 0.20 planets with 1 M ⊕ < M pl sin i < 1000 M ⊕ and 1 day < P orb < 1000 days per star, and indicating that nearly every M dwarf hosts at least one planet"
- [abstract] DR1 high-level RV data products: "The data were pipeline-processed, and high-level data products, including 18 642 precise RVs for 345 targets, were derived."
- [intro] raccoon capabilities used to derive RVs: "`raccoon` is an open source python package the implements the cross-correlation function (CCF) method."
- [readme] raccoon usage from README: "It is able to
- build weighted binary masks from a stellar spectrum template
- compute the CCF of stellar spectra with a mask
- derive radial velocities (RVs) and activity indicators from the CCF"
