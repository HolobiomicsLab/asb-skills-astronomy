---
name: cross-code-runtime-benchmarking
description: 'Use when validating a new forward-modelling tool against an established
  code: you need a single-realization runtime ratio plus fractional-agreement of model
  outputs (e.g. rotation/dispersion curves) on matched synthetic observations.'
when_to_use_negative:
- You need to benchmark on real observational IFU data rather than synthetic mock
  observations with a controlled, identical PSF and sky-orientation across codes
- The codes cannot be given the same fiducial mass model or produce comparable outputs
  (e.g. one lacks dispersion-curve reconstruction), making fractional-agreement computation
  ill-defined
- The system is not a rotationally supported axisymmetric disk galaxy (V/sigma_0 >~
  2) with a well-defined kinematic axis, which is outside the mass-model scope both
  codes share
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: RotCurves
  role: 'New code under test: constructs the mass model, generates the equilibrium
    rotation velocity, and mock-observes it via beam projection/convolution; timed
    at ~10 ms per realization'
  repo: github.com/nestoramit/RotCurves
- name: dysmalpy
  role: 'Established reference code: repeats the identical mock observation (same
    sky-orientation and PSF) to provide the baseline curves and the ~250x slower runtime
    reference'
- name: Python
  role: Implementation language for the benchmark driver that times realizations and
    computes fractional differences
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stag134
    title: '<tt>RotCurves</tt> : a <tt>PYTHON</tt> package for efficient modelling
      and fitting of galactic rotation curves at high- <i>z</i>'
  source_content_hash: sha256:07dc339a0118626c519e6c108d650d4c4b82f53b9e88e983033e077e2eb82a2c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RotCurves
  - dysmalpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# cross-code-runtime-benchmarking

## Summary

Benchmark a new parametric modelling code (RotCurves) against an established reference code (dysmalpy) by (a) timing single realizations and (b) quantifying agreement of mock-observed rotation and dispersion curves under identical sky-orientation and PSF conditions.

## When to use

Use when validating a new forward-modelling tool against an established code: you need a single-realization runtime ratio plus fractional-agreement of model outputs (e.g. rotation/dispersion curves) on matched synthetic observations.

## When NOT to use

- You need to benchmark on real observational IFU data rather than synthetic mock observations with a controlled, identical PSF and sky-orientation across codes
- The codes cannot be given the same fiducial mass model or produce comparable outputs (e.g. one lacks dispersion-curve reconstruction), making fractional-agreement computation ill-defined
- The system is not a rotationally supported axisymmetric disk galaxy (V/sigma_0 >~ 2) with a well-defined kinematic axis, which is outside the mass-model scope both codes share

## Inputs

- Fiducial galaxy mass model definition (disk, bulge, dark matter halo components with analytical density profiles) shared by both codes
- Sky-orientation (inclination, position angle) and instrumental PSF specification for the mock observations
- Two resolution configurations: well-resolved (PSF FWHM < R_eff) and marginally resolved (PSF FWHM >= 1.5 R_eff)

## Outputs

- benchmark_results.csv containing single-realization runtimes and runtime ratio for both codes
- Fractional-difference curves (RotCurves vs dysmalpy mock-observed rotation and dispersion curves) as a function of radius up to 3 R_eff for both resolution cases

## How to apply

Construct the same fiducial galaxy mass model (disk, bulge, dark matter halo) in both codes and generate the intrinsic equilibrium rotation velocity out to at least 3 R_eff. Mock-observe the fiducial galaxy in each code with identical sky-orientation and instrumental PSF, projecting and convolving the beam PSF in the plane of the galaxy to reconstruct beam-smeared rotation and dispersion curves. Run the comparison at two resolution regimes: well-resolved (PSF FWHM < R_eff) and marginally resolved (PSF FWHM >= 1.5 R_eff). Time a single realization of each code and record the runtime ratio (~250x, RotCurves at ~10 ms per realization). Compute the fractional difference between the codes' curves as a function of radius out to 3 R_eff for both cases, and write runtimes and fractional-agreement values to benchmark_results.csv.

## Related tools

- **RotCurves** (New code under test: constructs the mass model, generates the equilibrium rotation velocity, and mock-observes it via beam projection/convolution; timed at ~10 ms per realization) — github.com/nestoramit/RotCurves
- **dysmalpy** (Established reference code: repeats the identical mock observation (same sky-orientation and PSF) to provide the baseline curves and the ~250x slower runtime reference)
- **Python** (Implementation language for the benchmark driver that times realizations and computes fractional differences)

## Examples

```
pip install rotcurves
```

## Evaluation signals

- Runtime ratio RotCurves/dysmalpy for a single realization is approximately 250 in favor of RotCurves, with RotCurves at ~10 ms per realization
- Well-resolved case (PSF FWHM < R_eff): fractional difference in mock-observed rotation and dispersion curves within 5 per cent up to 3 R_eff
- Marginally resolved case (PSF FWHM >= 1.5 R_eff): discrepancies grow to at most about 15 per cent, larger than but consistent with the well-resolved case
- benchmark_results.csv exists and contains runtimes and fractional-agreement values for both resolution configurations
- Both codes were run with identical sky-orientation and PSF so curve differences isolate code effects, not setup differences

## Limitations

- Agreement degrades in marginally resolved systems (PSF FWHM >= 1.5 R_eff), with discrepancies up to ~15 per cent out to 3 R_eff
- The runtime ratio (~250x, ~10 ms per realization) is hardware- and configuration-dependent and should be re-measured on the benchmark machine
- The benchmark covers a fiducial axisymmetric disk+bulge+halo model; results may not transfer to mass models with strong non-circular motions or unusual profile choices

## Evidence

- [abstract] Runtime ratio and RotCurves speed: "The typical runtime with RotCurves is a few $\sim 10$ ms, a factor $\approx 250$ faster than dysmalpy for a single realization."
- [abstract] Curve agreement thresholds by resolution: "For well-resolved systems (PSF FWHM $< R_{\rm eff}$), the mock observed rotation and dispersion curves agree to within 5 per cent up to $3R_{\rm eff}$"
- [intro] Mass model definition shared across codes: "The code constructs a mass model of an idealized, axisymmetric galaxy with mass components following defined analytical density distributions"
- [intro] Mock-observation procedure (beam projection): "We simulate the projected beam on the plane of the galaxy (i.e., "beam projection"), reconstructing the beam-smeared rotation curve"
- [readme] PSF-conditioned applicability for benchmark cases: "The fitting procedure peforms best for resolved systems, with a beam size that is of the same size or smaller than the effective radius of the disk"
