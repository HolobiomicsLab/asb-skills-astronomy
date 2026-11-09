---
name: electron-density-fixed-point-iteration
description: Use when equilibrium ionization fractions of H/He gas are needed for
  given (T, nH, nHe) but eliminating n_e from the implicit photo-collisional equilibrium
  equations yields an intractable high-order polynomial.
when_to_use_negative:
- Non-equilibrium (time-dependent) ionization states are required — rabacus provides
  only equilibrium solvers; non-equilibrium solvers are a planned future improvement.
- Metal line cooling must be included in the thermal balance — it is not yet supported
  by the solvers.
- The problem already provides n_e directly or the system reduces to the closed-form
  pure-hydrogen case, making iteration unnecessary.
tools:
- name: rabacus
  role: Package implementing the fixed-point equilibrium solvers (Solve_CE, Solve_PCE,
    Solve_PCTE) with the bounded iterative n_e scheme of §4.2 and temperature search
    of §4.3
  repo: https://bitbucket.org/galtay/rabacus
- name: Fortran 90
  role: Language for the numerical core of the iterative solvers, giving thousands-of-times
    faster execution than pure Python
- name: Numpy f2py
  role: Wraps the Fortran 90 solver core into Python
- name: Quantities
  role: Attaches physical units to all solver variables (T, nH, nHe, n_e)
  repo: python-quantities/python-quantities
- name: Python
  role: High-level language in which the wrapped solvers are called and results evaluated
- name: SciPy
  role: Required prerequisite for the rabacus package (e.g. root-finding support)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:bfe376ced29b732cd0a5f7ab6a931991f1517ac04a75190bf0d73898694d2ad9
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# electron-density-fixed-point-iteration

## Summary

Robust iterative fixed-point scheme (§4.2 of rabacus) that solves the coupled H/He equilibrium ionization rate equations by repeatedly evaluating ionization fractions (Eqs. 21-22), re-deriving the electron density n_e (Eq. 4), and iterating until changes in n_e fall below a tolerance, avoiding an explicit fourth-order polynomial in n_e with thousands of terms.

## When to use

Use when equilibrium ionization fractions of H/He gas are needed for given (T, nH, nHe) but eliminating n_e from the implicit photo-collisional equilibrium equations yields an intractable high-order polynomial.

## When NOT to use

- Non-equilibrium (time-dependent) ionization states are required — rabacus provides only equilibrium solvers; non-equilibrium solvers are a planned future improvement.
- Metal line cooling must be included in the thermal balance — it is not yet supported by the solvers.
- The problem already provides n_e directly or the system reduces to the closed-form pure-hydrogen case, making iteration unnecessary.

## Inputs

- Temperature T (physical units, e.g. K)
- Hydrogen number density nH (physical units, e.g. 1/cm^3)
- Helium number density nHe (physical units, e.g. 1/cm^3)
- Initial guess for electron density n_e from the closed-form hydrogen solution (Eqs. 19-20)
- Photo-ionization and photo-heating rates (e.g. from the HM12 background) for the PC/PCTE variants

## Outputs

- Converged equilibrium ionization fractions for H and He species
- Converged electron density n_e (Eq. 4)
- For Solve_PCT: equilibrium temperature where photoheating equals radiative cooling
- Verification table (e.g. solve_ce_verification.csv) of solver outputs vs closed-form values with absolute and relative differences

## How to apply

Seed n_e with the closed-form pure-hydrogen collisional equilibrium solution (Eqs. 19-20), then evaluate all ionization fractions via the collisional (Eqs. 6-14) or photo-collisional (Eqs. 21-22) rate equations. Recompute n_e from charge neutrality (Eq. 4) using the updated fractions and repeat until the change in n_e drops below tolerance. For photo-collisional-thermal problems, wrap this scheme inside the temperature-search of §4.3, root-finding the temperature where photoheating balances radiative cooling. Implement the numerical core in Fortran 90 and wrap it into Python with Numpy f2py, attaching physical units via the Quantities package.

## Related tools

- **rabacus** (Package implementing the fixed-point equilibrium solvers (Solve_CE, Solve_PCE, Solve_PCTE) with the bounded iterative n_e scheme of §4.2 and temperature search of §4.3) — https://bitbucket.org/galtay/rabacus
- **Fortran 90** (Language for the numerical core of the iterative solvers, giving thousands-of-times faster execution than pure Python)
- **Numpy f2py** (Wraps the Fortran 90 solver core into Python)
- **Quantities** (Attaches physical units to all solver variables (T, nH, nHe, n_e)) — python-quantities/python-quantities
- **Python** (High-level language in which the wrapped solvers are called and results evaluated)
- **SciPy** (Required prerequisite for the rabacus package (e.g. root-finding support))

## Examples

```
import rabacus as ra; ce = ra.Solve_CE(T=1.0e4*pq.K, nH=0.01*pq.cm**-3, nHe=1.0e-2*pq.cm**-3)  # fixed-point n_e convergence per §4.2
```

## Evaluation signals

- Solve_CE outputs must agree with the closed-form pure-hydrogen expressions of Eqs. 19-20 at every sample point to within tolerance of the last significant figure.
- Iteration terminates when changes in n_e drop below the tolerance (convergence criterion of the fixed-point scheme).
- Recorded absolute and relative differences from the closed-form values (solve_ce_verification.csv) fall within tolerance at all (T, nH, nHe) test points.
- For Solve_PCT, the root-found temperature must satisfy photoheating = radiative cooling at the returned equilibrium state.
- The verification script runs without error and all variables carry Quantities units.

## Limitations

- Metal line cooling is not yet included in the solvers (planned future improvement).
- Non-equilibrium solvers are not yet included (planned future improvement).
- Building the Fortran core requires a Fortran compiler in the path and the F90 environment variable set; rabacus has been tested only with the intel compiler and gnu gfortran.
- Quantities has incomplete test coverage and is not suggested for mission-critical applications (per its README).

## Evidence

- [full_text] fixed-point n_e iteration: [evidence span withheld — anchor status: not_found]
- [intro] Fortran core wrapping: [evidence span withheld — anchor status: not_found]
