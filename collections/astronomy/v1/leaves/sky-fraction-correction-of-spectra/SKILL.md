---
name: sky-fraction-correction-of-spectra
description: Use when comparing angular power spectra measured on a masked/partial-sky
  map (e.g., destriped B-Machine Q/U maps with ~50% sky cut) against full-sky predictions
  such as the white-noise C_l = Omega_pix * sigma^2 * f_samp / hits.
when_to_use_negative:
- The maps cover the full sky, so no sky-fraction rescaling of anafast output is required.
- Comparing spectra of maps computed on different sky cuts, where a cross-scale correlation
  correction would be needed instead of a simple sky-fraction rescale.
- The map noise is dominated by correlated noise or systematic effects (e.g., gain
  variations near sunrise/sunset) rather than white noise, since the white-noise prediction
  C_l = Omega_pix * sigma^2 * f_samp / hits would not be a valid reference.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3399
- http://edamontology.org/topic_3315
tools:
- name: HEALPix
  role: Pixelization scheme of the B-Machine maps; anafast computes the EE/BB spectra
    on the partial sky
- name: Python
  role: Loading the FITS maps with healpy, applying the mask, computing spectra, and
    writing power_spectra.csv
- name: B-Machine
  role: Source instrument providing the destriped 37.5 GHz polarimeter Q/U maps, mask,
    and hitcount map
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:c47bf5af72bdcd6a7bfc3bfdd4148bfbc56f14bf97b093f690ead18413d117aa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - B-Machine
  - HEALPix
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3399
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Sky-fraction correction of spectra

## Summary

Computes angular power spectra of Q/U maps on a partial sky (e.g., the ~44.8% sky coverage of the B-Machine hitcount map) using HEALPix anafast, then rescales by the sky fraction to obtain equivalent full-sky C_l for comparison with white-noise predictions.

## When to use

Use when comparing angular power spectra measured on a masked/partial-sky map (e.g., destriped B-Machine Q/U maps with ~50% sky cut) against full-sky predictions such as the white-noise C_l = Omega_pix * sigma^2 * f_samp / hits.

## When NOT to use

- The maps cover the full sky, so no sky-fraction rescaling of anafast output is required.
- Comparing spectra of maps computed on different sky cuts, where a cross-scale correlation correction would be needed instead of a simple sky-fraction rescale.
- The map noise is dominated by correlated noise or systematic effects (e.g., gain variations near sunrise/sunset) rather than white noise, since the white-noise prediction C_l = Omega_pix * sigma^2 * f_samp / hits would not be a valid reference.

## Inputs

- Destriped B-Machine channel 6 Q and U maps, HEALPix Nside 512 FITS (figshare doi:10.6084/m9.figshare.644507)
- Partial-sky mask consistent with the observed area (~50% sky coverage)
- Hitcount map from the B-Machine observation
- Pixel area (Omega_pix) and sampling frequency (33.4 Hz) of the B-Machine polarimeter

## Outputs

- C_l vs l table (power_spectra.csv) containing measured EE and BB spectra, sky-fraction-corrected, alongside the predicted white-noise angular power spectrum

## How to apply

Apply a partial-sky mask consistent with the observed area (for B-Machine channel 6, ~50% coverage, sky fraction about 44.8% per the hitcount map), then compute the Q/U angular power spectra with HEALPix anafast on the masked maps. Correct the measured spectra by dividing by the sky fraction to obtain equivalent full-sky spectra. Compute the predicted white-noise spectrum as C_l = Omega_pix * sigma^2 * f_samp / hits using the pixel area, sampling frequency (33.4 Hz), and the hitcount map. Because only maps on the same sky cut are compared, no cross-scale correlation correction is needed. Validate that the measured EE/BB spectra agree with the white-noise prediction down to l = 250, with the ratio within a factor of ~2 for l <= 250.

## Related tools

- **HEALPix** (Pixelization scheme of the B-Machine maps; anafast computes the EE/BB spectra on the partial sky)
- **Python** (Loading the FITS maps with healpy, applying the mask, computing spectra, and writing power_spectra.csv)
- **B-Machine** (Source instrument providing the destriped 37.5 GHz polarimeter Q/U maps, mask, and hitcount map)

## Examples

```
cl = healpy.anafast(mapQ*mask, mapU*mask); cl_corrected = cl / sky_fraction  # sky_fraction ~ 0.448 from hitcount map
```

## Evaluation signals

- The ratio of measured C_l to the white-noise prediction stays within a factor of ~2 for l <= 250.
- Destriping removes excess power at intermediate scales (50 < l < 200) relative to the binned/filtered maps.
- The sky fraction used for the correction matches the hitcount map (~44.8% for the B-Machine observed area).
- power_spectra.csv contains C_l vs l columns for EE, BB, and the white-noise prediction.
- Spectra are computed with HEALPix anafast on the masked (partial-sky) maps and corrected by the same sky fraction.

## Limitations

- The sky-fraction correction assumes maps compared on the same sky cut; no cross-scale correlation correction is applied, so it is not valid when comparing spectra on different masks.
- The validation criterion (agreement with white noise within a factor of ~2 down to l = 250) applies only where noise is white; systematic effects like gain variations must be masked before map-making.
- Software/data versions (Python, HEALPix) are unspecified in the article, limiting reproducibility.

## Evidence

- [intro] Effect of destriping visible in power spectra: [evidence span withheld — anchor status: not_found]
