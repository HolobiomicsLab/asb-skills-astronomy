---
name: cigale-sed-grid-fitting
description: Use when you need to turn pre-transient broadband host-galaxy images
  (e.g. PS1 grizy cutouts) into a spectrally resolved galaxy cube, or when fitting
  per-pixel SEDs on a fixed parameter grid with an external redshift prior.
when_to_use_negative:
- No external host redshift is available — HyperGal explicitly requires an external
  redshift measurement of the host as a priori input
- 'The goal is a posteriori host measurements (redshift or local Halpha flux): the
  article notes the residual host component precludes such use, so the SED-fit model
  is the wrong basis for those studies'
- Pixels lack sufficient SNR — pixels below SNR>3 in any band are excluded from the
  photometric input
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3393
- http://edamontology.org/topic_3452
tools:
- name: CIGALE
  role: SED fitter run per pixel over a parameter grid, used as a physically-motivated
    spectral interpolator converting broadband photometry into hyperspectral SEDs
- name: HyperGal
  role: Scene-modeling pipeline that consumes the CIGALE-derived hyperspectral host
    cube as its galaxy component projected onto the SEDm observation space
  repo: https://github.com/JeremyLezmy/hypergal
- name: PS1 images (Pan-STARRS)
  role: Pre-transient multi-band photometric images from which the per-pixel photometric
    SED input is built
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:25bb502410c373ed5986f71191815b6c18ffb96842f98013f42257dbddd101e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JeremyLezmy/hypergal
  tools:
  - CIGALE
  - HyperGal
  - PS1 images (Pan-STARRS)
  - PanSTARRS-1
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3393
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# cigale-sed-grid-fitting

## Summary

Per-pixel SED grid fitting with CIGALE used as a physically-motivated spectral interpolator to convert multi-band host-galaxy photometry (e.g. PS1 grizy) into a hyperspectral galaxy model cube for SEDm scene modeling.

## When to use

Use when you need to turn pre-transient broadband host-galaxy images (e.g. PS1 grizy cutouts) into a spectrally resolved galaxy cube, or when fitting per-pixel SEDs on a fixed parameter grid with an external redshift prior.

## When NOT to use

- No external host redshift is available — HyperGal explicitly requires an external redshift measurement of the host as a priori input
- The goal is a posteriori host measurements (redshift or local Halpha flux): the article notes the residual host component precludes such use, so the SED-fit model is the wrong basis for those studies
- Pixels lack sufficient SNR — pixels below SNR>3 in any band are excluded from the photometric input

## Inputs

- Archival PS1 grizy stack cutouts at the target position (from the PS1 Image Cutout Server)
- External host-galaxy redshift (fixed input, e.g. from SDSS-class spectroscopy)
- CIGALE module configuration with the Table 1 parameter grid (sfhdelayed, bc03, nebular, dustatt_modified_CF00, dale2014)

## Outputs

- hostmodel_cube.fits — hyperspectral host model cube rebinned to ~26 A/px SEDm sampling over [3700, 9300] A
- hostmodel_quality.fits — per-pixel pull and spectral relative RMS quality map
- Per-pixel best-fit SED parameters from the CIGALE grid

## How to apply

Retrieve archival PS1 grizy stack cutouts at the target position from the Pan-STARRS Image Cutout Server, then build a multi-band photometric input keeping only pixels with SNR>3 in all bands. Run CIGALE per pixel over the paper's parameter grid using the sfhdelayed star-formation history, bc03 stellar populations, nebular emission, dustatt_modified_CF00 attenuation, and dale2014 dust emission modules, holding the external host redshift fixed. Convert the best-fit per-pixel SEDs into a hyperspectral host cube rebinned to the ~26 A/px SEDm sampling, truncated to the [3700, 9300] A domain, and save it as hostmodel_cube.fits. Compute per-pixel pull and spectral relative RMS (Eq. 14) quality maps saved as hostmodel_quality.fits. Rationale: CIGALE acts as a physically-motivated spectral interpolator, letting sparse broadband photometry constrain the full optical spectrum that the low-resolution SEDm IFS cannot measure directly.

## Related tools

- **CIGALE** (SED fitter run per pixel over a parameter grid, used as a physically-motivated spectral interpolator converting broadband photometry into hyperspectral SEDs)
- **HyperGal** (Scene-modeling pipeline that consumes the CIGALE-derived hyperspectral host cube as its galaxy component projected onto the SEDm observation space) — https://github.com/JeremyLezmy/hypergal
- **PS1 images (Pan-STARRS)** (Pre-transient multi-band photometric images from which the per-pixel photometric SED input is built)

## Examples

```
git clone https://github.com/JeremyLezmy/hypergal.git && cd hypergal && python setup.py install && python run_hypergal.py -t ZTF20aamifit
```

## Evaluation signals

- Median spectral relative RMS must be below 3% in the host core
- Spectral relative RMS up to ~10% is tolerated in the outer host regions (the paper reports averaged spectral RMS generally <3% in the core, reaching ~10% in outer parts)
- hostmodel_cube.fits spectral axis matches the ~26 A/px SEDm sampling truncated to [3700, 9300] A
- hostmodel_quality.fits contains per-pixel pull and spectral relative RMS (Eq. 14) values for all retained pixels
- All pixels entering the fit had SNR>3 in every PS1 band

## Limitations

- The need for an external redshift measurement of the host galaxy is described as the most limiting constraint of HyperGal
- Fit quality degrades with surface brightness: relative RMS reaches ~10% in outer galaxy regions versus <3% in the core
- Correction of emission line fluxes by monochromatic galaxy amplitudes G(lambda) depends on the relative distribution of stellar and gaseous components, an effect not studied extensively for HyperGal

## Evidence

- [abstract] CIGALE SED fitter used as a physically-motivated spectral interpolator: "it is based on the CIGALE SED fitter used as a physically-motivated spectral interpolator"
- [discussion] host redshift requirement: "the most limiting constraint from Hyper Gal is the need for an external redshift measurement of the host galaxy"
- [methods] PS1 background estimation: "Since only B gal is estimated from PS1 images, an additional B sky is estimated using two di ff erent sky levels"
- [full_text] workflow grid and modules: [evidence span withheld — anchor status: not_found]
- [readme] install instructions: [evidence span withheld — anchor status: external_doc]
