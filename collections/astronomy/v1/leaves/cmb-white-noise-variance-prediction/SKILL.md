---
name: cmb-white-noise-variance-prediction
description: 'Use when validating a destriped single-dish polarimeter map (e.g. B-Machine
  37.5 GHz Q/U HEALPix maps): the measured spectrum must match the predicted white-noise
  variance so that excess power reveals residual correlated noise.'
when_to_use_negative:
- When comparing maps observed on different sky cuts, since no cross-scale correlation
  correction is available and spectra are only comparable on the same cut
- When the map is dominated by sky signal rather than noise (e.g. foreground-contaminated
  or beam-smoothed maps where the white-noise null hypothesis does not hold)
- 'Before map-making: destriping limitations mean gain variations or spin-synchronous
  systematic effects must be handled separately and cannot be diagnosed by this white-noise
  comparison'
edam_operation: http://edamontology.org/operation_0242
tools:
- name: HEALPix
  role: anafast computes the partial-sky EE/BB spectra that are corrected by the sky
    fraction and compared to the white-noise prediction
- name: B-Machine
  role: the 37.5 GHz polarimeter whose destriped Q/U maps and hitcount data are the
    analysis inputs
- name: Python
  role: host language for the healpy-based spectrum computation and CSV output; the
    dst destriper itself is a parallel Python implementation
  repo: http://github.com/zonca/dst
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:8075f5b84ee9d57730bd83a8df888d01d8d4f602dea4b0ad2c4033fde3811a5a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - B-Machine
  - HEALPix
  - Python
  edam_operation: http://edamontology.org/operation_0242
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# CMB white-noise variance prediction for angular power spectra

## Summary

Computes the expected white-noise angular power spectrum of a partially observed CMB polarimetry map as C_l = Omega_pix * sigma^2 * f_samp / hits, and uses it as a null reference against which the measured EE/BB spectra (computed on the partial sky with HEALPix anafast and corrected by sky fraction) are validated.

## When to use

Use when validating a destriped single-dish polarimeter map (e.g. B-Machine 37.5 GHz Q/U HEALPix maps): the measured spectrum must match the predicted white-noise variance so that excess power reveals residual correlated noise.

## When NOT to use

- When comparing maps observed on different sky cuts, since no cross-scale correlation correction is available and spectra are only comparable on the same cut
- When the map is dominated by sky signal rather than noise (e.g. foreground-contaminated or beam-smoothed maps where the white-noise null hypothesis does not hold)
- Before map-making: destriping limitations mean gain variations or spin-synchronous systematic effects must be handled separately and cannot be diagnosed by this white-noise comparison

## Inputs

- Destriped Q and U maps for channel 6, HEALPix Nside 512 FITS (figshare doi:10.6084/m9.figshare.644507)
- Hitcount map for the same channel defining per-pixel observation counts
- Pixel area (Omega_pix) and sampling frequency (33.4 Hz) for the B-Machine data
- Sky mask defining ~44.8% sky coverage

## Outputs

- power_spectra.csv: table of C_l vs l with EE, BB, and the white-noise prediction columns
- Ratio of measured C_l to white-noise prediction per multipole (validation diagnostic)

## How to apply

Download the destriped Q and U maps (HEALPix Nside 512 FITS from figshare doi:10.6084/m9.figshare.644507) together with the hitcount map, and load them with healpy. Apply the partial-sky mask consistent with the observed area (~50% coverage; sky fraction about 44.8% per the hitcount map). Compute EE and BB spectra on the partial sky with HEALPix anafast, then divide by the sky fraction to obtain equivalent full-sky spectra; no cross-scale correlation correction is needed since only maps on the same sky cut are compared. Build the white-noise prediction per multipole as C_l = Omega_pix * sigma^2 * f_samp / hits, using the pixel area, the 33.4 Hz sampling frequency, and per-pixel hit counts. The decision point is agreement: the ratio of measured C_l to the white-noise prediction should stay within a factor of ~2 up to l = 250, and destriping should remove excess power at intermediate scales (50 < l < 200) relative to binned/filtered maps.

## Related tools

- **HEALPix** (anafast computes the partial-sky EE/BB spectra that are corrected by the sky fraction and compared to the white-noise prediction)
- **B-Machine** (the 37.5 GHz polarimeter whose destriped Q/U maps and hitcount data are the analysis inputs)
- **Python** (host language for the healpy-based spectrum computation and CSV output; the dst destriper itself is a parallel Python implementation) — http://github.com/zonca/dst

## Examples

```
import healpy as hp; cl_ee = hp.anafast(masked_Q, masked_Q, lmax=250); cl_bb = hp.anafast(masked_Q, masked_U, lmax=250)
```

## Evaluation signals

- Ratio of measured C_l to white-noise prediction stays within a factor of ~2 for all l <= 250
- Destriped maps show reduced excess power at intermediate scales (50 < l < 200) relative to binned/filtered maps
- Sky-fraction correction produces spectra consistent with the ~44.8% sky fraction implied by the hitcount map
- Output power_spectra.csv contains aligned EE, BB, and white-noise prediction columns for the same l range
- Spectra computed from maps on the identical sky cut require no cross-scale correlation correction

## Limitations

- Agreement is only expected down to l = 250; above that, beam effects and other systematics break the white-noise model
- Systematic effects such as gain variations or spin-synchronous effects are ineffective or harmful under destriping and must be dealt with before map-making; this skill cannot diagnose them
- Scalability claims apply to the destriping pipeline, not this validation; no software version numbers for Python, Trilinos, HDF5, or HEALPix are given, limiting reproducibility

## Evidence

- [intro] validation target of the analysis: [evidence span withheld — anchor status: not_found]
