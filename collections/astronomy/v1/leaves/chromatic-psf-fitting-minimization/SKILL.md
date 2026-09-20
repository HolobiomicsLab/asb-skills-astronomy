---
name: chromatic-psf-fitting-minimization
description: Use when extracting a transient point source from an integral-field spectrograph
  (e.g. SEDm) data cube whose host galaxy morphology and background models must have
  their spatial/chromatic shapes fixed prior to a linear 3D amplitude reconstruction.
when_to_use_negative:
- The target is genuinely an isolated point source far from its host — the standard
  pysedm extraction already assumes this and the added scene-modeling machinery is
  unnecessary.
- The host galaxy redshift is unknown — HyperGal's hyperspectral galaxy model requires
  an external redshift measurement as a priori input.
- The input is imaging data rather than a hyperspectral (spectro-spatial) cube — the
  chromatic PSF/ADR fit is defined over monochromatic slices of an IFS cube.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3360
tools:
- name: iminuit
  role: Performs the per-metaslice 2D weighted chi-square minimization of the SN position
    over the chromatic PSF, differential PSF, and background model parameters
  repo: https://github.com/scikit-hep/iminuit
- name: HyperGal
  role: Implements the progressive scene-modeling pipeline whose 2D/chromatic 1D PSF-ADR
    fitting stage this skill describes
  repo: https://github.com/JeremyLezmy/hypergal
- name: SEDm (SEDmachine)
  role: Integral field spectrograph providing the spectro-spatial data cube onto which
    the PSF and ADR models are fit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:9ebcd72a646a82b5cd1ed04dd4ec2a5017f73776e9ba7d1c76722e4c0437438d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scikit-hep/iminuit
  tools:
  - HyperGal
  - HyperGal (hypergal)
  - SEDm (SEDmachine)
  - iminuit
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# chromatic-psf-fitting-minimization

## Summary

HyperGal's PSF/ADR fitting stage: a per-metaslice 2D weighted chi-square minimization of the point-spread function and atmospheric differential refraction models, followed by a chromatic 1D fit that refines the wavelength-dependent PSF parameters before the final 3D amplitude fit.

## When to use

Use when extracting a transient point source from an integral-field spectrograph (e.g. SEDm) data cube whose host galaxy morphology and background models must have their spatial/chromatic shapes fixed prior to a linear 3D amplitude reconstruction.

## When NOT to use

- The target is genuinely an isolated point source far from its host — the standard pysedm extraction already assumes this and the added scene-modeling machinery is unnecessary.
- The host galaxy redshift is unknown — HyperGal's hyperspectral galaxy model requires an external redshift measurement as a priori input.
- The input is imaging data rather than a hyperspectral (spectro-spatial) cube — the chromatic PSF/ADR fit is defined over monochromatic slices of an IFS cube.

## Inputs

- SEDm (SEDmachine) integral-field spectrograph data cube of the transient field
- Projected hyperspectral host galaxy model (from pre-transient photometric images via CIGALE SED fitting)
- Constrained chromatic Gaussian+Moffat PSF model with differential PSF (ADR) component
- 6-parameter background model per metaslice
- Approximate SN position (e.g. from detection coordinates)

## Outputs

- Wavelength-dependent PSF and ADR (differential PSF) parameter fits
- Fixed scene morphology (host, SN, background shapes) entering the final 3D linear amplitude fit
- Extracted transient spectrum from the 3D linear amplitude adjustment of host/SN/background components

## How to apply

First run a 2D weighted chi-square minimization with iminuit on each metaslice around the SN position, fitting the SN position using the constrained chromatic Gaussian+Moffat PSF model, a differential PSF component, and a 6-parameter background model. Then perform the chromatic 1D fit across wavelength to refine the wavelength-dependent point-spread (PSF/ADR) parameters. Once the 2D and 1D fits fix the scene morphology, only the scaling amplitudes of the host galaxy, SN, and background polynomial components remain free for the final 3D linear fit over all monochromatic slices. The rationale is that separating non-linear shape/position estimation (chromatic PSF and ADR) from the linear amplitude problem stabilizes extraction even under strong host contamination.

## Related tools

- **iminuit** (Performs the per-metaslice 2D weighted chi-square minimization of the SN position over the chromatic PSF, differential PSF, and background model parameters) — https://github.com/scikit-hep/iminuit
- **HyperGal** (Implements the progressive scene-modeling pipeline whose 2D/chromatic 1D PSF-ADR fitting stage this skill describes) — https://github.com/JeremyLezmy/hypergal
- **SEDm (SEDmachine)** (Integral field spectrograph providing the spectro-spatial data cube onto which the PSF and ADR models are fit)

## Examples

```
python run_hypergal.py -t ZTF20abhrmxh
```

## Evaluation signals

- The chromatic 1D fit yields smooth, physically plausible wavelength-dependent PSF parameters (chromatic Gaussian+Moffat and ADR) across the spectral range.
- After the 2D and 1D fits, the final 3D fit should be linear in the remaining free parameters — only host, SN, and background scaling amplitudes adjust.
- Continuum-divided spectral relative RMS (Eq. 14) between injected and extracted spectra over [4000, 8000] Å should fall below ~5% for contrast c > 0.5.
- Across simulated SEDm cubes, extraction reaches the 5% spectral relative RMS level for about 60% of simulated observations.
- Downstream SNID typing (rlap_min = 6, 50% top-10 model agreement) on the extracted spectrum should classify SNe Ia correctly in ~95% of cases.

## Limitations

- The 3D fit's monochromatic galaxy amplitudes G(lambda) only correct host redshift offsets and emission-line fluxes to first order; interaction with emission-line positions/intensities depends on the relative distribution of stellar and gaseous components and has not been studied extensively.
- An external host-galaxy redshift measurement is required as input — the most limiting constraint of the method.
- Accuracy degrades at low contrast: the ~5% relative RMS validation applies to the c > 0.5 case; core-host contamination remains hardest near the host center.

## Evidence

- [methods] Per-metaslice 2D fit uses iminuit chi-square minimization: [evidence span withheld — anchor status: not_found]
- [methods] Final 3D linear fit fixes morphology, frees amplitudes: [evidence span withheld — anchor status: not_found]
- [readme] Scene modeler design from README: "HyperGal is a fully chromatic scene modeler, which uses pre-transient photometric images to generate a hyperspectral model of the host galaxy"
- [methods] Extraction accuracy metric and domain: [evidence span withheld — anchor status: not_found]
