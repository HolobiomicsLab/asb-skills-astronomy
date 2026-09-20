---
name: radial-profile-parameter-estimation
description: Use when you have visibilities exported from an ALMA ms file (text or
  .npy) of a disc and need fast posterior estimates of radial-profile parameters with
  fixed geometry (offsets, PA, inclination).
when_to_use_negative:
- The data are not interferometric visibilities but already-made images — vis-r fits
  parametric radial profiles in the visibility plane, not dirty images
- The disc requires radially dependent vertical structure or multiple disk planes/components
  with independent orientations out of the box — vis-r assumes a single disk plane
  and vertical scale height fixed with radius
- Only the stan version's features are needed but per-dataset astrometry is required
  — --astrom is implemented only in the emcee version
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: vis-r
  role: executable that models multiple sets of exported visibilities with parametric
    radial profiles, in emcee and stan backends
  repo: https://github.com/drgmk/vis-r
- name: emcee
  role: default MCMC sampling backend for the vis-r fit, with optional per-dataset
    astrometry fitting
- name: stan
  role: alternative backend (via --stan) that first estimates parameter scales with
    the Pathfinder algorithm
- name: conda
  role: creates the vis-r environment (python, emcee, matplotlib, scipy=1.11.4, etc.)
    before pip-installing the package
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:bece0193e5b529fe5ffb3f8df40fddbcee4df7f7d10b2bee452188cae7f264dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drgmk/vis-r
  tools:
  - conda
  - emcee
  - stan
  - vis-r
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# radial-profile-parameter-estimation

## Summary

Estimate radial-profile parameters (flux, r0, dr, dz) of a circumstellar disc by fitting parametric surface brightness profiles directly to ALMA visibilities with vis-r, using u,v averaging and the Discrete Hankel Transform for orders-of-magnitude speedup over image-based fitting.

## When to use

Use when you have visibilities exported from an ALMA ms file (text or .npy) of a disc and need fast posterior estimates of radial-profile parameters with fixed geometry (offsets, PA, inclination).

## When NOT to use

- The data are not interferometric visibilities but already-made images — vis-r fits parametric radial profiles in the visibility plane, not dirty images
- The disc requires radially dependent vertical structure or multiple disk planes/components with independent orientations out of the box — vis-r assumes a single disk plane and vertical scale height fixed with radius
- Only the stan version's features are needed but per-dataset astrometry is required — --astrom is implemented only in the emcee version

## Inputs

- visibility files exported from an ms file matching e.g. data/HD109573.12m*npy (text or numpy format, calibration and flagged data discarded, time/channel averaged)
- fixed disc geometry parameters (offsets, inclination, position angle) passed via -g
- radial-profile starting parameters (flux, r0, dr, dz) passed via -p

## Outputs

- best-fit radial-profile parameters (flux, r0, dr, dz) from the posterior
- best-fit model visibilities written to a *vismod.npy file

## How to apply

Export visibilities from the ms file to text or numpy files, discarding calibration and flagged data and first averaging (e.g. 30s time intervals, a few channels per spw) to keep file sizes down. Run vis-r with geometry fixed via -g (offsets 0.012 -0.035, inclination 26.6, PA 76.5 for HD 109573) and profile starting parameters -p (flux 0.013, r0 1.07, dr 0.06, dz 0.01), defaulting to the python/emcee backend. Start with hard u,v averaging via --sz set to roughly the disc size for fast, reasonable parameter estimates, then repeat with a larger --sz and longer emcee runs or the --stan implementation for robust posteriors. The stan variant estimates parameter scales with the Pathfinder algorithm first; only emcee supports per-dataset astrometry fitting with --astrom. Collect best-fit parameters from the posterior and write model visibilities to the *vismod.npy output.

## Related tools

- **vis-r** (executable that models multiple sets of exported visibilities with parametric radial profiles, in emcee and stan backends) — https://github.com/drgmk/vis-r
- **emcee** (default MCMC sampling backend for the vis-r fit, with optional per-dataset astrometry fitting)
- **stan** (alternative backend (via --stan) that first estimates parameter scales with the Pathfinder algorithm)
- **conda** (creates the vis-r environment (python, emcee, matplotlib, scipy=1.11.4, etc.) before pip-installing the package)

## Examples

```
vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01
```

## Evaluation signals

- The *vismod.npy output file exists after the run
- Subtracting *vismod.npy from the source ms and inspecting dirty images of the residuals shows no obvious residual structure
- Initial runs with --sz similar to the disc size complete quickly and give reasonable parameter estimates
- Robust posterior estimates are reproducible with larger --sz and either longer emcee runs or the stan implementation

## Limitations

- Vertical scale height can be included but is fixed with radius; radially dependent vertical structure is beyond ALMA's capability for most debris discs
- Both versions assume a single disk plane; multiple radial components must all be of the same type (e.g. Gaussian, power)
- The exported file contains whatever is in the ms, so calibration and flagged data must be discarded and averaging applied first, or the export becomes large
- Options differ between versions: stan uses Pathfinder scale estimation while only emcee has --astrom

## Evidence

- [methods] Fixed-geometry example invocation: "vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01"
- [readme] Export and averaging guidance: "discard any calibration and flagged data, and it is best to do some averaging first (e.g. 30s time intervals, a few channels per spw)"
- [readme] Recommended staged fitting strategy: "initially use rather hard u,v averaging with the `--sz` parameter, setting it to something similar to the disk size"
- [methods] Model validation by residual imaging: "Subtract the best fit model (`*vismod.npy`) from the ms from which the visibilities were exported"
- [abstract] Speed claim: "these ingredients yield a method that is one to two orders of magnitude faster than image based methods and can be run on a laptop"
