---
name: resolution-dependent-model-agreement-analysis
description: Use when validating a new or faster galactic kinematic modelling tool
  against an established code, or when you need to know how beam smearing (PSF FWHM
  relative to R_eff) affects agreement between two mock-observed rotation/dispersion
  curve predictions.
when_to_use_negative:
- The galaxy is not an axisymmetric, rotationally supported disk (V/sigma_0 >= 2)
  with a well-defined kinematic axis and no strong deviations from circular motions
- The system is unresolved (beam size much larger than the disk effective radius),
  where beam-smearing corrections break down and discrepancies reach ~15 per cent
  even between codes
- You need full 3D data-cube forward modelling rather than 1D rotation-curve comparison,
  since this benchmark only compares mock-observed 1D rotation and dispersion curves
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: RotCurves
  role: Fast parametric forward-modelling tool under test; constructs the fiducial
    mass model and generates beam-smeared rotation and dispersion curves in ~10 ms
    per realization
  repo: https://github.com/nestoramit/RotCurves
- name: dysmalpy
  role: Established reference parametric code used as the ground-truth comparison
    for the identical mock observations
- name: Python
  role: Implementation language for the benchmark workflow, timing, and fractional-difference
    computations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stag134
    title: '<tt>RotCurves</tt> : a <tt>PYTHON</tt> package for efficient modelling
      and fitting of galactic rotation curves at high- <i>z</i>'
  source_content_hash: sha256:df0a11a09301cbbe761a1fddb591cd8a7641af1dbc01c7e59b32ea34ab9e4c60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nestoramit/RotCurves
  tools:
  - Python
  - RotCurves
  - dysmalpy
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# resolution-dependent-model-agreement-analysis

## Summary

Benchmark a fast parametric rotation-curve forward-modelling code (RotCurves) against an established reference code (dysmalpy) by mock-observing an identical fiducial galaxy mass model at different resolutions and comparing beam-smeared rotation and dispersion curves out to 3R_eff, while recording single-realization runtime ratios.

## When to use

Use when validating a new or faster galactic kinematic modelling tool against an established code, or when you need to know how beam smearing (PSF FWHM relative to R_eff) affects agreement between two mock-observed rotation/dispersion curve predictions.

## When NOT to use

- The galaxy is not an axisymmetric, rotationally supported disk (V/sigma_0 >= 2) with a well-defined kinematic axis and no strong deviations from circular motions
- The system is unresolved (beam size much larger than the disk effective radius), where beam-smearing corrections break down and discrepancies reach ~15 per cent even between codes
- You need full 3D data-cube forward modelling rather than 1D rotation-curve comparison, since this benchmark only compares mock-observed 1D rotation and dispersion curves

## Inputs

- Fiducial galaxy mass model definition (disk, bulge, dark matter halo with selectable analytical density profiles) in RotCurves
- Sky-orientation and instrumental PSF specifications for the mock observation
- Matched dysmalpy model configuration using the same sky orientation and PSF
- Resolution configurations: well-resolved (PSF FWHM < R_eff) and marginally resolved (PSF FWHM >= 1.5 R_eff)

## Outputs

- Mock-observed beam-smeared rotation and dispersion curves from RotCurves and dysmalpy out to 3 R_eff
- Fractional-difference curves between the two codes as a function of radius for both resolution cases
- Single-realization runtimes and the runtime ratio (target ~250 in favor of RotCurves)
- benchmark_results.csv containing runtimes and fractional-agreement values for both resolution cases

## How to apply

Construct the fiducial axisymmetric galaxy mass model (disk, bulge, dark matter halo) in RotCurves and generate the intrinsic equilibrium rotation velocity out to at least 3 R_eff. Mock-observe it with a well-resolved configuration (PSF FWHM < R_eff) by projecting and convolving the beam PSF in the plane of the galaxy to reconstruct beam-smeared rotation and dispersion curves, then repeat the identical mock observation with dysmalpy for the same sky orientation and PSF. Time a single realization of each code and record the runtime ratio (expected ~250, with RotCurves at ~10 ms per realization). Repeat for a marginally resolved configuration (PSF FWHM >= 1.5 R_eff) and compute the fractional difference between the two codes' mock-observed curves as a function of radius up to 3 R_eff for both cases. Write runtimes and fractional-agreement values to benchmark_results.csv; a well-resolved run agreeing within 5 per cent and a marginally resolved run showing up to ~15 per cent discrepancy confirms the reported benchmark behavior.

## Related tools

- **RotCurves** (Fast parametric forward-modelling tool under test; constructs the fiducial mass model and generates beam-smeared rotation and dispersion curves in ~10 ms per realization) — https://github.com/nestoramit/RotCurves
- **dysmalpy** (Established reference parametric code used as the ground-truth comparison for the identical mock observations)
- **Python** (Implementation language for the benchmark workflow, timing, and fractional-difference computations)

## Examples

```
pip install rotcurves
```

## Evaluation signals

- Single-realization runtime ratio is approximately 250 in favor of RotCurves, with RotCurves at ~10 ms per realization
- Well-resolved case (PSF FWHM < R_eff): fractional difference between RotCurves and dysmalpy rotation and dispersion curves stays within 5 per cent up to 3 R_eff
- Marginally resolved case (PSF FWHM >= 1.5 R_eff): discrepancies grow but remain up to about 15 per cent
- Fractional differences are computed consistently as a function of radius out to 3 R_eff for both resolution configurations
- All runtimes and fractional-agreement values are present in benchmark_results.csv for both cases

## Limitations

- Marginally resolved systems (PSF FWHM >= 1.5 R_eff) show agreement discrepancies of up to ~15 per cent, so results in that regime carry larger uncertainty
- RotCurves fits best for resolved systems with beam size the same size as or smaller than the disk effective radius (PSF <~ R_eff,disk)
- The mass model assumes idealized axisymmetric, rotationally supported disks (V/sigma_0 >~ 2) with a well-defined kinematic axis and no strong deviations from circular motions
- Low disc masses (M_disc <~ 3x10^9 M_sun) and the effective radius show systematic biases in parameter recovery in related MCMC tests

## Evidence

- [abstract] For well-resolved systems (PSF FWHM < R_eff), mock rotation and dispersion curves agree with dysmalpy within 5 per cent up to 3 R_eff; marginally resolved systems show discrepancies up to 15 per cent: "For well-resolved systems (PSF FWHM $< R_{\rm eff}$), the mock observed rotation and dispersion curves agree to within 5 per cent up to $3R_{\rm eff}$"
- [abstract] RotCurves runtime is ~10 ms per realization, a factor ~250 faster than dysmalpy: "The typical runtime with RotCurves is a few $\sim 10$ ms, a factor $\approx 250$ faster than dysmalpy for a single realization."
- [abstract] Beam-smearing correction via beam projection and PSF convolution: "correcting for ‘beam smearing’ by projecting and convolving the beam PSF in the plane of the galaxy"
- [intro] The code constructs a mass model of an idealized, axisymmetric galaxy with selectable analytical density profiles: "The code constructs a mass model of an idealized, axisymmetric galaxy with mass components following defined analytical density distributions"
- [readme] Fitting procedure works best for resolved systems with beam size no larger than the disk effective radius: "The fitting procedure peforms best for resolved systems, with a beam size that is of the same size or smaller than the effective radius of the disk $PSF \lesssim R_{\rm eff,disk}$."
- [intro] Scope restricted to rotationally supported axisymmetric disks with no strong deviations from circular motions: "model axisymmetric disk galaxies that are rotationally supported ($V/\sigma_0 \gtrsim 2$) and have a well-defined kinematic axis"
