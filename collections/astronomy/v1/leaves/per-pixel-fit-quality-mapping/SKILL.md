---
name: per-pixel-fit-quality-mapping
description: Use when a hyperspectral host model cube has been generated from broadband
  photometry (e.g. PS1 grizy via CIGALE) and you need to verify per-pixel fit quality
  before projecting the model into SEDm observation space.
when_to_use_negative:
- When the goal is to measure host properties from the SEDm residual cube — the residual
  host component cannot be used for a posteriori measurements such as redshift or
  local Halpha flux
- When no per-pixel SED fit exists yet (this is a post-fit quality map, not the SED
  fitting itself)
- When evaluating the transient extraction quality — the spectral relative RMS here
  quantifies the host model, not SN extraction
edam_operation: http://edamontology.org/operation_2945
tools:
- name: CIGALE
  role: Per-pixel SED fitter whose best-fit spectra are the input to the quality computation;
    used as a physically-motivated spectral interpolator
- name: HyperGal
  role: Downstream scene modeler consuming hostmodel_cube.fits and hostmodel_quality.fits
    to project the galaxy model onto SEDm spectro-spatial observation space
  repo: https://github.com/JeremyLezmy/hypergal
- name: PS1 images (Pan-STARRS)
  role: Pre-transient broadband photometric input from which the host model and its
    quality map are derived
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202244740
    title: 'HyperGal: Hyperspectral scene modeling for supernova typing with the SED
      Machine integral field spectrograph'
  source_content_hash: sha256:37010d57d9ddbd424e3668fd723369fdfb1f65cb20b6995acc04bfb5f35e0697
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JeremyLezmy/hypergal
  tools:
  - CIGALE
  - HyperGal
  - PS1 images (Pan-STARRS)
  - PanSTARRS-1
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# per-pixel-fit-quality-mapping

## Summary

Validates the CIGALE-derived hyperspectral host-galaxy model by computing a per-pixel pull map and spectral relative RMS (Eq. 14), quantifying SED-fit fidelity spatially across the galaxy before the cube is used for SEDm scene modeling.

## When to use

Use when a hyperspectral host model cube has been generated from broadband photometry (e.g. PS1 grizy via CIGALE) and you need to verify per-pixel fit quality before projecting the model into SEDm observation space.

## When NOT to use

- When the goal is to measure host properties from the SEDm residual cube — the residual host component cannot be used for a posteriori measurements such as redshift or local Halpha flux
- When no per-pixel SED fit exists yet (this is a post-fit quality map, not the SED fitting itself)
- When evaluating the transient extraction quality — the spectral relative RMS here quantifies the host model, not SN extraction

## Inputs

- PS1 grizy stack cutouts at the transient position (SNR>3 in all bands per pixel)
- Best-fit per-pixel CIGALE SED cube rebinned to ~26 A/px over [3700, 9300] A (hostmodel_cube.fits)
- Fixed external host-galaxy redshift used as CIGALE input

## Outputs

- Per-pixel pull and spectral relative RMS (Eq. 14) quality map (hostmodel_quality.fits)

## How to apply

After running CIGALE per pixel over the PS1 grizy cutouts (pixels restricted to SNR>3 in all bands) and rebinning the best-fit SEDs to the ~26 A/px SEDm sampling truncated to the [3700, 9300] A domain (hostmodel_cube.fits), compute the per-pixel pull and the spectral relative RMS (Eq. 14) comparing the best-fit SED to the input photometric constraints, saving the result as hostmodel_quality.fits. Judge the mapping correct if the median spectral relative RMS is below 3% in the host core, with values up to ~10% tolerated in the outer host regions. The rationale is that CIGALE acts only as a physically-motivated spectral interpolator, so a spatially-resolved quality map exposes where the interpolation is unconstrained (typically low-SNR outskirts) before HyperGal's 3D fit relies on the model as background.

## Related tools

- **CIGALE** (Per-pixel SED fitter whose best-fit spectra are the input to the quality computation; used as a physically-motivated spectral interpolator)
- **HyperGal** (Downstream scene modeler consuming hostmodel_cube.fits and hostmodel_quality.fits to project the galaxy model onto SEDm spectro-spatial observation space) — https://github.com/JeremyLezmy/hypergal
- **PS1 images (Pan-STARRS)** (Pre-transient broadband photometric input from which the host model and its quality map are derived)

## Examples

```
python run_hypergal.py -t ZTF20aamifit  # hostmodel_cube.fits -> hostmodel_quality.fits
```

## Evaluation signals

- Median spectral relative RMS is below 3% in the host core of the quality map
- Outer host regions show RMS up to ~10%, which is tolerated but should be flagged as lower-fidelity
- hostmodel_quality.fits contains a per-pixel pull map with the same spatial grid as hostmodel_cube.fits
- Pixels failing SNR>3 in any PS1 band are excluded from the quality statistics
- RMS is computed over the [3700, 9300] A SEDm domain rather than the full CIGALE wavelength range

## Limitations

- Fit quality degrades to ~10% spectral RMS in outer galaxy parts where PS1 photometry is low-SNR
- An external host-galaxy redshift measurement is required as a priori input; without it the model cannot be built
- The monochromatic galaxy amplitude correction G(lambda) during the 3D fit is only first-order, and its interaction with emission line position/intensity depends on the relative distribution of stellar and gaseous components

## Evidence

- [methods] per-pixel pull and spectral relative RMS (Eq. 14) quality map: [evidence span withheld — anchor status: not_found]
- [methods] median spectral relative RMS below 3% in the host core: [evidence span withheld — anchor status: not_found]
- [readme] CIGALE as physically-motivated spectral interpolator: "uses pre-transient photometric images to generate a hyperspectral model of the host galaxy; it is based on the CIGALE SED fitter used as a physically-motivated spectral interpolator"
- [methods] spectral relative RMS metric definition: "of the spectral relative RMS between simulation input spectra and extracted spectra, averaged over the [4000,8000] Å domain"
