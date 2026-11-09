---
name: analytic-inverse-solution-derivation
description: Use when validating a numerical equilibrium radiative-transfer solver
  (e.g. rabacus Slab_Pln) against an analytic benchmark, or when a closed-form ionization-fraction-vs-column-depth
  profile is needed for a uniform pure-H slab under monochromatic plane-parallel illumination.
when_to_use_negative:
- The problem requires non-equilibrium chemistry or metal line cooling, which rabacus
  explicitly does not yet provide (planned future improvements).
- The radiation field is polychromatic or the gas includes helium, so the monochromatic
  pure-hydrogen inversion of Eqs. 23-25 no longer applies.
- The geometry is spherical (central point source or isotropic background) rather
  than a plane-parallel slab, where no planar column-depth inversion exists.
edam_operation: http://edamontology.org/operation_0438
tools:
- name: rabacus
  role: Provides the Slab_Pln numerical solver whose hydrogen ionization-fraction
    profile is validated against the derived closed-form inverse solution
  repo: https://bitbucket.org/galtay/rabacus
- name: Quantities
  role: Supplies the unit-carrying variables required for all slab and radiation-field
    inputs to Slab_Pln
  repo: https://www.github.com/python-quantities/python-quantities
- name: Python
  role: Host language in which Eqs. 23-25 are implemented and the column-depth-grid
    comparison is scripted
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:d241737d1c3b2ebca701f7a7ba12201401a307c3e971727479e3eff0297b8ade
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_0438
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# analytic-inverse-solution-derivation

## Summary

Derives the algebraic inverse of the hydrogen ionization balance (Eqs. 23-25) for a monochromatic, plane-parallel, uniform pure-hydrogen slab, and uses that closed-form solution as the benchmark for the rabacus Slab_Pln numerical solver. Agreement is quantified by pointwise relative difference on a common column-depth grid and must stay below 7e-3.

## When to use

Use when validating a numerical equilibrium radiative-transfer solver (e.g. rabacus Slab_Pln) against an analytic benchmark, or when a closed-form ionization-fraction-vs-column-depth profile is needed for a uniform pure-H slab under monochromatic plane-parallel illumination.

## When NOT to use

- The problem requires non-equilibrium chemistry or metal line cooling, which rabacus explicitly does not yet provide (planned future improvements).
- The radiation field is polychromatic or the gas includes helium, so the monochromatic pure-hydrogen inversion of Eqs. 23-25 no longer applies.
- The geometry is spherical (central point source or isotropic background) rather than a plane-parallel slab, where no planar column-depth inversion exists.

## Inputs

- Uniform pure-hydrogen slab parameters (density, thickness) as unit-carrying Quantities variables
- Monochromatic plane-parallel ionizing radiation source spectrum
- Common column-depth grid spanning the ionization front
- Implementation of the analytic inverse closed-form solution (Eqs. 23-25) in Python

## Outputs

- solution_comparison.csv: depth-resolved table of analytic vs Slab_Pln hydrogen ionization-fraction profiles on the shared column-depth grid
- Maximum pointwise relative difference near the ionization front (target < 7e-3)

## How to apply

Build the environment first: pip install scipy, numpy and quantities, set the F90 environment variable (e.g. export F90=gfortran), install an OpenMP-enabled rabacus, and confirm with a quick 'import rabacus as ra'. Set up the Slab_Pln problem for a uniform pure-hydrogen slab illuminated by monochromatic plane-parallel radiation, passing unit-carrying Quantities variables throughout. Implement the analytic inverse closed-form solution (Eqs. 23-25), which algebraically inverts the hydrogen ionization balance to give the ionization state as a function of column depth through the slab. Evaluate both the inverse analytic solution and the Slab_Pln numerical solution on one common column-depth grid spanning the ionization front, then compute the pointwise relative difference between the hydrogen ionization-fraction profiles. Record the maximum near the front — it must be below 7e-3, with differences two orders of magnitude smaller over most of the slab — and write the depth-resolved comparison table plus the maximum to solution_comparison.csv.

## Related tools

- **rabacus** (Provides the Slab_Pln numerical solver whose hydrogen ionization-fraction profile is validated against the derived closed-form inverse solution) — https://bitbucket.org/galtay/rabacus
- **Quantities** (Supplies the unit-carrying variables required for all slab and radiation-field inputs to Slab_Pln) — https://www.github.com/python-quantities/python-quantities
- **Python** (Host language in which Eqs. 23-25 are implemented and the column-depth-grid comparison is scripted)

## Examples

```
import rabacus as ra; import quantities as pq
```

## Evaluation signals

- Quick-test 'import rabacus as ra' runs without error in a fresh ipython session.
- Maximum relative difference between the analytic and Slab_Pln hydrogen ionization-fraction profiles near the ionization front is below 7e-3.
- Over most of the slab away from the front, the relative difference is roughly two orders of magnitude smaller than the near-front maximum.
- solution_comparison.csv exists and contains depth-resolved rows for both profiles evaluated on the same column-depth grid.
- All inputs to Slab_Pln carry Quantities units rather than bare floats.

## Limitations

- The closed-form inversion holds only for the monochromatic, equilibrium, pure-hydrogen plane-parallel case; spectral shape of the ionizing source, helium, and transferred recombination radiation fall outside it.
- The largest discrepancy (~7e-3) occurs near the ionization front, so tighter agreement should not be expected at the front.
- The build requires a Fortran 90 compiler with OpenMP and the F90 environment variable set (e.g. gfortran); otherwise a manual setup.py build with a detected compiler (via 'f2py -c --help-fcompiler') is needed.
- The Quantities package README notes test coverage is incomplete, so unit handling is 'not suggested for mission-critical applications'.

## Evidence

- [abstract] abstract: [evidence span withheld — anchor status: not_found]
- [intro] intro: "all variables in rabacus have units through use of the Quantities Python package"
- [results] results: [evidence span withheld — anchor status: not_found]
- [readme] readme: "begin an ipython session and attempt an import"
- [discussion] discussion: "the addition of metal line cooling and non-equilibrium solvers"
