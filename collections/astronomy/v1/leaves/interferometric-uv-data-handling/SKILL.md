---
name: interferometric-uv-data-handling
description: Use when you have a parametric radial brightness profile (e.g. '-p 0.013
  1.07 0.06 0.01' style parameters) for a circumstellar disc and need model visibilities
  at sampled u,v points without rendering images.
when_to_use_negative:
- The disc requires radially dependent vertical structure or multiple disc planes
  — vis-r assumes a single disk plane with a fixed (radius-independent) vertical scale
  height
- The geometry is non-axisymmetric or needs a full 2D image-plane model; the DHT route
  only produces visibilities of a radial surface-brightness profile
- The input is not yet an averaged u,v dataset exported from an ms file — calibration/flagged
  data must first be discarded and averaged (e.g. 30 s intervals, a few channels per
  spw)
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: vis-r
  role: implements the Discrete Hankel Transform as a matrix product mapping radial
    profile values at Hankel nodes Rnk to visibilities at Qzero (h.transform(f, q=Qzero))
  repo: https://github.com/drgmk/vis-r
- name: Python
  role: host language for the minimal example script that applies the DHT and writes
    the vismod numpy file
  repo: https://github.com/drgmk/vis-r
- name: numpy
  role: reads the hr4796.selfcal.npy uv dataset and serialises the computed model
    visibilities as '*vismod.npy'
  repo: https://github.com/drgmk/vis-r
- name: frank
  role: dependency of the vis-r install (wants scipy<1.12); related Hankel-transform
    visibility modelling context
  repo: https://github.com/drgmk/vis-r
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:042df0ae63a83c0022ba467d63fd8b17a6a00a952fb34cd3b80ad131787d060e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drgmk/vis-r
  tools:
  - Python
  - frank
  - numpy
  - vis-r
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# interferometric-uv-data-handling

## Summary

Converts an arbitrary parametric radial surface-brightness profile I(r) directly into complex interferometric visibilities V(q) using the Discrete Hankel Transform, as implemented in vis-r as a matrix product, bypassing image-plane formation entirely.

## When to use

Use when you have a parametric radial brightness profile (e.g. '-p 0.013 1.07 0.06 0.01' style parameters) for a circumstellar disc and need model visibilities at sampled u,v points without rendering images.

## When NOT to use

- The disc requires radially dependent vertical structure or multiple disc planes — vis-r assumes a single disk plane with a fixed (radius-independent) vertical scale height
- The geometry is non-axisymmetric or needs a full 2D image-plane model; the DHT route only produces visibilities of a radial surface-brightness profile
- The input is not yet an averaged u,v dataset exported from an ms file — calibration/flagged data must first be discarded and averaged (e.g. 30 s intervals, a few channels per spw)

## Inputs

- parametric radial profile specification (vis-r '-p' style parameters, e.g. '0.013 1.07 0.06 0.01')
- uv coordinate dataset as a numpy file (e.g. hr4796.selfcal.npy exported from an ms file)
- Discrete Hankel Transform object/plan (Hankel nodes Rnk, Ykm matrix, target q points Qzero)

## Outputs

- complex model visibilities V(q) at the sampled u,v points
- numpy file of model visibilities following the '*vismod.npy' naming convention

## How to apply

Load the uv coordinate dataset (e.g. the numpy file hr4796.selfcal.npy from the vis-r examples folder) and the parametric radial profile specification for the single-disc-plane model. Implement the Discrete Hankel Transform as a matrix product: evaluate the radial profile I(r) at the Hankel nodes Rnk, then multiply by the Ykm matrix to obtain visibilities at Qzero, which is equivalent to calling h.transform(f, q=Qzero). Evaluate the result at the sampled u,v points honoured by the single-disk-plane assumption of vis-r, so multiple radial components may have independent vertical scales but all must be of the same type. Write the complex model visibilities to a numpy file using the '*vismod.npy' naming convention so they can be subtracted from the measurement set downstream. Rationale: this DHT ingredient, combined with u,v space visibility averaging, makes vis-r one to two orders of magnitude faster than image-based methods and laptop-runnable.

## Related tools

- **vis-r** (implements the Discrete Hankel Transform as a matrix product mapping radial profile values at Hankel nodes Rnk to visibilities at Qzero (h.transform(f, q=Qzero))) — https://github.com/drgmk/vis-r
- **Python** (host language for the minimal example script that applies the DHT and writes the vismod numpy file) — https://github.com/drgmk/vis-r
- **numpy** (reads the hr4796.selfcal.npy uv dataset and serialises the computed model visibilities as '*vismod.npy') — https://github.com/drgmk/vis-r
- **frank** (dependency of the vis-r install (wants scipy<1.12); related Hankel-transform visibility modelling context) — https://github.com/drgmk/vis-r

## Examples

```
V = h.transform(f, q=Qzero)  # radial profile f at Hankel nodes Rnk times Ykm gives visibilities at Qzero
```

## Evaluation signals

- The matrix product (radial profile at Hankel nodes Rnk) x Ykm agrees with h.transform(f, q=Qzero) at the target q points
- A loadable '*vismod.npy' numpy file of complex visibilities is produced
- The minimal example script completes in less than 10 s on an M2 Mac
- Subtracting the vismod file from the ms yields clean dirty-image residuals
- The evaluation honours the single disk plane assumption, i.e. visibilities are evaluated only at the sampled u,v points of the exported dataset

## Limitations

- Assumes a single disk plane; multiple radial components can have independent vertical scales but must all be of the same type (e.g. Gaussian, power)
- Vertical scale height is fixed with radius — radially dependent vertical structure is beyond this method
- If many profile functions must be added, the code should be cloned and installed with 'pip install -e .' so the vis-r executable picks up local edits

## Evidence

- [abstract] DHT converts profiles to visibilities: "using the Discrete Hankel Transform to convert arbitrary parametric radial surface brightness profiles to visibilities"
- [abstract] Speed advantage of the DHT route: "Combined, these ingredients yield a method that is one to two orders of magnitude faster than image based methods and can be run on a laptop"
- [readme] Single disk plane assumption: "Both assume a single disk plane, but can include multiple radial components with independent vertical scales, though these must all be of the same type"
- [readme] vismod output convention for subtraction: "Subtract the best fit model (`*vismod.npy`) from the ms from which the visibilities were exported"
- [results] Runtime budget of the minimal example: "# minimal example script, this runs in less than 10s on an M2 Mac"
