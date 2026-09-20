---
name: integral-field-spectroscopy-scene-modeling
description: Use when a transient spectrum must be extracted from an IFS data cube
  (e.g. SEDm) and the transient lies near its host galaxy core, so standard isolated
  point-source extraction suffers host-galaxy contamination.
when_to_use_negative:
- When the transient is an isolated point source far from the host core — the standard
  pysedm extraction already handles this case without scene modeling.
- When an external host-galaxy redshift is unavailable, since HyperGal requires it
  as a priori input and it is its most limiting constraint.
- When a posteriori host measurements (e.g. redshift or local Halpha flux) are needed
  from the residual host component — scene modeling precludes such use.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: HyperGal
  role: 'Main scene-modeling pipeline: builds the hyperspectral host model and performs
    the 2D, 1D chromatic, and 3D amplitude fits to extract the transient spectrum'
  repo: https://github.com/Jeremy_Lezmy/Hyper_Gal
- name: iminuit
  role: Used for the per-metaslice 2D weighted chi-square minimization of the SN position,
    PSF, and background parameters
  repo: https://github.com/scikit-hep/iminuit
- name: CIGALE
  role: SED fitter used as a physically-motivated spectral interpolator to convert
    pre-transient photometry into the host galaxy cube
- name: pysedm
  role: Standard SEDm data reduction pipeline whose isolated point-source extraction
    is the baseline being improved upon
- name: SNID
  role: Classifier applied to the extracted spectrum for supernova typing
- name: SEDm (SEDmachine)
  role: The integral field spectrograph whose cubes are modeled
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:69680cc5fe438c171cf166df36cabb1f93b8fce1e7484ec6509031ac393330e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Jeremy_Lezmy/Hyper_Gal
  tools:
  - CIGALE
  - HyperGal
  - HyperGal (hypergal)
  - SEDm (SEDmachine)
  - SNID
  - iminuit
  - pysedm
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# integral-field-spectroscopy-scene-modeling

## Summary

Hyperspectral scene modeling for extracting a transient's spectrum from an integral field spectrograph (SEDm) cube by jointly fitting a projected host-galaxy hyperspectral model, a point-source (SN) component, and a diffuse background polynomial, instead of assuming an isolated point source.

## When to use

Use when a transient spectrum must be extracted from an IFS data cube (e.g. SEDm) and the transient lies near its host galaxy core, so standard isolated point-source extraction suffers host-galaxy contamination.

## When NOT to use

- When the transient is an isolated point source far from the host core — the standard pysedm extraction already handles this case without scene modeling.
- When an external host-galaxy redshift is unavailable, since HyperGal requires it as a priori input and it is its most limiting constraint.
- When a posteriori host measurements (e.g. redshift or local Halpha flux) are needed from the residual host component — scene modeling precludes such use.

## Inputs

- SEDm (SEDmachine) IFS data cube of the transient field (e.g. from pysedm reduction)
- Pre-transient multi-band photometric images of the host galaxy (e.g. PS1 images)
- External host-galaxy redshift measurement (a priori input)
- Host-galaxy SED model generated with CIGALE as spectral interpolator

## Outputs

- Extracted transient (SN) spectrum from the 3D amplitude fit
- Residual host / background component decomposition (host, SN, background amplitudes G(lambda))
- Continuum-divided spectral relative RMS (Eq. 14) quantifying extraction accuracy over [4000, 8000] A
- SNID typing classification with confidence criteria (rlap_min = 6, 50% top-10 agreement)

## How to apply

Build a hyperspectral model of the host galaxy from pre-transient photometric images (e.g. PS1), using the CIGALE SED fitter as a physically-motivated spectral interpolator, with host redshift supplied externally. Project this galaxy model plus a point-source transient and a diffuse background component onto the SEDm spectro-spatial observation space and fit progressively: first a per-metaslice 2D weighted chi-square minimization (with iminuit) over the SN position using a constrained chromatic Gaussian+Moffat PSF, a differential PSF, and a 6-parameter background model; then a chromatic 1D fit of the PSF/ADR models across wavelength; finally a 3D linear fit over all monochromatic slices in which only the monochromatic scaling amplitudes G(lambda) of the host, SN, and background components are free, the morphology having been fixed by the earlier 2D/1D fits. The extracted SN spectrum is the fitted transient amplitude spectrum; accuracy is assessed via the continuum-divided spectral relative RMS between input and extracted spectra averaged over [4000, 8000] Angstrom.

## Related tools

- **HyperGal** (Main scene-modeling pipeline: builds the hyperspectral host model and performs the 2D, 1D chromatic, and 3D amplitude fits to extract the transient spectrum) — https://github.com/Jeremy_Lezmy/Hyper_Gal
- **iminuit** (Used for the per-metaslice 2D weighted chi-square minimization of the SN position, PSF, and background parameters) — https://github.com/scikit-hep/iminuit
- **CIGALE** (SED fitter used as a physically-motivated spectral interpolator to convert pre-transient photometry into the host galaxy cube)
- **pysedm** (Standard SEDm data reduction pipeline whose isolated point-source extraction is the baseline being improved upon)
- **SNID** (Classifier applied to the extracted spectrum for supernova typing)
- **SEDm (SEDmachine)** (The integral field spectrograph whose cubes are modeled)

## Examples

```
python run_hypergal.py -t ZTF20abhrmxh
```

## Evaluation signals

- Continuum-normalized spectral relative RMS between injected input spectrum and extracted spectrum over [4000, 8000] A should be below ~5% for contrast c > 0.5 (HyperGal reaches the 5% level for 60% of simulated observations)
- Validation against 5000 simulated cubes with injected point sources of known input spectra at fixed contrast
- SN Ia typing success rate of ~95% on the real SEDm contrast distribution, ~10% more SNe Ia correctly classified than the standard extraction method
- False positive rate (SN Ia) below 2%, half that of the standard extraction method
- Assuming a similar contrast distribution, 14% (11%) additional SNe II (Ibc) correctly classified relative to standard extraction

## Limitations

- Requires an external redshift measurement of the host galaxy — the most limiting constraint of HyperGal
- The true distance distribution between SN and host was not explicitly modeled; it was marginalized uniformly between 0 and 5 arcsec
- The G(lambda) correction of emission line position/intensity depends on the relative distribution of stellar and gaseous components and has not been studied extensively
- The residual host component cannot be used for a posteriori measurements (e.g. redshift or local Halpha flux)
- pysedm-based limitations: about 30% of observed SNe show severe host contamination and ~10% are unusable, motivating the scene-modeling approach

## Evidence

- [abstract] CIGALE as spectral interpolator: "uses pre-transient photometric images to generate a hyperspectral model of the host galaxy; it is based on the CIGALE SED fitter used as a physically-motivated spectral interpolator"
- [methods] 3D linear fit fixes morphology: [evidence span withheld — anchor status: not_found]
- [abstract] validation on simulated cubes: "The full procedure is validated on 5000 simulated cubes."
- [methods] spectral RMS metric: "of the spectral relative RMS between simulation input spectra and extracted spectra, averaged over the [4000,8000] Å domain"
- [discussion] redshift requirement: "the most limiting constraint from Hyper Gal is the need for an external redshift measurement of the host galaxy"
- [intro] motivation: host contamination: "Since this pipeline intrinsically assumes the target is an isolated point source, it cannot properly handle the situation where the transient is close to its host galaxy core."
- [readme] readme description: "HyperGal is a fully chromatic scene modeler, which uses pre-transient photometric images to generate a hyperspectral model of the host galaxy"
