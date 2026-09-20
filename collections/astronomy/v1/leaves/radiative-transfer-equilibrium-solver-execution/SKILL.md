---
name: radiative-transfer-equilibrium-solver-execution
description: Use when you need equilibrium hydrogen/helium ionization fractions and
  temperature profiles from a central point source (e.g. T_eff=1e5 K blackbody, 1-10
  Rydbergs, Ln=5e48 s^-1) in a spherically symmetric gas distribution.
when_to_use_negative:
- The problem requires non-equilibrium (time-dependent) ionization or temperature
  evolution — rabacus only ships equilibrium solvers.
- Metal line cooling is physically important — metal line cooling is not yet included
  in rabacus.
- The geometry is not one of the three supported cases (semi-infinite slab in isotropic
  background, spherical gas with central point source, or spherical gas in isotropic
  background).
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3452
tools:
- name: rabacus
  role: Python package providing the analytic radiative transfer solvers for equilibrium
    ionization and temperature states in planar or spherical geometries
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: Interpreted wrapper language driving the rabacus solver and post-processing
    of profiles
- name: Fortran 90
  role: Language of the core solver routines, compiled with OpenMP directives for
    speed
- name: Numpy f2py
  role: Wraps the Fortran 90 core routines so they are callable from Python
- name: Quantities
  role: Provides units for all variables passed to and returned from the solver
  repo: python-quantities/python-quantities
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:f70cdd3cc94c90b9da4cb45dc92a958b91d7348fbc0af524a6961eecbe643905
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
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# radiative-transfer-equilibrium-solver-execution

## Summary

Executes the rabacus radiative transfer solver (Fortran 90 core wrapped in Python via Numpy f2py with OpenMP) to compute equilibrium ionization and temperature profiles for well-posed planar or spherical problems, such as reproducing the Iliev et al. (2006) Test 2 sphere against Cloudy 13.03.

## When to use

Use when you need equilibrium hydrogen/helium ionization fractions and temperature profiles from a central point source (e.g. T_eff=1e5 K blackbody, 1-10 Rydbergs, Ln=5e48 s^-1) in a spherically symmetric gas distribution.

## When NOT to use

- The problem requires non-equilibrium (time-dependent) ionization or temperature evolution — rabacus only ships equilibrium solvers.
- Metal line cooling is physically important — metal line cooling is not yet included in rabacus.
- The geometry is not one of the three supported cases (semi-infinite slab in isotropic background, spherical gas with central point source, or spherical gas in isotropic background).

## Inputs

- Central source spectrum parameters (T_eff=1e5 K blackbody, 1-10 Rydberg band, Ln=5e48 photons/s)
- Spherically symmetric gas density distribution with radial grid
- Unit-carrying Quantities objects for all physical variables
- Reference solver profiles (e.g. Cloudy 13.03) for comparison

## Outputs

- Converged radial ionization fraction profiles for H I/II and He I/II/III
- Self-consistent equilibrium temperature profile
- Aligned comparison table of relative differences (e.g. test2_sphere_comparison.csv)

## How to apply

Instantiate the rabacus spherical-geometry solver class for a gas distribution with a central point source, configuring the source spectrum (here a T_eff=1e5 K blackbody truncated to the 1-10 Rydberg band with photon emission rate Ln=5e48 s^-1) and enabling the self-consistent equilibrium temperature evolution option. All variables must carry units through the Quantities package (e.g. distance in kpc, fluxes in cgs) since rabacus enforces dimensional consistency. Run the converged solver, whose Fortran 90 core with OpenMP directives executes thousands of times faster than pure Python equivalents. To validate, load reference profiles (e.g. Cloudy 13.03 for the identical problem), interpolate the rabacus radial profiles onto the reference grid, and compute pointwise relative differences for each ion species (H I, H II, He I, He II, He III) and temperature. Agreement on the order of a few percent outside the innermost quarter kpc (where x_HI approaches the cloudy ionization-fraction deviation delta_H) confirms correct execution.

## Related tools

- **rabacus** (Python package providing the analytic radiative transfer solvers for equilibrium ionization and temperature states in planar or spherical geometries) — https://bitbucket.org/galtay/rabacus
- **Python** (Interpreted wrapper language driving the rabacus solver and post-processing of profiles)
- **Fortran 90** (Language of the core solver routines, compiled with OpenMP directives for speed)
- **Numpy f2py** (Wraps the Fortran 90 core routines so they are callable from Python)
- **Quantities** (Provides units for all variables passed to and returned from the solver) — python-quantities/python-quantities

## Examples

```
export F90=gfortran; pip install rabacus; python -c "import rabacus as ra; print('rabacus OK')"
```

## Evaluation signals

- Relative differences in ionization fractions and temperature between rabacus and Cloudy 13.03 remain within a few percent outside the innermost quarter kpc.
- The solver run converges to a self-consistent equilibrium solution for all radial cells (hydrogen and helium ionization fractions plus temperature).
- All inputs and outputs carry physical units through the Quantities package; unitless arrays raise errors or are rejected.
- The installation check passes: importing rabacus and computing a He I photo-heating rate (e.g. at redshift 3.0 from the HM12 background) succeeds.
- The aligned profiles and relative differences are written with matching radial grids (interpolated rabacus profiles onto the reference grid).

## Limitations

- Metal line cooling is not yet included (planned future improvement).
- Non-equilibrium solvers are not yet included (planned future improvement).
- Comparison accuracy degrades in the innermost quarter kpc where x_HI approaches the cloudy ionization-fraction deviation delta_H.
- Quantities has incomplete test coverage and is not suggested for mission-critical applications.

## Evidence
