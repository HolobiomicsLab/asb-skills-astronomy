---
name: ionization-front-relative-difference-quantification
description: Use when validating a plane-parallel radiative-transfer solver against
  a known closed-form solution, i.e. when you have both a numerical and an analytic
  ionization-fraction profile as a function of column depth and need to quantify their
  agreement near the ionization front.
when_to_use_negative:
- The problem involves non-equilibrium ionization, metal-line cooling, or spectral
  (multi-frequency) radiation, which this monochromatic plane-parallel hydrogen-only
  validation does not cover
- The geometry is spherical (central point source or isotropic background) rather
  than a plane-parallel semi-infinite slab, since the closed-form slab solution applies
  only to the Slab_Pln case
- You need time-dependent radiative transfer — rabacus computes equilibrium ionization
  and temperature states only
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: rabacus
  role: Provides the Slab_Pln solver whose numerical hydrogen ionization-fraction
    profile is compared against the analytic closed-form solution
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: Implementation language for the analytic inverse solution and the pointwise
    relative-difference computation
- name: Quantities
  role: Provides unit-carrying variables for all slab and radiation-field inputs in
    the Slab_Pln setup
  repo: https://github.com/python-quantities/python-quantities
- name: Fortran 90
  role: Core routines of rabacus (wrapped in Python via f2py) that execute the Slab_Pln
    transfer calculation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:18948fb1b7e7e08c381790630d9fe1520a0d4f4c8a30398a8cee90442266f405
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Fortran 90
  - Python
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# ionization-front-relative-difference-quantification

## Summary

Validates the rabacus Slab_Pln solver by computing the pointwise relative difference between its numerical hydrogen ionization-fraction profile and the closed-form analytic inverse solution (Eqs. 23–25) on a common column-depth grid through a monochromatically illuminated, uniform pure-hydrogen slab.

## When to use

Use when validating a plane-parallel radiative-transfer solver against a known closed-form solution, i.e. when you have both a numerical and an analytic ionization-fraction profile as a function of column depth and need to quantify their agreement near the ionization front.

## When NOT to use

- The problem involves non-equilibrium ionization, metal-line cooling, or spectral (multi-frequency) radiation, which this monochromatic plane-parallel hydrogen-only validation does not cover
- The geometry is spherical (central point source or isotropic background) rather than a plane-parallel semi-infinite slab, since the closed-form slab solution applies only to the Slab_Pln case
- You need time-dependent radiative transfer — rabacus computes equilibrium ionization and temperature states only

## Inputs

- Uniform pure-hydrogen slab definition (density, thickness) with unit-carrying Quantities variables
- Monochromatic plane-parallel ionizing radiation field specification
- Closed-form analytic inverse solution (Eqs. 23-25) evaluated on a column-depth grid
- rabacus Slab_Pln numerical solution evaluated on the same column-depth grid

## Outputs

- Depth-resolved comparison table of pointwise relative difference (solution_comparison.csv)
- Maximum relative difference value near the ionization front (validation threshold: < 7e-3)

## How to apply

First implement the analytic inverse solution (Eqs. 23–25), which algebraically inverts the hydrogen ionization balance to give the ionization state as a function of column depth through the slab. Set up the corresponding Slab_Pln problem in rabacus for a uniform pure-hydrogen slab illuminated by monochromatic plane-parallel radiation, using unit-carrying variables from the Quantities package. Evaluate both the analytic and the rabacus numerical solution on a common column-depth grid that spans the ionization front. Compute the pointwise relative difference between the two hydrogen ionization-fraction profiles and record its maximum value near the ionization front. Write the depth-resolved comparison table and the maximum relative difference to solution_comparison.csv. The validation passes if the maximum relative difference near the ionization front is below 7e-3; over most of the slab the difference should be two orders of magnitude smaller than that peak.

## Related tools

- **rabacus** (Provides the Slab_Pln solver whose numerical hydrogen ionization-fraction profile is compared against the analytic closed-form solution) — https://bitbucket.org/galtay/rabacus
- **Python** (Implementation language for the analytic inverse solution and the pointwise relative-difference computation)
- **Quantities** (Provides unit-carrying variables for all slab and radiation-field inputs in the Slab_Pln setup) — https://github.com/python-quantities/python-quantities
- **Fortran 90** (Core routines of rabacus (wrapped in Python via f2py) that execute the Slab_Pln transfer calculation)

## Examples

```
export F90=gfortran; sudo pip install rabacus; ipython -c "import rabacus as ra; print('ok')"
```

## Evaluation signals

- The quick-test import `import rabacus as ra` runs without error in a fresh Python/ipython session
- Maximum pointwise relative difference between the analytic and Slab_Pln hydrogen ionization-fraction profiles near the ionization front is below 7e-3
- The relative difference is two orders of magnitude smaller than the peak over most of the slab (away from the ionization front)
- solution_comparison.csv contains a depth-resolved table on the shared column-depth grid spanning the ionization front
- Both analytic and numerical profiles were evaluated on the identical column-depth grid before differencing

## Limitations

- The 7e-3 agreement bound is established only for monochromatic plane-parallel radiation on a uniform pure-hydrogen slab; other configurations are not covered
- Metal line cooling is not yet included in rabacus (planned future improvement), so validation cannot address metal-enriched gas
- Non-equilibrium solvers are not yet included (planned future improvement); only equilibrium solutions are validated
- Quantities package test coverage is incomplete and the package is not suggested for mission-critical applications

## Evidence

- [methods] Maximum relative difference near the ionization front below 7e-3: [evidence span withheld — anchor status: not_found]
- [methods] Solver machinery validated: [evidence span withheld — anchor status: not_found]
