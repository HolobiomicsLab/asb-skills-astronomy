---
name: units-aware-scientific-computing
description: Use when writing or running Python scientific code (e.g. rabacus solvers)
  whose inputs and outputs are physical quantities (densities, rates, temperatures)
  and unit consistency must be enforced.
when_to_use_negative:
- The computation is pure unitless linear algebra or machine-learning where dimensioned
  arithmetic adds overhead without benefit
- The underlying library does not support Quantities objects and cannot be wrapped
  so that units survive the call (e.g. code expecting bare numpy floats in its hot
  loop)
tools:
- name: Quantities
  role: Provides physical units (magnitude, dimensionality, uncertainty) for all rabacus
    variables; builds on numpy and works with numpy ufuncs
  repo: https://github.com/python-quantities/python-quantities
- name: rabacus
  role: Scientific package whose variables all carry units via Quantities, demonstrating
    the units-aware pattern in cosmological radiative transfer solvers
  repo: https://bitbucket.org/galtay/rabacus
- name: numpy
  role: Base numerical library on which Quantities is built; prerequisite installed
    with pip before rabacus
provenance:
  source_task_ids:
  - task_008
  - task_001
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.004
    title: 'Rabacus: A Python package for analytic cosmological radiative transfer
      calculations'
  source_content_hash: sha256:19a29a533fa7384cc66e0dc62284ecea2aecabcf93396102d463a7c6ea3b1d01
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/python-quantities/python-quantities
  tools:
  - Quantities
  - SciPy
  - numpy
  - rabacus
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# units-aware-scientific-computing

## Summary

Attach physical units to every numerical variable in a scientific Python workflow using the Quantities package (pq), as rabacus does for its radiative-transfer variables, so unit errors are caught by dimensionality checks rather than silent numeric mistakes.

## When to use

Use when writing or running Python scientific code (e.g. rabacus solvers) whose inputs and outputs are physical quantities (densities, rates, temperatures) and unit consistency must be enforced.

## When NOT to use

- The computation is pure unitless linear algebra or machine-learning where dimensioned arithmetic adds overhead without benefit
- The underlying library does not support Quantities objects and cannot be wrapped so that units survive the call (e.g. code expecting bare numpy floats in its hot loop)

## Inputs

- Numerical variables representing physical quantities (magnitude, dimensionality/units, optionally uncertainty)
- rabacus solver objects and HM12 radiation background tables consumed/produced with unit-carrying variables
- numpy arrays to be wrapped in Quantities objects

## Outputs

- Unit-carrying variables (e.g. ionization fractions, temperatures, column densities) with explicit dimensionality
- ValueError exceptions on dimensionally inconsistent operations, used as a correctness guard

## How to apply

Install the Quantities package with pip (it builds on numpy and supports many ufuncs), then express every physical variable as a Quantities object, e.g. a halo density profile built for an NFW model with virial mass 1e10 Msun or a photo-heating rate read from the HM12_Photorates_Table. Wrap the fast numerical core (rabacus's Fortran 90 routines via f2py) so unit-carrying Python variables pass into and out of the solver. Rely on Quantities' dimensionality checks: arithmetic between incompatible units raises a ValueError instead of silently producing wrong numbers, as in the README example where 'velocity + 3' fails with "Unable to convert between units of 'dimensionless' and 'm/s'". Validate that reported values carry the expected dimensionality, e.g. confirming the documented He I photo-heating rate He1h(z=3.0) = 3.39163517433e-12 eV/s from the HM12 table before accepting solver output.

## Related tools

- **Quantities** (Provides physical units (magnitude, dimensionality, uncertainty) for all rabacus variables; builds on numpy and works with numpy ufuncs) — https://github.com/python-quantities/python-quantities
- **rabacus** (Scientific package whose variables all carry units via Quantities, demonstrating the units-aware pattern in cosmological radiative transfer solvers) — https://bitbucket.org/galtay/rabacus
- **numpy** (Base numerical library on which Quantities is built; prerequisite installed with pip before rabacus)

## Examples

```
import quantities as pq; velocity = (42*pq.metre)/(17*pq.second); print("%.3f %s" % (velocity.magnitude, velocity.dimensionality))
```

## Evaluation signals

- Every input and output variable in the workflow reports a dimensionality (e.g. eV/s for photo-heating rates) when inspected via Quantities' dimensionality attribute
- Incompatible-unit arithmetic raises a ValueError (e.g. adding a dimensionless number to a m/s quantity) rather than returning a number
- Import check passes: 'import rabacus as ra' runs without error after quantities is installed
- The HM12 table reproduces the documented He I photo-heating rate 3.39163517433e-12 eV/s at z=3.0 with correct units
- pip's install output ends with 'Successfully installed rabacus' / 'Cleaning up...' indicating prerequisites including quantities were satisfied

## Limitations

- Quantities' test coverage is incomplete, so the package is 'not suggested for mission-critical applications' per its README
- Unit handling adds overhead relative to bare numpy; the speed advantage of rabacus comes from the Fortran 90 core, not from the Python-level unit layer
- Quantities only enforces dimensionality on operations it supports; not all numpy ufuncs are supported

## Evidence

- [readme] Quantities README: incompatible-unit error example: [evidence span withheld — anchor status: not_found]
