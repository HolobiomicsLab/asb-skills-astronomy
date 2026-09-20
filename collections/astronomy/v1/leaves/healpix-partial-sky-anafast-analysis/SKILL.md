---
name: healpix-partial-sky-anafast-analysis
description: Use when you have partial-sky Q/U (or T) maps in HEALPix FITS format
  plus a hitcount map, and you need to validate map noise properties by comparing
  measured EE/BB spectra to a white-noise prediction.
when_to_use_negative:
- Maps cover the full sky or different maps are compared across different sky cuts,
  where a sky-fraction correction alone (and the 'no cross-scale correlation correction'
  assumption) is invalid
- The data are raw, un-destriped timelines with strong correlated noise, so the measured
  spectrum reflects 1/f excess power rather than map noise properties
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: HEALPix
  role: Pixelization scheme of the maps and provider of the anafast routine used to
    compute partial-sky angular power spectra corrected by sky fraction
- name: B-Machine
  role: UCSB 37.5 GHz polarimeter whose destriped Q/U maps and hitcount map are the
    spectra input
- name: dst
  role: Python parallel destriper that produced the destriped maps whose power spectra
    are analyzed
  repo: http://github.com/zonca/dst
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:66c59bbe579286b5b1c0c1b0781ab8a16ed2b417b33985801c296adf4ee12174
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - B-Machine
  - HEALPix
  - Python
  - dst
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# healpix-partial-sky-anafast-analysis

## Summary

Computes Q/U angular power spectra of partially masked HEALPix maps (e.g. the B-Machine destriped maps at Nside 512) with anafast and corrects by sky fraction, then benchmarks the measured spectra against an analytic white-noise prediction.

## When to use

Use when you have partial-sky Q/U (or T) maps in HEALPix FITS format plus a hitcount map, and you need to validate map noise properties by comparing measured EE/BB spectra to a white-noise prediction.

## When NOT to use

- Maps cover the full sky or different maps are compared across different sky cuts, where a sky-fraction correction alone (and the 'no cross-scale correlation correction' assumption) is invalid
- The data are raw, un-destriped timelines with strong correlated noise, so the measured spectrum reflects 1/f excess power rather than map noise properties

## Inputs

- Destriped B-Machine channel 6 Q and U maps in HEALPix Nside 512 FITS format (figshare doi:10.6084/m9.figshare.644507)
- Hitcount map defining the observed sky area and per-pixel coverage (~44.8% sky fraction)
- Pixel area Omega_pix and polarimeter sampling frequency (33.4 Hz) for the white-noise prediction

## Outputs

- Partial-sky EE and BB angular power spectra corrected by the sky fraction (C_l vs l)
- Predicted white-noise angular power spectrum C_l = Omega_pix * sigma^2 * f_samp / hits
- power_spectra.csv containing C_l vs l for EE, BB, and the white-noise prediction

## How to apply

Download the destriped B-Machine channel 6 Q and U maps (HEALPix Nside 512 FITS, figshare doi:10.6084/m9.figshare.644507) and load them with healpy. Apply the partial-sky mask consistent with the observed area (~50% coverage; sky fraction about 44.8% from the hitcount map), then compute the EE and BB spectra on the partial sky with HEALPix anafast and divide by the sky fraction to obtain equivalent full-sky spectra; no cross-scale correlation correction is needed since only maps on the same sky cut are compared. Compute the predicted white-noise spectrum as C_l = Omega_pix * sigma^2 * f_samp / hits using the pixel area, sampling frequency (33.4 Hz), and the per-pixel hitcount map. Write the C_l vs l table (EE, BB, white-noise prediction) to power_spectra.csv and validate: the ratio of measured C_l to the white-noise prediction must stay within a factor of ~2 for l <= 250, and destriping must remove excess power at intermediate scales (50 < l < 200) relative to the binned/filtered maps.

## Related tools

- **HEALPix** (Pixelization scheme of the maps and provider of the anafast routine used to compute partial-sky angular power spectra corrected by sky fraction)
- **B-Machine** (UCSB 37.5 GHz polarimeter whose destriped Q/U maps and hitcount map are the spectra input)
- **dst** (Python parallel destriper that produced the destriped maps whose power spectra are analyzed) — http://github.com/zonca/dst

## Examples

```
import healpy as hp; cl_ee, cl_bb = hp.anafast(masked_q, masked_u); cl_ee /= fsky
```

## Evaluation signals

- Ratio of measured C_l to the white-noise prediction stays within a factor of ~2 for all l <= 250
- Destriped map spectra show reduced excess power at intermediate scales (50 < l < 200) relative to binned/filtered (un-destriped) maps
- Sky fraction used for correction matches the hitcount map (about 44.8%)
- power_spectra.csv contains aligned l, EE, BB, and white-noise prediction columns over the analyzed multipole range

## Limitations

- Only maps on the same sky cut are compared, so no cross-scale correlation correction is applied; other partial-sky comparisons would need mode-coupling corrections
- Systematic effects such as thermally induced gain variations near sunrise and sunset must be masked before map-making; destriping alone cannot remove them
- Validation against the white-noise prediction is only demonstrated down to l = 250

## Evidence

- [methods] The sections of the data nearby sunrise and sunset were contaminated by large thermally induced gain variations and were masked out.: [evidence span withheld — anchor status: not_found]
