---
name: cross-code-profile-interpolation-and-difference-analysis
description: Use when two codes have produced radial profiles (x_HII, x_HeII, x_HeIII,
  T) for the same well-posed problem and you must quantify agreement, e.g. validating
  rabacus against Cloudy 13.03 reference profiles.
when_to_use_negative:
- The problem is not one of rabacus's supported geometries — a semi-infinite slab
  in an isotropic background, a spherically symmetric gas with a point source at the
  center, or a spherically symmetric gas in an isotropic background.
- 'You need non-equilibrium or metal-line-cooling physics: the paper states these
  solvers are planned future improvements and not yet included.'
- The two codes were not run on the same physical setup (source spectrum, band truncation,
  luminosity, and temperature-evolution toggle must match), making pointwise differences
  meaningless.
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: rabacus
  role: 'Primary solver: computes equilibrium hydrogen/helium ionization fractions
    and temperature profiles for the spherical point-source geometry, executed via
    its Fortran 90 core with OpenMP'
  repo: https://bitbucket.org/galtay/rabacus
- name: Quantities
  role: Unit handling for all profile variables so interpolated values and relative
    differences are computed on dimensionally consistent quantities
  repo: https://github.com/python-quantities/python-quantities
- name: Numpy f2py
  role: Wraps the Fortran 90 core routines in Python so the solver is callable from
    the comparison script
- name: Cloudy 13.03
  role: Reference code supplying the benchmark radial profiles against which rabacus
    differences are computed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:588e958784235e885d2fccde3fad71317516d8c9d39b0f5165060f9eecc8a1a8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cloudy 13.03
  - Numpy f2py
  - Quantities
  - rabacus
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Cross-code profile interpolation and difference analysis

## Summary

A validation skill that compares radial ionization-fraction and temperature profiles from two radiative-transfer codes (e.g. rabacus and Cloudy 13.03) by interpolating one solver's outputs onto the other's radial grid and computing pointwise relative differences, as done for the Iliev et al. (2006) Test 2 sphere.

## When to use

Use when two codes have produced radial profiles (x_HII, x_HeII, x_HeIII, T) for the same well-posed problem and you must quantify agreement, e.g. validating rabacus against Cloudy 13.03 reference profiles.

## When NOT to use

- The problem is not one of rabacus's supported geometries — a semi-infinite slab in an isotropic background, a spherically symmetric gas with a point source at the center, or a spherically symmetric gas in an isotropic background.
- You need non-equilibrium or metal-line-cooling physics: the paper states these solvers are planned future improvements and not yet included.
- The two codes were not run on the same physical setup (source spectrum, band truncation, luminosity, and temperature-evolution toggle must match), making pointwise differences meaningless.

## Inputs

- radial ionization-fraction (x_HI, x_HII, x_HeI, x_HeII, x_HeIII) and temperature profiles from rabacus for the Test 2 sphere (T_eff=1e5 K blackbody, 1–10 Ryd, L_n=5e48 s^-1, temperature evolution on)
- Cloudy 13.03 reference profiles for the identical problem setup (radial grid + ionization/temperature values)

## Outputs

- test2_sphere_comparison.csv with aligned (interpolated) profiles on the Cloudy radial grid
- pointwise relative-difference series for each ion species and for temperature

## How to apply

Run the primary solver (rabacus, whose Fortran 90 core wrapped in Python solves the spherical gas distribution with a central point source) configured to the benchmark setup — here a T_eff = 1e5 K blackbody truncated to 1–10 Rydbergs with L_n = 5e48 photons/s and self-consistent equilibrium temperature evolution enabled. Load the reference code's profiles (Cloudy 13.03) for the identical problem. Interpolate the primary profiles onto the reference code's radial grid so both are sampled at identical radii, keeping all variables unit-aware via the Quantities package. Compute pointwise relative differences for each ion species (H, He) and for temperature. Interpret results in light of the known inner-region deviation: relative differences should be on the order of a few percent except in the innermost quarter kpc where x_HI approaches the cloudy ionization-fraction deviation δ_H. Write the aligned profiles and relative differences to a CSV (e.g. test2_sphere_comparison.csv).

## Related tools

- **rabacus** (Primary solver: computes equilibrium hydrogen/helium ionization fractions and temperature profiles for the spherical point-source geometry, executed via its Fortran 90 core with OpenMP) — https://bitbucket.org/galtay/rabacus
- **Quantities** (Unit handling for all profile variables so interpolated values and relative differences are computed on dimensionally consistent quantities) — https://github.com/python-quantities/python-quantities
- **Numpy f2py** (Wraps the Fortran 90 core routines in Python so the solver is callable from the comparison script)
- **Cloudy 13.03** (Reference code supplying the benchmark radial profiles against which rabacus differences are computed)

## Examples

```
import rabacus as ra; import quantities as pq; import numpy as np  # then instantiate the spherical point-source solver and interpolate onto the Cloudy radial grid
```

## Evaluation signals

- Pointwise relative differences in ionization fractions and temperature remain on the order of a few percent outside the innermost quarter kpc, matching the paper's reported agreement.
- Near-radial-center behavior is understood: x_HI relative difference approaches the cloudy ionization-fraction deviation δ_H within the innermost ~0.25 kpc and is flagged, not treated as failure.
- Rabacus profiles converge (radial hydrogen/helium ionization and temperature solve completes) before interpolation.
- All profile arrays carry units through the Quantities package, and interpolation lands exactly on the Cloudy radial grid points (alignment check on radii).
- The output test2_sphere_comparison.csv contains one relative-difference column per ion species plus temperature, each finite and physically bounded (0 ≤ x ≤ 1).

## Limitations

- Metal line cooling is not yet included in rabacus (planned future improvement), which can affect equilibrium temperature profiles.
- Non-equilibrium solvers are not yet included (planned future improvement); comparisons are limited to equilibrium solutions.
- The Quantities README notes test coverage is incomplete and the package 'is not suggested for mission-critical applications'.
- Innermost-quarter-kpc differences are inherently limited by the cloudy ionization-fraction deviation δ_H and cannot be expected to reach the few-percent level.

## Evidence

- [intro] Scope of the solver being validated: [evidence span withheld — anchor status: not_found]
