---
name: closed-form-analytic-solution-verification
description: Use when implementing a new equilibrium solver (e.g. Solve_CE) and a
  closed-form expression exists (Eqs. 19-20) that lets you validate outputs at sample
  (T, nH, nHe) points before trusting the iterative code.
when_to_use_negative:
- No closed-form solution exists for the configuration (e.g. photo-collisional Solve_PC
  or photo-collisional-thermal Solve_PCT with radiation fields) — use independent
  cross-checks or convergence tests instead
- The problem includes helium or non-equilibrium physics, since Eqs. 19-20 are valid
  only for pure-hydrogen collisional equilibrium
- Metal line cooling or non-equilibrium effects matter, as these are not included
  in rabacus solvers
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: rabacus
  role: Provides the iterative Solve_CE solver being verified against the closed-form
    expressions; installed via pip
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: Host language driving the comparison script and writing solve_ce_verification.csv
- name: Fortran 90
  role: Language of the numerical core implementing the rate equations and fixed-point
    n_e convergence being verified
- name: Numpy f2py
  role: Wraps the Fortran 90 solver core so it can be called from the Python verification
    script
- name: Quantities
  role: Attaches physical units to all variables in the comparison so T, nH, nHe and
    outputs are dimensionally consistent
  repo: https://www.github.com/python-quantities/python-quantities
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:95b2a15c1a79082a902329d26005992fe0fbb113ce6bddfe758f1d4728a52666
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Fortran 90
  - Numpy f2py
  - Python
  - Quantities
  - SciPy
  - rabacus
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# closed-form-analytic-solution-verification

## Summary

Verifies an iterative numerical solver against an available closed-form analytic solution by evaluating both at shared sample points and recording absolute/relative differences, as rabacus does for Solve_CE vs. the pure-hydrogen collisional equilibrium expressions (Eqs. 19-20).

## When to use

Use when implementing a new equilibrium solver (e.g. Solve_CE) and a closed-form expression exists (Eqs. 19-20) that lets you validate outputs at sample (T, nH, nHe) points before trusting the iterative code.

## When NOT to use

- No closed-form solution exists for the configuration (e.g. photo-collisional Solve_PC or photo-collisional-thermal Solve_PCT with radiation fields) — use independent cross-checks or convergence tests instead
- The problem includes helium or non-equilibrium physics, since Eqs. 19-20 are valid only for pure-hydrogen collisional equilibrium
- Metal line cooling or non-equilibrium effects matter, as these are not included in rabacus solvers

## Inputs

- test_points.csv with sample (T, nH, nHe) points
- Closed-form pure-hydrogen collisional equilibrium expressions (Eqs. 19-20)
- Implemented iterative Solve_CE solver (Fortran 90 core wrapped via f2py)

## Outputs

- solve_ce_verification.csv containing solver outputs, closed-form values, and absolute and relative differences per sample point
- Pass/fail verdict per sample point against last-significant-figure tolerance

## How to apply

First evaluate the closed-form analytic expressions (Eqs. 19-20 for pure-hydrogen collisional equilibrium) directly at each (T, nH, nHe) point from test_points.csv. Then run the iterative solver (Solve_CE, the fixed-point scheme of §4.2 that converges n_e from an initial closed-form hydrogen guess) at the same points and record solver outputs alongside the analytic values. Compute absolute and relative differences per quantity per point and write them to solve_ce_verification.csv. The verification passes only if the iterative solver agrees with the closed-form values at every sample point to within a tolerance of the last significant figure, and the script runs without error. This works because the closed form is only available for the restricted pure-hydrogen CE case, so it validates the shared numerical core (rate equations Eqs. 6-14, n_e fixed-point convergence) that the more general Solve_PC and Solve_PCT solvers build on.

## Related tools

- **rabacus** (Provides the iterative Solve_CE solver being verified against the closed-form expressions; installed via pip) — https://bitbucket.org/galtay/rabacus
- **Python** (Host language driving the comparison script and writing solve_ce_verification.csv)
- **Fortran 90** (Language of the numerical core implementing the rate equations and fixed-point n_e convergence being verified)
- **Numpy f2py** (Wraps the Fortran 90 solver core so it can be called from the Python verification script)
- **Quantities** (Attaches physical units to all variables in the comparison so T, nH, nHe and outputs are dimensionally consistent) — https://www.github.com/python-quantities/python-quantities

## Examples

```
export F90=gfortran; sudo pip install scipy numpy quantities rabacus; python verify_solve_ce.py --points test_points.csv --out solve_ce_verification.csv
```

## Evaluation signals

- Solve_CE outputs agree with closed-form Eqs. 19-20 values at every sample point to within tolerance of the last significant figure
- solve_ce_verification.csv contains absolute and relative difference columns for every (T, nH, nHe) entry in test_points.csv with no missing rows
- The verification script runs end-to-end without error
- The iterative n_e fixed-point loop converges (changes in n_e drop below tolerance) at every sample point rather than hitting a maximum-iteration cap
- All quantities carry physical units (Quantities package) so unit mismatches would raise errors rather than silently corrupt the comparison

## Limitations

- The closed-form check only covers pure-hydrogen collisional equilibrium; Solve_PC and Solve_PCT have no analytic counterpart and cannot be verified this way
- Metal line cooling and non-equilibrium solvers are not included in rabacus, so the verified solver scope excludes those processes
- Rabacus requires a Fortran 90 compiler (tested with intel and gnu gfortran) and the F90 environment variable set correctly; the build can fail without it

## Evidence

- [other] rabacus uses a robust iterative approach: an initial guess for n_e from the closed-form hydrogen ionization fraction is used to evaluate all ionization fractions: [evidence span withheld — anchor status: not_found]
- [methods] Reconstruct the three single-zone equilibrium solvers (Solve_CE, Solve_PCE, Solve_PCTE): [evidence span withheld — anchor status: not_found]
- [abstract] Speed rationale for the Fortran core under verification: "The core routines are written in Fortran 90 and then wrapped in Python"
- [intro] Units enforcement in the comparison workflow: "all variables in rabacus have units through use of the Quantities Python package"
