---
name: radial-profile-relative-difference-analysis
description: Use when a solved rabacus sphere/slab profile must be benchmarked against
  a reference ionization or temperature profile (e.g. Cloudy 13.03 Test 1) on a shared
  radial grid.
when_to_use_negative:
- The problem requires non-equilibrium solvers or metal line cooling, which rabacus
  explicitly does not yet include (planned future improvements).
- The goal is a self-consistent equilibrium temperature calculation rather than comparison
  against a fixed-temperature reference profile.
- Geometry is not one of the three supported cases (slab in isotropic background,
  sphere with point source, sphere in isotropic background).
tools:
- name: rabacus
  role: Solves the equilibrium ionization and recombination-radiation transfer problem
    to produce the radial xHII profile being compared
  repo: https://bitbucket.org/galtay/rabacus
- name: Quantities
  role: Provides units for all problem variables (radius, density, photon luminosity)
    passed into the sphere problem
  repo: https://www.github.com/python-quantities/python-quantities
- name: Cloudy 13.03
  role: Supplies the reference ionization profile that the rabacus solution is benchmarked
    against
- name: Numpy f2py
  role: Wraps the Fortran 90 core routines in Python so the solver can be invoked
    from a Python session
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:3441af451ad73661d9bc0eae9cc375c34d00d5b2bab9900a5b1414b7a6d33910
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cloudy 13.03
  - Numpy f2py
  - Quantities
  - rabacus
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# radial-profile-relative-difference-analysis

## Summary

Compares a computed radial ionization profile (e.g. rabacus xHII for a Strömgren sphere) against a reference code profile (e.g. Cloudy) via point-by-point relative differences, validating agreement at the sub-percent level away from the ionization front.

## When to use

Use when a solved rabacus sphere/slab profile must be benchmarked against a reference ionization or temperature profile (e.g. Cloudy 13.03 Test 1) on a shared radial grid.

## When NOT to use

- The problem requires non-equilibrium solvers or metal line cooling, which rabacus explicitly does not yet include (planned future improvements).
- The goal is a self-consistent equilibrium temperature calculation rather than comparison against a fixed-temperature reference profile.
- Geometry is not one of the three supported cases (slab in isotropic background, sphere with point source, sphere in isotropic background).

## Inputs

- Rabacus Sphere_Stromgren problem configuration (15 eV monochromatic point source, case-B recombination, fixed temperature, radius 6.6 kpc, nH 1e-3 cm^-3, Ndot 5.0e48 s^-1)
- Reference ionization profile (e.g. Cloudy 13.03 output) for the identical test configuration
- Unit-specified parameters via the Quantities Python package

## Outputs

- sphere_stromgren_test1_comparison.csv containing rabacus xHII profile, interpolated Cloudy profile, and point-by-point relative differences
- Radial xHII profile from the rabacus Fortran 90 solver

## How to apply

Build the test problem in rabacus — for Test 1 use a monochromatic 15 eV point source, case-B recombination radiation treatment, fixed temperature, spherical radius 6.6 kpc, hydrogen number density 1e-3 cm^-3, and source photon luminosity 5.0e48 s^-1, with units supplied through the Quantities package. Solve the equilibrium ionization and recombination-radiation transfer problem with the Fortran 90 core routines (wrapped via Numpy f2py) to obtain the radial xHII profile. Load the reference (Cloudy 13.03) profile for the same configuration and interpolate it onto the rabacus radial grid. Compute the point-by-point relative difference between the two hydrogen ionization fractions and write both profiles plus the differences to a CSV (e.g. sphere_stromgren_test1_comparison.csv). Accept the comparison when the typical relative difference is below one percent, tolerating only a few percent near the ionization front.

## Related tools

- **rabacus** (Solves the equilibrium ionization and recombination-radiation transfer problem to produce the radial xHII profile being compared) — https://bitbucket.org/galtay/rabacus
- **Quantities** (Provides units for all problem variables (radius, density, photon luminosity) passed into the sphere problem) — https://www.github.com/python-quantities/python-quantities
- **Cloudy 13.03** (Supplies the reference ionization profile that the rabacus solution is benchmarked against)
- **Numpy f2py** (Wraps the Fortran 90 core routines in Python so the solver can be invoked from a Python session)

## Examples

```
export F90=gfortran; sudo pip install rabacus; python -c "import rabacus as ra"
```

## Evaluation signals

- Typical relative difference between rabacus and Cloudy 13.03 hydrogen ionization fractions is below one percent
- Relative difference rises to only a few percent in the region near the ionization front
- The installation was verified by importing rabacus (e.g. 'import rabacus as ra') without errors before solving
- Output CSV contains three aligned series on the same radial grid: rabacus profile, interpolated reference profile, and relative differences

## Limitations

- Metal line cooling is not yet included in rabacus (planned future improvement).
- Non-equilibrium solvers are not yet included (planned future improvement).
- Discrepancies of a few percent are expected near the ionization front, so the threshold is not uniform across the radial grid.
- A Fortran 90 compiler (gfortran or ifort) must be in the executable path and the F90 environment variable set before installation.

## Evidence
