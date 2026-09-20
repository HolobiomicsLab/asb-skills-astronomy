---
name: radio-interferometric-visibility-fitting
description: Use when fitting parametric radial surface brightness profiles directly
  to ALMA (or other radio interferometric) visibilities exported from an ms file,
  especially when image-plane parametric modelling is computationally too expensive.
when_to_use_negative:
- The target requires radially dependent vertical structure or multiple disk planes
  — vis-r assumes a single disk plane and vertical scale height fixed with radius
- Different disk components need independent orientations — the packaged code assumes
  a single orientation (would require code modification)
- Input is a visibility ms file directly — visibilities must first be exported to
  text/numpy with calibration and flagged data discarded and averaging applied
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3303
tools:
- name: vis-r
  role: Primary executable for radial-profile fitting of visibilities via DHT and
    MCMC sampling
  repo: https://github.com/drgmk/vis-r
- name: emcee
  role: Default ensemble sampler backend for the python version of vis-r
  repo: https://github.com/drgmk/vis-r
- name: stan
  role: Alternative --stan backend using Pathfinder scale estimation followed by gradient-based
    HMC/NUTS sampling
  repo: https://github.com/drgmk/vis-r
- name: alma
  role: Provides example CASA functions for exporting visibilities from ms files and
    subtracting *vismod.npy model visibilities
  repo: https://github.com/drgmk/alma
- name: conda
  role: Environment creation for vis-r install (python, emcee, matplotlib, scipy=1.11.4,
    cmdstanpy, arviz, etc.)
- name: frank
  role: Related visibility-fitting package pinned by the scipy<1.12 requirement of
    the vis-r environment
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_008
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:d14a8fd748b5083caf95660eb73905238a71ec4c3ae8e52d605db596f656cd5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drgmk/vis-r
  tools:
  - ALMA
  - Python
  - alma
  - conda
  - emcee
  - frank
  - stan
  - vis-r
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3303
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# radio-interferometric-visibility-fitting

## Summary

Rapid radial-profile modelling of radio interferometric visibilities with the vis-r package, which combines u,v-space averaging and a Discrete Hankel Transform to fit ALMA visibility data one-to-two orders of magnitude faster than image-based methods.

## When to use

Use when fitting parametric radial surface brightness profiles directly to ALMA (or other radio interferometric) visibilities exported from an ms file, especially when image-plane parametric modelling is computationally too expensive.

## When NOT to use

- The target requires radially dependent vertical structure or multiple disk planes — vis-r assumes a single disk plane and vertical scale height fixed with radius
- Different disk components need independent orientations — the packaged code assumes a single orientation (would require code modification)
- Input is a visibility ms file directly — visibilities must first be exported to text/numpy with calibration and flagged data discarded and averaging applied

## Inputs

- Visibilities exported from an ms file as text or numpy (.npy) files, e.g. data/HD109573.12m*npy
- Self-calibrated visibility arrays such as hr4796.selfcal.npy
- Disk geometry parameters (offsets, inclination, position angle) and radial-profile starting parameters (flux, r0, dr, dz)

## Outputs

- Best-fit radial-profile parameters (flux, r0, dr, dz) from the emcee or stan posterior
- Best-fit model visibilities written to the *vismod.npy file
- Posterior samples (many more independent samples with stan for a given step count)

## How to apply

Export visibilities from the ms file to a text or numpy file, discarding calibration and flagged data and averaging first (e.g. 30 s time intervals, a few channels per spw) to keep the exported file small. Run the vis-r executable with geometry options -g (offsets, inclination, position angle, e.g. '0.012 -0.035 26.6 76.5') and profile starting parameters -p (flux, r0, dr, dz, e.g. '0.013 1.07 0.06 0.01'), using either the python/emcee or the --stan (Pathfinder + HMC/NUTS) sampler. Start with hard u,v averaging via --sz set near the disk size for a fast initial run, then increase --sz for robust posterior estimates. Subtract the best-fit model (*vismod.npy) from the source ms and inspect dirty images of the residuals to validate the fit. Both versions assume a single disk plane; only the emcee version supports independent astrometry/offset fitting per visibility dataset (--astrom).

## Related tools

- **vis-r** (Primary executable for radial-profile fitting of visibilities via DHT and MCMC sampling) — https://github.com/drgmk/vis-r
- **emcee** (Default ensemble sampler backend for the python version of vis-r) — https://github.com/drgmk/vis-r
- **stan** (Alternative --stan backend using Pathfinder scale estimation followed by gradient-based HMC/NUTS sampling) — https://github.com/drgmk/vis-r
- **alma** (Provides example CASA functions for exporting visibilities from ms files and subtracting *vismod.npy model visibilities) — https://github.com/drgmk/alma
- **conda** (Environment creation for vis-r install (python, emcee, matplotlib, scipy=1.11.4, cmdstanpy, arviz, etc.))
- **frank** (Related visibility-fitting package pinned by the scipy<1.12 requirement of the vis-r environment)

## Examples

```
vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01
```

## Evaluation signals

- The *vismod.npy best-fit model file is produced and, when subtracted from the source ms, dirty images of the residuals show no obvious residual structure
- Fitted radial-profile parameters (flux, r0, dr, dz) are present in the posterior/output and consistent with starting values in -p
- Wall-clock runtime falls in the seconds-to-minutes range on a laptop (the minimal hello_world.py example completes in under 10 s on an M2 Mac)
- Sampler diagnostics converge: R-hat near 1 and higher effective sample size for stan vs emcee at equal step count, with posteriors agreeing within credible intervals
- stan yields many more independent posterior samples than emcee for the same number of warmup/sampling steps

## Limitations

- Both emcee and stan versions assume a single disk plane; multiple radial components are allowed but all vertical scales must be of the same type (e.g. Gaussian, power)
- Vertical scale height is fixed with radius; radially dependent vertical structure cannot be fitted
- Only the emcee version supports independent offset/astrometry parameters (--astrom) for each set of visibilities
- The stan version first estimates parameter scales with the Pathfinder algorithm; the two versions differ in minor ways beyond that

## Evidence

- [abstract] u,v averaging and DHT rationale: [evidence span withheld — anchor status: not_found]
- [methods] example fit command: "vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01"
- [readme] export and averaging guidance: "discard any calibration and flagged data, and it is best to do some averaging first (e.g. 30s time intervals, a few channels per spw)"
- [readme] staged --sz fitting strategy: "initially use rather hard u,v averaging with the `--sz` parameter, setting it to something similar to the disk size, or even smaller"
- [readme] residual validation: "Subtract the best fit model (`*vismod.npy`) from the ms from which the visibilities were exported"
- [abstract] speedup claim: "yield a method that is one to two orders of magnitude faster than image based methods and can be run on a laptop"
- [abstract] single plane and vertical structure limitation: "Vertical scale height can be included, but is fixed with radius; in principle this is a limitation but in practice radially dependent vertical structure information is beyond even ALMA’s capability"
- [readme] emcee/stan differences: "The `stan` version first estimates the parameter scales with the Pathfinder algorithm, and only the `emcee` version has independent astrometry fitting"
