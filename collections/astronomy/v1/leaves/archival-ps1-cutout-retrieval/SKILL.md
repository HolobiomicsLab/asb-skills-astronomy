---
name: archival-ps1-cutout-retrieval
description: 'Use when a pre-transient host-galaxy reference is needed for SEDm scene
  modeling: downloading PS1 grizy stack cutouts at the transient position to feed
  per-pixel CIGALE SED fitting or host contamination estimation.'
when_to_use_negative:
- The host galaxy has no usable PS1 coverage or the cutouts are saturated/deeply blended,
  since the pre-transient photometric images are the sole host reference
- The transient is far from its host core and host contamination is negligible, so
  a standard isolated point-source extraction (e.g. plain pysedm) suffices without
  a host model
- The goal is a posteriori host measurements (e.g. redshift or local Halpha flux)
  from residuals, which the HyperGal pipeline precludes
edam_operation: http://edamontology.org/operation_2422
tools:
- name: PanSTARRS-1
  role: Public multi-band wide photometric survey supplying the pre-transient grizy
    host reference images via the Image Cutout Server
- name: PS1 images (Pan-STARRS)
  role: Source of the host-galaxy background level B_gal estimated from the retrieved
    cutouts
- name: HyperGal
  role: Scene modeler that consumes the PS1 cutouts to generate the hyperspectral
    host model before projecting it onto SEDm observation space
  repo: https://github.com/JeremyLezmy/hypergal
- name: CIGALE
  role: SED fitter applied per pixel on the PS1 photometric input as a physically-motivated
    spectral interpolator
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:df3a709e21533a017d1c129d687b7d854a15025c7fcffdcd0d0843dad3f3c285
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JeremyLezmy/hypergal
  tools:
  - CIGALE
  - HyperGal
  - PS1 images (Pan-STARRS)
  - PanSTARRS-1
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Archival PS1 cutout retrieval

## Summary

Retrieve pre-transient Pan-STARRS-1 (PS1) grizy broadband stack cutouts at a transient's sky position from the public PS1 Image Cutout Server, to serve as the photometric input for building a hyperspectral host-galaxy model with CIGALE in the HyperGal workflow.

## When to use

Use when a pre-transient host-galaxy reference is needed for SEDm scene modeling: downloading PS1 grizy stack cutouts at the transient position to feed per-pixel CIGALE SED fitting or host contamination estimation.

## When NOT to use

- The host galaxy has no usable PS1 coverage or the cutouts are saturated/deeply blended, since the pre-transient photometric images are the sole host reference
- The transient is far from its host core and host contamination is negligible, so a standard isolated point-source extraction (e.g. plain pysedm) suffices without a host model
- The goal is a posteriori host measurements (e.g. redshift or local Halpha flux) from residuals, which the HyperGal pipeline precludes

## Inputs

- Transient sky coordinates (e.g. ZTF20aamifit position)
- External host-galaxy redshift measurement (fixed prior for downstream CIGALE fitting)
- Public PS1 Image Cutout Server (Pan-STARRS-1 stacked g/r/i/z/y imagery)

## Outputs

- PS1 grizy FITS stack cutouts of the host galaxy at the transient position
- Multi-band photometric input (pixels with SNR>3 in all bands) for per-pixel CIGALE SED fitting
- Downstream hyperspectral host model cube (hostmodel_cube.fits) and quality map (hostmodel_quality.fits)

## How to apply

Query the public PS1 Image Cutout Server with the target's sky coordinates (e.g. the ZTF20aamifit position) and download the stacked g, r, i, z, y FITS cutouts of the host galaxy. These pre-transient broadband images provide the only host spectral reference before the transient appears, so they become the multi-band photometric input for the per-pixel CIGALE fit (sfhdelayed SFH, bc03 populations, nebular emission, dustatt_modified_CF00, dale2014) using the external host redshift as a fixed input. When building the photometric input, keep only pixels with SNR>3 in all five bands, and rebin the resulting best-fit SEDs to the ~26 A/px SEDm sampling over [3700, 9300] A. Because only the galaxy background level B_gal can be estimated from the PS1 images, plan for an additional sky level B_sky to be estimated from two different sky levels during the later scene fit.

## Related tools

- **PanSTARRS-1** (Public multi-band wide photometric survey supplying the pre-transient grizy host reference images via the Image Cutout Server)
- **PS1 images (Pan-STARRS)** (Source of the host-galaxy background level B_gal estimated from the retrieved cutouts)
- **HyperGal** (Scene modeler that consumes the PS1 cutouts to generate the hyperspectral host model before projecting it onto SEDm observation space) — https://github.com/JeremyLezmy/hypergal
- **CIGALE** (SED fitter applied per pixel on the PS1 photometric input as a physically-motivated spectral interpolator)

## Examples

```
python run_hypergal.py -t ZTF20aamifit  # target cube auto-downloaded; PS1 grizy cutouts feed the CIGALE host model
```

## Evaluation signals

- All five PS1 bands (g, r, i, z, y) are retrieved as readable FITS cutouts with WCS centered on the target position
- The SNR>3-in-all-bands pixel mask yields a non-empty multi-band photometric input covering the host
- Downstream per-pixel CIGALE fit achieves a median spectral relative RMS below 3% in the host core, with outer regions tolerated up to ~10% (Eq. 14)
- The hyperspectral cube is rebinned to the ~26 A/px SEDm sampling and truncated to the [3700, 9300] A domain as hostmodel_cube.fits

## Limitations

- Only B_gal is estimated from PS1 images; an additional B_sky must be estimated from two different sky levels
- HyperGal requires an external host-galaxy redshift measurement as a priori input, so the PS1 photometry alone cannot provide it
- Archival PS1 resolution and depth limit fit quality: the averaged spectral RMS reaches about 10% in the outer parts of the galaxy even when below 3% in the core

## Evidence

- [methods] Since only B gal is estimated from PS1 images, an additional B sky is estimated using two di ff erent sky levels: "Since only B gal is estimated from PS1 images, an additional B sky is estimated using two di ff erent sky levels"
- [intro] public multi-band wide photometric surveys provide host reference: "public multi-band wide photometric surveys can provide reference information on the host galaxy before the transient event"
- [readme] HyperGal uses pre-transient photometric images for a hyperspectral host model: "HyperGal is a fully chromatic scene modeler, which uses pre-transient photometric images to generate a hyperspectral model of the host galaxy"
- [full_text] Retrieve the archival PS1 grizy stack cutouts from the PS1 Image Cutout Server: [evidence span withheld — anchor status: not_found]
