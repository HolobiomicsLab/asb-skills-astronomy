---
name: debris-disc-radial-profile-modelling
description: Use when high-resolution ALMA observations of a (debris) disc need fast
  parametric radial-profile fitting directly in the visibility plane, e.g. from an
  exported ms to .npy/.txt visibilities with disc geometry (-g) and radial profile
  (-p) parameters.
when_to_use_negative:
- When radially dependent vertical scale height must be modelled — vis-r fixes vertical
  scale height with radius, and the article notes such structure is beyond ALMA's
  capability for most debris discs anyway
- When a single disc plane assumption is invalid, e.g. multiple disc components needing
  independent orientations — the code assumes one disk plane and shared profile type
- Image-based modelling is required (e.g. full imaging/MCMC workflows like dlmatra/miao)
  rather than rapid visibility-plane radial-profile fits
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3322
tools:
- name: vis-r
  role: 'Primary executable: u,v averaging + Discrete Hankel Transform visibility
    fitting of parametric radial profiles; installed via conda + pip and run as the
    vis-r shell command'
  repo: https://github.com/drgmk/vis-r
- name: emcee
  role: MCMC sampler in the default python/emcee version of vis-r for posterior sampling
    of profile parameters
  repo: https://github.com/drgmk/vis-r
- name: stan
  role: Gradient-based HMC/NUTS sampling backend (--stan flag), with Pathfinder-based
    parameter scale estimation; alternative to emcee with more independent samples
    per step budget
  repo: https://github.com/drgmk/vis-r
- name: arviz
  role: Convergence diagnostics (R-hat, effective sample size) on the emcee and stan
    posterior runs
- name: alma
  role: Helper scripts for exporting visibilities from an ms file and subtracting
    best-fit vismodel visibilities to form residual dirty images
  repo: https://github.com/drgmk/alma
- name: frank
  role: Pinned dependency (scipy<1.12 constraint) during vis-r installation; the conda
    env creation also creates the vis-r shell command
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:257b1b6dbe7b490e87f8eb4501ece0f3adc5b9da3612c9d7b0391ba91641d6e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drgmk/vis-r
  tools:
  - Python
  - alma
  - arviz
  - conda
  - emcee
  - frank
  - stan
  - vis-r
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3322
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# debris-disc-radial-profile-modelling

## Summary

Rapid parametric radial-profile fitting of ALMA debris-disc visibilities using vis-r, which combines u,v-space visibility averaging with a Discrete Hankel Transform to convert parametric radial surface-brightness profiles into visibilities, avoiding image-plane modelling.

## When to use

Use when high-resolution ALMA observations of a (debris) disc need fast parametric radial-profile fitting directly in the visibility plane, e.g. from an exported ms to .npy/.txt visibilities with disc geometry (-g) and radial profile (-p) parameters.

## When NOT to use

- When radially dependent vertical scale height must be modelled — vis-r fixes vertical scale height with radius, and the article notes such structure is beyond ALMA's capability for most debris discs anyway
- When a single disc plane assumption is invalid, e.g. multiple disc components needing independent orientations — the code assumes one disk plane and shared profile type
- Image-based modelling is required (e.g. full imaging/MCMC workflows like dlmatra/miao) rather than rapid visibility-plane radial-profile fits

## Inputs

- Self-calibrated visibilities exported from an ms file to a numpy file (e.g. hr4796.selfcal.npy, HD109573.12m*npy)
- Disc geometry parameters (offset RA/Dec in arcsec, inclination in degrees, position angle in degrees) via -g
- Radial profile parameters (e.g. radius, exponent, width, amplitude) via -p
- Optional --sz u,v-averaging parameter set near the disc size

## Outputs

- Fitted radial profile posterior parameter estimates (emcee or stan/HMC samples)
- Best-fit model visibilities (*vismod.npy) for subtraction from the ms
- Convergence diagnostics computable with arviz (R-hat, effective sample size, trace stability)

## How to apply

