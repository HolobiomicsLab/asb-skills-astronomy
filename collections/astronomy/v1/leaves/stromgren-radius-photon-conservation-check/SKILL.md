---
name: stromgren-radius-photon-conservation-check
description: Use when you need to benchmark an equilibrium ionization solver against
  a published Strömgren sphere test, or to quantify how recombination radiation (case
  A vs case B vs full transfer) alters an H II region's ionization profile.
when_to_use_negative:
- The geometry is planar (slab) or an isotropic background on a sphere rather than
  a central point source — use the corresponding rabacus geometry solver instead
- The problem requires non-equilibrium ionization or metal line cooling — the article
  states these are planned future improvements and not yet included
- You need helium-only or temperature-free test verification — this skill is specifically
  the hydrogen Strömgren sphere photon-conservation check
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: rabacus
  role: Core package used to solve the case A, case B (on-the-spot), and full recombination-radiation
    transport (N_mu=32) ionization profiles for the point-source Strömgren sphere
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: Interpreted host language in which the Fortran 90 solvers are wrapped and
    the test script is written
- name: Numpy f2py
  role: Wraps the compiled Fortran 90 core routines so they are callable from the
    Python test workflow
- name: Quantities
  role: Provides units for all variables (cloud density, spectrum, photo-ionization
    rates) passed into and read out of rabacus
  repo: https://github.com/python-quantities/python-quantities
- name: Fortran 90
  role: OpenMP-enabled compiled core that executes the equilibrium ionization solves
    at high speed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:5ef49edfeec3c7e038a809ec6756a1af1670d944f96fe1e45e5aa809c00ea88c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Fortran 90
  - Numpy f2py
  - Python
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# stromgren-radius-photon-conservation-check

## Summary

Reproduces the Raičević et al. (2014) Test 1 Strömgren sphere with rabacus, comparing case A, case B (on-the-spot), and full isotropic recombination-radiation transport solutions, and validates the result via a constant ~0.6 diffuse-to-central photo-ionization rate ratio and photon-conserving agreement of Strömgren radii.

## When to use

Use when you need to benchmark an equilibrium ionization solver against a published Strömgren sphere test, or to quantify how recombination radiation (case A vs case B vs full transfer) alters an H II region's ionization profile.

## When NOT to use

- The geometry is planar (slab) or an isotropic background on a sphere rather than a central point source — use the corresponding rabacus geometry solver instead
- The problem requires non-equilibrium ionization or metal line cooling — the article states these are planned future improvements and not yet included
- You need helium-only or temperature-free test verification — this skill is specifically the hydrogen Strömgren sphere photon-conservation check

## Inputs

- Spherically symmetric, uniform-density hydrogen cloud specification (density, radius grid)
- Central point source ionizing spectrum for the Test 1 problem
- Angle quadrature setting for isotropic recombination-radiation transport (N_mu=32 Gauss-Legendre points)

## Outputs

- test1_ionization_profiles.csv containing three hydrogen ionization profiles (case A, case B, rt) and ratio diagnostics
- Diffuse-to-central photo-ionization rate ratio as a function of radius (expected constant ~0.6 for case B)
- Strömgren radii for the case B and rt solutions, checked for agreement (photon conservation)

## How to apply

Install rabacus with its prerequisites (scipy, numpy>=1.7, quantities) after setting the F90 environment variable (e.g. export F90=gfortran) so the OpenMP-enabled Fortran 90 core is built via f2py. Set up the Test 1 problem: a spherically symmetric, uniform-density hydrogen cloud with a central point source and a defined ionizing spectrum. Solve the equilibrium hydrogen ionization profile three ways with the rabacus point-source solver: case A (all recombinations radiate away), case B (on-the-spot approximation), and full recombination radiation transport using isotropic transport with N_mu=32 Gauss-Legendre angle quadrature points. Compute the diffuse-to-central photo-ionization rate ratio as a function of radius and write the three hydrogen ionization profiles plus ratio diagnostics to test1_ionization_profiles.csv. Accept the run if the case B ratio is a constant of approximately 0.6, the rt solution overlaps case A at small radii, and the rt and case B Strömgren radii agree, confirming photon conservation.

## Related tools

- **rabacus** (Core package used to solve the case A, case B (on-the-spot), and full recombination-radiation transport (N_mu=32) ionization profiles for the point-source Strömgren sphere) — https://bitbucket.org/galtay/rabacus
- **Python** (Interpreted host language in which the Fortran 90 solvers are wrapped and the test script is written)
- **Numpy f2py** (Wraps the compiled Fortran 90 core routines so they are callable from the Python test workflow)
- **Quantities** (Provides units for all variables (cloud density, spectrum, photo-ionization rates) passed into and read out of rabacus) — https://github.com/python-quantities/python-quantities
- **Fortran 90** (OpenMP-enabled compiled core that executes the equilibrium ionization solves at high speed)

## Examples

```
import rabacus as ra
```

## Evaluation signals

- Case B diffuse-to-central photo-ionization rate ratio is a constant of approximately 0.6 across radius in the Test 1 Strömgren sphere
- The full recombination-radiation (rt) solution overlaps the case A solution at small radii
- The rt Strömgren radius agrees with the case B Strömgren radius, confirming photon conservation
- test1_ionization_profiles.csv is written with all three hydrogen ionization profiles and ratio diagnostics
- rabacus imports successfully in a Python session, indicating the f2py-wrapped Fortran build completed (e.g. import rabacus as ra)

## Limitations

- The Fortran build requires an OpenMP-enabled Fortran 90 compiler in the path; rabacus has been tested with the intel compiler and gnu gfortran, and other compilers require the manual setup.py install path
- The case B result relies on the on-the-spot approximation; deviations from ~0.6 indicate the approximation or quadrature (N_mu=32) is inadequate for the problem
- Non-equilibrium solvers and metal line cooling are not included in rabacus (planned future improvements), so only equilibrium hydrogen/helium tests are supported

## Evidence

- [intro] Fortran 90 core wrapped in Python: [evidence span withheld — anchor status: not_found]
