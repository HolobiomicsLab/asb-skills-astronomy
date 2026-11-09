---
name: equilibrium-photoionization-solver-execution
description: Use when you need equilibrium ionization/temperature solutions for hydrogen
  and helium gas in a spherical geometry with a central point source, and when agreement
  with a Cloudy reference profile (e.g. Cloudy 13.03) must be quantified radius by
  radius.
when_to_use_negative:
- The problem requires metal line cooling — rabacus does not yet include metal line
  cooling (planned future improvement).
- Non-equilibrium (time-dependent) ionization evolution is needed — rabacus provides
  only equilibrium solvers; non-equilibrium solvers are planned future work.
- The geometry is not one of rabacus's supported cases (semi-infinite slab with isotropic
  background, spherical gas with central point source, spherical gas with isotropic
  background).
tools:
- name: rabacus
  role: Equilibrium photoionization/temperature solver for H and He in spherical geometry
    with a central point source; produces the radial ionization and temperature profiles
  repo: https://bitbucket.org/galtay/rabacus
- name: Quantities
  role: Provides units for all rabacus variables; model inputs (radius, densities,
    spectrum) must be passed as Quantities unit-bearing arrays
  repo: python-quantities/python-quantities
- name: Numpy f2py
  role: Wraps the Fortran 90 core routines into Python and detects/compiles with the
    chosen Fortran compiler (f2py -c --help-fcompiler)
- name: Fortran 90 compiler (gfortran/ifort)
  role: Compiles the OpenMP-enabled Fortran 90 core; selected via the F90 environment
    variable (export F90=gfortran) before pip install
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:d165e1a70eff790d35d4f889e10ce6a8f0937eafba0d6a8ae8c8c7f86f1e941d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Fortran 90 compiler (gfortran/ifort)
  - Numpy f2py
  - Quantities
  - rabacus
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# equilibrium-photoionization-solver-execution

## Summary

Executes the rabacus equilibrium photoionization solver (Fortran 90 core wrapped in Python via f2py) to compute coupled H/He ionization fractions and temperatures, and validates the radial profiles against Cloudy 13.03 reference solutions such as the Friedrich et al. (2012) helium sphere test.

## When to use

Use when you need equilibrium ionization/temperature solutions for hydrogen and helium gas in a spherical geometry with a central point source, and when agreement with a Cloudy reference profile (e.g. Cloudy 13.03) must be quantified radius by radius.

## When NOT to use

- The problem requires metal line cooling — rabacus does not yet include metal line cooling (planned future improvement).
- Non-equilibrium (time-dependent) ionization evolution is needed — rabacus provides only equilibrium solvers; non-equilibrium solvers are planned future work.
- The geometry is not one of rabacus's supported cases (semi-infinite slab with isotropic background, spherical gas with central point source, spherical gas with isotropic background).

## Inputs

- Spherical gas model specification: radius (20 kpc), constant densities nH=1e-3 cm^-3 and nHe=8.7e-5 cm^-3, as Quantities unit-bearing arrays
- Central point-source spectrum: 1e5 K blackbody
- Solver configuration: case A recombination rates, evolving (self-consistent) temperature, recombination radiation transported (no on-the-spot approximation)
- Cloudy 13.03 reference radial profiles of xH1, xHe1 and ionization fractions

## Outputs

- Radial profiles of xH1, xH2, xHe1, xHe2, xHe3 and temperature from the rabacus solver output object
- Radius-by-radius comparison table written to cloudy_comparison.csv (relative differences in neutral fractions, absolute differences in ionization fractions)

## How to apply

Install prerequisites (scipy, numpy, quantities), set the F90 environment variable (export F90=gfortran) so f2py compiles the OpenMP-enabled Fortran 90 core, then pip install rabacus and verify by importing it. Build a spherical constant-density model (e.g. radius 20 kpc, nH=1e-3 cm^-3, nHe=8.7e-5 cm^-3, 1e5 K blackbody central spectrum), passing all variables as Quantities unit-bearing arrays since every rabacus variable carries units. Solve the coupled ionization/temperature equilibrium choosing case A recombination rates, an evolving (self-consistent) temperature, and transfer of recombination radiation rather than the on-the-spot approximation. Extract radial profiles of xH1, xH2, xHe1, xHe2, xHe3 and temperature from the solver output object, then compute relative differences in neutral fractions (xH1, xHe1) and absolute differences in ionization fractions against the Cloudy 13.03 profiles and write the radius-by-radius comparison to cloudy_comparison.csv.

## Related tools

- **rabacus** (Equilibrium photoionization/temperature solver for H and He in spherical geometry with a central point source; produces the radial ionization and temperature profiles) — https://bitbucket.org/galtay/rabacus
- **Quantities** (Provides units for all rabacus variables; model inputs (radius, densities, spectrum) must be passed as Quantities unit-bearing arrays) — python-quantities/python-quantities
- **Numpy f2py** (Wraps the Fortran 90 core routines into Python and detects/compiles with the chosen Fortran compiler (f2py -c --help-fcompiler))
- **Fortran 90 compiler (gfortran/ifort)** (Compiles the OpenMP-enabled Fortran 90 core; selected via the F90 environment variable (export F90=gfortran) before pip install)

## Examples

```
export F90=gfortran && sudo pip install scipy numpy quantities && sudo pip install rabacus && python -c "import rabacus as ra"
```

## Evaluation signals

- Relative differences between rabacus and Cloudy 13.03 in hydrogen and helium neutral fractions (xH1, xHe1) are less than 5% at large radii.
- All ionization fractions (xH2, xHe2, xHe3) agree with Cloudy 13.03 within 0.02 absolute difference.
- Import test succeeds: 'import rabacus as ra' runs in an ipython session after 'Successfully installed rabacus'.
- Radius-by-radius comparison table is written to cloudy_comparison.csv with per-radius relative (neutral) and absolute (ionized) differences.
- All model variables (radius, nH, nHe, spectrum) were passed as Quantities unit-bearing arrays, since all variables in rabacus have units.

## Limitations

- Metal line cooling is not yet included in rabacus (planned future improvement).
- Non-equilibrium solvers are not yet included (planned future improvement).
- OpenMP-enabled builds require setting the F90 environment variable; for compilers other than intel and gfortran, the manual setup.py install path with edited f90_flags/omp_lib is needed.
- No version number for the rabacus package is stated in the article metadata.

## Evidence

- [abstract] a Python package for calculating the transfer of hydrogen ionizing radiation in simplified geometries relevant to astronomy and cosmology: [evidence span withheld — anchor status: not_found]
- [intro] we implemented the core functionality of rabacus in Fortran 90 and wrapped the resulting code in Python: [evidence span withheld — anchor status: not_found]
- [readme] sudo pip install scipy
  sudo pip install numpy
  sudo pip install quantities: "sudo pip install scipy
  sudo pip install numpy
  sudo pip install quantities"
