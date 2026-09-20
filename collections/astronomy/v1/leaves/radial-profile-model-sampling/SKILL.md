---
name: radial-profile-model-sampling
description: Use when you have visibilities exported from an ALMA ms file to text/numpy
  files and need posterior samples of radial-profile parameters (geometry, power/Gaussian
  profiles) via vis-r, deciding between emcee and stan samplers.
when_to_use_negative:
- The model requires multiple disk components with independent orientations or radially
  dependent vertical structure, since both vis-r versions assume a single disk plane
  with vertical scales of a single type
- The data are not interferometric visibilities exported from an ms file but an already-imaged
  map; use image-based fitting instead (vis-r's advantage is being 1–2 orders of magnitude
  faster than image-based methods)
- Independent offset/astrometry parameters per visibility set are required while running
  the stan backend, because only the emcee version implements --astrom
edam_operation: http://edamontology.org/operation_3949
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: vis-r
  role: Package providing the vis-r executable that fits parametric radial profiles
    to visibilities with emcee or stan backends
  repo: https://github.com/drgmk/vis-r
- name: emcee
  role: Affine-invariant ensemble sampler used by the default python version for posterior
    sampling
- name: stan
  role: Alternative backend (invoked with --stan) that uses the Pathfinder algorithm
    for parameter-scale estimation and yields more independent samples per step
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:1b118d58946de64491dd0b997d126d7c1b643bbdb922f51daccd8818ddb8e1e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drgmk/vis-r
  tools:
  - Python
  - emcee
  - stan
  - vis-r
  edam_operation: http://edamontology.org/operation_3949
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# radial-profile-model-sampling

## Summary

Run MCMC posterior sampling of a parametric radial disk profile directly against interferometric visibilities using the vis-r package, choosing between the python/emcee and stan backends for the same model and inputs.

## When to use

Use when you have visibilities exported from an ALMA ms file to text/numpy files and need posterior samples of radial-profile parameters (geometry, power/Gaussian profiles) via vis-r, deciding between emcee and stan samplers.

## When NOT to use

- The model requires multiple disk components with independent orientations or radially dependent vertical structure, since both vis-r versions assume a single disk plane with vertical scales of a single type
- The data are not interferometric visibilities exported from an ms file but an already-imaged map; use image-based fitting instead (vis-r's advantage is being 1–2 orders of magnitude faster than image-based methods)
- Independent offset/astrometry parameters per visibility set are required while running the stan backend, because only the emcee version implements --astrom

## Inputs

- Visibilities exported from an ms file to text or numpy files (e.g. data/HD109573.12m*npy)
- Initial geometry parameters via -g (dx, dy, inclination, position angle, e.g. 0.012 -0.035 26.6 76.5)
- Initial radial-profile parameters via -p (e.g. 0.013 1.07 0.06 0.01)
- uv-averaging size via --sz, set near or below the disk size for a first fast pass

## Outputs

- Posterior samples of the radial-profile parameters (emcee or stan chains)
- Best-fit model visibilities written as *vismod.npy
- Fitted parameter estimates and, optionally, residuals after model subtraction for dirty-image inspection

## How to apply

Export the visibilities from the ms to numpy or text files (after discarding calibration/flagged data and averaging, e.g. 30 s time intervals with a few channels per spw). Install vis-r in a conda environment (python, emcee, matplotlib, scipy=1.11.4, multiprocess, arviz, corner, pytest, numpy, cmdstanpy, ipython, jupyter) with pip from github.com/drgmk/vis-r. Run the vis-r executable with -v on the visibility files, passing geometry via -g (e.g. 0.012 -0.035 26.6 76.5) and profile parameters via -p (e.g. 0.013 1.07 0.06 0.01); both the emcee and stan versions default to the same number of warmup/sampling steps, but stan first estimates parameter scales with the Pathfinder algorithm and yields many more independent samples for a given number of steps. Choose --astrom (emcee-only) when each visibility dataset needs independent astrometry/offset fitting, and validate the fit by subtracting the best-fit model (*vismod.npy) from the ms and inspecting dirty images of residuals.

## Related tools

- **vis-r** (Package providing the vis-r executable that fits parametric radial profiles to visibilities with emcee or stan backends) — https://github.com/drgmk/vis-r
- **emcee** (Affine-invariant ensemble sampler used by the default python version for posterior sampling)
- **stan** (Alternative backend (invoked with --stan) that uses the Pathfinder algorithm for parameter-scale estimation and yields more independent samples per step)

## Examples

```
vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01
```

## Evaluation signals

- The emcee and stan runs on identical inputs produce generally consistent posteriors, with stan yielding many more independent samples for the same number of warmup/sampling steps
- The minimal example script (examples/hello_world.py) completes its full fit in under 10 s on an M2 Mac
- A *vismod.npy best-fit model file is produced and can be subtracted from the source ms, leaving residuals whose dirty images show no clear unmodelled disk structure
- A first pass with hard uv averaging (--sz near or below the disk size) completes quickly and gives reasonable parameter estimates before refining with larger --sz and longer runs

## Limitations

- Both versions assume a single disk plane; multiple radial components may have independent vertical scales but all must be of the same type (e.g. Gaussian, power)
- Vertical scale height can be included but is fixed with radius; radially dependent vertical structure is beyond ALMA's capability for most debris discs
- Only the emcee version supports independent offset/astrometry parameters (--astrom) for each set of visibilities
- frank dependency requires scipy<1.12 (install tested with scipy=1.11.4)
- Exported files reflect whatever is in the ms, so calibration and flagged data must be discarded and averaging done before export to keep file sizes manageable

## Evidence

- [readme] stan vs emcee sampler behavior: "Both default to the same number of 
warmup/sampling steps, but `stan` will yield many more independent samples
for a given number of steps."
- [readme] emcee-only astrometry: "only the `emcee` version has independent astrometry fitting for 
each visibility dataset `--astrom` implemented."
- [readme] stan Pathfinder pre-step: "The `stan` version first estimates the parameter scales with the Pathfinder algorithm"
- [readme] suggested fitting strategy: "The suggested method of fitting is to initially use rather hard u,v averaging
with the `--sz` parameter, setting it to something similar to the disk size"
- [intro] single-plane assumption: "Both assume a single disk plane, but can include multiple radial components 
with independent vertical scales, though these must all be of the same type"
- [methods] install command: "CONDA_SUBDIR=osx-arm64 conda create -n vis-r -c conda-forge python emcee matplotlib scipy=1.11.4"
