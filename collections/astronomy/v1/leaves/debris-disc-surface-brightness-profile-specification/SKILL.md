---
name: debris-disc-surface-brightness-profile-specification
description: Use when you have u,v visibilities exported to a numpy/text file from
  an ALMA ms (e.g. hr4796.selfcal.npy) and want to model the disc as a parametric
  radial profile evaluated directly in visibility space.
when_to_use_negative:
- The disc needs radially dependent vertical structure — vis-r's vertical scale height
  is fixed with radius and this information is beyond even ALMA's capability for most
  debris discs
- You need multiple disk components with independent orientations, or general 2D/image-plane
  structure — vis-r assumes a single disk plane
- Your model requires full image-plane forward-modelling (e.g. the galario-based workflow
  in the drgmk/alma repo) rather than a radial-profile approximation
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: vis-r
  role: Implements the DHT matrix-product transform (radial profile at Hankel nodes
    Rnk times Ykm) mapping I(r) to V(q) without image formation; installed via pip
    from GitHub
  repo: https://github.com/drgmk/vis-r/archive/refs/heads/main.zip
- name: emcee
  role: Python MCMC backend that fits the parametric profile specification to the
    exported visibilities
  repo: https://github.com/drgmk/vis-r
- name: stan
  role: Alternative sampling backend (via cmdstanpy) for the same parametric profile
    model; estimates parameter scales with the Pathfinder algorithm first
  repo: https://github.com/drgmk/vis-r
- name: frank
  role: 'Related radial-profile visibility package referenced by vis-r (dependency
    constraint: scipy<1.12, i.e. scipy=1.11.4)'
  repo: https://github.com/drgmk/vis-r
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:072a6ee7f19a86de04e9e5d7885e8cb46a689a48d6ffb70e8d23534456719f2c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - emcee
  - frank
  - stan
  - vis-r
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# debris-disc-surface-brightness-profile-specification

## Summary

Specify a parametric radial surface-brightness profile for a debris disc and convert it directly to complex visibilities with the Discrete Hankel Transform (DHT), bypassing image-plane formation. This is the second ingredient of the vis-r method and is what makes radial-profile fitting one to two orders of magnitude faster than image-based modelling of ALMA visibilities.

## When to use

Use when you have u,v visibilities exported to a numpy/text file from an ALMA ms (e.g. hr4796.selfcal.npy) and want to model the disc as a parametric radial profile evaluated directly in visibility space.

## When NOT to use

- The disc needs radially dependent vertical structure — vis-r's vertical scale height is fixed with radius and this information is beyond even ALMA's capability for most debris discs
- You need multiple disk components with independent orientations, or general 2D/image-plane structure — vis-r assumes a single disk plane
- Your model requires full image-plane forward-modelling (e.g. the galario-based workflow in the drgmk/alma repo) rather than a radial-profile approximation

## Inputs

- Parametric radial profile specification (vis-r '-p' parameters, e.g. 0.013 1.07 0.06 0.01) and geometry ('-g') parameters
- u,v coordinate dataset exported from an ms file as numpy array (e.g. hr4796.selfcal.npy)

## Outputs

- Complex model visibilities V(q) at the sampled u,v points, written to a '*vismod.npy' numpy file for downstream subtraction from the ms

## How to apply

Specify the profile with the vis-r '-p' style parameters (e.g. '-p 0.013 1.07 0.06 0.01' for a Gaussian-type radial component) together with the geometry ('-g 0.012 -0.035 26.6 76.5'). Implement the DHT as a matrix product: evaluate the radial profile I(r) at the Hankel nodes Rnk, then multiply by the Ykm matrix to obtain complex visibilities V(q) at the sampled Qzero points — equivalently h.transform(f, q=Qzero) — honouring vis-r's single-disk-plane assumption. Load the u,v coordinates from the exported numpy file, apply the transform at the sampled u,v points, and write the model visibilities to a '*vismod.npy' file so they can be subtracted from the ms downstream. Multiple radial components with independent vertical scales are allowed, but all must be of the same type (e.g. Gaussian, power); compact central star, point, and Gaussian sources can be added on top.

## Related tools

- **vis-r** (Implements the DHT matrix-product transform (radial profile at Hankel nodes Rnk times Ykm) mapping I(r) to V(q) without image formation; installed via pip from GitHub) — https://github.com/drgmk/vis-r/archive/refs/heads/main.zip
- **emcee** (Python MCMC backend that fits the parametric profile specification to the exported visibilities) — https://github.com/drgmk/vis-r
- **stan** (Alternative sampling backend (via cmdstanpy) for the same parametric profile model; estimates parameter scales with the Pathfinder algorithm first) — https://github.com/drgmk/vis-r
- **frank** (Related radial-profile visibility package referenced by vis-r (dependency constraint: scipy<1.12, i.e. scipy=1.11.4)) — https://github.com/drgmk/vis-r

## Examples

```
vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01
```

## Evaluation signals

- The transform's matrix product (profile at Hankel nodes Rnk × Ykm) matches the reference implementation h.transform(f, q=Qzero) at the sampled Qzero points
- A '*vismod.npy' file is produced and loadable with numpy for each fitted visibility set
- The minimal example script completes in less than 10 s on an M2 Mac, confirming the transform's speed advantage
- Subtracting the model visibilities from the ms and imaging residuals yields dirty images that support a good fit
- Profile parameters are constrained when fitting starts with hard u,v averaging (--sz set to roughly the disk size or smaller)

## Limitations

- Assumes a single disk plane; multiple radial components must all be of the same type (e.g. Gaussian, power), even if vertical scales differ
- Vertical scale height can be included but is fixed with radius; radially dependent vertical structure cannot be modelled
- Profile evaluation is only valid at the sampled u,v points (Qzero); arbitrary u,v locations require re-running the transform

## Evidence

- [abstract] DHT converts profiles to visibilities without images: "using the Discrete Hankel Transform to convert arbitrary parametric radial surface brightness profiles to visibilities"
- [intro] vis-r purpose: "`vis-r` is a method and the associated code for rapid radial profile modelling of interferometric visibilities."
- [intro] Single-plane / component-type restriction: "Both assume a single disk plane, but can include multiple radial components with independent vertical scales, though these must all be of the same type"
- [readme] vismod output used for subtraction: "Subtract the best fit model (`*vismod.npy`) from the ms from which the visibilities were exported"
- [results] Runtime validation budget: "# minimal example script, this runs in less than 10s on an M2 Mac"
- [methods] CLI profile/geometry specification: "vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01"
