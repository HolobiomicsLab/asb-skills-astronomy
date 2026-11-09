---
name: weak-neutron-proton-rate-integration
description: Use when computing precision weak n<->p conversion rates for BBN neutron
  freeze-out where Born-approximation rates are insufficient and QED radiative, weak-magnetism,
  recoil, or finite-temperature corrections are required.
when_to_use_negative:
- When a rough Born-approximation rate suffices and sub-percent rate precision is
  not needed — the full corrected integrand is unnecessary overhead
- When studying BSM physics that modifies the thermal background only (not the weak
  rates), where PRyMordial can instead load pre-stored rate files and the ab-initio
  integration is redundant
- When the target is a codebase other than Python 3/Julia (e.g. the FORTRAN-based
  PArthENoPE or the C++ AlterBBN), where this module's implementation is not directly
  portable
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3452
- http://edamontology.org/topic_0204
tools:
- name: PRyMordial
  role: Host package; implements the n<->p weak-rate computation in the dedicated
    module PRyM_evalnTOp.py, with pre-stored rate tables and neutron-lifetime or modified-Fermi-constant
    normalization options
  repo: https://github.com/vallima/PRyMordial
- name: Python 3
  role: Language in which the Born-limit integrand and its corrections are coded and
    numerically integrated with adaptive quadrature
- name: PRIMAT
  role: 'Reference recipe source: the treatment of QED radiative corrections, finite
    nucleon-mass/weak-magnetism effects, and finite-temperature effects follows Appendix
    B of Ref. [47]'
- name: Julia
  role: Optional acceleration via SciML kit routines for the numerical integration
    of the rate integrals
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:756a7362739054fc0b5d7b6a0c45ba6af8c8364670cbb8069c8260ba56458efe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallima/PRyMordial
  tools:
  - Julia
  - PRIMAT
  - PRyMordial
  - Python 3
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3452
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# weak-neutron-proton-rate-integration

## Summary

Computes the weak neutron<->proton interconversion rates driving neutron freeze-out in BBN beyond the Born approximation, following the PRIMAT (Appendix B of Ref. [47]) treatment of QED radiative corrections, finite nucleon-mass/weak-magnetism, and finite-temperature effects, as implemented in PRyMordial's PRyM_evalnTOp.py module.

## When to use

Use when computing precision weak n<->p conversion rates for BBN neutron freeze-out where Born-approximation rates are insufficient and QED radiative, weak-magnetism, recoil, or finite-temperature corrections are required.

## When NOT to use

- When a rough Born-approximation rate suffices and sub-percent rate precision is not needed — the full corrected integrand is unnecessary overhead
- When studying BSM physics that modifies the thermal background only (not the weak rates), where PRyMordial can instead load pre-stored rate files and the ab-initio integration is redundant
- When the target is a codebase other than Python 3/Julia (e.g. the FORTRAN-based PArthENoPE or the C++ AlterBBN), where this module's implementation is not directly portable

## Inputs

- Grid of T_gamma/T_nu values spanning the BBN temperature range
- Charged-current weak matrix element and Eq. (13) Born-limit integrand definition
- PRIMAT Appendix B prescriptions for radiative, recoil, weak-magnetism, and finite-temperature corrections
- Normalization input: measured neutron lifetime or modified Fermi constant

## Outputs

- Tabulated weak n<->p rates beyond the Born approximation in n_to_p_weak_rates.csv
- Rate table usable as input to the PRyMordial BBN abundance and N_eff computation

## How to apply

First code the Eq. (13) Born-limit integrand for neutron-to-proton and proton-to-neutron interconversion in Python 3, with the electron and neutrino phase-space factors and the standard weak matrix element, and integrate the Born-limit rate integrals over the lepton energy variables using adaptive quadrature. Then sequentially add the corrections: QED radiative corrections to the charged-current integrand, the finite-nucleon-mass recoil and weak-magnetism terms prescribed by PRIMAT Appendix B, and the finite-temperature corrections. Evaluate the corrected rates on a grid of T_gamma/T_nu values and write the table to n_to_p_weak_rates.csv (or load a pre-stored rate file). Finally, normalize the rates either via the neutron lifetime or the modified Fermi constant — this normalization choice is the key decision point for SM precision analyses.

## Related tools

- **PRyMordial** (Host package; implements the n<->p weak-rate computation in the dedicated module PRyM_evalnTOp.py, with pre-stored rate tables and neutron-lifetime or modified-Fermi-constant normalization options) — https://github.com/vallima/PRyMordial
- **Python 3** (Language in which the Born-limit integrand and its corrections are coded and numerically integrated with adaptive quadrature)
- **PRIMAT** (Reference recipe source: the treatment of QED radiative corrections, finite nucleon-mass/weak-magnetism effects, and finite-temperature effects follows Appendix B of Ref. [47])
- **Julia** (Optional acceleration via SciML kit routines for the numerical integration of the rate integrals)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- Tabulated rates in n_to_p_weak_rates.csv are smooth and positive across the BBN temperature range
- Rates are physically consistent across the full grid of T_gamma/T_nu values
- Consistency of the implemented corrections with the PRIMAT Appendix B prescription (qualitative, since the paper does not report a quantitative reference threshold for this module)
- Switching normalization between the neutron lifetime and the modified Fermi constant yields consistent rate normalization
- Born-limit integrand reproduces the Eq. (13) phase-space structure before corrections are added

## Limitations

- The paper describes but does not separately benchmark the PRyM_evalnTOp module, so no quantitative reference threshold is reported and acceptance is qualitative consistency with the PRIMAT Appendix B prescription
- Rates must either be recomputed on a grid of T_gamma/T_nu values (costly) or loaded from pre-stored files; the pre-stored tables may not cover arbitrary thermal-history scenarios
- Full usage quality depends on optional dependencies (Numba, Numdifftools, Vegas, PyJulia/diffeqpy); the code can avoid Numba/Numdifftools and does not use Vegas by default, potentially at reduced performance

## Evidence

- [supplementary] PRyMordial implements the refinements to the Born-level n<->p rates — QED radiative corrections, finite nucleon-mass/weak-magnetism effects, and finite-temperature effects: [evidence span withheld — anchor status: not_found]
- [supplementary] The rates are computed in the dedicated module PRyM_evalnTOp.py (or loaded from pre-stored files) and can be normalized either via the neutron lifetime or the modified Fermi constant.: [evidence span withheld — anchor status: not_found]
- [intro] PRIMAT is an user-friendly Mathematica package containing all the inputs and ingredients for an ab-initio computation: "PRIMAT [47, 48] is an user-friendly Mathematica package containing all the inputs and ingredients for an ab-initio computation"
- [intro] interconnect first-principle thermal background with ab-initio neutron-to-proton conversion calculations: [evidence span withheld — anchor status: not_found]
- [readme] The code can easily avoid dependencies on Numba and Numdifftools and it does not make use of Vegas by default.: "The code can easily avoid dependencies on Numba and Numdifftools and it does not make use of Vegas by default."
