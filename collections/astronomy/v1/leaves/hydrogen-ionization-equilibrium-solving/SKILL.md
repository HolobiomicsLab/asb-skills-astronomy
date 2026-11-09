---
name: hydrogen-ionization-equilibrium-solving
description: Use when you need equilibrium ionization/temperature profiles or column
  densities for H/He gas under a central point source, an isotropic UV background
  (e.g. HM12), or single-zone conditions, and must choose between case A, case B (on-the-spot),
  or full recombination radiation transfer.
when_to_use_negative:
- You need non-equilibrium (time-dependent) ionization — the article states future
  improvements will include non-equilibrium solvers, which are not yet included.
- Metal line cooling is essential to your calculation — metal line cooling is not
  yet included in rabacus.
- The geometry is neither planar nor spherical with a point source or isotropic background
  — rabacus targets these simplified geometries specifically.
tools:
- name: rabacus
  role: Provides the equilibrium ionization/temperature solvers (Solve_CE, Solve_PCE,
    Solve_PCTE), point-source and background sphere solvers, and the HM12 photorates
    table.
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: High-level interface in which the solvers are invoked and from which profile
    outputs and diagnostics are collected.
  repo: https://pythonhosted.org//rabacus
- name: Quantities
  role: Attaches physical units to all solver inputs and outputs so profiles carry
    consistent dimensionality.
  repo: https://www.github.com/python-quantities/python-quantities
- name: Numpy f2py
  role: Wraps the OpenMP-enabled Fortran 90 numerical core so the iterative convergence
    schemes run at compiled speed from Python.
  repo: https://bitbucket.org/galtay/rabacus
- name: SciPy
  role: Prerequisite package used by rabacus (e.g. root-finding for the equilibrium
    temperature in the photo-collisional-thermal solver).
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:b5632390f251f1f0e3517fd2d0606eba64e97cb707351e017fb94222f83b134a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Numpy f2py
  - Python
  - Quantities
  - SciPy
  - numpy
  - rabacus
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# hydrogen-ionization-equilibrium-solving

## Summary

Solves equilibrium hydrogen (and helium) ionization and temperature states for well-posed radiative transfer problems in planar or spherical geometries using rabacus, whose Fortran 90 core is wrapped in Python via f2py. The skill covers iterative fixed-point convergence of the implicit photo-collisional equilibrium equations and validation against closed-form limits and photon-conservation tests.

## When to use

Use when you need equilibrium ionization/temperature profiles or column densities for H/He gas under a central point source, an isotropic UV background (e.g. HM12), or single-zone conditions, and must choose between case A, case B (on-the-spot), or full recombination radiation transfer.

## When NOT to use

- You need non-equilibrium (time-dependent) ionization — the article states future improvements will include non-equilibrium solvers, which are not yet included.
- Metal line cooling is essential to your calculation — metal line cooling is not yet included in rabacus.
- The geometry is neither planar nor spherical with a point source or isotropic background — rabacus targets these simplified geometries specifically.

## Inputs

- Ionizing source spectrum or radiation background (e.g. HM12 photorates table via ra.HM12_Photorates_Table)
- Gas density structure (uniform sphere, NFW halo profile from Eqs. 41-43, slab) with Quantities units
- Sample (T, nH, nHe) points for single-zone equilibrium solvers (e.g. test_points.csv)
- Solver-mode choices: case A, case B (on-the-spot), or rt with N_mu=32 Gauss-Legendre quadrature

## Outputs

- Hydrogen and helium ionization fraction profiles vs radius
- Equilibrium temperature profiles (self-consistent or fixed-T)
- Diffuse-to-central photo-ionization rate ratio diagnostics (e.g. test1_ionization_profiles.csv)
- Verified solver output tables (e.g. solve_ce_verification.csv, halo_ionization_profiles.h5)

## How to apply

Set the F90 environment variable (e.g. export F90=gfortran) and install the OpenMP-enabled rabacus build via pip; verify by importing rabacus and computing the He I photo-heating rate from the HM12 table at z=3.0 (expected 3.39163517433e-12 eV/s). For single-zone problems, exploit that rabacus avoids solving the fourth-order polynomial from eliminating n_e in Eqs. 21-22: instead seed n_e from the closed-form hydrogen ionization fraction, evaluate all ionization fractions, recompute n_e (Eq. 4), and iterate until changes in n_e fall below tolerance (Solve_CE / Solve_PC / Solve_PCTE, the latter root-finding the temperature where photoheating balances cooling). For sphere tests, solve case A (all recombinations radiate away), case B with the on-the-spot approximation, and the full isotropic recombination-radiation transport with N_mu=32 Gauss-Legendre angle quadrature points. Attach units to every variable through the Quantities package and write profile outputs (e.g. CSV or HDF5) with consistent units.

## Related tools

- **rabacus** (Provides the equilibrium ionization/temperature solvers (Solve_CE, Solve_PCE, Solve_PCTE), point-source and background sphere solvers, and the HM12 photorates table.) — https://bitbucket.org/galtay/rabacus
- **Python** (High-level interface in which the solvers are invoked and from which profile outputs and diagnostics are collected.) — https://pythonhosted.org//rabacus
- **Quantities** (Attaches physical units to all solver inputs and outputs so profiles carry consistent dimensionality.) — https://www.github.com/python-quantities/python-quantities
- **Numpy f2py** (Wraps the OpenMP-enabled Fortran 90 numerical core so the iterative convergence schemes run at compiled speed from Python.) — https://bitbucket.org/galtay/rabacus
- **SciPy** (Prerequisite package used by rabacus (e.g. root-finding for the equilibrium temperature in the photo-collisional-thermal solver).)

## Examples

```
export F90=gfortran; sudo pip install rabacus; python -c "import rabacus as ra"
```

## Evaluation signals

- Case B diffuse-to-central photo-ionization rate ratio is a constant of approximately 0.6 in the Test 1 Strömgren sphere.
- The rt (recombination radiation transfer) solution overlaps case A at small radii and its Strömgren radius agrees with the case B radius, confirming photon conservation.
- Solve_CE outputs agree with the closed-form pure-hydrogen collisional equilibrium expressions of Eqs. 19-20 at every sample point to within tolerance of the last significant figure.
- The HM12 table reproduces the documented He I photo-heating rate of 3.39163517433e-12 eV/s at z=3.0 before accepting solver runs.
- `import rabacus as ra` succeeds after installation, and output files contain profiles for all requested models with consistent physical units.

## Limitations

- Metal line cooling is not yet included (planned future improvement).
- Non-equilibrium (time-dependent) solvers are not yet included (planned future improvement).
- Rabacus has been tested with the intel compiler and gnu gfortran; other compilers may require manual install with adjusted f90_flags and omp_lib in setup.py.
- No version number for the rabacus package is stated in the source metadata.

## Evidence

- [intro] Fortran 90 core wrapped via f2py: [evidence span withheld — anchor status: not_found]
- [results] Test 1 ratio validation: [evidence span withheld — anchor status: not_found]
- [methods] Install test via He1 photo-heating rate: [evidence span withheld — anchor status: not_found]
