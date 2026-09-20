---
name: primat-correction-prescription-implementation
description: Use when BBN neutron freeze-out or helium-4 abundance modeling requires
  weak n<->p rates beyond the Born approximation, e.g. building or validating a PRyMordial-style
  n_to_p_weak_rates table.
when_to_use_negative:
- When Born-approximation rates suffice (e.g. order-of-magnitude freeze-out estimates)
  — the PRIMAT Appendix B corrections add complexity without changing the qualitative
  answer
- When an existing pre-stored n_to_p_weak_rates table already covers the requested
  T_gamma/T_nu range and physics assumptions — loading it is preferred over recomputation
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3450
- http://edamontology.org/topic_0153
tools:
- name: PRyMordial
  role: Host package; computes the corrected n<->p weak rates in the dedicated module
    PRyM_evalnTOp.py (or loads them from pre-stored files) and feeds them into the
    BBN abundance calculation
  repo: github.com/vallima/PRyMordial
- name: PRIMAT
  role: 'Reference prescription: Appendix B of Ref. [47] combines state-of-the-art
    recipes for electroweak rates beyond the Born approximation that this skill implements'
- name: Python 3
  role: Implementation language for the Born-limit integrand, adaptive quadrature
    of the rate integrals, and generation of n_to_p_weak_rates.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:ae4057b9120d40f1c04dcec2abf386661e36486360f416f90edb1508c10b2da5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRIMAT
  - PRyMordial
  - Python 3
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3450
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# primat-correction-prescription-implementation

## Summary

Implements the ab-initio neutron<->proton weak interconversion rates beyond the Born approximation by coding the Eq. (13) Born-limit phase-space integrand and then layering the corrections prescribed in PRIMAT (Appendix B of Ref. [47]): QED radiative corrections, finite-nucleon-mass recoil/weak-magnetism terms, and finite-temperature effects.

## When to use

Use when BBN neutron freeze-out or helium-4 abundance modeling requires weak n<->p rates beyond the Born approximation, e.g. building or validating a PRyMordial-style n_to_p_weak_rates table.

## When NOT to use

- When Born-approximation rates suffice (e.g. order-of-magnitude freeze-out estimates) — the PRIMAT Appendix B corrections add complexity without changing the qualitative answer
- When an existing pre-stored n_to_p_weak_rates table already covers the requested T_gamma/T_nu range and physics assumptions — loading it is preferred over recomputation

## Inputs

- Grid of T_gamma/T_nu values spanning the BBN temperature range
- Electron and neutrino phase-space variables for the Eq. (13) Born-limit integrand
- Weak matrix element parameters (neutron lifetime or modified Fermi constant for normalization)

## Outputs

- n_to_p_weak_rates.csv tabulating corrected n->p and p->n weak rates vs. T_gamma/T_nu
- PRyM_evalnTOp.py module (or pre-stored rate files) consumable by PRyMordial's BBN integration

## How to apply

Code the Eq. (13) Born-limit integrand for neutron-to-proton and proton-to-neutron interconversion in Python 3, including the electron and neutrino phase-space factors and the standard weak matrix element. Numerically integrate the Born-limit rate integrals over the lepton energy variables with adaptive quadrature (SciPy; optionally Numba/Numba-accelerated or Vegas as recommended by the PRyMordial README). Then, following PRIMAT Appendix B, add the QED radiative corrections to the charged-current integrand, the finite-nucleon-mass recoil and weak-magnetism terms, and the finite-temperature corrections. Evaluate the corrected rates on a grid of T_gamma/T_nu values and write the table to n_to_p_weak_rates.csv; PRyMordial computes these rates in the dedicated module PRyM_evalnTOp.py (or loads them from pre-stored files). Rates can be normalized either via the neutron lifetime or the modified Fermi constant — choose one normalization and state it.

## Related tools

- **PRyMordial** (Host package; computes the corrected n<->p weak rates in the dedicated module PRyM_evalnTOp.py (or loads them from pre-stored files) and feeds them into the BBN abundance calculation) — github.com/vallima/PRyMordial
- **PRIMAT** (Reference prescription: Appendix B of Ref. [47] combines state-of-the-art recipes for electroweak rates beyond the Born approximation that this skill implements)
- **Python 3** (Implementation language for the Born-limit integrand, adaptive quadrature of the rate integrals, and generation of n_to_p_weak_rates.csv)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- Tabulated rates in n_to_p_weak_rates.csv are smooth and positive across the BBN temperature range
- Rates are physically consistent across the T_gamma/T_nu grid, in qualitative agreement with the PRIMAT Appendix B prescription
- Rates computed with the neutron-lifetime normalization agree with those using the modified Fermi constant within the expected precision
- Corrected rates exceed the Eq. (13) Born-limit integrals by an amount consistent with QED radiative, recoil/weak-magnetism, and finite-temperature corrections

## Limitations

- The PRyMordial paper describes but does not separately benchmark this module, so no quantitative reference threshold is reported; acceptance is qualitative consistency with the PRIMAT Appendix B prescription
- Depends on external pre-stored rate files if PRyM_evalnTOp.py is not run, and on optional dependencies (NumPy, SciPy mandatory; Numba, Numdifftools, Vegas recommended) for best performance

## Evidence

- [full_text] PRyMordial implements the refinements to the Born-level n<->p rates — QED radiative corrections, finite nucleon-mass/weak-magnetism effects, and finite-temperature effects: [evidence span withheld — anchor status: not_found]
- [full_text] The rates are computed in the dedicated module PRyM_evalnTOp.py (or loaded from pre-stored files) and can be normalized either via the neutron lifetime or the modified Fermi constant.: [evidence span withheld — anchor status: not_found]
- [intro] PRIMAT [47, 48] is an user-friendly Mathematica package containing all the inputs and ingredients for an ab-initio computation: "PRIMAT [47, 48] is an user-friendly Mathematica package containing all the inputs and ingredients for an ab-initio computation"
- [intro] The package is written and runs entirely with Python 3: "The package is written and runs entirely with Python 3"
- [readme] NumPy (mandatory) – pip install numpy
- SciPy (mandatory) – pip install scipy
- Numba (recommended) – pip install numba: "NumPy (mandatory) – pip install numpy
- SciPy (mandatory) – pip install scipy
- Numba (recommended) – pip install numba"
