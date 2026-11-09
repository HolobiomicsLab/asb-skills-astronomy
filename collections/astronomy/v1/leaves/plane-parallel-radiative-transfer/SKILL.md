---
name: plane-parallel-radiative-transfer
description: Use when you need to verify a plane-parallel hydrogen ionization-front
  solution on a slab, or benchmark a numerical radiative-transfer solver against a
  closed-form column-depth solution for monochromatic irradiation.
when_to_use_negative:
- The problem involves spherical geometry with a central point source or an isotropic
  background rather than plane-parallel radiation on a slab
- The radiation field is broadband (e.g. the HM12 metagalactic background) rather
  than monochromatic, so the closed-form inverse solution does not apply
- Non-equilibrium ionization or metal line cooling is needed — the article notes these
  are planned future improvements not yet included
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: rabacus
  role: Provides the Slab_Pln solver for equilibrium hydrogen ionization in a slab
    irradiated by plane-parallel monochromatic radiation
  repo: https://bitbucket.org/galtay/rabacus
- name: Python
  role: Hosts the rabacus API, the analytic inverse solution (Eqs. 23-25), and the
    CSV comparison script
- name: Quantities
  role: Supplies unit-carrying variables required by all rabacus inputs and outputs
  repo: https://github.com/python-quantities/python-quantities
- name: Fortran 90
  role: Core solver routines wrapped in Python via Numpy f2py, compiled with gfortran
    for OpenMP speed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:798fc0dc1db33d68322bc4764ac6eed64854335c2bc199255fb8d30a0f845c68
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Fortran 90
  - Python
  - Quantities
  - rabacus
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# plane-parallel-radiative-transfer

## Summary

Validates the rabacus Slab_Pln solver, which computes equilibrium ionization for a uniform pure-hydrogen slab illuminated by monochromatic plane-parallel radiation, against a closed-form analytic inverse solution (Eqs. 23-25) expressed as ionization fraction vs. hydrogen column depth.

## When to use

Use when you need to verify a plane-parallel hydrogen ionization-front solution on a slab, or benchmark a numerical radiative-transfer solver against a closed-form column-depth solution for monochromatic irradiation.

## When NOT to use

- The problem involves spherical geometry with a central point source or an isotropic background rather than plane-parallel radiation on a slab
- The radiation field is broadband (e.g. the HM12 metagalactic background) rather than monochromatic, so the closed-form inverse solution does not apply
- Non-equilibrium ionization or metal line cooling is needed — the article notes these are planned future improvements not yet included

## Inputs

- Uniform pure-hydrogen slab density profile (unit-carrying Quantities variables)
- Monochromatic plane-parallel ionizing radiation field (photon rate/flux with units)
- Common hydrogen column-depth grid spanning the ionization front

## Outputs

- Numerical hydrogen ionization-fraction profile from Slab_Pln on the column-depth grid
- Analytic closed-form hydrogen ionization-fraction profile (Eqs. 23-25) on the same grid
- Pointwise relative-difference profile and its maximum value near the ionization front
- Depth-resolved comparison table written to solution_comparison.csv

## How to apply

Install the prerequisites (scipy, numpy, quantities) with pip, set the F90 environment variable to gfortran, and build an OpenMP-enabled rabacus. In a Python session with unit-carrying variables (via the Quantities package), set up the Slab_Pln problem for a uniform pure-hydrogen slab under monochromatic plane-parallel incident radiation. Independently implement the analytic inverse closed-form solution (Eqs. 23-25), which algebraically inverts the hydrogen ionization balance to give ionization state as a function of column depth. Evaluate both solutions on a common column-depth grid spanning the ionization front and compute the pointwise relative difference between hydrogen ionization-fraction profiles; the maximum difference near the ionization front must stay below 7e-3, two orders of magnitude smaller over most of the slab.

## Related tools

- **rabacus** (Provides the Slab_Pln solver for equilibrium hydrogen ionization in a slab irradiated by plane-parallel monochromatic radiation) — https://bitbucket.org/galtay/rabacus
- **Python** (Hosts the rabacus API, the analytic inverse solution (Eqs. 23-25), and the CSV comparison script)
- **Quantities** (Supplies unit-carrying variables required by all rabacus inputs and outputs) — https://github.com/python-quantities/python-quantities
- **Fortran 90** (Core solver routines wrapped in Python via Numpy f2py, compiled with gfortran for OpenMP speed)

## Examples

```
export F90=gfortran; sudo pip install quantities; sudo pip install rabacus; python -c "import rabacus as ra; sol = ra.Slab_Pln()"
```

## Evaluation signals

- Maximum relative difference between the analytic and Slab_Pln hydrogen ionization-fraction profiles near the ionization front is below 7e-3
- The relative difference is two orders of magnitude smaller than 7e-3 over most of the slab (away from the ionization front)
- `import rabacus as ra` in an ipython session runs without error (quick-test install check)
- Both solutions are evaluated on an identical column-depth grid so the comparison is pointwise
- solution_comparison.csv contains the depth-resolved table and the recorded maximum relative difference

## Limitations

- rabacus does not yet include metal line cooling or non-equilibrium solvers (planned future improvements)
- Test coverage of the Quantities package is incomplete, so it is 'not suggested for mission-critical applications'
- No version number for the rabacus package is stated in the provided source metadata
- The 7e-3 agreement bound applies to the monochromatic plane-parallel slab case; other geometries/spectra require different validation

## Evidence

- [abstract] rabacus, a Python package for calculating the transfer of hydrogen ionizing radiation in simplified geometries relevant to astronomy and cosmology: [evidence span withheld — anchor status: not_found]
- [intro] Fortran 90 core wrapped in Python: [evidence span withheld — anchor status: not_found]
