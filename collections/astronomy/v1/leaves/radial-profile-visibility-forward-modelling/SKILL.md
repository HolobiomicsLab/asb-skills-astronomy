---
name: radial-profile-visibility-forward-modelling
description: Use when fitting parametric radial brightness profiles to ALMA/exported
  visibilities (numpy or text files from an ms) and you need model visibilities computed
  without ever gridding or imaging the sky.
when_to_use_negative:
- The modelling target requires radially varying vertical structure — vis-r assumes
  a single disk plane with vertical scale height fixed with radius, which the paper
  notes is beyond ALMA's capability for most debris discs anyway.
- Each disk component needs independent orientation — vis-r assumes a single disk
  plane (independent orientations per component require code changes).
- The analysis goal is image-plane reconstruction or imaging rather than visibility-domain
  radial profile fitting.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: vis-r
  role: Provides the DHT-based radial profile to visibilities machinery and the '*vismod.npy'
    output convention used in this forward-modelling step
  repo: github.com/drgmk/vis-r
- name: Python
  role: Language in which the DHT matrix product (profile at Hankel nodes Rnk times
    Ykm) is implemented, e.g. via h.transform(f, q=Qzero)
- name: emcee
  role: Optional MCMC backend driving repeated forward-model evaluations of the radial
    profile to visibilities
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/rasti/rzaf004
    title: '<tt>vis-r</tt>: Quick radial profile modelling for radio interferometric
      visibilities'
  source_content_hash: sha256:d0aaec98bbb57ff69a2b348c3436d41a99e31d067bf7188c57e0cb37e7bf2bc6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - emcee
  - vis-r
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# radial-profile-visibility-forward-modelling

## Summary

Forward-models arbitrary parametric radial surface-brightness profiles I(r) directly into interferometric visibilities V(q) via a Discrete Hankel Transform, bypassing image-plane formation. This is the second ingredient of the vis-r method and underpins its 1–2 order-of-magnitude speed-up over image-based fitting for ALMA debris-disc data.

## When to use

Use when fitting parametric radial brightness profiles to ALMA/exported visibilities (numpy or text files from an ms) and you need model visibilities computed without ever gridding or imaging the sky.

## When NOT to use

- The modelling target requires radially varying vertical structure — vis-r assumes a single disk plane with vertical scale height fixed with radius, which the paper notes is beyond ALMA's capability for most debris discs anyway.
- Each disk component needs independent orientation — vis-r assumes a single disk plane (independent orientations per component require code changes).
- The analysis goal is image-plane reconstruction or imaging rather than visibility-domain radial profile fitting.

## Inputs

- uv coordinates exported from an ms file as a numpy or text file (e.g. hr4796.selfcal.npy)
- parametric radial surface-brightness profile specification (vis-r '-p' style parameters)
- optional auxiliary model terms: offsets, central star/point/Gaussian sources

## Outputs

- complex model visibilities V(q) evaluated at the sampled u,v points
- model visibility file written with the '*vismod.npy' naming convention

## How to apply

Load the exported u,v dataset (e.g. a numpy file such as hr4796.selfcal.npy) and the parametric profile specification (the '-p 0.013 1.07 0.06 0.01' style parameters of vis-r). Implement the Discrete Hankel Transform as a matrix product: evaluate the radial profile at the Hankel nodes Rnk, multiply by the Ykm matrix, and obtain complex visibilities at the sampled q values — equivalently h.transform(f, q=Qzero). The transform honours vis-r's single-disk-plane assumption, though multiple radial components with independent vertical scales (of the same type, e.g. Gaussian or power) and compact central star/point/Gaussian sources can be included. Write the computed model visibilities to a numpy file using the '*vismod.npy' convention so they can later be subtracted from the source ms. The minimal example script completes in under 10 s on an M2 Mac, so the transform should be essentially instantaneous per evaluation, enabling use inside gradient-based samplers such as Hamiltonian Monte-Carlo.

## Related tools

- **vis-r** (Provides the DHT-based radial profile to visibilities machinery and the '*vismod.npy' output convention used in this forward-modelling step) — github.com/drgmk/vis-r
- **Python** (Language in which the DHT matrix product (profile at Hankel nodes Rnk times Ykm) is implemented, e.g. via h.transform(f, q=Qzero))
- **emcee** (Optional MCMC backend driving repeated forward-model evaluations of the radial profile to visibilities)

## Examples

```
vis-r -v data/HD109573.12m*npy -g 0.012 -0.035 26.6 76.5 -p 0.013 1.07 0.06 0.01
```

## Evaluation signals

- The '*vismod.npy' file is produced and loadable with numpy.
- The full minimal example script completes in under 10 s on an M2 Mac, as reported for vis-r.
- Model visibilities are complex-valued and evaluated at exactly the q values of the sampled u,v points.
- Subtracting the best-fit model from the ms and making dirty images shows no strong structured residuals.
- Subsequent fitting runs complete quickly and give reasonable parameter estimates with hard u,v averaging (--sz set to roughly the disk size).

## Limitations

- Assumes a single disk plane; multiple radial components can have independent vertical scales but must all be of the same type (e.g. Gaussian, power).
- Vertical scale height is fixed with radius; radially dependent vertical structure cannot be modelled.
- Independent astrometry fitting per dataset and independent offset parameters per dataset are only available in the emcee version, not the stan version.
- Exported ms data must have calibration and flagged data discarded and be averaged first (e.g. 30 s intervals, a few channels per spw) to keep file sizes manageable.

## Evidence

- [results] DHT matrix product: [evidence span withheld — anchor status: not_found]
- [abstract] DHT ingredient of vis-r: "(ii) using the Discrete Hankel Transform to convert arbitrary parametric radial surface brightness profiles to visibilities"
- [methods] output convention: "Subtract the best fit model (`*vismod.npy`) from the ms from which the visibilities were exported"
- [readme] single-plane assumption: "Both assume a single disk plane, but can include multiple radial components with independent vertical scales"
- [results] speed budget: "# minimal example script, this runs in less than 10s on an M2 Mac"
