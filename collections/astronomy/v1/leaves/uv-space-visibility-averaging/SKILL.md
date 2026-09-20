---
name: uv-space-visibility-averaging
description: Use when fitting parametric radial profiles to ALMA visibilities (e.g.
  exported ms data like HD109573.12m*npy or hr4796.selfcal.npy) and you need fast
  initial runs or coarse-but-robust radially averaged profiles.
when_to_use_negative:
- 'When you need robust, high-precision posterior parameter estimates: aggressive
  --sz near the disk size yields coarse fits; re-run with larger --sz instead.'
- When radially dependent vertical structure must be modelled — vis-r assumes a single
  disk plane with vertical scale height fixed with radius, beyond ALMA's capability
  for most debris discs regardless of averaging.
- When the data require image-plane analysis (e.g. morphological features that are
  not axisymmetric radial profiles) — this skill is specific to u,v-space radial averaging.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: vis-r
  role: provides the vis_r.bin_uv binning function and the --sz CLI parameter that
    implement hard u,v-space averaging of visibility samples
  repo: https://github.com/drgmk/vis-r
- name: emcee
  role: consumes the averaged u,v profile downstream as the default MCMC sampling
    backend for the radial-profile fit
  repo: https://github.com/drgmk/vis-r
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:1a2df97fbb4eb50842b52a70650b2368bd9e8e1bb78ab6f73f4892b3156ad04e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drgmk/vis-r
  tools:
  - ALMA
  - Python
  - emcee
  - vis-r
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# uv-space-visibility-averaging

## Summary

Aggressive hard averaging of ALMA interferometric visibilities in u,v space (element (i) of the vis-r method), controlled by the --sz parameter, to produce a radially averaged visibility profile that makes radial-profile fitting fast enough to run on a laptop.

## When to use

Use when fitting parametric radial profiles to ALMA visibilities (e.g. exported ms data like HD109573.12m*npy or hr4796.selfcal.npy) and you need fast initial runs or coarse-but-robust radially averaged profiles.

## When NOT to use

- When you need robust, high-precision posterior parameter estimates: aggressive --sz near the disk size yields coarse fits; re-run with larger --sz instead.
- When radially dependent vertical structure must be modelled — vis-r assumes a single disk plane with vertical scale height fixed with radius, beyond ALMA's capability for most debris discs regardless of averaging.
- When the data require image-plane analysis (e.g. morphological features that are not axisymmetric radial profiles) — this skill is specific to u,v-space radial averaging.

## Inputs

- Exported visibilities from an ms file in text or numpy format (e.g. data/HD109573.12m*npy, hr4796.selfcal.npy) with calibration/flagged data removed and coarse time/channel averaging applied
- Arrays of u, v, real and imaginary visibility parts and weights
- --sz parameter (angular scale in arcsec, set near the disk size, e.g. a value similar to the disk size or smaller)

## Outputs

- Binned u, v, Re, Im, w arrays forming a radially averaged visibility profile (e.g. saved as radially_averaged_visibilities.npy)
- A smoothed u,v-radius profile covering the range expected for a disk of the assumed size, suitable as input for initial vis-r parameter estimation

## How to apply

After exporting visibilities from the ms file to a text or numpy file (calibration and flagged data discarded, ~30 s time averaging and a few channels per spw applied), group the u, v, Re, Im, w samples as a function of u,v radius using vis_r's binning function, invoked as vis_r.bin_uv(u, v, Re, Im, w, size_arcsec=sz). Set --sz to something similar to the disk size (or even smaller) so the averaging is 'hard': each u,v-radius bin contains many samples, the radially averaged profile is smooth, and the fit completes quickly with reasonable parameter estimates. The averaged profile then feeds the Discrete Hankel Transform stage for vis-r's emcee or stan parameter estimation. Once initial success is confirmed (via residual dirty images), re-run with a larger --sz to obtain more robust posterior estimates. The rationale is that u,v averaging is one of the two main ingredients that makes vis-r one to two orders of magnitude faster than image-based methods.

## Related tools

- **vis-r** (provides the vis_r.bin_uv binning function and the --sz CLI parameter that implement hard u,v-space averaging of visibility samples) — https://github.com/drgmk/vis-r
- **emcee** (consumes the averaged u,v profile downstream as the default MCMC sampling backend for the radial-profile fit) — https://github.com/drgmk/vis-r

## Examples

```
vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01 --sz 0.06
```

## Evaluation signals

- The averaging run completes quickly (the package's minimal example runs in under 10 s on an M2 Mac), consistent with the 10x-100x speedup over image-based methods.
- The resulting radially averaged profile is smooth and covers the u,v-radius range expected for a disk of the assumed size.
- A first fit on the averaged profile gives reasonable parameter estimates, verified by subtracting the *vismod.npy model from the ms and checking dirty images of residuals for no obvious residual structure.
- Setting --sz similar to the disk size (or smaller) yields faster runs than larger --sz; the final robust run should use a larger --sz value.

## Limitations

- Hard u,v averaging (--sz near disk size) gives coarse profiles and quick runs; posterior estimates with larger --sz are needed for robust results.
- The export should discard calibration and flagged data and apply some averaging first (e.g. 30 s time intervals, a few channels per spw), otherwise the exported file is unnecessarily large.
- vis-r assumes a single disk plane; multiple radial components are allowed but must be of the same type with independent vertical scales, and vertical scale height is fixed with radius.

## Evidence

- [methods] hard u,v averaging with --sz: "initially use rather hard u,v averaging with the `--sz` parameter, setting it to something similar to the disk size"
- [abstract] two main elements: "The two main elements are (i) $u,v$ space visibility averaging"
- [abstract] fast and laptop-runnable: "these ingredients yield a method that is one to two orders of magnitude faster than image based methods and can be run on a laptop"
- [readme] export with averaging: "discard any calibration and flagged data, and it is best to do some averaging first (e.g. 30s time intervals, a few channels per spw)"
- [readme] larger sz for robust posteriors: "Given some initial success, more robust posterior
estimates can be obtained with larger `--sz`"
