---
name: hydrogen-ionization-equilibrium-slab-analysis
description: Use when reproducing or validating plane-parallel (slab) hydrogen ionization
  equilibrium solutions against a closed-form analytic solution, or when benchmarking
  the Slab_Pln solver near the ionization front.
when_to_use_negative:
- The problem involves helium, spectral (non-monochromatic) radiation, recombination
  radiation transfer, or equilibrium temperature solving — the closed-form solution
  Eqs. 23-25 covers only monochromatic hydrogen-only slabs, while those features require
  the full rabacus solution machinery
- The geometry is spherically symmetric with a point source or an isotropic background
  — those correspond to rabacus's other solver cases, not the plane-parallel Slab_Pln
  case
- A time-dependent (non-equilibrium) ionization state is needed — rabacus provides
  equilibrium solvers only, and non-equilibrium solvers are explicitly a planned future
  improvement
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3316
tools:
- name: rabacus
  role: Provides the Slab_Pln plane-parallel hydrogen ionization equilibrium solver
    whose output is compared pointwise with the analytic closed-form solution
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: Interpreted host language in which the Slab_Pln problem is set up and the
    analytic inverse solution (Eqs. 23-25) and comparison are implemented
- name: Quantities
  role: Supplies unit-carrying variables required by all rabacus inputs and outputs
    in the slab problem setup
  repo: https://github.com/python-quantities/python-quantities
- name: Fortran 90
  role: Language of the core rabacus routines, compiled with OpenMP directives via
    f2py to give the fast Slab_Pln solver being validated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:f479ac8f124aaa0c62b9180b432b9c5d75738da9279e7a7a864b76bc6d28a435
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Fortran 90
  - Python
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# hydrogen-ionization-equilibrium-slab-analysis

## Summary

Validates the rabacus Slab_Pln plane-parallel radiative transfer solver against the closed-form analytic inverse solution for hydrogen ionization equilibrium in a monochromatically illuminated uniform pure-hydrogen slab, confirming the numerical machinery is correct to better than 7 × 10^-3 relative difference.

## When to use

Use when reproducing or validating plane-parallel (slab) hydrogen ionization equilibrium solutions against a closed-form analytic solution, or when benchmarking the Slab_Pln solver near the ionization front.

## When NOT to use

- The problem involves helium, spectral (non-monochromatic) radiation, recombination radiation transfer, or equilibrium temperature solving — the closed-form solution Eqs. 23-25 covers only monochromatic hydrogen-only slabs, while those features require the full rabacus solution machinery
- The geometry is spherically symmetric with a point source or an isotropic background — those correspond to rabacus's other solver cases, not the plane-parallel Slab_Pln case
- A time-dependent (non-equilibrium) ionization state is needed — rabacus provides equilibrium solvers only, and non-equilibrium solvers are explicitly a planned future improvement

## Inputs

- Uniform pure-hydrogen slab definition (density, thickness)
- Monochromatic plane-parallel incident ionizing radiation field (unit-carrying Quantities variables)
- Common column-depth grid spanning the ionization front

## Outputs

- Hydrogen ionization-fraction profiles from rabacus Slab_Pln and from the analytic inverse solution (Eqs. 23-25)
- Depth-resolved comparison table written to solution_comparison.csv
- Maximum pointwise relative difference between the two profiles near the ionization front

## How to apply

Build an OpenMP-enabled rabacus by setting the F90 environment variable (e.g. export F90=gfortran) and installing prerequisites (scipy, numpy, quantities) via pip; verify the install with 'import rabacus as ra'. Set up the Slab_Pln problem for a uniform pure-hydrogen slab illuminated by monochromatic plane-parallel radiation, using unit-carrying Quantities variables as rabacus requires. Independently implement the analytic inverse closed-form solution (Eqs. 23-25), which algebraically inverts the hydrogen ionization balance to give the ionization fraction as a function of column depth through the slab. Evaluate both the analytic and numerical hydrogen ionization-fraction profiles on a common column-depth grid spanning the ionization front, then compute the pointwise relative difference and record its maximum near the ionization front. Write the depth-resolved comparison table and maximum relative difference to solution_comparison.csv.

## Related tools

- **rabacus** (Provides the Slab_Pln plane-parallel hydrogen ionization equilibrium solver whose output is compared pointwise with the analytic closed-form solution) — https://bitbucket.org/galtay/rabacus
- **Python** (Interpreted host language in which the Slab_Pln problem is set up and the analytic inverse solution (Eqs. 23-25) and comparison are implemented)
- **Quantities** (Supplies unit-carrying variables required by all rabacus inputs and outputs in the slab problem setup) — https://github.com/python-quantities/python-quantities
- **Fortran 90** (Language of the core rabacus routines, compiled with OpenMP directives via f2py to give the fast Slab_Pln solver being validated)

## Examples

```
export F90=gfortran; sudo pip install scipy numpy quantities rabacus; python -c "import rabacus as ra"
```

## Evaluation signals

- The maximum relative difference between the analytic and Slab_Pln hydrogen ionization-fraction profiles near the ionization front must be below 7e-3
- Over most of the slab (away from the ionization front) the relative difference should be two orders of magnitude smaller than the peak
- The quick-test import 'import rabacus as ra' must run without error after installation
- The comparison output solution_comparison.csv must contain the depth-resolved table plus the recorded maximum relative difference
- Both profiles must be evaluated on the same column-depth grid spanning the ionization front for the pointwise comparison to be valid

## Limitations

- The agreement bound (max relative difference < 7 × 10^-3) holds specifically near the ionization front; deviations concentrate there and are smaller elsewhere
- The validation covers only the monochromatic, pure-hydrogen, plane-parallel case — the closed-form inverse solution does not extend to helium, broadband spectra, or recombination radiation
- Non-equilibrium solvers and metal line cooling are not included in rabacus (planned future improvements)
- Building the OpenMP-enabled Fortran core requires a Fortran 90 compiler in the path and the F90 environment variable set correctly; other compilers require the manual setup.py install path

## Evidence

- [results] The maximum relative and absolute difference between the rabacus solution and the closed-form solution occurs near the ionization front but is always less than 7 × 10^-3: [evidence span withheld — anchor status: not_found]
