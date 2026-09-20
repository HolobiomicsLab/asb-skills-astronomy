---
name: hm12-uv-background-initialization
description: Use when a radiative-transfer run needs a homogeneous isotropic UV background
  (e.g. grey_fixT_B, hm12_fixT_B, hm12_evoT_B/A/rt models) and you must build and
  verify the HM12 photorates table before feeding the solvers.
when_to_use_negative:
- The problem requires a point-source radiation field rather than a homogeneous isotropic
  background (use the spherical point-source case instead)
- Metal line cooling or non-equilibrium ionization are required, since rabacus does
  not yet include them
- A custom or time-evolving radiation spectrum not represented by HM12 or the grey
  background is needed
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: rabacus
  role: provides the HM12_Photorates_Table used to initialize the meta-galactic UV
    background photorates
  repo: https://bitbucket.org/galtay/rabacus
- name: Quantities
  role: attaches physical units to all photorate values so they interoperate with
    solver inputs
  repo: https://github.com/python-quantities/python-quantities
- name: Python
  role: environment in which rabacus and the HM12 table are imported and queried
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:8cca5a5b91914cde4186cf564a298e510833764b4e679f87e964d3dda4eb5880
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - Quantities
  - numpy
  - rabacus
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# hm12-uv-background-initialization

## Summary

Initializes and validates the Haardt & Madau 2012 (HM12) meta-galactic UV radiation background photorates table from rabacus, which supplies the redshift-dependent photo-ionization and photo-heating rates needed by the slab and Sphere_Bgnd solvers.

## When to use

Use when a radiative-transfer run needs a homogeneous isotropic UV background (e.g. grey_fixT_B, hm12_fixT_B, hm12_evoT_B/A/rt models) and you must build and verify the HM12 photorates table before feeding the solvers.

## When NOT to use

- The problem requires a point-source radiation field rather than a homogeneous isotropic background (use the spherical point-source case instead)
- Metal line cooling or non-equilibrium ionization are required, since rabacus does not yet include them
- A custom or time-evolving radiation spectrum not represented by HM12 or the grey background is needed

## Inputs

- Target redshift (e.g. z=3.0) at which photorates are needed
- Rabacus installation with the OpenMP-enabled Fortran 90 core (F90 compiler set, e.g. export F90=gfortran, then pip install rabacus)
- Ionizing species of interest for rate checks (e.g. He I photo-heating rate)

## Outputs

- ra.HM12_Photorates_Table object exposing redshift-dependent photo-ionization and photo-heating rates for H I, He I, and He II as Quantities-carrying values
- Validated He1h(z=3.0) photo-heating rate of 3.39163517433e-12 eV/s confirming table integrity

## How to apply

Import rabacus as ra and instantiate the HM12 meta-galactic radiation background object via ra.HM12_Photorates_Table, which gives access to redshift-dependent photo-ionization (H1h, He1h, He2h) and photo-heating rates. Before using the table in any solver run, validate it by asking for the He I photo-heating rate at a redshift of 3.0 and confirming it reproduces the documented value of 3.39163517433e-12 eV/s; only accept the background model if this check passes. All rates and quantities handled by rabacus carry physical units through the Quantities package, so express subsequent inputs (halo density profiles, redshifts) with units rather than bare floats. Once validated, pass the table to the five background models (grey_fixT_B, hm12_fixT_B, hm12_evoT_B, hm12_evoT_A, hm12_evoT_rt) when running the slab or Sphere_Bgnd solvers.

## Related tools

- **rabacus** (provides the HM12_Photorates_Table used to initialize the meta-galactic UV background photorates) — https://bitbucket.org/galtay/rabacus
- **Quantities** (attaches physical units to all photorate values so they interoperate with solver inputs) — https://github.com/python-quantities/python-quantities
- **Python** (environment in which rabacus and the HM12 table are imported and queried)

## Examples

```
import rabacus as ra; hm12 = ra.HM12_Photorates_Table(); print(hm12.He1h(3.0))  # expect 3.39163517433e-12 eV/s
```

## Evaluation signals

- The He1h photo-heating rate at z=3.0 returned by the HM12 table equals 3.39163517433e-12 eV/s exactly
- The table object instantiates without import errors after 'import rabacus as ra', confirming a working OpenMP-enabled Fortran build
- All returned rates carry Quantities units (e.g. eV/s) rather than bare floats
- The validated table feeds all five background models (grey_fixT_B, hm12_fixT_B, hm12_evoT_B, hm12_evoT_A, hm12_evoT_rt) without unit-conversion errors in downstream solver runs

## Limitations

- HM12 is fixed to the Haardt & Madau 2012 spectrum; other background shapes are not covered by this table
- Rabacus solutions are equilibrium ionization/temperature states only; non-equilibrium solvers and metal line cooling are planned future improvements and not available
- The Fortran 90 core must be compiled with the F90 environment variable set correctly (gfortran or ifort); other compilers require a manual setup.py build

## Evidence
