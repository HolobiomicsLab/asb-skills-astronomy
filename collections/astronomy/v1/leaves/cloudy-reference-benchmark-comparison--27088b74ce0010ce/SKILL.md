---
name: cloudy-reference-benchmark-comparison
description: Use when an analytic radiative-transfer code's ionization or temperature
  profiles must be validated against a published Cloudy benchmark case (e.g. a 1e5
  K blackbody, case A recombination, evolving-temperature sphere) with quantitative
  agreement thresholds.
when_to_use_negative:
- The problem requires non-equilibrium (time-dependent) ionization solvers or metal
  line cooling, which rabacus does not yet provide (planned future improvements)
- The geometry or radiation field is outside rabacus's three supported cases (semi-infinite
  slab in isotropic background, sphere with central point source, sphere in isotropic
  background), so no comparable Cloudy benchmark can be set up
- You need a code-to-code agreement metric other than profile comparison — this skill
  specifically compares radial ionization/temperature profiles against precomputed
  Cloudy 13.03 output
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3360
tools:
- name: rabacus
  role: Solves the equilibrium ionization/temperature problem for the spherical constant-density
    gas with a central point source, producing radial profiles of xH1, xH2, xHe1,
    xHe2, xHe3 and temperature
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: Host language for the rabacus API; comparison script loads Cloudy 13.03 profiles,
    computes differences, and writes cloudy_comparison.csv
  repo: https://bitbucket.org/galtay/rabacus
- name: Fortran 90
  role: Core routines of rabacus, wrapped in Python via f2py; must be built OpenMP-enabled
    (export F90=gfortran) for the solver
  repo: https://bitbucket.org/galtay/rabacus
- name: Quantities
  role: Provides unit-bearing arrays; all model parameters (20 kpc radius, densities)
    must be passed to rabacus as Quantities
  repo: https://github.com/python-quantities/python-quantities
- name: Numpy f2py
  role: Wraps the Fortran 90 core into the importable Python package (import rabacus
    as ra)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:7d2400e7e8a01ca844bc672b88516516a53d053cba28242f15235aff7414dc58
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Fortran 90
  - Numpy f2py
  - Python
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cloudy-reference-benchmark-comparison

## Summary

Benchmark rabacus equilibrium ionization/temperature solutions against Cloudy 13.03 reference profiles, e.g. the Friedrich et al. (2012) Test 1A 20 kpc helium sphere, by computing radius-by-radius relative and absolute differences in hydrogen and helium ionization fractions and temperature.

## When to use

Use when an analytic radiative-transfer code's ionization or temperature profiles must be validated against a published Cloudy benchmark case (e.g. a 1e5 K blackbody, case A recombination, evolving-temperature sphere) with quantitative agreement thresholds.

## When NOT to use

- The problem requires non-equilibrium (time-dependent) ionization solvers or metal line cooling, which rabacus does not yet provide (planned future improvements)
- The geometry or radiation field is outside rabacus's three supported cases (semi-infinite slab in isotropic background, sphere with central point source, sphere in isotropic background), so no comparable Cloudy benchmark can be set up
- You need a code-to-code agreement metric other than profile comparison — this skill specifically compares radial ionization/temperature profiles against precomputed Cloudy 13.03 output

## Inputs

- Cloudy 13.03 reference radial profiles of ionization fractions (xH1, xH2, xHe1, xHe2, xHe3) and temperature
- Benchmark problem specification (Friedrich et al. 2012 Test 1A): 20 kpc helium sphere, nH=1e-3 cm^-3, nHe=8.7e-5 cm^-3, 1e5 K blackbody spectrum
- rabacus solver output object containing radial profiles of xH1, xH2, xHe1, xHe2, xHe3 and temperature

## Outputs

- cloudy_comparison.csv with radius-by-radius relative differences in neutral fractions (xH1, xHe1) and absolute differences in ionization fractions
- Accept/reject verdict based on the 5% relative and 0.02 absolute agreement thresholds

## How to apply

Set up the benchmark geometry in rabacus as a spherical constant-density model with a central point source (radius 20 kpc, nH=1e-3 cm^-3, nHe=8.7e-5 cm^-3, 1e5 K blackbody spectrum), passing all values as Quantities unit-bearing arrays. Solve the coupled ionization/temperature equilibrium with case A recombination rates and self-consistent (evolving) temperature, transporting recombination radiation rather than applying the on-the-spot approximation. Extract radial profiles of xH1, xH2, xHe1, xHe2, xHe3 and temperature from the solver output object. Load the Cloudy 13.03 reference profiles and compute relative differences in the neutral fractions (xH1, xHe1) and absolute differences in the ionization fractions as a function of radius, then write the radius-by-radius table to cloudy_comparison.csv. Accept the run if neutral H/He fractions agree within 5% relative difference at large radii and all ionization fractions agree within 0.02 absolute difference.

## Related tools

- **rabacus** (Solves the equilibrium ionization/temperature problem for the spherical constant-density gas with a central point source, producing radial profiles of xH1, xH2, xHe1, xHe2, xHe3 and temperature) — https://bitbucket.org/galtay/rabacus
- **Python** (Host language for the rabacus API; comparison script loads Cloudy 13.03 profiles, computes differences, and writes cloudy_comparison.csv) — https://bitbucket.org/galtay/rabacus
- **Fortran 90** (Core routines of rabacus, wrapped in Python via f2py; must be built OpenMP-enabled (export F90=gfortran) for the solver) — https://bitbucket.org/galtay/rabacus
- **Quantities** (Provides unit-bearing arrays; all model parameters (20 kpc radius, densities) must be passed to rabacus as Quantities) — https://github.com/python-quantities/python-quantities
- **Numpy f2py** (Wraps the Fortran 90 core into the importable Python package (import rabacus as ra))

## Examples

```
export F90=gfortran; pip install rabacus; import rabacus as ra  # then solve sphere w/ central point source and diff vs Cloudy 13.03 profiles
```

## Evaluation signals

- At large radii, relative differences between rabacus and Cloudy in the hydrogen and helium neutral fractions (xH1, xHe1) are always less than 5%
- All ionization fractions (xH2, xHe2, xHe3) agree with Cloudy 13.03 within 0.02 absolute difference across the radius grid
- cloudy_comparison.csv exists and contains a radius-indexed table of relative and absolute differences for all five ionization fractions
- The rabacus import test succeeds (import rabacus as ra in an ipython session), confirming the OpenMP-enabled Fortran build is functional
- Temperature profiles are self-consistently solved (evolving temperature, case A recombination rates, recombination radiation transported), matching the benchmark configuration rather than a fixed-temperature or on-the-spot run

## Limitations

- Metal line cooling is not yet included in rabacus (planned future improvement), so temperature comparisons may diverge where metal cooling matters
- Non-equilibrium solvers are not yet included in rabacus (planned future improvement); only equilibrium ionization/temperature states can be benchmarked
- No version number for the rabacus package is stated in the article, so the exact benchmarked version must be confirmed from the repository
- Agreement is only asserted for neutral fractions at large radii (<5% relative); near the ionization front or at small radii differences may exceed thresholds

## Evidence
