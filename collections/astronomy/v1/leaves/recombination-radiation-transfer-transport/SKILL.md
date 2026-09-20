---
name: recombination-radiation-transfer-transport
description: 'Use when reproducing or validating an isotropic recombination-radiation
  sphere test (e.g. Raičević et al. 2014 Fig. 8): a uniform-density H cloud with a
  central point source where diffuse vs. direct photo-ionization rates must be compared.'
when_to_use_negative:
- The problem is a plane-parallel slab or an isotropic background on a sphere rather
  than a central point source in a spherical cloud — use the corresponding rabacus
  geometry solver instead
- Non-equilibrium ionization or metal line cooling is required — these are planned
  future improvements, not available in rabacus
- Full 3D or multi-frequency radiative transfer beyond rabacus's simplified analytic
  geometries is the goal
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: rabacus
  role: Provides the point-source hydrogen ionization solvers for case A, case B (on-the-spot),
    and full recombination radiation transport with isotropic Gauss-Legendre angle
    quadrature
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: Interpreted host language driving the rabacus solvers and CSV diagnostics;
    requires scipy, numpy, and quantities
  repo: http://pythonhosted.org//rabacus
- name: Fortran 90
  role: Core rabacus routines compiled via numpy f2py; OpenMP-enabled build needed
    for execution speed
  repo: https://github.com/galtay/rabacus
- name: Numpy f2py
  role: Wraps the Fortran 90 core into importable Python modules during installation
- name: Quantities
  role: Attaches physical units to all rabacus variables (cloud density, radius, spectrum,
    rates)
  repo: python-quantities/python-quantities
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:7a8b898b071f11cf3d41d49e0df20cf8bbae750773ff6dc588dc02985690629d
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
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# recombination-radiation-transfer-transport

## Summary

Solve the Test 1 Strömgren sphere with rabacus under the on-the-spot (case B) approximation and full recombination radiation transport (rt) with isotropic angle quadrature, then quantify the diffuse-to-central photo-ionization rate ratio (~0.6). Validates photon conservation in recombination radiation transfer.

## When to use

Use when reproducing or validating an isotropic recombination-radiation sphere test (e.g. Raičević et al. 2014 Fig. 8): a uniform-density H cloud with a central point source where diffuse vs. direct photo-ionization rates must be compared.

## When NOT to use

- The problem is a plane-parallel slab or an isotropic background on a sphere rather than a central point source in a spherical cloud — use the corresponding rabacus geometry solver instead
- Non-equilibrium ionization or metal line cooling is required — these are planned future improvements, not available in rabacus
- Full 3D or multi-frequency radiative transfer beyond rabacus's simplified analytic geometries is the goal

## Inputs

- Spherically symmetric, uniform-density hydrogen cloud parameters (density, radius grid)
- Central point-source ionizing spectrum with units (via Quantities)
- rabacus point-source solver configuration (case A, case B on-the-spot, and rt with N_mu=32 Gauss-Legendre angles)

## Outputs

- test1_ionization_profiles.csv containing the three hydrogen ionization profiles (case A, case B, rt) and ratio diagnostics
- Diffuse-to-central photo-ionization rate ratio as a function of radius
- Case B and rt Strömgren radii for photon-conservation comparison

## How to apply

Set up a spherically symmetric, uniform-density hydrogen cloud illuminated by a central point source with a defined ionizing spectrum. Solve the hydrogen ionization equilibrium profile three ways with the rabacus point-source solver: case A (all recombinations radiate away), case B (on-the-spot approximation, recombination photons absorbed locally), and a full recombination radiation transport (rt) solution using isotropic transport with N_mu=32 Gauss-Legendre angle quadrature points. Compute the ratio of diffuse (recombination-driven) to central source photo-ionization rates as a function of radius. Accept the result if the case B ratio is constant at approximately 0.6, the rt solution tracks case A at small radii, and the rt Strömgren radius agrees with the case B radius, confirming photon conservation.

## Related tools

- **rabacus** (Provides the point-source hydrogen ionization solvers for case A, case B (on-the-spot), and full recombination radiation transport with isotropic Gauss-Legendre angle quadrature) — https://bitbucket.org/galtay/rabacus
- **Python** (Interpreted host language driving the rabacus solvers and CSV diagnostics; requires scipy, numpy, and quantities) — http://pythonhosted.org//rabacus
- **Fortran 90** (Core rabacus routines compiled via numpy f2py; OpenMP-enabled build needed for execution speed) — https://github.com/galtay/rabacus
- **Numpy f2py** (Wraps the Fortran 90 core into importable Python modules during installation)
- **Quantities** (Attaches physical units to all rabacus variables (cloud density, radius, spectrum, rates)) — python-quantities/python-quantities

## Examples

```
import rabacus as ra; sol_B = ra.SphereIsovolved.HI([...])  # then compare diffuse/central photo-ionization rates; write test1_ionization_profiles.csv
```

## Evaluation signals

- Case B diffuse-to-central photo-ionization rate ratio is approximately constant at ~0.6 across radius
- The rt (full transport) solution overlaps the case A solution at small radii
- The rt Strömgren radius agrees with the case B Strömgren radius, confirming photon conservation
- Import of rabacus succeeds in a Python session with an OpenMP-enabled Fortran 90 build compiled via f2py
- Output CSV (test1_ionization_profiles.csv) contains the three hydrogen ionization profiles plus ratio diagnostics

## Limitations

- Metal line cooling is not yet included (planned future improvement)
- Non-equilibrium solvers are not yet included (planned future improvement)
- No version number for the rabacus package is stated in the source metadata
- Requires a Fortran 90 compiler (gfortran or ifort) in the path and the F90 environment variable set; other compilers need the manual setup.py install path

## Evidence

- [other]  isotropic transport with N_mu=32 Gauss-Legendre angle quadrature points: [evidence span withheld — anchor status: not_found]
- [other]  ratio of approximately 0.6 between diffuse and central photo-ionization rates: [evidence span withheld — anchor status: not_found]
- [intro]  Fortran 90 core wrapped in Python via f2py: [evidence span withheld — anchor status: not_found]
