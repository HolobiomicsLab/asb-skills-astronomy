---
name: photoionization-temperature-profile-verification
description: Use when validating an analytic radiative-transfer code's H/He ionization
  fractions and equilibrium temperature profiles against a published benchmark (e.g.
  Iliev Test 2, T_eff=1e5 K blackbody, 1-10 Ryd, L_n=5e48/s) or a Cloudy run.
when_to_use_negative:
- The problem requires metal line cooling or non-equilibrium (time-dependent) ionization
  — rabacus does not yet include these (planned future improvements)
- The geometry is not one of rabacus's supported cases (semi-infinite slab in isotropic
  background, spherical gas with central point source, spherical gas in isotropic
  background)
- Quantitative agreement is needed in the innermost quarter kpc, where x_HI approaches
  the cloudy ionization-fraction deviation δ_H and differences exceed the few-percent
  tolerance
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3314
tools:
- name: rabacus
  role: Runs the analytic spherical point-source ionization and equilibrium temperature
    solver used to generate the benchmark profiles
  repo: https://bitbucket.org/galtay/rabacus
- name: Quantities
  role: Carries physical units on all solver variables so radial grids and profiles
    are dimensionally consistent during interpolation
  repo: python-quantities/python-quantities
- name: Numpy f2py
  role: Wraps the Fortran 90 (OpenMP) core routines for execution from Python
- name: Python
  role: Host language for the solver, interpolation onto the Cloudy radial grid, and
    relative-difference computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:ee759a0e450b45a368b94a1f862d73db05cc8fd10ba5725e839a161c048bee2a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Numpy f2py
  - Python
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# photoionization-temperature-profile-verification

## Summary

Verifies analytic photoionization and equilibrium temperature solutions by reproducing benchmark problems (e.g. Iliev et al. 2006 Test 2 sphere) in rabacus and comparing radial ionization fractions and temperature profiles against Cloudy 13.03 reference results.

## When to use

Use when validating an analytic radiative-transfer code's H/He ionization fractions and equilibrium temperature profiles against a published benchmark (e.g. Iliev Test 2, T_eff=1e5 K blackbody, 1-10 Ryd, L_n=5e48/s) or a Cloudy run.

## When NOT to use

- The problem requires metal line cooling or non-equilibrium (time-dependent) ionization — rabacus does not yet include these (planned future improvements)
- The geometry is not one of rabacus's supported cases (semi-infinite slab in isotropic background, spherical gas with central point source, spherical gas in isotropic background)
- Quantitative agreement is needed in the innermost quarter kpc, where x_HI approaches the cloudy ionization-fraction deviation δ_H and differences exceed the few-percent tolerance

## Inputs

- Central source specification: T_eff = 1e5 K blackbody spectrum truncated to 1–10 Rydbergs, photon emission rate L_n = 5e48 s^-1
- Gas density distribution for the spherical (Iliev Test 2) geometry
- Solver configuration: self-consistent equilibrium temperature evolution option enabled
- Cloudy 13.03 reference radial profiles for the identical problem setup

## Outputs

- Converged radial profiles of hydrogen and helium ionization fractions and gas temperature from rabacus
- Aligned profiles and pointwise relative differences written to test2_sphere_comparison.csv

## How to apply

Instantiate the rabacus spherical-geometry solver for a gas distribution with a central point source, configure the benchmark exactly: blackbody source with T_eff=1e5 K truncated to the 1–10 Rydberg band, photon emission rate L_n=5e48 s^-1, and self-consistent equilibrium temperature evolution enabled. Execute the Fortran 90/OpenMP solver (wrapped via Numpy f2py, all variables carrying units through the Quantities package) to converge the radial hydrogen and helium ionization fractions and temperature profile. Load the Cloudy 13.03 reference profiles for the identical setup, interpolate the rabacus profiles onto the Cloudy radial grid, and compute pointwise relative differences for each ion species (H I, H II, He I, He II, He III) and temperature. Accept the comparison when relative differences remain on the order of a few percent everywhere except the innermost quarter kpc, where x_HI approaches the cloudy ionization-fraction deviation δ_H.

## Related tools

- **rabacus** (Runs the analytic spherical point-source ionization and equilibrium temperature solver used to generate the benchmark profiles) — https://bitbucket.org/galtay/rabacus
- **Quantities** (Carries physical units on all solver variables so radial grids and profiles are dimensionally consistent during interpolation) — python-quantities/python-quantities
- **Numpy f2py** (Wraps the Fortran 90 (OpenMP) core routines for execution from Python)
- **Python** (Host language for the solver, interpolation onto the Cloudy radial grid, and relative-difference computation)

## Examples

```
export F90=gfortran; sudo pip install rabacus; python -c "import rabacus as ra; sol = ra.IlievTest2Sphere()"
```

## Evaluation signals

- Pointwise relative differences in ionization fractions and temperature vs Cloudy 13.03 are on the order of a few percent outside the innermost quarter kpc
- Deviation in the innermost quarter kpc is expected and explained by x_HI approaching the cloudy ionization-fraction deviation δ_H
- Solver converges on radial H and He ionization fractions and temperature with the temperature-evolution option enabled
- All profile arrays carry Quantities units; interpolation onto the Cloudy radial grid is dimensionally consistent
- test2_sphere_comparison.csv contains aligned profiles plus per-species and temperature relative-difference columns

## Limitations

- Metal line cooling is not yet included (planned future improvement)
- Non-equilibrium solvers are not yet included (planned future improvement)
- Agreement degrades in the innermost quarter kpc where x_HI approaches the cloudy ionization-fraction deviation δ_H
- Quantities package test coverage is incomplete and is not suggested for mission-critical applications

## Evidence

- [intro] implementation and units: [evidence span withheld — anchor status: not_found]