Export visibilities from the measurement set to a numpy or text file, discarding calibration and flagged data and averaging first (e.g. 30 s time intervals, a few channels per spw). Run the vis-r executable supplying disc geometry parameters (-g: offset RA/Dec, inclination, position angle) and radial profile parameters (-p), fitting multiple visibility sets at once. Follow the README strategy: start with hard u,v averaging via --sz set to roughly the disc size (or smaller) for quick runs and parameter estimates, then increase --sz for robust posteriors. Sample posteriors with the python/emcee version by default, or add the --stan flag for gradient-based HMC/NUTS sampling (after Pathfinder scale estimation), which yields many more independent samples for the same step budget. Judge the fit by subtracting the best-fit model visibilities (*vismod.npy) from the ms and inspecting dirty images of the residuals.

## Related tools

- **vis-r** (Primary executable: u,v averaging + Discrete Hankel Transform visibility fitting of parametric radial profiles; installed via conda + pip and run as the vis-r shell command) — https://github.com/drgmk/vis-r
- **emcee** (MCMC sampler in the default python/emcee version of vis-r for posterior sampling of profile parameters) — https://github.com/drgmk/vis-r
- **stan** (Gradient-based HMC/NUTS sampling backend (--stan flag), with Pathfinder-based parameter scale estimation; alternative to emcee with more independent samples per step budget) — https://github.com/drgmk/vis-r
- **arviz** (Convergence diagnostics (R-hat, effective sample size) on the emcee and stan posterior runs)
- **alma** (Helper scripts for exporting visibilities from an ms file and subtracting best-fit vismodel visibilities to form residual dirty images) — https://github.com/drgmk/alma
- **frank** (Pinned dependency (scipy<1.12 constraint) during vis-r installation; the conda env creation also creates the vis-r shell command)

## Examples

```
vis-r -v data/hr4796.selfcal.npy --stan -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01
```

## Evaluation signals

- Residual dirty images after subtracting *vismod.npy from the source ms show no structured emission at the disc location
- Sampler convergence: R-hat near 1 and substantially higher effective sample size (arviz) — the stan/HMC run should exceed emcee ESS at equal step count
- Posterior parameter posteriors from stan consistent with emcee estimates within fitted credible intervals
- Initial hard-averaging runs (--sz near disc size) complete quickly and give reasonable parameter estimates before larger --sz runs
- Minimal example script completes in less than 10 s on an M2 Mac, confirming the install and environment (scipy=1.11.4, frank's scipy<1.12 requirement) work

## Limitations

- Vertical scale height is fixed with radius; radially dependent vertical structure cannot be modelled
- Both stan and emcee versions assume a single disk plane; multiple radial components can have independent vertical scales but must all be of the same type (e.g. Gaussian, power)
- Independent astrometry fitting per visibility dataset (--astrom) is implemented only in the emcee version, not the stan version
- frank dependency requires scipy<1.12 (env pins scipy=1.11.4); install instructions target an M2 Mac (CONDA_SUBDIR=osx-arm64)

## Evidence

- [abstract] Two main algorithmic ingredients: [evidence span withheld — anchor status: not_found]
- [abstract] HMC capability and speed claim: "is sufficient for most ALMA observations of debris discs, and can be used with gradient-based fitting methods such as Hamiltonian Monte-Carlo."
- [readme] Export and averaging guidance: "These will export whatever is in the ms, so discard any calibration and flagged data, and it is best to do some averaging first (e.g. 30s time intervals, a few channels per spw)"
- [readme] stan vs emcee differences: "The `stan` version first estimates the parameter scales with the Pathfinder algorithm, and only the `emcee` version has independent astrometry fitting for each visibility dataset `--astrom`"
- [readme] Recommended fitting strategy: "The suggested method of fitting is to initially use rather hard u,v averaging with the `--sz` parameter, setting it to something similar to the disk size, or even smaller."
- [intro] Assumption of single disk plane: "Both assume a single disk plane, but can include multiple radial components with independent vertical scales, though these must all be of the same type"
