---
name: radial-profile-discrete-hankel-transform-modeling
description: Use when you need fast radial-profile fitting of debris-disk (or similar)
  structure from high-resolution ALMA visibilities and image-plane parametric MCMC
  is computationally too expensive.
when_to_use_negative:
- You need radially dependent vertical scale height structure — vis-r assumes a single
  disk plane with a vertical scale height fixed with radius
- You need per-component independent orientations for a multi-component disk model
  without editing the code
- The data are not interferometric visibilities (e.g. already-imaged maps) — this
  skill operates in u,v space
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3934
tools:
- name: vis-r
  role: Primary package that performs u,v averaging and Discrete Hankel Transform
    radial-profile fitting of exported visibilities; installed via pip and run through
    the vis-r shell command
  repo: https://github.com/drgmk/vis-r/archive/refs/heads/main.zip
- name: emcee
  role: Python MCMC backend of vis-r supporting independent per-dataset offsets and
    astrometry fitting (--astrom)
  repo: https://github.com/drgmk/vis-r
- name: stan
  role: Alternative gradient-based (Hamiltonian Monte-Carlo) backend of vis-r that
    first estimates parameter scales with the Pathfinder algorithm and yields more
    independent samples per step
- name: alma
  role: Helper library with CASA functions to export visibilities from an ms to numpy/text
    and to subtract best-fit model visibilities for residual dirty images
  repo: https://github.com/drgmk/alma
- name: conda
  role: Environment setup for vis-r (python, emcee, matplotlib, scipy=1.11.4, cmdstanpy,
    etc.)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:f2d720679db79b1e4d0a269cacd85e2818c2e297e2092d926620c4febd47d9b6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALMA
  - Python
  - alma
  - conda
  - emcee
  - stan
  - vis-r
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# radial-profile-discrete-hankel-transform-modeling

## Summary

Rapidly fit parametric radial surface brightness profiles directly to ALMA radio-interferometric visibilities by combining hard u,v-space averaging with a Discrete Hankel Transform (the vis-r package), achieving 10x–100x speedups over image-based parametric modelling on a laptop.

## When to use

Use when you need fast radial-profile fitting of debris-disk (or similar) structure from high-resolution ALMA visibilities and image-plane parametric MCMC is computationally too expensive.

## When NOT to use

- You need radially dependent vertical scale height structure — vis-r assumes a single disk plane with a vertical scale height fixed with radius
- You need per-component independent orientations for a multi-component disk model without editing the code
- The data are not interferometric visibilities (e.g. already-imaged maps) — this skill operates in u,v space

## Inputs

- Self-calibrated ALMA visibilities exported from an ms file to a numpy or text file (e.g. hr4796.selfcal.npy)
- Initial guesses for disk geometry and radial-profile parameters passed as -g and -p flags
- The --sz u,v-averaging parameter (set near the disk size for initial fast runs)

## Outputs

- Posterior samples for radial-profile disk parameters (from emcee or stan backends)
- Best-fit model visibilities written as *vismod.npy for subtraction from the ms
- Timing/speedup comparison versus an image-based baseline (e.g. timing_comparison.csv)

## How to apply

First export self-calibrated visibilities from the measurement set to a .npy or text file (e.g. hr4796.selfcal.npy), discarding calibration/flagged data and pre-averaging (e.g. 30 s time intervals, a few channels per spw) to keep file size down. Run the vis-r executable on the exported visibilities, initially using hard u,v averaging via the --sz parameter set near the disk size so runs complete quickly with reasonable parameter estimates; then increase --sz and use longer emcee runs or the stan implementation (with Pathfinder scale estimation) for robust posteriors. The method's two ingredients are u,v-space visibility averaging and the Discrete Hankel Transform, which converts arbitrary parametric radial profiles to visibilities without a costly image-space Fourier step. Evaluate the fit by subtracting the best-fit model (*vismod.npy) from the originating ms and making dirty images of the residuals.

## Related tools

- **vis-r** (Primary package that performs u,v averaging and Discrete Hankel Transform radial-profile fitting of exported visibilities; installed via pip and run through the vis-r shell command) — https://github.com/drgmk/vis-r/archive/refs/heads/main.zip
- **emcee** (Python MCMC backend of vis-r supporting independent per-dataset offsets and astrometry fitting (--astrom)) — https://github.com/drgmk/vis-r
- **stan** (Alternative gradient-based (Hamiltonian Monte-Carlo) backend of vis-r that first estimates parameter scales with the Pathfinder algorithm and yields more independent samples per step)
- **alma** (Helper library with CASA functions to export visibilities from an ms to numpy/text and to subtract best-fit model visibilities for residual dirty images) — https://github.com/drgmk/alma
- **conda** (Environment setup for vis-r (python, emcee, matplotlib, scipy=1.11.4, cmdstanpy, etc.))

## Examples

```
vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01
```

## Evaluation signals

- Wall-clock speedup of the vis-r fit relative to an image-based parametric-model baseline falls between 10x and 100x
- The vis-r fit completes in seconds to minutes on a laptop; the package's minimal example runs in under 10 s on an M2 Mac
- Runs with hard --sz averaging give reasonable parameter estimates quickly, and subsequent larger --sz runs yield consistent posteriors
- Dirty images of residuals after subtracting *vismod.npy from the ms show no obvious structured residuals

## Limitations

- Assumes a single disk plane; vertical scale height can be included but is fixed with radius, and radially dependent vertical structure is beyond ALMA's capability for most debris discs
- Radial components with independent vertical scales must all be of the same type (e.g. Gaussian, power)
- stan version lacks independent per-dataset astrometry fitting (--astrom is emcee-only); emcee and stan versions have minor behavioural differences
- Exported ms contents include calibration and flagged data unless discarded, and large un-averaged exports slow fitting — pre-average (e.g. 30 s, few channels per spw)

## Evidence

- [abstract] one to two orders of magnitude faster: "Combined, these ingredients yield a method that is one to two orders of magnitude faster than image based methods and can be run on a laptop"
- [abstract] method ingredients: [evidence span withheld — anchor status: not_found]
- [readme] suggested fitting procedure: "The suggested method of fitting is to initially use rather hard u,v averaging with the `--sz` parameter, setting it to something similar to the disk size"
- [readme] pre-export ms hygiene: "discard any calibration and flagged data, and it is best to do some averaging first (e.g. 30s time intervals, a few channels per spw)"
- [readme] residual evaluation: "Subtract the best fit model (`*vismod.npy`) from the ms from which the visibilities were exported"
- [abstract] vertical structure limitation: [evidence span withheld — anchor status: not_found]
