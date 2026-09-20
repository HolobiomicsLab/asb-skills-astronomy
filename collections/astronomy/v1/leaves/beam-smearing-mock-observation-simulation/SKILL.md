---
name: beam-smearing-mock-observation-simulation
description: Use when you need synthetic beam-smeared rotation curves from a galaxy
  mass model to test MCMC parameter recovery, benchmark against codes like dysmalpy,
  or create realistic mock data for high-z IFU-style observations.
when_to_use_negative:
- The galaxy is not axisymmetric and rotationally supported (V/sigma_0 <~ 2), lacks
  a well-defined kinematic axis, or shows strong non-circular motions
- The system is unresolved in the sense that the beam size substantially exceeds the
  disc effective radius, or inclination falls outside 10 deg < i < 80 deg
- You need full 3D forward-modelling fidelity for marginally resolved systems where
  mock curves may disagree with dedicated codes like dysmalpy by up to ~15 per cent
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: RotCurves
  role: Forward-modelling package that constructs the mass model, applies beam projection/PSF
    convolution, and generates the mock beam-smeared rotation curve (~10 ms per realization)
  repo: github.com/nestoramit/RotCurves
- name: emcee
  role: MCMC sampler used with the generated mock curves in the built-in Bayesian
    fitting procedure to recover intrinsic parameters and assess biases
- name: dysmalpy
  role: Established parametric code used to benchmark the mock rotation and dispersion
    curves produced by this simulation step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/mnras/stag134
    title: '<tt>RotCurves</tt> : a <tt>PYTHON</tt> package for efficient modelling
      and fitting of galactic rotation curves at high- <i>z</i>'
  source_content_hash: sha256:693940fe3e8cadce25c7021f9b32dfd05b0fad418dfd5026cb38d0945d0bbc87
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RotCurves
  - dysmalpy
  - emcee
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# beam-smearing-mock-observation-simulation

## Summary

Generates mock, beam-smeared 1D rotation curves from an axisymmetric disc+halo galaxy mass model using RotCurves, by projecting the observational beam PSF onto the galaxy plane ('beam projection'). This simulates realistic instrumental degradation so that forward-model fits can be compared directly with observed rotation-curve data and parameter-recovery biases can be quantified.

## When to use

Use when you need synthetic beam-smeared rotation curves from a galaxy mass model to test MCMC parameter recovery, benchmark against codes like dysmalpy, or create realistic mock data for high-z IFU-style observations.

## When NOT to use

- The galaxy is not axisymmetric and rotationally supported (V/sigma_0 <~ 2), lacks a well-defined kinematic axis, or shows strong non-circular motions
- The system is unresolved in the sense that the beam size substantially exceeds the disc effective radius, or inclination falls outside 10 deg < i < 80 deg
- You need full 3D forward-modelling fidelity for marginally resolved systems where mock curves may disagree with dedicated codes like dysmalpy by up to ~15 per cent

## Inputs

- Axisymmetric galaxy mass model specification (disc, bulge, dark-matter halo components with selectable analytical density profiles) via RotCurves.galaxy_model / RotCurves.mass_model
- Sky-orientation parameters (inclination, position angle) for the mock observation
- Instrumental point-spread-function (PSF) / beam definition for beam-smearing projection

## Outputs

- Mock beam-smeared 1D rotation curve (and dispersion curve) for the input mass model
- Per-realization intrinsic vs. recovered parameter comparisons (e.g. fractional bias of R_eff and M_disc) suitable for aggregation into tables such as bias_table.csv

## How to apply

First build the mass model with RotCurves.galaxy_model and RotCurves.mass_model, selecting analytical density profiles for the disc, bulge, and dark-matter halo components of an axisymmetric, rotationally supported galaxy (V/sigma_0 >~ 2). Then call RotCurves.rotation_curve with the sky orientation (inclination, position angle) and the instrumental PSF: the code computes the equilibrium rotation velocity and mock-observes it by projecting and convolving the beam PSF in the plane of the galaxy, reconstructing the beam-smeared rotation curve. Choose the PSF/beam size and galaxy effective radius so the system is at least marginally resolved; the fitting performs best when PSF <~ R_eff,disk. Each realization takes ~10 ms, a factor ~250 faster than dysmalpy, so large parameter grids (e.g. low-mass M_disc <~ 3e9 Msun vs. higher-mass control sets) can be swept efficiently to measure fractional biases in recovered parameters such as R_eff and M_disc via the built-in emcee MCMC fitter.

## Related tools

- **RotCurves** (Forward-modelling package that constructs the mass model, applies beam projection/PSF convolution, and generates the mock beam-smeared rotation curve (~10 ms per realization)) — github.com/nestoramit/RotCurves
- **emcee** (MCMC sampler used with the generated mock curves in the built-in Bayesian fitting procedure to recover intrinsic parameters and assess biases)
- **dysmalpy** (Established parametric code used to benchmark the mock rotation and dispersion curves produced by this simulation step)

## Examples

```
pip install rotcurves
```

## Evaluation signals

- For well-resolved systems (PSF FWHM < R_eff), the mock rotation and dispersion curves should agree with dysmalpy within 5 per cent up to 3 R_eff; discrepancies up to 15 per cent are acceptable only for marginally resolved systems
- Typical runtime per realization should be of order ~10 ms; a factor ~250 slower output suggests the fast RotCurves pipeline is not being used as intended
- Parameter recovery from the mock curves should be excellent at S/N >~ 25, with deviations only increasing at lower S/N
- Validation from the source task: the low-mass set (M_disc <= ~3e9 Msun) should reproduce the paper's reported systematic biases in R_eff and M_disc recovery, while the higher-mass control set recovers parameters without bias

## Limitations

- Systematic biases emerge in recovered effective radius and for low disc masses (M_disc <~ 3 x 10^9 Msun), so parameter recovery is not unbiased in all regimes
- Best suited for resolved systems with beam size same size or smaller than the disc effective radius (PSF <~ R_eff,disk)
- Restricted to inclinations of 10 deg < i < 80 deg and marginally resolved systems can show up to 15 per cent curve discrepancies

## Evidence

- [intro] The equilibrium rotation velocity is mock-observed given the sky-orientation and the instrumental point-spread-function (PSF).: "The equilibrium rotation velocity is mock-observed given the sky-orientation and the instrumental point-spread-function (PSF)."
- [readme] README forward-modelling scope: "The equilibrium rotation velocity is mock-observed given the sky-orientation and the instrumental point-spread-function (PSF)."
- [abstract] Beam-smearing correction description: "correcting for ‘beam smearing’ by projecting and convolving the beam PSF in the plane of the galaxy"
- [abstract] Benchmark agreement for well-resolved systems: "For well-resolved systems (PSF FWHM $< R_{\rm eff}$), the mock observed rotation and dispersion curves agree to within 5 per cent up to $3R_{\rm eff}$"
- [abstract] Runtime advantage over dysmalpy: "The typical runtime with RotCurves is a few $\sim 10$ ms, a factor $\approx 250$ faster than dysmalpy for a single realization."
