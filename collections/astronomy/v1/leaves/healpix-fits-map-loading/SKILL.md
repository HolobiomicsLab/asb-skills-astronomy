---
name: healpix-fits-map-loading
description: Use when the input is a HEALPix FITS map of polarized CMB data (Q/U)
  that must be loaded into healpy for partial-sky analysis, e.g. B-Machine destriped
  Nside 512 maps from figshare DOI 10.6084/m9.figshare.644507.
when_to_use_negative:
- Input data is still a raw polarimeter timestream (HDF5 with TIME, PHI, THETA, PSI,
  FLAG, TEMP, Q, U columns) that has not yet been destriped and map-made into HEALPix
  FITS maps.
- The maps are simulated full-sky products where no hit-count-based sky-fraction correction
  is needed.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: healpy
  role: Reads the HEALPix FITS Q/U maps and computes partial-sky spectra with anafast
- name: HEALPix
  role: Pixelization scheme of the B-Machine destriped maps (Nside 512) that defines
    the map grid being loaded
- name: dst
  role: Destriping pipeline whose output HEALPix FITS maps (via figshare DOI 10.6084/m9.figshare.644507)
    are the files loaded by this skill
  repo: http://github.com/zonca/dst
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:df44dbee86495dca6b82423b4717ff1afd9cdf0081ff402081c049e2afd9fceb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - B-Machine
  - HEALPix
  - Python
  - dst
  - healpy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# healpix-fits-map-loading

## Summary

Loads HEALPix FITS-format Q/U sky maps (e.g. the B-Machine destriped Nside 512 maps released via figshare DOI 10.6084/m9.figshare.644507) into healpy so downstream analyses such as partial-sky angular power spectra can be computed.

## When to use

Use when the input is a HEALPix FITS map of polarized CMB data (Q/U) that must be loaded into healpy for partial-sky analysis, e.g. B-Machine destriped Nside 512 maps from figshare DOI 10.6084/m9.figshare.644507.

## When NOT to use

- Input data is still a raw polarimeter timestream (HDF5 with TIME, PHI, THETA, PSI, FLAG, TEMP, Q, U columns) that has not yet been destriped and map-made into HEALPix FITS maps.
- The maps are simulated full-sky products where no hit-count-based sky-fraction correction is needed.

## Inputs

- HEALPix FITS-format destriped Q and U maps, Nside 512 (B-Machine 40 GHz polarimeter, figshare doi:10.6084/m9.figshare.644507)
- Hitcount map defining the observed sky area (~50% coverage, sky fraction ~44.8%)

## Outputs

- In-memory Q/U map arrays (healpy map objects) ready for anafast
- Partial-sky EE and BB angular power spectra corrected by sky fraction, written as a C_l vs l table (power_spectra.csv)

## How to apply

Read the destriped B-Machine channel 6 Q and U maps from the HEALPix FITS files (Nside 512) downloaded from figshare doi:10.6084/m9.figshare.644507 using healpy, preserving the HEALPix pixelization scheme used by the dst destriping pipeline. Pair the map load with the associated hitcount map to derive the observed sky fraction (about 44.8%) needed to correct spectra computed on the partial sky. Once loaded, feed the Q/U arrays directly to HEALPix anafast to compute EE and BB spectra, dividing by the sky fraction to obtain equivalent full-sky spectra. Validate the load by checking the maps are Nside 512, Q/U field order is correct, and that the measured spectrum agrees with the white-noise prediction C_l = Omega_pix * sigma^2 * f_samp / hits within a factor of ~2 for l <= 250.

## Related tools

- **healpy** (Reads the HEALPix FITS Q/U maps and computes partial-sky spectra with anafast)
- **HEALPix** (Pixelization scheme of the B-Machine destriped maps (Nside 512) that defines the map grid being loaded)
- **dst** (Destriping pipeline whose output HEALPix FITS maps (via figshare DOI 10.6084/m9.figshare.644507) are the files loaded by this skill) — http://github.com/zonca/dst

## Examples

```
import healpy as hp; q_map, u_map = hp.read_map('bmachine_ch6_destriped.fits', field=[0,1], dtype=None)
```

## Evaluation signals

- Loaded maps have Nside 512 and contain both Q and U fields from the figshare doi:10.6084/m9.figshare.644507 release.
- Sky fraction derived from the hitcount map is about 44.8% (~50% observed sky coverage).
- Measured EE/BB spectra, after sky-fraction correction, stay within a factor of ~2 of the predicted white-noise power C_l = Omega_pix * sigma^2 * f_samp / hits for l <= 250.
- Destriped maps show reduced excess power at intermediate scales (50 < l < 200) relative to the binned/filtered maps.
- The C_l vs l table (EE, BB, white-noise prediction) is written to power_spectra.csv with consistent l values.

## Limitations

- Spectra are computed on the partial sky (~44.8% coverage) and corrected only by the sky fraction; no cross-scale correlation correction is applied, so comparisons must be between maps on the same sky cut.
- No version numbers are given for Python, Trilinos, HDF5, or HEALPix, limiting reproducibility of the loading/analysis environment.
- The skill covers only map loading and downstream spectral analysis; systematic effects like gain variations must already have been handled before map-making.

## Evidence

- [intro] HEALPix pixelization scheme: [evidence span withheld — anchor status: not_found]
