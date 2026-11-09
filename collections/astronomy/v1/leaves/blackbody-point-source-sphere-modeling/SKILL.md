---
name: blackbody-point-source-sphere-modeling
description: Use when reproducing analytic radiative transfer benchmark problems such
  as a spherical gas distribution with a central point source (e.g., 20 kpc helium
  sphere), where radial H/He ionization and temperature profiles against Cloudy are
  needed.
when_to_use_negative:
- The problem is a plane-parallel slab in an isotropic background or a sphere bathed
  in an isotropic background rather than a sphere with a central point source — use
  the corresponding rabacus geometry class instead
- Metal line cooling or non-equilibrium (time-dependent) ionization physics is required
  — rabacus explicitly does not yet include these
- The domain requires full 3D radiative transfer rather than one of rabacus's three
  simplified geometries relevant to cosmology
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3318
tools:
- name: rabacus
  role: Python package whose spherical constant-density, central point-source solver
    computes the coupled equilibrium ionization and temperature radial profiles
  repo: https://bitbucket.org/galtay/rabacus
- name: Quantities
  role: Prerequisite package providing units on all rabacus variables; all model inputs
    (radius, densities, spectrum) must be passed as Quantities unit-bearing arrays
  repo: https://pypi.python.org/pypi/quantities/
- name: Numpy f2py
  role: Wraps the Fortran 90 core routines into Python during installation of rabacus
- name: Fortran 90 compiler (gfortran)
  role: Compiles the OpenMP-enabled Fortran 90 core; set F90=gfortran before pip install
    rabacus
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:6696da8d444b56680191eb5da16034212369e471e4e2874710806066f912efc8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Fortran 90 compiler (gfortran)
  - Numpy f2py
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# blackbody-point-source-sphere-modeling

## Summary

Configure and solve a spherically symmetric, constant-density gas cloud irradiated by a central point source (e.g., the Friedrich et al. 2012 20 kpc helium sphere) with rabacus, obtaining coupled ionization/temperature radial profiles under case A recombination with evolving temperature and recombination radiation transport.

## When to use

Use when reproducing analytic radiative transfer benchmark problems such as a spherical gas distribution with a central point source (e.g., 20 kpc helium sphere), where radial H/He ionization and temperature profiles against Cloudy are needed.

## When NOT to use

- The problem is a plane-parallel slab in an isotropic background or a sphere bathed in an isotropic background rather than a sphere with a central point source — use the corresponding rabacus geometry class instead
- Metal line cooling or non-equilibrium (time-dependent) ionization physics is required — rabacus explicitly does not yet include these
- The domain requires full 3D radiative transfer rather than one of rabacus's three simplified geometries relevant to cosmology

## Inputs

- Sphere geometry parameters as Quantities arrays: radius 20 kpc, constant densities nH=1e-3 cm^-3 and nHe=8.7e-5 cm^-3
- Central point source spectrum: 1e5 K blackbody
- Solver settings: case A recombination rates, evolving temperature, recombination radiation transport (no on-the-spot approximation)
- Cloudy 13.03 reference radial profiles for xH1, xHe1 and ionization fractions

## Outputs

- Radial profiles of xH1, xH2, xHe1, xHe2, xHe3 and temperature from the rabacus solver output object
- Radius-by-radius comparison table written to cloudy_comparison.csv
- Relative differences (neutral fractions) and absolute differences (ionization fractions) vs Cloudy 13.03

## How to apply

Build the model in rabacus as a spherical constant-density slab with a central point source: radius 20 kpc, nH=1e-3 cm^-3, nHe=8.7e-3-derived densities (nHe=8.7e-5 cm^-3 per the benchmark), and a 1e5 K blackbody spectrum, passing all values as Quantities unit-bearing arrays since every rabacus variable carries units. Solve the coupled ionization/temperature equilibrium using case A recombination rates, evolving (self-consistent) temperature rather than fixed temperature, and transporting recombination radiation instead of applying the on-the-spot approximation. Extract radial profiles of xH1, xH2, xHe1, xHe2, xHe3 and temperature from the solver output object. Compare against Cloudy 13.03 reference profiles by computing relative differences in neutral fractions (xH1, xHe1) and absolute differences in ionization fractions radius by radius, and write the comparison table to cloudy_comparison.csv.

## Related tools

- **rabacus** (Python package whose spherical constant-density, central point-source solver computes the coupled equilibrium ionization and temperature radial profiles) — https://bitbucket.org/galtay/rabacus
- **Quantities** (Prerequisite package providing units on all rabacus variables; all model inputs (radius, densities, spectrum) must be passed as Quantities unit-bearing arrays) — https://pypi.python.org/pypi/quantities/
- **Numpy f2py** (Wraps the Fortran 90 core routines into Python during installation of rabacus)
- **Fortran 90 compiler (gfortran)** (Compiles the OpenMP-enabled Fortran 90 core; set F90=gfortran before pip install rabacus)

## Examples

```
export F90=gfortran; pip install rabacus; python -c "import rabacus as ra"
```

## Evaluation signals

- Neutral hydrogen and helium fractions (xH1, xHe1) agree with Cloudy 13.03 within 5% relative difference at large radii
- All ionization fractions (xH2, xHe2, xHe3) agree with Cloudy 13.03 within 0.02 absolute difference at all radii
- Install sanity check succeeds: `import rabacus as ra` in ipython returns no error after the OpenMP-enabled pip install
- cloudy_comparison.csv contains a radius-by-radius table with finite relative/absolute difference columns for all tracked fractions
- All solver inputs are Quantities unit-bearing arrays; mixed unitless/unit inputs raise unit-conversion errors rather than silently succeeding

## Limitations

- Metal line cooling is not yet included in rabacus (planned future improvement)
- Non-equilibrium (time-dependent) solvers are not yet included; only equilibrium ionization/temperature states are computed
- No version number for the rabacus package is stated in the paper; only hydrogen and helium are treated, no heavier elements
- Quantities package has incomplete test coverage and is 'not suggested for mission-critical applications' per its README

## Evidence

- [abstract] Spherical point-source geometry is a supported case: "2) a spherically symmetric gas distribution with a point source at the center"
- [abstract] Solver includes H/He, temperature, and recombination radiation: "The solutions include a treatment of both hydrogen and helium, a self-consistent calculation of equilibrium temperatures, and the transfer of recombination radiation"
- [intro] All variables carry units via Quantities: "all variables in rabacus have units through use of the Quantities Python package"
- [readme] Install with F90 set for OpenMP build: "The simplest way to do this is to set the environment variable ``F90`` before following the installation instructions below."
- [readme] Import check after install: "To double check, begin an ipython session and attempt an import, ::

  import rabacus as ra"
- [intro] Attention to on-the-spot vs recombination radiation transfer: [evidence span withheld — anchor status: not_found]
