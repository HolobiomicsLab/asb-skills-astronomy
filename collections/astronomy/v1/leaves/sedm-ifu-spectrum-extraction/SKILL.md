---
name: sedm-ifu-spectrum-extraction
description: Use when a reduced SEDm (SEDmachine) IFU data cube of a ZTF transient
  is contaminated by host-galaxy flux near the host core, and a clean SN spectrum
  is needed for SNID typing.
when_to_use_negative:
- The transient is an isolated point source far from its host core — the standard
  PSF extraction (pysedm) suffices since contamination is not the limiting factor
- No pre-transient photometric host images are available to build the CIGALE hyperspectral
  galaxy model
- No external host redshift exists and in-pipeline redshift estimation is required
  — the paper notes 'the most limiting constraint from Hyper Gal is the need for an
  external redshift measurement of the host galaxy'
tools:
- name: HyperGal
  role: Chromatic scene modeler that projects the CIGALE host model, SN point source
    and diffuse background onto the SEDm observation space and fits them to extract
    the SN spectrum
  repo: https://github.com/Jeremy_Lezmy/Hyper_Gal
- name: CIGALE
  role: SED fitter used as a physically-motivated spectral interpolator converting
    pre-transient PS1 photometry into the hyperspectral host galaxy cube
- name: SNID
  role: Spectral classifier applied to the extracted spectrum with rlap_min = 6 and
    50% top-10 model-type agreement
- name: pysedm
  role: Baseline SEDm IFS data-reduction pipeline producing the cubes; its isolated-point-source
    extraction is the comparison method
- name: iminuit
  role: Least-squares/likelihood fitting interface used within the HyperGal fitting
    machinery
  repo: https://github.com/scikit-hep/iminuit
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:9d6ded27ae073ecc0e5db6d6dc5c20de12da331de7cf6aa1f6e624425fa5448e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Jeremy_Lezmy/Hyper_Gal
  tools:
  - CIGALE
  - HyperGal
  - SNID
  - iminuit
  - pysedm
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# sedm-ifu-spectrum-extraction

## Summary

Extracts a clean transient spectrum from SEDm IFU cubes using HyperGal, a fully chromatic scene modeler that fits a CIGALE-based hyperspectral host model plus a point-source SN and diffuse background, mitigating host contamination near the galaxy core.

## When to use

Use when a reduced SEDm (SEDmachine) IFU data cube of a ZTF transient is contaminated by host-galaxy flux near the host core, and a clean SN spectrum is needed for SNID typing.

## When NOT to use

- The transient is an isolated point source far from its host core — the standard PSF extraction (pysedm) suffices since contamination is not the limiting factor
- No pre-transient photometric host images are available to build the CIGALE hyperspectral galaxy model
- No external host redshift exists and in-pipeline redshift estimation is required — the paper notes 'the most limiting constraint from Hyper Gal is the need for an external redshift measurement of the host galaxy'

## Inputs

- Reduced SEDm (pysedm) IFU data cube of the transient (or a target name such as ZTF20abhrmxh for automatic cube download)
- Pre-transient PS1 (Pan-STARRS) multi-band photometric images of the host galaxy
- External host-galaxy redshift measurement (e.g. from SDSS; sigma_z ~ 10^-5 for z < 0.1, or SNID-derived ~5e-3)

## Outputs

- Extracted 1D supernova spectrum over the [4000,8000] Å SEDm domain
- Fitted scene model components (hyperspectral host cube, SN point source, diffuse background)
- SNID classification (Ia / II / Ibc / uncertain) with best-fit rlap
- Contrast and spectral relative RMS diagnostics per target

## How to apply

Build a hyperspectral host-galaxy model cube from pre-transient PS1 multi-band photometric images using CIGALE as a physically-motivated spectral interpolator, estimating the galaxy background B_gal from PS1 and an additional B_sky from two sky levels. Complement this with a point-source (SN) component and a smooth diffuse background, project all three onto the SEDm spectro-spatial observation space, and adjust them to the observations. Extract the SN spectrum over the SEDm [4000,8000] Å domain and assess extraction quality via the spectral relative RMS between simulation input and extracted spectra in contrast bins. Feed the extracted spectrum to SNID with stricter-than-default confidence criteria (rlap_min = 6 and at least 50% of the top-10 models agreeing with the best-fit type); if either criterion fails, label the result 'uncertain'. The external host redshift must be supplied a priori since HyperGal does not estimate it internally.

## Related tools

- **HyperGal** (Chromatic scene modeler that projects the CIGALE host model, SN point source and diffuse background onto the SEDm observation space and fits them to extract the SN spectrum) — https://github.com/Jeremy_Lezmy/Hyper_Gal
- **CIGALE** (SED fitter used as a physically-motivated spectral interpolator converting pre-transient PS1 photometry into the hyperspectral host galaxy cube)
- **SNID** (Spectral classifier applied to the extracted spectrum with rlap_min = 6 and 50% top-10 model-type agreement)
- **pysedm** (Baseline SEDm IFS data-reduction pipeline producing the cubes; its isolated-point-source extraction is the comparison method)
- **iminuit** (Least-squares/likelihood fitting interface used within the HyperGal fitting machinery) — https://github.com/scikit-hep/iminuit

## Examples

```
python setup.py install && python run_hypergal.py -t ZTF20abhrmxh
```

## Evaluation signals

- Spectral relative RMS between simulation input and extracted spectra, averaged over [4000,8000] Å, is low across contrast bins
- SNID classification meets the stricter criteria: rlap_min = 6 and ≥50% of the top-10 models share the best-fit type, otherwise output is 'uncertain'
- Benchmark: ~95% of SNe Ia correctly classified from the estimated contrast distribution of real SEDm observations
- False positive rate for SN Ia classification stays below 2% (half the standard extraction method)
- Isolated host component's main spectral features align with the input redshift (e.g. z = 0.045 for SN ZTF20aamifit)

## Limitations

- Requires an external host-galaxy redshift measurement as a priori input — the paper's most limiting constraint
- SN-host distance distribution is not explicitly modeled; it is marginalized uniformly between 0 and 5 arcsec
- Correction of redshift offset and emission-line fluxes by monochromatic galaxy amplitudes G(lambda) depends on the relative stellar/gaseous distribution and has not been studied extensively
- The residual host component cannot be used for a posteriori measurements (e.g. redshift or local Halpha flux) without the dedicated in-pipeline redshift extension

## Evidence

- [readme] HyperGal is a fully chromatic scene modeler, which uses pre-transient photometric images to generate a hyperspectral model of the host galaxy: "HyperGal is a fully chromatic scene modeler, which uses pre-transient photometric images to generate a hyperspectral model of the host galaxy"
- [abstract] galaxy model projected onto SEDm observation space: "The galaxy model, complemented by a point source and a diffuse background component, is projected onto the SEDm spectro-spatial observation space and adjusted to observations"
- [methods] SNID stricter classification criteria: "The minimum rlap is set to rlap min = 6 (rather than 5) for the best-fit model; furthermore, at least 50% of the top-10 models have to be of the same type as the best one"
- [methods] extraction quality metric: "of the spectral relative RMS between simulation input spectra and extracted spectra, averaged over the [4000,8000] Å domain"
- [intro] scope: isolated point source assumption: "Since this pipeline intrinsically assumes the target is an isolated point source, it cannot properly handle the situation where the transient is close to its host galaxy core."
- [discussion] external redshift requirement: "the most limiting constraint from Hyper Gal is the need for an external redshift measurement of the host galaxy"
