---
name: stromgren-sphere-ionization-profile-simulation
description: Use when you need to compute the radial hydrogen ionization fraction
  (xHII) for a spherically symmetric, uniform-density gas cloud illuminated by a central
  monochromatic ionizing source, e.g. reproducing the Iliev et al. (2006) Test 1 Strömgren
  sphere benchmark.
when_to_use_negative:
- The problem involves non-equilibrium ionization or temperature evolution — rabacus
  computes equilibrium states and its non-equilibrium solvers are not yet included
  (planned future improvement)
- The gas requires metal line cooling, which is not yet included in rabacus
- The geometry is not one of rabacus's supported cases (semi-infinite slab in isotropic
  background, spherical gas with central point source, or spherical gas in isotropic
  background)
edam_operation: http://edamontology.org/operation_0282
tools:
- name: rabacus
  role: Python package that computes the equilibrium ionization and recombination-radiation
    transfer solution for the spherical gas distribution with a point source at the
    center
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: Host language in which the Fortran 90 core routines are wrapped and the comparison
    workflow is scripted
- name: Numpy f2py
  role: Wrapping mechanism used to expose the Fortran 90 core routines to Python
- name: Quantities
  role: Provides units for all problem parameters (radius, density, luminosity) passed
    into the Sphere_Stromgren problem
  repo: python-quantities/python-quantities
- name: Cloudy
  role: External code whose version 13.03 reference ionization profile is used as
    the validation benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:c390121bfbe018a032c2fc48f3f8516157e504eb2579a54f490886add3ac0e6a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cloudy
  - Numpy f2py
  - Python
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# stromgren-sphere-ionization-profile-simulation

## Summary

Simulate the equilibrium ionization profile of a Strömgren sphere (spherically symmetric gas with a central point source) using the rabacus package, whose Fortran 90 core routines wrapped in Python via Numpy f2py solve hydrogen ionizing radiative transfer with recombination radiation, and validate against the Cloudy 13.03 reference profile.

## When to use

Use when you need to compute the radial hydrogen ionization fraction (xHII) for a spherically symmetric, uniform-density gas cloud illuminated by a central monochromatic ionizing source, e.g. reproducing the Iliev et al. (2006) Test 1 Strömgren sphere benchmark.

## When NOT to use

- The problem involves non-equilibrium ionization or temperature evolution — rabacus computes equilibrium states and its non-equilibrium solvers are not yet included (planned future improvement)
- The gas requires metal line cooling, which is not yet included in rabacus
- The geometry is not one of rabacus's supported cases (semi-infinite slab in isotropic background, spherical gas with central point source, or spherical gas in isotropic background)

## Inputs

- Point-source photon luminosity (5.0e48 s^-1) and monochromatic source energy (15 eV) for the Test 1 configuration
- Spherical gas cloud parameters: radius 6.6 kpc and hydrogen number density 1e-3 cm^-3 (as Quantities-unit variables)
- Physics options: case-B recombination radiation treatment and fixed (non-evolving) temperature
- Cloudy 13.03 reference ionization profile for the same Test 1 configuration

## Outputs

- Radial hydrogen ionization fraction profile (xHII vs. radius) from the rabacus Sphere_Stromgren solution
- sphere_stromgren_test1_comparison.csv containing rabacus and Cloudy ionization profiles plus point-by-point relative differences

## How to apply

First install an OpenMP-enabled rabacus with pip after setting the F90 environment variable (export F90=gfortran or F90=ifort), since the core routines are Fortran 90 wrapped in Python via Numpy f2py; verify by importing rabacus in a Python session. Build a Sphere_Stromgren problem instance with the Test 1 configuration: monochromatic 15 eV point source, case-B recombination radiation treatment, fixed (non-evolving) temperature, spherical radius 6.6 kpc, hydrogen number density 1e-3 cm^-3, and source photon luminosity 5.0e48 s^-1, expressing all parameters as Quantities-unit variables because all variables in rabacus carry units. Solve the equilibrium ionization and recombination-radiation transfer problem to obtain the radial xHII profile, then load the Cloudy 13.03 reference profile for the same configuration and interpolate it onto the rabacus radial grid. Compute the point-by-point relative difference between the rabacus and Cloudy hydrogen ionization fractions and write both profiles plus the relative differences to sphere_stromgren_test1_comparison.csv.

## Related tools

- **rabacus** (Python package that computes the equilibrium ionization and recombination-radiation transfer solution for the spherical gas distribution with a point source at the center) — https://bitbucket.org/galtay/rabacus
- **Python** (Host language in which the Fortran 90 core routines are wrapped and the comparison workflow is scripted)
- **Numpy f2py** (Wrapping mechanism used to expose the Fortran 90 core routines to Python)
- **Quantities** (Provides units for all problem parameters (radius, density, luminosity) passed into the Sphere_Stromgren problem) — python-quantities/python-quantities
- **Cloudy** (External code whose version 13.03 reference ionization profile is used as the validation benchmark)

## Examples

```
export F90=gfortran; sudo pip install rabacus; python -c "import rabacus as ra"
```

## Evaluation signals

- The typical relative difference between the rabacus and Cloudy 13.03 hydrogen ionization fractions is below one percent
- The relative difference rises to only a few percent, confined to the region near the ionization front
- The pip installation reports 'Successfully installed rabacus' and `import rabacus as ra` succeeds in a Python session
- The output file sphere_stromgren_test1_comparison.csv contains both profiles and the point-by-point relative differences on a common radial grid
- All problem parameters are accepted as Quantities-unit variables without unit errors

## Limitations

- Metal line cooling is not yet included (planned future improvement)
- Non-equilibrium solvers are not yet included (planned future improvement)
- The build requires a supported Fortran 90 compiler (gfortran or ifort) in the executable path, with F90 set before installation; other compilers require the manual setup.py install path
- No version number for the rabacus package is stated in the article text

## Evidence

- [intro] Fortran core wrapped via f2py: [evidence span withheld — anchor status: not_found]
