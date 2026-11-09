---
name: spherical-hii-region-benchmark-comparison
description: Use when validating an analytic equilibrium radiative-transfer solver
  against a reference code for a spherically symmetric gas distribution with a central
  point source, by comparing radial ionization-fraction and temperature profiles.
when_to_use_negative:
- The problem geometry is a semi-infinite slab in an isotropic background rather than
  a spherical gas distribution with a central point source — use rabacus's slab solver
  setup instead.
- Non-equilibrium (time-dependent) ionization or temperature evolution is required
  — rabacus only provides equilibrium solvers and metal line cooling is not yet included.
- A cosmological radiation background (e.g. HM12) rather than a central blackbody
  point source drives the ionization field.
edam_operation: http://edamontology.org/operation_2426
tools:
- name: rabacus
  role: Equilibrium ionization/temperature solver for the spherical point-source geometry;
    Fortran 90 core wrapped in Python
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: Drives the solver, interpolates rabacus profiles onto the Cloudy radial grid,
    computes relative differences, and writes the comparison CSV
  repo: http://pythonhosted.org//rabacus
- name: Numpy f2py
  role: Wraps the Fortran 90 core routines for calling from Python
  repo: https://bitbucket.org/galtay/rabacus
- name: Quantities
  role: Attaches physical units to all solver variables (radial coordinates, densities,
    temperatures) so unit-safe comparisons are possible
  repo: https://github.com/python-quantities/python-quantities
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:9b8939ad4a676ef654e245402f0de1c23bca9f78b3d996c740a7d6257b9bf4d1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
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
yes_rate: 0.9090909090909091
n_claims: 11
---

# spherical-hii-region-benchmark-comparison

## Summary

Reproduces a published Strömgren-sphere style H II region benchmark (Iliev et al. 2006 Test 2: spherically symmetric gas with a central T_eff = 1e5 K blackbody point source, Ln = 5e48 photons/s over 1–10 Rydbergs, with self-consistent equilibrium temperature evolution) using rabacus, then quantifies agreement against Cloudy 13.03 reference profiles.

## When to use

Use when validating an analytic equilibrium radiative-transfer solver against a reference code for a spherically symmetric gas distribution with a central point source, by comparing radial ionization-fraction and temperature profiles.

## When NOT to use

- The problem geometry is a semi-infinite slab in an isotropic background rather than a spherical gas distribution with a central point source — use rabacus's slab solver setup instead.
- Non-equilibrium (time-dependent) ionization or temperature evolution is required — rabacus only provides equilibrium solvers and metal line cooling is not yet included.
- A cosmological radiation background (e.g. HM12) rather than a central blackbody point source drives the ionization field.

## Inputs

- Benchmark specification: Iliev et al. (2006) Test 2 parameters (T_eff = 1e5 K blackbody, 1–10 Rydberg band, Ln = 5e48 photons/s)
- Cloudy 13.03 reference radial profiles of ionization fractions and temperature for the identical problem
- Radially symmetric gas density distribution with central point source geometry

## Outputs

- Converged rabacus radial profiles of hydrogen and helium ionization fractions and gas temperature
- Pointwise relative-difference profiles (rabacus vs. Cloudy) per ion species and for temperature
- Aligned comparison table, e.g. test2_sphere_comparison.csv

## How to apply

Instantiate the rabacus spherical-geometry solver class for a gas distribution with a central point source and configure the Test 2 problem: T_eff = 1e5 K blackbody truncated to the 1–10 Rydberg band, Ln = 5e48 s^-1, with the self-consistent equilibrium temperature calculation enabled (including helium and recombination radiation transfer). Run the Fortran 90 core (wrapped in Python via Numpy f2py, OpenMP enabled) to convergence, with all variables carrying units via the Quantities package. Load the Cloudy 13.03 reference profiles for the identical setup, interpolate the rabacus radial profiles onto the Cloudy grid, and compute pointwise relative differences for each ion species (H I, He I, He II) and for temperature, writing the aligned profiles to a CSV such as test2_sphere_comparison.csv. Judge success by the paper's reported agreement level: relative differences of a few percent outside the innermost quarter kpc, where x_HI deviations are dominated by the cloudy ionization-fraction deviation δ_H.

## Related tools

- **rabacus** (Equilibrium ionization/temperature solver for the spherical point-source geometry; Fortran 90 core wrapped in Python) — https://bitbucket.org/galtay/rabacus
- **Python** (Drives the solver, interpolates rabacus profiles onto the Cloudy radial grid, computes relative differences, and writes the comparison CSV) — http://pythonhosted.org//rabacus
- **Numpy f2py** (Wraps the Fortran 90 core routines for calling from Python) — https://bitbucket.org/galtay/rabacus
- **Quantities** (Attaches physical units to all solver variables (radial coordinates, densities, temperatures) so unit-safe comparisons are possible) — https://github.com/python-quantities/python-quantities

## Examples

```
sudo pip install rabacus && python -c "import rabacus as ra"
```

## Evaluation signals

- Pointwise relative differences in x_HII, x_HeII, x_HeIII and temperature between rabacus and Cloudy 13.03 remain on the order of a few percent beyond ~0.25 kpc radius.
- Any larger deviation near the center is confined to the innermost quarter kpc and coincides with x_HI approaching the cloudy ionization-fraction deviation δ_H.
- Rabacus output objects carry physical units (Quantities package); radial interpolation onto the Cloudy grid is performed before differencing.
- Solver converged: radial ionization-fraction and temperature profiles are stable and monotonic where expected for a Strömgren sphere.
- The aligned profiles and relative differences are written to test2_sphere_comparison.csv with one row per radius and per-species difference columns.

## Limitations

- Rabacus equilibrium solvers do not include metal line cooling or non-equilibrium (time-dependent) solvers; both are listed as future improvements.
- Agreement with Cloudy degrades in the innermost quarter kpc where x_HI approaches the cloudy ionization-fraction deviation δ_H, so few-percent validation holds only outside that region.
- Rabacus is restricted to simplified, well-posed geometries (semi-infinite slab, spherical gas with central point source, spherical gas in isotropic background) and cannot treat arbitrary geometries.

## Evidence

- [intro] F90 core with f2py wrapping justifies install/build requirements: [evidence span withheld — anchor status: not_found]
