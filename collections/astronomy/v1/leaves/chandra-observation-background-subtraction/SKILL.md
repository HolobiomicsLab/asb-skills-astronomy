---
name: chandra-observation-background-subtraction
description: Use when building an X-ray spectral data cube from one or more Chandra
  ACIS observations (e.g. merged ObsIDs of a supernova remnant) and a background spectrum
  must be estimated to separate instrumental/sky background from source emission before
  spectral fitting or unmixing.
when_to_use_negative:
- The target's background is dominated by genuinely extended diffuse source emission
  that a blank-sky spectrum cannot represent (e.g. large-scale cluster emission),
  where local background estimation is required instead.
- The data are not Chandra ACIS observations — the blank-sky calibration product is
  instrument-specific and cannot be applied to other telescopes' data.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Chandra X-ray telescope
  role: Instrument providing the ACIS observations merged into the data cube; its
    blank-sky calibration observations supply the background spectrum
- name: SUSHI
  role: Consumes the merged data cube plus the blank-sky background spectrum as inputs
    to its semi-blind unmixing cost function
  repo: https://github.com/jmlascar/SUSHI
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202347518
    title: 'SUSHI: An algorithm for source separation of hyperspectral images with
      non-stationary spectral variation'
  source_content_hash: sha256:82bda7a6bdd91c7addf34472d5631689136b06b16c9d8439d53ecdc5ceb640d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmlascar/SUSHI
  tools:
  - Chandra X-ray telescope
  - JAX
  - Python
  - SUSHI
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Chandra observation background subtraction

## Summary

Constructing Chandra X-ray hyperspectral (ACIS) data cubes from merged observations and characterizing the instrumental background using the Chandra ACIS calibration team's blank-sky observations, so that the background spectrum can be accounted for during spectral unmixing (e.g. with SUSHI).

## When to use

Use when building an X-ray spectral data cube from one or more Chandra ACIS observations (e.g. merged ObsIDs of a supernova remnant) and a background spectrum must be estimated to separate instrumental/sky background from source emission before spectral fitting or unmixing.

## When NOT to use

- The target's background is dominated by genuinely extended diffuse source emission that a blank-sky spectrum cannot represent (e.g. large-scale cluster emission), where local background estimation is required instead.
- The data are not Chandra ACIS observations — the blank-sky calibration product is instrument-specific and cannot be applied to other telescopes' data.

## Inputs

- Chandra ACIS observation event files (multiple ObsIDs, e.g. 1994, 1995, 1996, 1997, 1999, 2000, 2001 for the Crab Nebula)
- Chandra blank-sky observations from the Chandra ACIS calibration team
- Chandra instrumental response files (ARF and RMF)

## Outputs

- Merged Chandra hyperspectral data cube (spatial pixels x spectral energy channels)
- Background spectrum derived from blank-sky observations, supplied to the spectral unmixing/fitting step

## How to apply

Merge the target's Chandra observations (e.g. the Crab Nebula data cube from the 2000–2001 ObsIDs 1994, 1995, 1996, 1997, 1999, 2000, 2001) into a single hyperspectral cube, and derive the background spectrum from the Chandra ACIS calibration team's blank-sky observations rather than from a local off-source annulus. The background spectrum is then supplied alongside the data cube to the analysis (in the SUSHI workflow it enters the likelihood term so that per-voxel fitting accounts for the background contribution). Instrumental response files (ARF and RMF) must be folded with the spectral model when constructing or fitting the cube, as done for the Chandra telescope response in the paper's toy model. This background treatment is what makes the single-component synchrotron fit of the Crab Nebula comparable to published photon-index maps (Mori et al. 2004) on the same data set.

## Related tools

- **Chandra X-ray telescope** (Instrument providing the ACIS observations merged into the data cube; its blank-sky calibration observations supply the background spectrum)
- **SUSHI** (Consumes the merged data cube plus the blank-sky background spectrum as inputs to its semi-blind unmixing cost function) — https://github.com/jmlascar/SUSHI

## Evaluation signals

- The resulting source spectral map (e.g. the Crab Nebula synchrotron photon-index map) is consistent with past analyses of the same data set (Mori et al. 2004) while resolving finer spatial detail than re-binned per-pixel 1D fits.
- The merged cube uses the correct ObsID set for the epoch of interest (for the Crab: 1994, 1995, 1996, 1997, 1999, 2000, 2001).
- The background spectrum traces the Chandra ACIS blank-sky calibration product rather than an ad hoc local estimate, so fitted source parameters are not biased by residual background counts.

## Limitations

- The article does not detail how the blank-sky spectrum is normalized to the target's exposure/region, so re weighting for differing exposure times must be handled by the practitioner.
- Blank-sky background spectra may not capture background variability specific to the target's sky position or observation epoch.

## Evidence

- [results] Data cube construction from merged Chandra ObsIDs: "The data cube was constructed from the Chandra X-ray observation from the years 2000 and 2001 (ObsID: 1994, 1995, 1996, 1997, 1999, 2000, 2001)"
- [results] Background spectrum from ACIS calibration blank-sky observations: [evidence span withheld — anchor status: not_found]
- [methods] Instrumental response (ARF/RMF) convolved with spectral models: "These were convolved by the Chandra instrumental response (ARF and RMF files)."
- [abstract] Background handling enables consistency with past Crab Nebula results: "The results obtained are realistic and in accordance with past findings but have a much better spatial resolution."
