---
name: gauss-legendre-angular-quadrature-transfer
description: Use when a central point source ionizes a spherical hydrogen cloud and
  the analysis requires the effect of diffuse recombination photons rather than the
  on-the-spot approximation, e.g. reproducing the Raičević et al. (2014) Test 1 sphere.
when_to_use_negative:
- The problem geometry is a semi-infinite slab or a spherical gas in an isotropic
  background rather than a spherical gas with a central point source, since rabacus
  treats these as distinct problems
- Non-equilibrium ionization or metal line cooling is required — the paper states
  these are future improvements not yet included
- Only the on-the-spot approximation is needed, in which case the case B solver alone
  suffices without the N_mu=32 quadrature transport
edam_operation: http://edamontology.org/operation_2426
tools:
- name: rabacus
  role: Provides the point-source spherical solver for case A, case B (on-the-spot),
    and the isotropic recombination radiation transfer solution with Gauss-Legendre
    quadrature
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: Interpreted driver language in which rabacus solvers are called and in which
    the ratio diagnostics and CSV export are computed
  repo: https://bitbucket.org/galtay/rabacus
- name: Fortran 90
  role: Language of the rabacus core routines that execute the transfer calculation,
    wrapped in Python via f2py
  repo: https://bitbucket.org/galtay/rabacus
- name: Numpy f2py
  role: Wraps the Fortran 90 core routines so they are callable from Python
- name: Quantities
  role: Supplies units for all variables passed to and returned from the rabacus solvers
  repo: python-quantities/python-quantities
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:68fdcaba4ad9827ff6b3ca2f25c228e29e366f5bd5ad33d03da96e6740bac186
schema_version: 0.2.0
metadata:
  license_tier: unknown
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
yes_rate: 0.7272727272727273
n_claims: 11
---

# gauss-legendre-angular-quadrature-transfer

## Summary

Solve equilibrium hydrogen ionization in a spherically symmetric, uniform-density Strömgren sphere with full transfer of isotropic recombination radiation, using rabacus with N_mu=32 Gauss-Legendre angle quadrature points, and benchmark against case A and on-the-spot (case B) solutions.

## When to use

Use when a central point source ionizes a spherical hydrogen cloud and the analysis requires the effect of diffuse recombination photons rather than the on-the-spot approximation, e.g. reproducing the Raičević et al. (2014) Test 1 sphere.

## When NOT to use

- The problem geometry is a semi-infinite slab or a spherical gas in an isotropic background rather than a spherical gas with a central point source, since rabacus treats these as distinct problems
- Non-equilibrium ionization or metal line cooling is required — the paper states these are future improvements not yet included
- Only the on-the-spot approximation is needed, in which case the case B solver alone suffices without the N_mu=32 quadrature transport

## Inputs

- Spherically symmetric, uniform-density hydrogen cloud definition with a central point source and an ionizing spectrum
- Angle quadrature setting for isotropic transport (N_mu=32 Gauss-Legendre points)

## Outputs

- Hydrogen ionization profiles for case A, case B, and the isotropic rt solution
- Diffuse-to-central photo-ionization rate ratio as a function of radius
- test1_ionization_profiles.csv containing the three profiles and ratio diagnostics

## How to apply

Set up a spherically symmetric, uniform-density hydrogen cloud illuminated by a central point source with a defined ionizing spectrum, and solve it three ways in rabacus: case A (all recombinations radiate away), case B (on-the-spot approximation, recombination photons absorbed locally), and the full isotropic recombination radiation transport solution using N_mu=32 Gauss-Legendre angle quadrature points. Compute the diffuse-to-central photo-ionization rate ratio as a function of radius from the rt solution and check it is approximately constant at ~0.6 for the case B comparison. Verify that the rt solution tracks case A at small radii and that the rt Strömgren radius agrees with the case B radius, confirming photon conservation. Write the three hydrogen ionization profiles and ratio diagnostics to test1_ionization_profiles.csv.

## Related tools

- **rabacus** (Provides the point-source spherical solver for case A, case B (on-the-spot), and the isotropic recombination radiation transfer solution with Gauss-Legendre quadrature) — https://bitbucket.org/galtay/rabacus
- **Python** (Interpreted driver language in which rabacus solvers are called and in which the ratio diagnostics and CSV export are computed) — https://bitbucket.org/galtay/rabacus
- **Fortran 90** (Language of the rabacus core routines that execute the transfer calculation, wrapped in Python via f2py) — https://bitbucket.org/galtay/rabacus
- **Numpy f2py** (Wraps the Fortran 90 core routines so they are callable from Python)
- **Quantities** (Supplies units for all variables passed to and returned from the rabacus solvers) — python-quantities/python-quantities

## Examples

```
import rabacus as ra  # after export F90=gfortran; pip install rabacus — then solve Test 1 point-source case A, case B, and isotropic rt with N_mu=32
```

## Evaluation signals

- Case B diffuse-to-central photo-ionization rate ratio is a constant of approximately 0.6 across the Strömgren sphere
- The rt solution overlaps the case A profile at small radii
- The rt Strömgren radius agrees with the case B Strömgren radius, confirming photon conservation
- All solver variables carry units through the Quantities package (no unit errors raised)
- test1_ionization_profiles.csv contains three hydrogen ionization profiles plus the ratio diagnostics

## Limitations

- Metal line cooling is not yet included in rabacus (planned future improvement)
- Non-equilibrium solvers are not yet included (planned future improvement)
- An OpenMP-enabled Fortran 90 build compiled via f2py is required; install may fail without a Fortran compiler (e.g. gfortran or ifort) in the executable path and the F90 environment variable set

## Evidence

- [full_text] N_mu=32 quadrature and ratio ~0.6: [evidence span withheld — anchor status: not_found]
- [full_text] expected ratio finding: [evidence span withheld — anchor status: not_found]
