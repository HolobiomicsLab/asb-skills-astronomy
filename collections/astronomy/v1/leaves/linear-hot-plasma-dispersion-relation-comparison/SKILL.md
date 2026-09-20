---
name: linear-hot-plasma-dispersion-relation-comparison
description: 'Use when validating a plasma-wave simulation code: 1D low-β wave-test
  runs are available and numerical dispersion curves must be checked against linearised
  hot-plasma theory (WHAMP) per propagation angle.'
when_to_use_negative:
- The simulation regime is non-linear or strongly kinetic (e.g. growing instabilities
  past the linear stage) — linear WHAMP theory no longer describes the dynamics
- Plasma conditions are not low-β wave-test setups, so the linearised hot-plasma reference
  solutions do not apply
- Only single-fluid MHD solvers are being validated, whose wave physics differs from
  the hybrid-Vlasov description
tools:
- name: Vlasiator
  role: Hybrid-Vlasov solver used to produce the 1D low-β wave-test simulations whose
    electromagnetic field fluctuations are decomposed into numerical dispersion curves
  repo: https://github.com/fmihpc/vlasiator
- name: WHAMP
  role: Independently computes the reference linear hot-plasma dispersion solutions
    (Rönnmark 1982) for the same plasma parameters and propagation angles
- name: analysator
  role: Python tools to read and analyze Vlasiator output files (VLSV), used to extract
    field fluctuations for spectral decomposition
  repo: https://github.com/fmihpc/analysator
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:7ad486cacbea381f52fdcca4327d18f3cfe4d871348677f753aec9fce891101c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/vlasiator
  tools:
  - Vlasiator
  - WHAMP
  - analysator
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# linear-hot-plasma-dispersion-relation-comparison

## Summary

Verifies a Vlasov/hybrid-Vlasov plasma solver by comparing numerically extracted ω(k) dispersion curves from simulated electromagnetic field fluctuations against independently computed linear hot-plasma dispersion solutions (WHAMP, Rönnmark 1982) for parallel, perpendicular, and oblique propagation angles.

## When to use

Use when validating a plasma-wave simulation code: 1D low-β wave-test runs are available and numerical dispersion curves must be checked against linearised hot-plasma theory (WHAMP) per propagation angle.

## When NOT to use

- The simulation regime is non-linear or strongly kinetic (e.g. growing instabilities past the linear stage) — linear WHAMP theory no longer describes the dynamics
- Plasma conditions are not low-β wave-test setups, so the linearised hot-plasma reference solutions do not apply
- Only single-fluid MHD solvers are being validated, whose wave physics differs from the hybrid-Vlasov description

## Inputs

- 1D low-β Vlasiator wave-test simulation runs (parallel, perpendicular, oblique propagation geometries)
- Simulated electromagnetic field fluctuation time series from Vlasiator output files (VLSV)
- Plasma parameters (density, background magnetic field, ion/electron temperatures) matching between simulation and theory
- WHAMP (Rönnmark 1982) reference linear hot-plasma dispersion solutions for the same parameters and angles

## Outputs

- dispersion_curves.csv containing ω vs k per propagation angle
- Maximum relative frequency error score between simulated and WHAMP curves
- Comparison of simulated vs linear-theory dispersion curves for parallel, perpendicular, and oblique propagation

## How to apply

Configure and launch 1D low-β wave-test runs of the Vlasiator solver for three propagation geometries (parallel, perpendicular, oblique). Extract the simulated electromagnetic field fluctuations and apply a spatiotemporal Fourier/spectral decomposition to obtain numerical ω(k) dispersion curves per angle. Independently compute the reference linear hot-plasma dispersion solutions with WHAMP (Rönnmark 1982) for the same plasma parameters and angles. Compare simulated and theoretical curves and compute the maximum relative frequency error, targeting the excellent agreement reported by Kempf et al. 2013; the only noticeable difference should appear at high frequencies and wave numbers. Write the ω-vs-k data per angle to dispersion_curves.csv and record the error score in a structured output.

## Related tools

- **Vlasiator** (Hybrid-Vlasov solver used to produce the 1D low-β wave-test simulations whose electromagnetic field fluctuations are decomposed into numerical dispersion curves) — https://github.com/fmihpc/vlasiator
- **WHAMP** (Independently computes the reference linear hot-plasma dispersion solutions (Rönnmark 1982) for the same plasma parameters and propagation angles)
- **analysator** (Python tools to read and analyze Vlasiator output files (VLSV), used to extract field fluctuations for spectral decomposition) — https://github.com/fmihpc/analysator

## Examples

```
export PYTHONPATH=[path-to-analysator]; python -c "import analysator as pt; f=pt.vlsvfile.VlsvReader('bulk.0000000.vlsv')"
```

## Evaluation signals

- Simulated dispersion curves agree with WHAMP analytical solutions for parallel, perpendicular, and oblique propagation, with only noticeable differences at high frequencies and wave numbers
- Maximum relative frequency error between simulated and linear-theory curves is small, consistent with the agreement reported in Kempf et al. 2013
- dispersion_curves.csv contains ω vs k pairs for each of the three propagation angles
- Spectral decomposition recovers clean, noiseless ω(k) ridges, consistent with Vlasiator's distribution-function-based, numerically noiseless ion representation
- Error score recorded in a structured output matches the per-angle curve comparison

## Limitations

- Agreement degrades at high frequencies and high wave numbers, where the only noticeable difference between simulation and WHAMP solutions appears
- Verification is limited to 1D low-β wave-test configurations; global magnetospheric runs (Earth's foreshock, magnetosheath) are outside this linear-theory check
- Reference solutions depend on matching plasma parameters exactly between the WHAMP input and the simulation setup

## Evidence

- [abstract] Vlasov-system numerical solvers and verification: "A short review of verification, validation and physical results is included"
- [abstract] Vlasiator as the simulation engine: "Vlasiator, the world’s first global hybrid-Vlasov simulation for the Earth’s magnetic domain, the magnetosphere."
- [readme] Noiseless ion representation enables clean spectra: [evidence span withheld — anchor status: external_doc]
- [readme] Reading simulation outputs for spectral analysis: [evidence span withheld — anchor status: not_found]
- [readme] Self-consistent plasma model being verified: [evidence span withheld — anchor status: external_doc]
