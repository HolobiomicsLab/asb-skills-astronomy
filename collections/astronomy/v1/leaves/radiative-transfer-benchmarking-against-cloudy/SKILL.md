---
name: radiative-transfer-benchmarking-against-cloudy
description: Use when a new or updated equilibrium ionization/radiative-transfer solver
  needs code verification against an established photoionization reference (Cloudy)
  using a well-posed benchmark geometry with published parameters.
when_to_use_negative:
- Problems requiring metal line cooling or non-equilibrium (time-dependent) solvers
  — these are planned future rabacus improvements and absent from the code.
- Ill-posed transfer problems outside rabacus's three supported geometries (semi-infinite
  slab with isotropic background, spherical gas with central point source, spherical
  gas in isotropic background).
- Cases where the reference code configuration (spectral shape, case A/B treatment,
  temperature handling) cannot be matched exactly, since benchmark acceptance depends
  on a like-for-like comparison.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3360
tools:
- name: rabacus
  role: Solves the Test 1 Strömgren sphere equilibrium ionization and recombination-radiation
    transfer problem to produce the xHII profile under comparison
  repo: https://bitbucket.org/galtay/rabacus
- name: Cloudy
  role: Independent photoionization code (version 13.03) supplying the reference ionization
    profile for the benchmark comparison
- name: Quantities
  role: Provides physical units on all Sphere_Stromgren input variables so configurations
    are dimensionally consistent
  repo: python-quantities/python-quantities
- name: Numpy f2py
  role: Wraps the Fortran 90 core routines for invocation from Python during the benchmark
    run
- name: Python
  role: 'Hosts the benchmark workflow: builds the problem instance, interpolates the
    Cloudy profile onto the rabacus grid, and writes the comparison CSV'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:035cd4bdba091927a9f6e9d887ad638141d0cec775e7857bb054b88689bdea36
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cloudy
  - Numpy f2py
  - Python
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# radiative-transfer-benchmarking-against-cloudy

## Summary

Validates an analytic radiative transfer solver (rabacus) against the Cloudy 13.03 photoionization code on the Iliev et al. (2006) Test 1 Strömgren sphere — a pure-hydrogen, monochromatic 15 eV, case-B, fixed-temperature problem — by quantifying the point-by-point relative difference in hydrogen ionization fraction profiles.

## When to use

Use when a new or updated equilibrium ionization/radiative-transfer solver needs code verification against an established photoionization reference (Cloudy) using a well-posed benchmark geometry with published parameters.

## When NOT to use

- Problems requiring metal line cooling or non-equilibrium (time-dependent) solvers — these are planned future rabacus improvements and absent from the code.
- Ill-posed transfer problems outside rabacus's three supported geometries (semi-infinite slab with isotropic background, spherical gas with central point source, spherical gas in isotropic background).
- Cases where the reference code configuration (spectral shape, case A/B treatment, temperature handling) cannot be matched exactly, since benchmark acceptance depends on a like-for-like comparison.

## Inputs

- Sphere_Stromgren problem parameters: monochromatic 15 eV point source, case-B recombination, fixed temperature, spherical radius 6.6 kpc, nH = 1e-3 cm^-3, source photon luminosity 5.0e48 s^-1 (with Quantities units)
- Cloudy 13.03 reference hydrogen ionization profile for the same Test 1 configuration
- Quantities-wrapped physical unit variables (length, density, luminosity)

## Outputs

- Radial xHII (hydrogen ionization fraction) profile from the rabacus Fortran 90 solver
- Point-by-point relative difference between rabacus and Cloudy ionization fractions
- sphere_stromgren_test1_comparison.csv containing both profiles plus the relative differences

## How to apply

Install OpenMP-enabled rabacus via pip after exporting F90=gfortran (or ifort), and verify with an import in a Python session. Build a Sphere_Stromgren instance mirroring Iliev et al. Test 1: monochromatic 15 eV point source, case-B recombination treatment, fixed (non-evolving) temperature, radius 6.6 kpc, hydrogen number density 1e-3 cm^-3, source photon luminosity 5.0e48 s^-1, with units supplied through the Quantities package. Solve the equilibrium ionization plus recombination-radiation transfer problem with the Fortran 90 core routines wrapped in Python via Numpy f2py to obtain the radial xHII profile. Load the Cloudy 13.03 reference profile for the identical configuration, interpolate it onto the rabacus radial grid, and compute the point-by-point relative difference in hydrogen ionization fraction. Accept the benchmark when the typical relative difference is below one percent, allowing a few percent only near the ionization front.

## Related tools

- **rabacus** (Solves the Test 1 Strömgren sphere equilibrium ionization and recombination-radiation transfer problem to produce the xHII profile under comparison) — https://bitbucket.org/galtay/rabacus
- **Cloudy** (Independent photoionization code (version 13.03) supplying the reference ionization profile for the benchmark comparison)
- **Quantities** (Provides physical units on all Sphere_Stromgren input variables so configurations are dimensionally consistent) — python-quantities/python-quantities
- **Numpy f2py** (Wraps the Fortran 90 core routines for invocation from Python during the benchmark run)
- **Python** (Hosts the benchmark workflow: builds the problem instance, interpolates the Cloudy profile onto the rabacus grid, and writes the comparison CSV)

## Examples

```
export F90=gfortran; pip install rabacus; python -c "import rabacus as ra"
```

## Evaluation signals

- Typical point-by-point relative difference between rabacus and Cloudy 13.03 hydrogen ionization fractions is below one percent over most of the profile.
- Relative difference rises to only a few percent in the region near the ionization front, and nowhere else exceeds that envelope.
- sphere_stromgren_test1_comparison.csv contains both profiles and the relative differences on a common radial grid (Cloudy profile interpolated onto the rabacus grid).
- Rabacus import succeeds after pip installation and the F90 environment variable was set to a supported compiler (gfortran or ifort), confirming the OpenMP-enabled Fortran build.
- Problem configuration matches Test 1 exactly: 15 eV monochromatic source, case-B recombination, fixed temperature, 6.6 kpc radius, 1e-3 cm^-3 density, 5.0e48 s^-1 luminosity.

## Limitations

- Metal line cooling is not yet included in rabacus (planned future improvement).
- Non-equilibrium solvers are not yet included in rabacus (planned future improvement).
- Agreement degrades near the ionization front, where the relative difference reaches a few percent rather than the sub-one-percent typical level.
- Rabacus has been tested with the intel and gnu gfortran compilers; other compilers require the manual install path and may not work with the single-command pip install.

## Evidence

- [abstract] Test 1 configuration is well-posed with fixed parameters: [evidence span withheld — anchor status: not_found]
