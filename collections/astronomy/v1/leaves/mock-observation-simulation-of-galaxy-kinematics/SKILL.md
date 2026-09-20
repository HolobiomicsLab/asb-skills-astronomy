---
name: mock-observation-simulation-of-galaxy-kinematics
description: Use when you have an intrinsic 1D rotation curve (disk+bulge+halo mass
  model) and need beam-smeared mock-observed rotation/dispersion curves for a given
  sky-orientation and PSF, e.g. for benchmarking or forward-model fitting.
when_to_use_negative:
- The galaxy is not rotationally supported (V/sigma_0 <~ 2) or lacks a well-defined
  kinematic axis / has strong deviations from circular motions — the axisymmetric
  forward model assumes none
- 'The system is poorly resolved (beam size much larger than R_eff): the fitting procedure
  performs best for PSF <~ R_eff, and mock-observed curves can deviate up to ~15%
  even in marginally resolved cases'
- Inclination is outside 10° < i < 80°, where RotCurves is best suited
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: RotCurves
  role: Parametric forward-modelling tool that constructs the mass model, mock-observes
    the equilibrium rotation velocity given sky-orientation and PSF, and reconstructs
    beam-smeared rotation/dispersion curves (~10 ms per realization)
  repo: github.com/nestoramit/RotCurves
- name: dysmalpy
  role: Established parametric code used to repeat the identical mock observation
    for benchmark validation of curve agreement and runtime comparison
- name: Python
  role: Implementation language of the RotCurves package used for the mock-observation
    simulation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stag134
    title: '<tt>RotCurves</tt> : a <tt>PYTHON</tt> package for efficient modelling
      and fitting of galactic rotation curves at high- <i>z</i>'
  source_content_hash: sha256:c2e92e2ad987c602bf15a85c46ac95a698bae51932c44416b49095f5fae9ab77
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RotCurves
  - dysmalpy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mock-observation-simulation-of-galaxy-kinematics

## Summary

Simulates the instrumental observation of an idealized axisymmetric galaxy model by projecting and convolving the beam PSF in the plane of the galaxy, reconstructing beam-smeared rotation and dispersion curves from the intrinsic equilibrium rotation velocity.

## When to use

Use when you have an intrinsic 1D rotation curve (disk+bulge+halo mass model) and need beam-smeared mock-observed rotation/dispersion curves for a given sky-orientation and PSF, e.g. for benchmarking or forward-model fitting.

## When NOT to use

- The galaxy is not rotationally supported (V/sigma_0 <~ 2) or lacks a well-defined kinematic axis / has strong deviations from circular motions — the axisymmetric forward model assumes none
- The system is poorly resolved (beam size much larger than R_eff): the fitting procedure performs best for PSF <~ R_eff, and mock-observed curves can deviate up to ~15% even in marginally resolved cases
- Inclination is outside 10° < i < 80°, where RotCurves is best suited

## Inputs

- Axisymmetric galaxy mass model (disk, bulge, dark matter halo components with analytical density profiles)
- Intrinsic equilibrium rotation velocity out to at least 3 R_eff
- Sky-orientation parameters (inclination, position angle)
- Instrumental point-spread-function (PSF) / beam specification
- Effective radius R_eff defining the resolution regime

## Outputs

- Beam-smeared mock-observed rotation curve
- Beam-smeared mock-observed dispersion curve
- Fractional difference curves vs radius up to 3 R_eff (when benchmarking against dysmalpy)
- benchmark_results.csv with runtimes and fractional-agreement values

## How to apply

Construct the axisymmetric galaxy mass model (disk, bulge, dark matter halo) with selectable analytical density profiles in RotCurves and compute the intrinsic equilibrium rotation velocity out to at least 3 R_eff. Mock-observe it by projecting and convolving the beam PSF in the plane of the galaxy given a sky-orientation and instrumental PSF, reconstructing the beam-smeared rotation and dispersion curves. Choose the resolution configuration deliberately: well-resolved means PSF FWHM < R_eff, marginally resolved means PSF FWHM >= 1.5 R_eff, since agreement with full 3D codes like dysmalpy degrades from ~5% to ~15% between these regimes. To validate the mock-observation machinery, repeat the identical mock observation with dysmalpy for the same sky-orientation and PSF, compute fractional differences as a function of radius up to 3 R_eff, and record runtimes (RotCurves ~10 ms, factor ~250 faster than dysmalpy per single realization).

## Related tools

- **RotCurves** (Parametric forward-modelling tool that constructs the mass model, mock-observes the equilibrium rotation velocity given sky-orientation and PSF, and reconstructs beam-smeared rotation/dispersion curves (~10 ms per realization)) — github.com/nestoramit/RotCurves
- **dysmalpy** (Established parametric code used to repeat the identical mock observation for benchmark validation of curve agreement and runtime comparison)
- **Python** (Implementation language of the RotCurves package used for the mock-observation simulation)

## Examples

```
pip install rotcurves
```

## Evaluation signals

- Well-resolved case (PSF FWHM < R_eff): fractional difference between RotCurves and dysmalpy mock-observed rotation and dispersion curves within 5% up to 3 R_eff
- Marginally resolved case (PSF FWHM >= 1.5 R_eff): expected discrepancies up to ~15%
- Single-realization runtime ratio ~250 in favor of RotCurves, with RotCurves at ~10 ms per realization
- Curves computed out to at least 3 R_eff for both resolution configurations
- Runtimes and fractional-agreement values for both cases written to benchmark_results.csv

## Limitations

- Agreement with dysmalpy degrades to discrepancies up to ~15% for marginally resolved systems (PSF FWHM >= 1.5 R_eff)
- Fitting/mock-observation procedure performs best when the beam size is the same size or smaller than the disk effective radius (PSF <~ R_eff)
- Model limited to axisymmetric, rotationally supported disks (V/sigma_0 >~ 2) with a well-defined kinematic axis and no strong deviations from circular motions
- Best suited for inclinations between 10 and 80 degrees

## Evidence

- [abstract] Mock-observation with PSF convolution: "correcting for ‘beam smearing’ by projecting and convolving the beam PSF in the plane of the galaxy"
- [abstract] Well-resolved agreement threshold: "For well-resolved systems (PSF FWHM $< R_{\rm eff}$), the mock observed rotation and dispersion curves agree to within 5 per cent up to $3R_{\rm eff}$"
- [abstract] Runtime comparison: "The typical runtime with RotCurves is a few $\sim 10$ ms, a factor $\approx 250$ faster than dysmalpy for a single realization."
- [intro] Forward-modelling of beam projection: "We simulate the projected beam on the plane of the galaxy (i.e., "beam projection"), reconstructing the beam-smeared rotation curve"
- [readme] Applicability to resolved systems: "The fitting procedure peforms best for resolved systems, with a beam size that is of the same size or smaller than the effective radius of the disk $PSF \lesssim R_{\rm eff,disk}$."
