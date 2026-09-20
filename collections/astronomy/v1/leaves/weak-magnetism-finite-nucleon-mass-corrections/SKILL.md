---
name: weak-magnetism-finite-nucleon-mass-corrections
description: Use when computing ab-initio n<->p weak interconversion rates beyond
  the Born approximation for neutron freeze-out or BBN helium-4 predictions, where
  Born-level rates are insufficient for precision studies.
when_to_use_negative:
- When only an order-of-magnitude estimate of the n<->p rates is needed and Born-level
  accuracy is sufficient
- When the analysis targets thermoprocesses other than the charged-current n<->p weak
  rates (e.g. thermonuclear reaction rates or the thermal bath itself), which PRyMordial
  handles in other modules
tools:
- name: PRyMordial
  role: Python 3 package whose PRyM_evalnTOp.py module computes the corrected weak
    n<->p rates (or loads pre-stored tables) with QED, finite-nucleon-mass/weak-magnetism,
    and finite-temperature corrections
  repo: github.com/vallima/PRyMordial
- name: PRIMAT
  role: Reference treatment (Appendix B of Ref. [47]) whose electroweak rate recipes
    beyond the Born approximation PRyMordial follows
- name: Python 3
  role: Language used to code the Born-limit integrand and perform adaptive quadrature
    integration of the rate integrals
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:e022945fe9939dd49a66837aeb93f6f31cbf5c6103c4d2fec89c86cf470c7308
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRIMAT
  - PRyMordial
  - Python 3
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# weak-magnetism-finite-nucleon-mass-corrections

## Summary

Refines the Born-level neutron-to-proton weak conversion rates used in BBN freeze-out by adding QED radiative corrections, finite nucleon-mass recoil and weak-magnetism terms, and finite-temperature effects, following the PRIMAT Appendix B prescription as implemented in PRyMordial's PRyM_evalnTOp.py module.

## When to use

Use when computing ab-initio n<->p weak interconversion rates beyond the Born approximation for neutron freeze-out or BBN helium-4 predictions, where Born-level rates are insufficient for precision studies.

## When NOT to use

- When only an order-of-magnitude estimate of the n<->p rates is needed and Born-level accuracy is sufficient
- When the analysis targets thermoprocesses other than the charged-current n<->p weak rates (e.g. thermonuclear reaction rates or the thermal bath itself), which PRyMordial handles in other modules

## Inputs

- Born-limit n<->p rate integrand (Eq. 13) with electron and neutrino phase-space factors
- Grid of T_gamma/T_nu values spanning the BBN temperature range
- Normalization choice: neutron lifetime or modified Fermi constant

## Outputs

- Tabulated corrected weak n<->p rates written to n_to_p_weak_rates.csv
- Rate values computed in PRyM_evalnTOp.py (or loaded from pre-stored files) for use in freeze-out/BBN abundance runs

## How to apply

First code the Eq. (13) Born-limit integrand for neutron-to-proton and proton-to-neutron interconversion with the electron and neutrino phase-space factors and the standard weak matrix element in Python 3, and integrate it over the lepton energy variables with adaptive quadrature. Then add the QED radiative corrections to the charged-current integrand, followed by the finite-nucleon-mass recoil and weak-magnetism terms prescribed by PRIMAT Appendix B, and finally the finite-temperature corrections. Evaluate the corrected rates on a grid of T_gamma/T_nu values across the BBN temperature range and write the table to n_to_p_weak_rates.csv. Rates can be normalized either via the neutron lifetime or the modified Fermi constant; PRyMordial either computes them in PRyM_evalnTOp.py or loads them from pre-stored files.

## Related tools

- **PRyMordial** (Python 3 package whose PRyM_evalnTOp.py module computes the corrected weak n<->p rates (or loads pre-stored tables) with QED, finite-nucleon-mass/weak-magnetism, and finite-temperature corrections) — github.com/vallima/PRyMordial
- **PRIMAT** (Reference treatment (Appendix B of Ref. [47]) whose electroweak rate recipes beyond the Born approximation PRyMordial follows)
- **Python 3** (Language used to code the Born-limit integrand and perform adaptive quadrature integration of the rate integrals)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- Tabulated rates are smooth, positive, and physically consistent across the BBN temperature range
- Corrected rates exceed the Born-limit Eq. (13) integrand results by the expected QED radiative, recoil/weak-magnetism, and finite-temperature increments
- Rate table in n_to_p_weak_rates.csv is defined on the full grid of T_gamma/T_nu values with no gaps or discontinuities
- Normalization via the neutron lifetime and via the modified Fermi constant yields mutually consistent rates
- Acceptance is qualitative consistency with the PRIMAT Appendix B prescription, since the paper describes but does not separately benchmark this module

## Limitations

- The PRyMordial paper describes but does not separately benchmark the PRyM_evalnTOp.py module, so no quantitative reference threshold is reported and validation is qualitative
- Rates must either be computed or loaded from pre-stored files, so workflows depending on freshly computed tables need the module evaluated before use

## Evidence

- [other] PRyMordial implements the refinements to the Born-level n<->p rates — QED radiative corrections, finite nucleon-mass/weak-magnetism effects, and finite-temperature effects: [evidence span withheld — anchor status: not_found]
- [other] rates computed in PRyM_evalnTOp.py and normalization options: [evidence span withheld — anchor status: not_found]
- [intro] PRIMAT as the source of the ab-initio rate ingredients: "PRIMAT [47, 48] is an user-friendly Mathematica package containing all the inputs and ingredients for an ab-initio computation"
- [readme] PRyMordial scope statement: "A Python package for precise computations of BBN within and beyond the Standard Model."
