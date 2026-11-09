---
name: spherical-isotropic-background-radiative-transfer
description: Use when a task needs radial ionization/temperature structure of a cosmological
  gaseous halo (e.g. an NFW profile) illuminated by a meta-galactic background rather
  than a central point source, and multiple background/temperature-treatment models
  must be compared.
when_to_use_negative:
- The geometry is a semi-infinite slab in an isotropic background or a sphere with
  a central point source — those are distinct rabacus cases, not Sphere_Bgnd
- A non-equilibrium (time-dependent) ionization solution is needed — the paper states
  non-equilibrium solvers are a planned future improvement
- Metal line cooling must be modeled — the article notes metal line cooling is not
  yet included
tools:
- name: rabacus
  role: Provides the Sphere_Bgnd solver computing hydrogen/helium ionization, equilibrium
    temperatures, and recombination radiation for a sphere in an isotropic background
  repo: https://bitbucket.org/galtay/rabacus
- name: Fortran 90
  role: Language of the OpenMP-enabled core routines wrapped into Python, giving execution
    speeds thousands of times faster than pure Python
  repo: https://github.com/galtay/rabacus
- name: Quantities
  role: Supplies units for all halo/profile variables so results carry consistent
    physical dimensionality
  repo: https://github.com/python-quantities/python-quantities
- name: Python
  role: Driver language used to build the NFW profile, call the solver, and collect
    profiles into an output file
- name: numpy f2py
  role: Wraps the Fortran 90 core into importable Python routines
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:9401d0b403f7f7de6b79fcb3b2d004881fbac6d59f85f8dbfddd0ce412d9ad53
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Fortran 90
  - Python
  - Quantities
  - numpy
  - numpy f2py
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

# spherical-isotropic-background-radiative-transfer

## Summary

Computes equilibrium ionization and temperature profiles for a spherically symmetric gaseous halo immersed in a homogeneous isotropic UV background (e.g. HM12) using rabacus's Sphere_Bgnd solver, including hydrogen/helium ionization states, equilibrium temperatures, and recombination radiation transfer.

## When to use

Use when a task needs radial ionization/temperature structure of a cosmological gaseous halo (e.g. an NFW profile) illuminated by a meta-galactic background rather than a central point source, and multiple background/temperature-treatment models must be compared.

## When NOT to use

- The geometry is a semi-infinite slab in an isotropic background or a sphere with a central point source — those are distinct rabacus cases, not Sphere_Bgnd
- A non-equilibrium (time-dependent) ionization solution is needed — the paper states non-equilibrium solvers are a planned future improvement
- Metal line cooling must be modeled — the article notes metal line cooling is not yet included

## Inputs

- Spherically symmetric density profile (e.g. NFW halo with virial mass, overdensity Δv=200, concentration from the c0=3.5 relation), with Quantities units
- Meta-galactic UV background definition, e.g. the HM12 (Haardt & Madau 2012) photorates table via ra.HM12_Photorates_Table
- Choice among the five benchmark models: grey_fixT_B, hm12_fixT_B, hm12_evoT_B, hm12_evoT_A, hm12_evoT_rt

## Outputs

- Radial hydrogen and helium ionization-fraction profiles for each background model
- Equilibrium (or fixed) temperature profiles per model
- Solution including transfer of recombination radiation (where the model treats it, e.g. hm12_evoT_rt)
- Aggregated profile table, e.g. halo_ionization_profiles.h5

## How to apply

Set export F90=gfortran and install the OpenMP-enabled rabacus Fortran 90 core via pip (wrapped with f2py); express all inputs with Quantities units. Build the halo density profile — for the paper's example, an NFW profile for Mvir=1e10 Msun with overdensity Δv=200 at z=0 using the c0=3.5 concentration relation (Eqs. 41-43). Initialize the HM12 background via ra.HM12_Photorates_Table and validate it against the documented He I photo-heating rate of 3.39163517433e-12 eV/s at z=3.0 before running any solver. Then run the Sphere_Bgnd solver for the paper's five models — grey_fixT_B, hm12_fixT_B, hm12_evoT_B, hm12_evoT_A, and hm12_evoT_rt — which span grey vs HM12 spectra, fixed vs equilibrium temperature evolution, and on-the-spot vs full transfer of recombination radiation. Collect the radial ionization-fraction and temperature profiles for all five models and persist them (e.g. to an HDF5 file such as halo_ionization_profiles.h5) with consistent physical units.

## Related tools

- **rabacus** (Provides the Sphere_Bgnd solver computing hydrogen/helium ionization, equilibrium temperatures, and recombination radiation for a sphere in an isotropic background) — https://bitbucket.org/galtay/rabacus
- **Fortran 90** (Language of the OpenMP-enabled core routines wrapped into Python, giving execution speeds thousands of times faster than pure Python) — https://github.com/galtay/rabacus
- **Quantities** (Supplies units for all halo/profile variables so results carry consistent physical dimensionality) — https://github.com/python-quantities/python-quantities
- **Python** (Driver language used to build the NFW profile, call the solver, and collect profiles into an output file)
- **numpy f2py** (Wraps the Fortran 90 core into importable Python routines)

## Examples

```
export F90=gfortran; pip install rabacus; then in Python: import rabacus as ra; hm12 = ra.HM12_Photorates_Table(); assert abs(hm12.He1h.thin(3.0) - 3.39163517433e-12) < 1e-20
```

## Evaluation signals

- ra.HM12_Photorates_Table reproduces the documented He1h(z=3.0) photo-heating rate of 3.39163517433e-12 eV/s
- Output file contains ionization and temperature profiles for all five background models (grey_fixT_B, hm12_fixT_B, hm12_evoT_B, hm12_evoT_A, hm12_evoT_rt)
- All profile quantities carry consistent physical units from the Quantities package
- NFW profile parameters match the intended halo (Mvir=1e10 Msun, Δv=200, z=0, c0=3.5 concentration relation)
- rabacus imports successfully after pip install, confirming the OpenMP-enabled build

## Limitations

- Metal line cooling is not yet included (planned future improvement)
- Non-equilibrium solvers are not yet included (planned future improvement)
- Requires a Fortran compiler in the path and the F90 environment variable for OpenMP-enabled compilation; other compilers need the manual setup.py install route

## Evidence

- [methods] install verification: [evidence span withheld — anchor status: not_found]
