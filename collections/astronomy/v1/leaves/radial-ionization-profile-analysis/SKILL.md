---
name: radial-ionization-profile-analysis
description: Use when validating an equilibrium radiative-transfer solver against
  Cloudy by comparing radial profiles of xH1, xH2, xHe1, xHe2, xHe3 and temperature
  for a constant-density sphere with a central source.
when_to_use_negative:
- The problem is a non-equilibrium (time-dependent) ionization or cooling calculation
  — rabacus only provides equilibrium solvers
- Metal line cooling matters for the temperature solution — rabacus does not yet include
  metal line cooling
- The geometry is not one of rabacus's supported cases (semi-infinite slab in isotropic
  background, sphere with central point source, or sphere in isotropic background)
tools:
- name: rabacus
  role: Solves the coupled equilibrium ionization/temperature problem for a spherically
    symmetric gas distribution with a point source at the center, including transfer
    of recombination radiation
  repo: https://bitbucket.org/galtay/rabacus
- name: Quantities
  role: Supplies units for all model variables; all values passed to rabacus must
    be Quantities unit-bearing arrays
  repo: https://www.github.com/python-quantities/python-quantities
- name: Numpy f2py
  role: Wraps the Fortran 90 core routines in Python during the rabacus build
  repo: https://bitbucket.org/galtay/rabacus
- name: Fortran 90
  role: Language of the compiled core routines; requires a compiler (e.g. gfortran)
    in the executable path with F90 set
  repo: https://bitbucket.org/galtay/rabacus
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:5db0d7dd460fc4bd16e1f2f1fde1f7d239c1ad20b22423abd3961b973a46286d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Fortran 90
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
n_claims: 11
---

# radial-ionization-profile-analysis

## Summary

Reproduce and validate equilibrium ionization and temperature radial profiles of spherical H/He gas clouds irradiated by a central point source using rabacus, then compare them radius-by-radius against Cloudy reference profiles (e.g. the Friedrich et al. 2012 20 kpc helium sphere test).

## When to use

Use when validating an equilibrium radiative-transfer solver against Cloudy by comparing radial profiles of xH1, xH2, xHe1, xHe2, xHe3 and temperature for a constant-density sphere with a central source.

## When NOT to use

- The problem is a non-equilibrium (time-dependent) ionization or cooling calculation — rabacus only provides equilibrium solvers
- Metal line cooling matters for the temperature solution — rabacus does not yet include metal line cooling
- The geometry is not one of rabacus's supported cases (semi-infinite slab in isotropic background, sphere with central point source, or sphere in isotropic background)

## Inputs

- Spherical constant-density gas model specification (radius 20 kpc, nH=1e-3 cm^-3, nHe=8.7e-5 cm^-3) as Quantities unit-bearing arrays
- Ionizing source spectrum (1e5 K blackbody, central point source)
- Cloudy 13.03 reference radial profiles of ionization fractions and temperature
-  rabacus installed with an OpenMP-enabled Fortran 90 build (F90=gfortran)

## Outputs

- Radial profiles of xH1, xH2, xHe1, xHe2, xHe3 and temperature from the rabacus solver output object
- cloudy_comparison.csv with radius-by-radius relative differences in neutral fractions and absolute differences in ionization fractions
- Pass/fail judgment against the 5% relative and 0.02 absolute validation thresholds

## How to apply

Install rabacus with an OpenMP-enabled Fortran 90 build (set the F90 environment variable, e.g. export F90=gfortran, then pip install rabacus after scipy, numpy and quantities prerequisites are present), and verify the install by importing rabacus. Configure a spherical constant-density model with a central point source (e.g. radius 20 kpc, nH=1e-3 cm^-3, nHe=8.7e-5 cm^-3, a 1e5 K blackbody spectrum), passing all values as Quantities unit-bearing arrays since all rabacus variables carry units. Solve the coupled ionization/temperature equilibrium with case A recombination rates and evolving (self-consistent) temperature, transporting recombination radiation rather than applying the on-the-spot approximation. Extract radial profiles of xH1, xH2, xHe1, xHe2, xHe3 and temperature from the solver output object, load the Cloudy 13.03 reference profiles, and compute relative differences in neutral fractions (xH1, xHe1) and absolute differences in ionization fractions versus radius. Write the radius-by-radius comparison to cloudy_comparison.csv and accept the run if neutral fractions agree within 5% relative difference at large radii and all ionization fractions agree within 0.02 absolute difference.

## Related tools

- **rabacus** (Solves the coupled equilibrium ionization/temperature problem for a spherically symmetric gas distribution with a point source at the center, including transfer of recombination radiation) — https://bitbucket.org/galtay/rabacus
- **Quantities** (Supplies units for all model variables; all values passed to rabacus must be Quantities unit-bearing arrays) — https://www.github.com/python-quantities/python-quantities
- **Numpy f2py** (Wraps the Fortran 90 core routines in Python during the rabacus build) — https://bitbucket.org/galtay/rabacus
- **Fortran 90** (Language of the compiled core routines; requires a compiler (e.g. gfortran) in the executable path with F90 set) — https://bitbucket.org/galtay/rabacus

## Examples

```
import rabacus as ra
```

## Evaluation signals

- Relative differences between rabacus and Cloudy 13.03 hydrogen and helium neutral fractions (xH1, xHe1) are less than 5% at large radii
- All ionization fractions agree with Cloudy 13.03 within 0.02 absolute difference
- rabacus imports successfully in a Python/ipython session after the pip install (building requires F90=gfortran and the scipy/numpy/quantities prerequisites)
- cloudy_comparison.csv is produced with one row per radius containing the neutral-fraction relative differences and ionization-fraction absolute differences

## Limitations

- Metal line cooling is not yet included in rabacus (planned future improvement)
- Non-equilibrium solvers are not yet included in rabacus (planned future improvement)
- The build requires a Fortran 90 compiler with OpenMP support; rabacus has been tested with the intel compiler and the gnu gfortran compiler, and other compilers need the manual install path

## Evidence
