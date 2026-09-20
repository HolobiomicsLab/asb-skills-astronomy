---
name: visibility-radial-profile-fitting
description: Use when you have ALMA visibilities of a debris/protoplanetary disc exported
  from an ms file to .npy/.txt and need a rapid parametric radial-profile fit with
  MCMC posteriors.
when_to_use_negative:
- The science requires radially dependent vertical structure — vis-r only supports
  a vertical scale height fixed with radius
- The model needs a single radial component with independent orientations per disk
  component — vis-r assumes a single disk plane and all radial components of the same
  type
- You need to fit arbitrary multi-disk morphologies with different model types; the
  README notes this requires modifying the code yourself
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: vis-r
  role: Core executable that bins visibilities in u,v space and fits parametric radial
    profiles via DHT and MCMC
  repo: https://github.com/drgmk/vis-r
- name: emcee
  role: Ensemble MCMC sampler used by the default Python version of vis-r
  repo: https://github.com/drgmk/vis-r
- name: stan
  role: Alternative backend (cmdstanpy) using Pathfinder for parameter-scale estimation;
    yields more independent samples for the same number of steps
  repo: https://github.com/drgmk/vis-r
- name: alma
  role: Provides example functions for exporting visibilities from ms files and subtracting
    the best-fit model (*vismod.npy) to form residuals
  repo: https://github.com/drgmk/alma
- name: frank
  role: Dependency pulling in scipy<1.12 requirement during conda environment creation
  repo: https://github.com/drgmk/vis-r
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:cc94a5c582306841da0d229f24ccdc1007f87d544354117c60cedc463d1db946
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drgmk/vis-r
  tools:
  - ALMA
  - Python
  - alma
  - emcee
  - frank
  - stan
  - vis-r
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# visibility-radial-profile-fitting

## Summary

Fit parametric radial surface brightness profiles (e.g. Gaussian, power-law) directly to interferometric visibilities using vis-r, which combines hard u,v-space averaging with a Discrete Hankel Transform, running one to two orders of magnitude faster than image-based modelling on a laptop.

## When to use

Use when you have ALMA visibilities of a debris/protoplanetary disc exported from an ms file to .npy/.txt and need a rapid parametric radial-profile fit with MCMC posteriors.

## When NOT to use

- The science requires radially dependent vertical structure — vis-r only supports a vertical scale height fixed with radius
- The model needs a single radial component with independent orientations per disk component — vis-r assumes a single disk plane and all radial components of the same type
- You need to fit arbitrary multi-disk morphologies with different model types; the README notes this requires modifying the code yourself

## Inputs

- Visibility data exported from an ms file to a text or numpy file (e.g. data/HD109573.12m*npy, hr4796.selfcal.npy), calibrated, flagged data removed, coarse averaging applied
- Disc geometry parameters (offsets, inclination, position angle) via -g
- Initial radial profile parameters via -p (e.g. 0.013 1.07 0.06 0.01)

## Outputs

- Best-fit model visibilities saved to *vismod.npy
- Posterior parameter estimates from the emcee chain (or stan samples)
- Radially averaged visibility profile when u,v averaging is applied (e.g. radially_averaged_visibilities.npy)

## How to apply

Export visibilities from the ms file to a numpy/text file, discarding calibration and flagged data and pre-averaging (e.g. 30 s time intervals, a few channels per spw) to keep file size down. Run the `vis-r` executable (default emcee version, or add `--stan` for Pathfinder scale estimation) supplying `-v` with the visibility file(s), `-g` with disc geometry (e.g. offsets, inclination 76.5 deg, position angle), and `-p` with radial profile parameters. Start with hard u,v averaging via `--sz` set to roughly the disk size for quick runs and reasonable parameter estimates, then increase `--sz` with longer emcee runs or stan sampling for robust posteriors. Use `--astrom` (emcee only) for independent astrometry per dataset; include point/Gaussian sources or a central star as needed, noting the model assumes a single disk plane with all radial components of the same type.

## Related tools

- **vis-r** (Core executable that bins visibilities in u,v space and fits parametric radial profiles via DHT and MCMC) — https://github.com/drgmk/vis-r
- **emcee** (Ensemble MCMC sampler used by the default Python version of vis-r) — https://github.com/drgmk/vis-r
- **stan** (Alternative backend (cmdstanpy) using Pathfinder for parameter-scale estimation; yields more independent samples for the same number of steps) — https://github.com/drgmk/vis-r
- **alma** (Provides example functions for exporting visibilities from ms files and subtracting the best-fit model (*vismod.npy) to form residuals) — https://github.com/drgmk/alma
- **frank** (Dependency pulling in scipy<1.12 requirement during conda environment creation) — https://github.com/drgmk/vis-r

## Examples

```
vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01
```

## Evaluation signals

- The *vismod.npy output file is produced by the run
- Runs with --sz set near the disk size complete quickly and give reasonable parameter estimates
- Subtracting the model visibilities from the ms and creating dirty images shows residuals consistent with a good fit
- The radially averaged visibility profile (after bin_uv with size_arcsec=sz) is smooth and spans the u,v-radius range expected for the assumed disk size
- Increasing --sz and run length yields more robust (tighter, stable) posterior estimates

## Limitations

- Vertical scale height is fixed with radius; radially dependent vertical structure is beyond ALMA's capability for most debris discs
- Both stan and emcee versions assume a single disk plane; multiple radial components must all be of the same type
- Independent offset parameters per visibility set are only available in the emcee version; --astrom is emcee-only
- frank dependency requires scipy<1.12 (installed with scipy=1.11.4)

## Evidence

- [abstract] Two main elements: averaging and DHT: "The two main elements are (i) $u,v$ space visibility averaging"
- [abstract] One to two orders of magnitude faster than image-based methods: "these ingredients yield a method that is one to two orders of magnitude faster than image based methods and can be run on a laptop"
- [methods] Run fitting on visibilities: "vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01"
- [methods] Hard u,v averaging with --sz: "initially use rather hard u,v averaging with the `--sz` parameter, setting it to something similar to the disk size"
- [methods] Subtract model and inspect residuals: "Subtract the best fit model (`*vismod.npy`) from the ms from which the visibilities were exported"
- [readme] emcee-only astrometry option: "only the `emcee` version has independent astrometry fitting for each visibility dataset `--astrom` implemented"
