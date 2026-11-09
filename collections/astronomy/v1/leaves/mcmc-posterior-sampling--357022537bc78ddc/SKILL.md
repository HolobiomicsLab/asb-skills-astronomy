---
name: mcmc-posterior-sampling
description: Use when fitting parametric radial profiles (e.g. Gaussian, power) to
  ALMA visibility sets exported from an ms file and you need posterior parameter estimates
  via MCMC rather than a single optimisation.
when_to_use_negative:
- When the science requires radially dependent vertical structure — vis-r fixes the
  vertical scale height with radius, and the article notes this information is beyond
  ALMA's capability for most debris discs
- When the model needs multiple disk planes or independent orientations per disk component
  — vis-r assumes a single disk plane (though emcee allows independent offsets per
  visibility dataset)
- When an image-based fitting approach is required or the analysis targets calibrated
  images rather than exported visibilities
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: vis-r
  role: Executable that performs the MCMC radial-profile fit to visibility files,
    selectable emcee or stan backend, writes *vismod.npy
  repo: https://github.com/drgmk/vis-r
- name: emcee
  role: Default python ensemble sampler backend used by vis-r for posterior sampling
    (e.g. 18 walkers, 1000 steps, 6 threads); supports --astrom per-dataset offsets
  repo: https://github.com/drgmk/vis-r
- name: stan
  role: Alternative vis-r backend (--stan flag) that first estimates parameter scales
    with the Pathfinder algorithm and yields more independent samples per step
  repo: https://github.com/drgmk/vis-r
- name: conda
  role: Environment creation for vis-r install, pinning scipy=1.11.4 because frank
    wants scipy<1.12
  repo: https://github.com/drgmk/vis-r
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:17d0e6c6845ccc265f83fb65e594c739c45fcd8fec4aff89efdbd14a7213ec2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drgmk/vis-r
  tools:
  - Python
  - conda
  - emcee
  - stan
  - vis-r
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# mcmc-posterior-sampling

## Summary

Posterior sampling of radial-profile model parameters fitted directly to interferometric visibilities with vis-r, using either the python/emcee ensemble sampler or the stan (Pathfinder-initialised HMC) backend, producing best-fit flux/r0/dr/dz (plus geometry) posteriors and *vismod.npy model visibilities.

## When to use

Use when fitting parametric radial profiles (e.g. Gaussian, power) to ALMA visibility sets exported from an ms file and you need posterior parameter estimates via MCMC rather than a single optimisation.

## When NOT to use

- When the science requires radially dependent vertical structure — vis-r fixes the vertical scale height with radius, and the article notes this information is beyond ALMA's capability for most debris discs
- When the model needs multiple disk planes or independent orientations per disk component — vis-r assumes a single disk plane (though emcee allows independent offsets per visibility dataset)
- When an image-based fitting approach is required or the analysis targets calibrated images rather than exported visibilities

## Inputs

- Visibility file(s) exported from an ms file as text or numpy (.npy), e.g. data/HD109573.12m*npy or hr4796.selfcal.npy
- Geometry parameters via -g (offsets, inclination, position angle, e.g. 0.012 -0.035 26.6 76.5)
- Radial-profile starting parameters via -p (flux, r0, dr, dz, e.g. 0.013 1.07 0.06 0.01)
- Optional --sz u,v averaging size (set near the disk size for a fast initial run)

## Outputs

- Posterior parameter estimates from the emcee chain (or independent stan samples)
- Best-fit model visibilities written to *vismod.npy
- Fitted radial-profile parameters (flux, r0, dr, dz) and geometry/offset posteriors

## How to apply

Export visibilities from the ms to a text or numpy file, discarding calibration and flagged data and pre-averaging (e.g. 30 s time intervals, a few channels per spw) to keep the file small. Run vis-r with -v on the visibility file(s), -g to fix or start the geometry (offsets, inclination, position angle) and -p for the radial-profile starting parameters (flux, r0, dr, dz), using the default emcee backend (18 walkers, 1000 steps, 6 threads in the HR 4796 example). For a fast first pass, set hard u,v averaging via --sz to a value similar to the disk size (or smaller), then repeat with larger --sz and either longer emcee runs or the --stan flag for more robust posteriors. Use the emcee-only --astrom option if multiple visibility datasets need independent offset fitting. Validate by checking *vismod.npy is produced, subtracting it from the source ms, and inspecting dirty images of the residuals for obvious structure.

## Related tools

- **vis-r** (Executable that performs the MCMC radial-profile fit to visibility files, selectable emcee or stan backend, writes *vismod.npy) — https://github.com/drgmk/vis-r
- **emcee** (Default python ensemble sampler backend used by vis-r for posterior sampling (e.g. 18 walkers, 1000 steps, 6 threads); supports --astrom per-dataset offsets) — https://github.com/drgmk/vis-r
- **stan** (Alternative vis-r backend (--stan flag) that first estimates parameter scales with the Pathfinder algorithm and yields more independent samples per step) — https://github.com/drgmk/vis-r
- **conda** (Environment creation for vis-r install, pinning scipy=1.11.4 because frank wants scipy<1.12) — https://github.com/drgmk/vis-r

## Examples

```
vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01
```

## Evaluation signals

- The *vismod.npy output file exists and contains best-fit model visibilities
- Subtracting *vismod.npy from the source ms and imaging dirty residuals leaves no obvious residual structure
- The emcee run completes with the expected walkers/steps (e.g. 18 walkers, 1000 steps in the HR 4796 example) and converges to reasonable parameter estimates
- A staged run: hard --sz u,v averaging first gives quick, reasonable estimates, and larger --sz repeats give consistent robust posteriors
- The minimal examples/hello_world.py fit completes end-to-end in less than 10 s on an M2 Mac with fitted parameters present

## Limitations

- Both vis-r backends assume a single disk plane; multiple radial components are allowed but their vertical scales must all be of the same type
- Vertical scale height is fixed with radius; radially dependent vertical structure is not modelled
- Only the emcee version implements independent astrometry fitting per visibility dataset (--astrom); the stan version first estimates parameter scales with Pathfinder
- Exported files can be large, so averaging in the ms (e.g. 30 s intervals, few channels per spw) and discarding calibration/flagged data is required beforehand

## Evidence

- [readme] emcee backend details: "The suggested method of fitting is to initially use rather hard u,v averaging
with the `--sz` parameter, setting it to something similar to the disk size,"
- [readme] geometry and profile parameters: "vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01"
- [readme] backend differences: "The `stan` version first estimates the parameter scales with the Pathfinder algorithm,
 and only the `emcee` version has independent astrometry fitting for 
 each visibility dataset `--astrom`"
- [methods] export procedure: "discard any calibration and flagged data, and it is best to do some averaging first (e.g. 30s time intervals, a few channels per spw)"
- [methods] residual validation: "Subtract the best fit model (`*vismod.npy`) from the ms from which the visibilities were exported"
