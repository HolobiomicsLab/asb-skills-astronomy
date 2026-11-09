---
name: on-the-spot-approximation-comparison
description: Use when validating a radiative transfer code's treatment of diffuse
  recombination photons, or when quantifying how the on-the-spot (case B) approximation
  biases photo-ionization rates in a central-source Strömgren sphere.
when_to_use_negative:
- The problem requires non-equilibrium (time-dependent) ionization solvers — these
  are planned future improvements, not yet included in rabacus
- Metal line cooling is physically important for the target cloud — metal line cooling
  is not yet included in rabacus
- The geometry is not one of rabacus's supported cases (semi-infinite slab in isotropic
  background, or spherically symmetric gas with a central point source or in an isotropic
  background)
edam_operation: http://edamontology.org/operation_2426
tools:
- name: rabacus
  role: Solves the equilibrium hydrogen ionization profiles for case A, case B (on-the-spot),
    and the isotropic recombination-radiation transport solution
  repo: https://github.com/galtay/rabacus
- name: Python
  role: Interpreted wrapper language in which the point-source solver is invoked and
    ratio diagnostics are computed
  repo: https://bitbucket.org/galtay/rabacus
- name: Quantities
  role: Provides units for all variables in the rabacus problem setup
  repo: https://github.com/python-quantities/python-quantities
- name: Numpy f2py
  role: Wraps the Fortran 90 core routines (with OpenMP) so they can be called from
    Python
  repo: https://bitbucket.org/galtay/rabacus
- name: Fortran 90
  role: Language of the core radiative transfer routines that must be compiled with
    an OpenMP-enabled f2py build
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:927d8456ef4945e1dfe1b6c177b2f62771058d2de315ebe73dc1dc6d95fea940
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/galtay/rabacus
  tools:
  - Fortran 90
  - Numpy f2py
  - Python
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# on-the-spot-approximation-comparison

## Summary

Reproduces the Raičević et al. (2014) isotropic recombination-radiation sphere benchmark with rabacus by solving the Test 1 Strömgren sphere under case A, case B (on-the-spot), and full recombination-radiation transport, then quantifying the diffuse-to-central photo-ionization rate ratio.

## When to use

Use when validating a radiative transfer code's treatment of diffuse recombination photons, or when quantifying how the on-the-spot (case B) approximation biases photo-ionization rates in a central-source Strömgren sphere.

## When NOT to use

- The problem requires non-equilibrium (time-dependent) ionization solvers — these are planned future improvements, not yet included in rabacus
- Metal line cooling is physically important for the target cloud — metal line cooling is not yet included in rabacus
- The geometry is not one of rabacus's supported cases (semi-infinite slab in isotropic background, or spherically symmetric gas with a central point source or in an isotropic background)

## Inputs

- Central point source ionizing spectrum for the Test 1 problem
- Spherically symmetric, uniform-density hydrogen cloud specification
- Rabacus build with scipy, numpy, quantities, and an OpenMP-enabled Fortran 90 core compiled via f2py

## Outputs

- Hydrogen ionization profiles for case A, case B, and the full rt solution (test1_ionization_profiles.csv)
- Diffuse-to-central photo-ionization rate ratio as a function of radius
- Case B and rt Strömgren radii for the photon-conservation check

## How to apply

Set up a spherically symmetric, uniform-density hydrogen cloud illuminated by a central point source with a defined ionizing spectrum. Solve the equilibrium hydrogen ionization profile three ways with the rabacus point-source solver: case A (all recombinations radiate away), case B using the on-the-spot approximation (recombination photons absorbed locally), and full recombination-radiation transport using isotropic transport with N_mu=32 Gauss-Legendre angle quadrature points. Compute the diffuse-to-central photo-ionization rate ratio as a function of radius for the case B solution. Write the three hydrogen ionization profiles and ratio diagnostics to test1_ionization_profiles.csv. Accept the run if the case B ratio is constant at approximately 0.6, the rt solution tracks case A at small radii, and the rt and case B Strömgren radii agree, confirming photon conservation.

## Related tools

- **rabacus** (Solves the equilibrium hydrogen ionization profiles for case A, case B (on-the-spot), and the isotropic recombination-radiation transport solution) — https://github.com/galtay/rabacus
- **Python** (Interpreted wrapper language in which the point-source solver is invoked and ratio diagnostics are computed) — https://bitbucket.org/galtay/rabacus
- **Quantities** (Provides units for all variables in the rabacus problem setup) — https://github.com/python-quantities/python-quantities
- **Numpy f2py** (Wraps the Fortran 90 core routines (with OpenMP) so they can be called from Python) — https://bitbucket.org/galtay/rabacus
- **Fortran 90** (Language of the core radiative transfer routines that must be compiled with an OpenMP-enabled f2py build)

## Examples

```
sudo pip install rabacus; python -c "import rabacus as ra"
```

## Evaluation signals

- Case B diffuse-to-central photo-ionization rate ratio is a constant of approximately 0.6 across radius
- The full rt solution overlaps the case A profile at small radii
- The rt Strömgren radius agrees with the case B Strömgren radius (photon conservation)
- The three hydrogen ionization profiles and ratio diagnostics are written to test1_ionization_profiles.csv
- The isotropic transport solution used N_mu=32 Gauss-Legendre angle quadrature points

## Limitations

- Metal line cooling is not yet included (planned future improvement)
- Non-equilibrium solvers are not yet included (planned future improvement)
- Fortran compilation requires the F90 environment variable to be set and a Fortran compiler in the path; only intel and gnu gfortran compilers are preconfigured, others need manual setup.py edits

## Evidence

- [intro] On-the-spot approximation is a focal process: [evidence span withheld — anchor status: not_found]
