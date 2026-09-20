---
name: discrete-hankel-transform-implementation
description: Use when fitting ALMA visibility data with parametric radial profiles
  (e.g. the '-p 0.013 1.07 0.06 0.01' style vis-r parameters) and a fast model-visibility
  prediction is needed without image generation.
when_to_use_negative:
- When radially dependent vertical (scale-height) structure must be modelled — vis-r
  fixes vertical scale height with radius, so the DHT radial-profile route is insufficient
- When the model requires multiple disk planes or independent orientations per disk
  component, since both vis-r versions assume a single disk plane
- When an image-plane product (e.g. a dirty image of the model itself) rather than
  visibilities is the required output
tools:
- name: vis-r
  role: Package providing the DHT-based radial-profile-to-visibilities method; the
    transform is exercised via its h.transform(f, q=Qzero) interface and installed
    from its GitHub repo
  repo: https://github.com/drgmk/vis-r
- name: Python
  role: 'Implementation language: the DHT is implemented as a numpy matrix product
    (profile at Hankel nodes Rnk times Ykm) and the result saved via numpy'
- name: ALMA
  role: Instrument whose high-resolution debris-disc visibilities motivate the DHT
    approach over expensive image-based fitting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:688fd3637a01308a6eff154536eacd8b2ecc7c46969edebb244085438e63def1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drgmk/vis-r
  tools:
  - ALMA
  - Python
  - vis-r
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# discrete-hankel-transform-implementation

## Summary

Implementation of the Discrete Hankel Transform (DHT) as used by vis-r to convert an arbitrary parametric radial surface-brightness profile I(r) directly into complex visibilities V(q) for radio interferometric data, bypassing image-plane formation entirely.

## When to use

Use when fitting ALMA visibility data with parametric radial profiles (e.g. the '-p 0.013 1.07 0.06 0.01' style vis-r parameters) and a fast model-visibility prediction is needed without image generation.

## When NOT to use

- When radially dependent vertical (scale-height) structure must be modelled — vis-r fixes vertical scale height with radius, so the DHT radial-profile route is insufficient
- When the model requires multiple disk planes or independent orientations per disk component, since both vis-r versions assume a single disk plane
- When an image-plane product (e.g. a dirty image of the model itself) rather than visibilities is the required output

## Inputs

- uv coordinate dataset as numpy file (e.g. hr4796.selfcal.npy from the vis-r examples folder)
- parametric radial surface-brightness profile specification (vis-r '-p' style parameters, e.g. 0.013 1.07 0.06 0.01)

## Outputs

- complex model visibilities V(q) at the sampled Qzero points
- numpy file written with the '*vismod.npy' naming convention for downstream subtraction from the ms

## How to apply

Load the sampled u,v coordinates (e.g. hr4796.selfcal.npy from the vis-r examples folder) and evaluate the parametric radial profile at the Hankel nodes Rnk. Implement the DHT as a matrix product: multiply the radial profile evaluated at Hankel nodes Rnk by the matrix Ykm to give visibilities at Qzero, equivalently calling h.transform(f, q=Qzero). The transform honours vis-r's single-disk-plane assumption, so orientation/offset handling stays outside the DHT itself. Write the computed model visibilities to a '*vismod.npy' numpy file so they can be subtracted from the measurement set downstream. The rationale is that image-based FFT fitting of high-resolution ALMA observations of debris discs is computationally expensive, whereas the DHT-based radial-profile route is one to two orders of magnitude faster and laptop-runnable.

## Related tools

- **vis-r** (Package providing the DHT-based radial-profile-to-visibilities method; the transform is exercised via its h.transform(f, q=Qzero) interface and installed from its GitHub repo) — https://github.com/drgmk/vis-r
- **Python** (Implementation language: the DHT is implemented as a numpy matrix product (profile at Hankel nodes Rnk times Ykm) and the result saved via numpy)
- **ALMA** (Instrument whose high-resolution debris-disc visibilities motivate the DHT approach over expensive image-based fitting)

## Examples

```
vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01
```

## Evaluation signals

- Minimal example script (hello_world.py style) completes in less than 10 s on an M2 Mac
- A loadable '*vismod.npy' numpy file is produced and can be read back with np.load
- Model visibilities are evaluated at the sampled Qzero points matching the loaded u,v dataset
- Downstream subtraction of the *vismod.npy model from the exported ms yields residual visibilities/dirty images consistent with a good radial-profile fit

## Limitations

- Assumes a single disk plane; multiple radial components can have independent vertical scales but must all be of the same type (e.g. Gaussian, power)
- Vertical scale height can be included but is fixed with radius; radially dependent vertical structure is not supported
- Compact central star, point, and Gaussian sources can be added, but arbitrary 2D structure (e.g. spirals, eccentricity beyond parametric radial terms) is outside the DHT scope

## Evidence

- [methods] DHT as matrix product: [evidence span withheld — anchor status: not_found]
- [abstract] DHT purpose: "using the Discrete Hankel Transform to convert arbitrary parametric radial surface brightness profiles to visibilities"
- [results] runtime budget: "# minimal example script, this runs in less than 10s on an M2 Mac"
- [readme] model output convention: "Subtract the best fit model (`*vismod.npy`) from the ms from which the visibilities were exported"
- [intro] single plane assumption: "Both assume a single disk plane, but can include multiple radial components with independent vertical scales"
