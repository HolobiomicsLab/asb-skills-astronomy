---
name: photoheating-cooling-temperature-root-search
description: Use when computing self-consistent equilibrium temperatures (and ionization
  fractions) for hydrogen/helium gas irradiated by a photo-ionizing spectrum, where
  temperature must be solved so photoheating equals radiative cooling rather than
  being fixed a priori.
when_to_use_negative:
- The gas is optically thick in a slab or sphere geometry needing full transfer of
  ionizing radiation — use rabacus's radiative transfer solvers instead of the single-zone
  thermal root-search.
- Metal line cooling is required — the article states metal line cooling is not yet
  included (planned future improvement).
- Time-dependent (non-equilibrium) evolution is needed — non-equilibrium solvers are
  not yet included in rabacus.
- Temperature is fixed a priori — use the fixed-temperature Solve_PC path rather than
  the PCT temperature search.
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: rabacus
  role: Provides the Solve_PCT photo-collisional-thermal solver with the §4.3 temperature-search
    scheme that root-finds the equilibrium temperature
  repo: https://bitbucket.org/galtay/rabacus
- name: Quantities
  role: Attaches physical units to all variables (temperatures, densities, rates)
    entering and leaving the thermal root-find
  repo: https://www.github.com/python-quantities/python-quantities
- name: SciPy
  role: Prerequisite Python package used in the scientific environment hosting the
    solver workflow
  repo: http://www.scipy.org/scipylib/index.html
- name: Numpy f2py
  role: Wraps the Fortran 90 numerical core so the iterative n_e / temperature root-find
    executes at compiled speed from Python
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:fd7aa8ac949e12531c75e55b64185c036eb0fe9c4ecf37433db0ad01ecafaf63
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Numpy f2py
  - Quantities
  - SciPy
  - rabacus
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# photoheating-cooling-temperature-root-search

## Summary

Root-find the equilibrium temperature at each spatial cell where integrated photoheating rates (e.g. from an HM12 background) balance radiative cooling for H/He gas, as implemented in rabacus's photo-collisional-thermal solver Solve_PCT via the temperature-search scheme of §4.3.

## When to use

Use when computing self-consistent equilibrium temperatures (and ionization fractions) for hydrogen/helium gas irradiated by a photo-ionizing spectrum, where temperature must be solved so photoheating equals radiative cooling rather than being fixed a priori.

## When NOT to use

- The gas is optically thick in a slab or sphere geometry needing full transfer of ionizing radiation — use rabacus's radiative transfer solvers instead of the single-zone thermal root-search.
- Metal line cooling is required — the article states metal line cooling is not yet included (planned future improvement).
- Time-dependent (non-equilibrium) evolution is needed — non-equilibrium solvers are not yet included in rabacus.
- Temperature is fixed a priori — use the fixed-temperature Solve_PC path rather than the PCT temperature search.

## Inputs

- (T, nH, nHe) gas state sample points (e.g. from test_points.csv)
- Photo-ionizing radiation field / background (e.g. HM12) with spectral shape and redshift
- Photo-heating and photo-ionization rates for H I, He I, He II
- Radiative cooling rates for H/He species
- Convergence tolerance on n_e for the inner iterative solve

## Outputs

- Equilibrium temperature (with physical units) at which photoheating balances radiative cooling
- Converged H/He ionization fractions and electron density n_e at the equilibrium temperature
- Comparison records (e.g. CSV) of solver outputs vs reference values

## How to apply

First solve the photo-collisional problem (Solve_PC) at a trial temperature by iteratively converging n_e: use the closed-form hydrogen ionization fraction as the initial n_e guess, evaluate ionization fractions from the implicit photo-collisional equations (Eqs. 21-22), recompute n_e from charge neutrality (Eq. 4), and repeat until changes in n_e drop below tolerance. Then apply the temperature-search scheme of §4.3 (Solve_PCT) to bracket and root-find the temperature at which the net heating-cooling balance vanishes, re-solving the ionization state at each trial temperature. Rates include photo-ionization and photo-heating from the radiation field (e.g. the Haardt & Madau 2012 background) plus radiative cooling; the numerical core runs in Fortran 90 wrapped to Python via Numpy f2py, and all inputs/outputs carry physical units through the Quantities package. Decisions include solver choice (CE vs PC vs PCT) and convergence tolerance on n_e; the result is the equilibrium temperature returned with units alongside the converged ionization fractions.

## Related tools

- **rabacus** (Provides the Solve_PCT photo-collisional-thermal solver with the §4.3 temperature-search scheme that root-finds the equilibrium temperature) — https://bitbucket.org/galtay/rabacus
- **Quantities** (Attaches physical units to all variables (temperatures, densities, rates) entering and leaving the thermal root-find) — https://www.github.com/python-quantities/python-quantities
- **SciPy** (Prerequisite Python package used in the scientific environment hosting the solver workflow) — http://www.scipy.org/scipylib/index.html
- **Numpy f2py** (Wraps the Fortran 90 numerical core so the iterative n_e / temperature root-find executes at compiled speed from Python)

## Examples

```
import rabacus as ra; bck = ra.HM12(); q = bck.He1.photoheating_rate(z=3.0)
```

## Evaluation signals

- At the returned equilibrium temperature, the sum of photoheating rates equals the sum of radiative cooling rates (net heating ≈ 0) to within solver tolerance.
- The inner fixed-point iteration on n_e converges: successive n_e changes fall below the stated tolerance before the temperature search completes.
- Outputs carry physical units via Quantities and are dimensionally consistent (e.g. K, cm^-3).
- Solve_CE at the same (T, nH, nHe) points agrees with the closed-form pure-hydrogen equilibrium expressions of Eqs. 19-20 to within the last significant figure.
- The script runs without error and produces a well-formed comparison table (e.g. solve_ce_verification.csv).

## Limitations

- Metal line cooling is not yet included (planned future improvement in rabacus).
- Non-equilibrium solvers are not yet included (planned future improvement in rabacus).
- OpenMP-enabled Fortran builds require the F90 environment variable and a supported compiler (gnu gfortran or Intel ifort); other compilers need the manual install path.
- Quantities has incomplete test coverage and is not suggested for mission-critical applications.

## Evidence

- [methods] PCT temperature search: [evidence span withheld — anchor status: not_found]
- [full_text] iterative n_e scheme: [evidence span withheld — anchor status: not_found]
- [methods] HM12 photo-heating test: [evidence span withheld — anchor status: not_found]
