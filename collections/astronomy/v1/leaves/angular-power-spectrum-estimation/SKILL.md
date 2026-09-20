---
name: angular-power-spectrum-estimation
description: Use when you have partial-sky CMB polarization maps (HEALPix Q/U FITS)
  and need to quantify residual noise or signal power, e.g. checking that destriping
  removed excess correlated-noise power at intermediate angular scales.
when_to_use_negative:
- 'When comparing maps on different sky cuts: the anafast-plus-sky-fraction correction
  assumes all compared maps use the same mask, and cross-scale correlation corrections
  would be needed otherwise.'
- When the goal is to characterize 1/f correlated noise in the raw time-ordered data
  itself — that is addressed by destriping, not by map-domain power spectrum estimation.
- When maps contain large systematics such as thermally induced gain variations near
  sunrise/sunset; those sections must be masked before map-making, as destriping alone
  does not remove them.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: HEALPix
  role: Pixelization scheme for the maps and provider of anafast for partial-sky spectrum
    computation
- name: B-Machine
  role: 'Source experiment: 30 days of 37.5 GHz polarized microwave data whose destriped
    maps are analyzed'
- name: Python
  role: Implementation language used for the destriping pipeline and downstream spectrum
    analysis
  repo: http://github.com/zonca/dst
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ascom.2013.10.002
    title: Destriping cosmic microwave background polarimeter data
  source_content_hash: sha256:19a4652756930886404291c734f64a12b35f12bb358c3b6e81626f2f7c2187eb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - B-Machine
  - HEALPix
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# angular-power-spectrum-estimation

## Summary

Compute EE and BB angular power spectra from partial-sky HEALPix polarimeter maps (e.g. B-Machine destriped Q/U maps at Nside 512) and validate them against a predicted white-noise spectrum built from the hitcount map.

## When to use

Use when you have partial-sky CMB polarization maps (HEALPix Q/U FITS) and need to quantify residual noise or signal power, e.g. checking that destriping removed excess correlated-noise power at intermediate angular scales.

## When NOT to use

- When comparing maps on different sky cuts: the anafast-plus-sky-fraction correction assumes all compared maps use the same mask, and cross-scale correlation corrections would be needed otherwise.
- When the goal is to characterize 1/f correlated noise in the raw time-ordered data itself — that is addressed by destriping, not by map-domain power spectrum estimation.
- When maps contain large systematics such as thermally induced gain variations near sunrise/sunset; those sections must be masked before map-making, as destriping alone does not remove them.

## Inputs

- Destriped B-Machine Q and U maps, HEALPix Nside 512 FITS (figshare doi:10.6084/m9.figshare.644507)
- Hitcount map defining sky coverage (~44.8% sky fraction)
- Pixel area Omega_pix and detector sampling frequency (e.g. 33.4 Hz)
- Optional: binned/filtered maps from the same pipeline for destriping-vs-noise comparison

## Outputs

- C_l vs l table with measured EE, measured BB, and predicted white-noise C_l (e.g. power_spectra.csv)
- Ratio of measured C_l to white-noise prediction per multipole bin

## How to apply

Download the destriped B-Machine channel 6 Q and U maps (HEALPix Nside 512 FITS from figshare doi:10.6084/m9.figshare.644507) and load them with healpy. Apply the partial-sky mask consistent with the observed area (~50% sky coverage; sky fraction about 44.8% per the hitcount map). Compute the EE and BB spectra on the partial sky using HEALPix anafast, then correct by the sky fraction to obtain equivalent full-sky spectra; no cross-scale correlation correction is needed when only maps on the same sky cut are compared. Build the predicted white-noise spectrum as C_l = Omega_pix * sigma^2 * f_samp / hits using the pixel area, the sampling frequency (33.4 Hz for B-Machine), and the hitcount map, then compare measured vs. predicted C_l as a function of multipole l.

## Related tools

- **HEALPix** (Pixelization scheme for the maps and provider of anafast for partial-sky spectrum computation)
- **B-Machine** (Source experiment: 30 days of 37.5 GHz polarized microwave data whose destriped maps are analyzed)
- **Python** (Implementation language used for the destriping pipeline and downstream spectrum analysis) — http://github.com/zonca/dst

## Examples

```
import healpy as hp; cl_ee, cl_bb = hp.anafast(map_q_masked, map_u_masked); cl_ee /= f_sky  # f_sky ~ 0.448 from hitcount map
```

## Evaluation signals

- The measured C_l must agree with the predicted white-noise variance down to l = 250: the ratio of measured C_l to the white-noise prediction stays within a factor of ~2 for l <= 250.
- Destriping must remove excess power at intermediate scales (50 < l < 200) relative to the binned/filtered maps.
- The applied sky-fraction correction is consistent with the hitcount map (~44.8% sky coverage).
- Spectra are computed on the partial sky with HEALPix anafast and only then corrected by the sky fraction, not corrected beforehand.

## Limitations

- The white-noise prediction assumes the noise is purely white after destriping; residual correlated noise or systematics (gain variations, spin-synchronous effects) handled outside destriping would bias the measured spectrum.
- The simple sky-fraction correction is only valid when all compared maps share the same sky cut; cross-scale correlations are not corrected.
- Destriping effectiveness requires a scanning strategy with frequent crossing points; other datasets may not support comparable noise removal.

## Evidence

- [methods] Maps in HEALPix FITS format are publicly available on the Internet via Figshare: "Maps in HEALPix FITS format are publicly available on the Internet via Figshare7 (Zonca)8."
- [methods] partial sky anafast with sky-fraction correction: "spectra were computed on partial sky using HEALPix anafast and then corrected by the sky fraction"
- [abstract] B-Machine data and destriped maps publicly available: "The B-Machine data and destriped maps are made publicly available."
- [readme] destriped maps figshare DOI in README: "Destriped maps from the B-Machine 40GHz polarimeter are available on figshare at: http://dx.doi.org/10.6084/m9.figshare.644507"
- [intro] destriping effectiveness depends on crossing points: "destriping requires that the data are acquired with a scanning strategy (Dupac and Tauber, 2005) which assures frequent crossing points"
