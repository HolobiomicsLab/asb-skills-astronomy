---
name: debris-disc-geometry-parameterization
description: Use when fitting an inclined debris disc in u,v space with vis-r and
  you need to fix or initialize disc geometry (inclination, PA, sky-plane offsets)
  from e.g. scattered-light or moment maps before sampling radial-profile parameters.
when_to_use_negative:
- Your model needs independent orientations for multiple disk components — vis-r assumes
  a single disk plane shared by all radial components
- You need a radially dependent vertical scale height — the article notes vertical
  scale height is fixed with radius
- The input is an image or an unexported ms file — vis-r expects visibilities first
  exported to a text or numpy file
edam_operation: http://edamontology.org/operation_0336
tools:
- name: vis-r
  role: Runs the parametric radial-profile fit with the -g disc geometry argument
    (offsets, inclination, PA) on the exported visibilities
  repo: https://github.com/drgmk/vis-r/archive/refs/heads/main.zip
- name: emcee
  role: Ensemble MCMC sampler behind the default vis-r mode that samples the geometry
    and radial-profile parameters (18 walkers, 1000 steps, 6 threads)
  repo: https://github.com/drgmk/vis-r
- name: CASA / ms export utilities
  role: Exports the visibilities from the ms to a text or numpy file, discarding calibration
    and flagged data, before geometry-parameterized fitting
  repo: https://github.com/drgmk/alma/blob/a10e4e40e84ff3a25dfa3b0067fec5f4c3713b1d/alma/casa.py#L226
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:315c543140aa86ab7e3625a65c5b7058b17ccfd3df8abf1a76272be48666ade0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CASA / ms export utilities
  - Python
  - emcee
  - vis-r
  edam_operation: http://edamontology.org/operation_0336
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# debris-disc-geometry-parameterization

## Summary

Parameterizes a debris disc's geometry (RA/Dec offsets, inclination, position angle) for vis-r radial-profile fitting of ALMA visibilities, supplied via the -g argument, so a single disk plane is shared by all radial components while offsets can vary per visibility dataset in the emcee version.

## When to use

Use when fitting an inclined debris disc in u,v space with vis-r and you need to fix or initialize disc geometry (inclination, PA, sky-plane offsets) from e.g. scattered-light or moment maps before sampling radial-profile parameters.

## When NOT to use

- Your model needs independent orientations for multiple disk components — vis-r assumes a single disk plane shared by all radial components
- You need a radially dependent vertical scale height — the article notes vertical scale height is fixed with radius
- The input is an image or an unexported ms file — vis-r expects visibilities first exported to a text or numpy file

## Inputs

- Self-calibrated ALMA visibilities exported to a numpy file (e.g. hr4796.selfcal.npy or HD109573.12m*npy)
- Disc geometry parameters: RA offset, Dec offset, inclination (deg), position angle (deg) supplied via -g
- Radial profile initial parameters supplied via -p (e.g. p0 = [0.03, 0.03, 26.7, 76.7, 0.015, 1.08, 0.06, 0.04])
- Optional --sz u,v averaging scale set similar to the disk size

## Outputs

- Best-fit model visibilities saved to the *vismod.npy output file
- Posterior parameter estimates from the emcee chain (18 walkers, 1000 steps, 6 threads)

## How to apply

Export the self-calibrated visibilities (e.g. hr4796.selfcal.npy) from the ms to a numpy file, discarding calibration and flagged data and averaging (e.g. 30s intervals, a few channels per spw). Supply the disc geometry to vis-r via the -g argument as RA offset, Dec offset, inclination (deg), and position angle (deg), e.g. the repository example uses -g 0.012 -0.035 26.6 76.5 for HD 109573; for HR 4796 the fit uses i≈76.7 deg, PA≈26.7 deg. Pass radial profile parameters via -p in the style p0 = [0.03, 0.03, 26.7, 76.7, 0.015, 1.08, 0.06, 0.04], running the default emcee mode (18 walkers, 1000 steps, 6 threads). vis-r assumes a single disk plane for all radial components; if multiple visibility sets are supplied, use the emcee version's --astrom option to fit independent astrometry per dataset. Start with hard u,v averaging using --sz set near the disk size, then increase --sz for more robust posteriors.

## Related tools

- **vis-r** (Runs the parametric radial-profile fit with the -g disc geometry argument (offsets, inclination, PA) on the exported visibilities) — https://github.com/drgmk/vis-r/archive/refs/heads/main.zip
- **emcee** (Ensemble MCMC sampler behind the default vis-r mode that samples the geometry and radial-profile parameters (18 walkers, 1000 steps, 6 threads)) — https://github.com/drgmk/vis-r
- **CASA / ms export utilities** (Exports the visibilities from the ms to a text or numpy file, discarding calibration and flagged data, before geometry-parameterized fitting) — https://github.com/drgmk/alma/blob/a10e4e40e84ff3a25dfa3b0067fec5f4c3713b1d/alma/casa.py#L226

## Examples

```
vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01
```

## Evaluation signals

- The *vismod.npy output file is produced by the vis-r run
- With --sz set similar to the disk size, the run completes quickly and gives reasonable parameter estimates
- Subtracting *vismod.npy from the source ms and making dirty images of the residuals shows the model fits the data well
- Recovered inclination and position angle agree with known disc geometry (HR 4796: i ≈ 76.7 deg, PA ≈ 26.7 deg)
- When multiple visibility datasets are supplied with --astrom, each dataset receives its own independent astrometric offset

## Limitations

- Both vis-r versions assume a single disk plane; multiple radial components with independent vertical scales must all be of the same type (e.g. Gaussian, power)
- Vertical scale height can be included but is fixed with radius; radially dependent vertical structure is beyond even ALMA's capability for most debris discs
- The stan and python/emcee versions differ: only the emcee version implements --astrom independent astrometry per dataset; stan first estimates parameter scales with the Pathfinder algorithm
- Geometry is shared across all radial components unless the user modifies the code (e.g. to allow independent orientations per component)

## Evidence

- [readme] Example -g invocation for HD 109573: "vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01"
- [intro] Single disk plane assumption: "Both assume a single disk plane, but can include multiple radial components with independent vertical scales, though these must all be of the same type"
- [readme] Per-dataset independent offsets in emcee version: "Each set of visibilities can have independent offset parameters in the `emcee` version."
- [readme] --astrom option is emcee-only: "only the `emcee` version has independent astrometry fitting for each visibility dataset `--astrom` implemented"
- [abstract] Vertical scale height fixed with radius: "Vertical scale height can be included, but is fixed with radius"
- [readme] Suggested initial --sz averaging strategy: "The suggested method of fitting is to initially use rather hard u,v averaging with the `--sz` parameter, setting it to something similar to the disk size"
