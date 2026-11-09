---
name: born-limit-phase-space-integration
description: Use when an ab-initio n<->p conversion-rate table for BBN freeze-out
  must be recomputed or validated beyond the Born approximation, e.g. regenerating
  PRyMordial's PRyM_evalnTOp rates.
when_to_use_negative:
- The analysis uses a fixed neutron lifetime or pre-stored rate tables and does not
  require recomputing n<->p rates from first principles.
- The question concerns BBN light-element abundance networks or N_eff thermal-bath
  computations rather than the electroweak rate integrals themselves.
- Rates outside the standard BBN temperature range or non-standard charged-current
  physics not covered by the PRIMAT Appendix B recipe are needed.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: PRyMordial
  role: Hosts the dedicated module PRyM_evalnTOp.py that implements (or pre-stores)
    the corrected n<->p weak-rate computation and writes n_to_p_weak_rates.csv
  repo: github.com/vallima/PRyMordial
- name: Python 3
  role: Language in which the Born-limit integrand and its corrections are coded and
    numerically integrated with adaptive quadrature
- name: PRIMAT
  role: Reference prescription (Appendix B) whose state-of-the-art electroweak-rate
    recipes beyond the Born approximation are followed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:043b4b28e822bf62c38a61e99ee2cb491458a687021420575b20495f1b955b60
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRIMAT
  - PRyMordial
  - Python 3
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# born-limit-phase-space-integration

## Summary

Reconstructs the ab-initio neutron–proton weak interconversion rates used for neutron freeze-out in PRyMordial, starting from the Eq. (13) Born-limit integrand and augmenting it with QED radiative corrections, finite-nucleon-mass/weak-magnetism terms, and finite-temperature corrections following PRIMAT Appendix B.

## When to use

Use when an ab-initio n<->p conversion-rate table for BBN freeze-out must be recomputed or validated beyond the Born approximation, e.g. regenerating PRyMordial's PRyM_evalnTOp rates.

## When NOT to use

- The analysis uses a fixed neutron lifetime or pre-stored rate tables and does not require recomputing n<->p rates from first principles.
- The question concerns BBN light-element abundance networks or N_eff thermal-bath computations rather than the electroweak rate integrals themselves.
- Rates outside the standard BBN temperature range or non-standard charged-current physics not covered by the PRIMAT Appendix B recipe are needed.

## Inputs

- Eq. (13) Born-limit rate integrand definition (electron and neutrino phase-space factors, standard weak matrix element)
- PRIMAT Appendix B (Ref. [47]) prescriptions for QED radiative corrections, finite-nucleon-mass/weak-magnetism, and finite-temperature effects
- Grid of T_gamma/T_nu values spanning the BBN temperature range
- Neutron lifetime or modified Fermi constant (normalization choice)

## Outputs

- n_to_p_weak_rates.csv: tabulated corrected n<->p weak rates on a grid of T_gamma/T_nu values
- Interconversion rates usable as ab-initio input to the BBN freeze-out network in PRyMordial

## How to apply

Implement the Eq. (13) Born-limit integrand for neutron-to-proton and proton-to-neutron interconversion in Python 3, including the electron and neutrino phase-space factors and the standard weak charged-current matrix element, and integrate the rate integrals over the lepton energy variables with adaptive quadrature. Then sequentially augment the integrand with (i) QED radiative corrections, (ii) the finite-nucleon-mass recoil and weak-magnetism terms prescribed by PRIMAT Appendix B (Ref. [47]), and (iii) finite-temperature corrections to the rate integrals. Evaluate the corrected rates on a grid of T_gamma/T_nu values and write the table to n_to_p_weak_rates.csv, mirroring PRyMordial's dedicated module PRyM_evalnTOp.py (or loading its pre-stored rate files). Normalize the computed rates either via the measured neutron lifetime or via the modified Fermi constant, and validate qualitative consistency with the PRIMAT Appendix B prescription rather than a numeric benchmark.

## Related tools

- **PRyMordial** (Hosts the dedicated module PRyM_evalnTOp.py that implements (or pre-stores) the corrected n<->p weak-rate computation and writes n_to_p_weak_rates.csv) — github.com/vallima/PRyMordial
- **Python 3** (Language in which the Born-limit integrand and its corrections are coded and numerically integrated with adaptive quadrature)
- **PRIMAT** (Reference prescription (Appendix B) whose state-of-the-art electroweak-rate recipes beyond the Born approximation are followed)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- Tabulated rates in n_to_p_weak_rates.csv are smooth and positive across the BBN temperature range
- Rates are physically consistent across the grid of T_gamma/T_nu values
- Corrected rates reproduce the full correction set: QED radiative corrections, finite nucleon-mass/weak-magnetism effects, and finite-temperature effects
- Qualitative consistency with the PRIMAT Appendix B prescription (the paper reports no quantitative reference threshold)
- Rates can be normalized either via the neutron lifetime or the modified Fermi constant without inconsistency

## Limitations

- The paper describes but does not separately benchmark this module; no quantitative reference threshold is reported, so acceptance is qualitative consistency with the PRIMAT Appendix B prescription.
- Rates may be loaded from pre-stored files instead of recomputed, so a user must verify which mode (live evaluation vs. table lookup) is active in PRyM_evalnTOp.py.

## Evidence

- [intro] PRIMAT [47, 48] is an user-friendly Mathematica package containing all the inputs and ingredients for an ab-initio computation: "PRIMAT [47, 48] is an user-friendly Mathematica package containing all the inputs and ingredients for an ab-initio computation"
- [methods] PRyMordial implements the refinements to the Born-level n<->p rates — QED radiative corrections, finite nucleon-mass/weak-magnetism effects, and finite-temperature effects: [evidence span withheld — anchor status: not_found]
- [methods] Code the Eq. (13) Born-limit integrand for neutron-to-proton and proton-to-neutron interconversion in Python 3: [evidence span withheld — anchor status: not_found]
- [readme] A Python package for precise computations of BBN within and beyond the Standard Model.: "A Python package for precise computations of BBN within and beyond the Standard Model."
