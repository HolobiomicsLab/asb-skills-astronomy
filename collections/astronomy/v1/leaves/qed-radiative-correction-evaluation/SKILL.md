---
name: qed-radiative-correction-evaluation
description: Use when a BBN freeze-out or neutron-abundance calculation requires weak
  n<->p rates accurate beyond the Born approximation, or when tabulating corrected
  rates as a function of T_gamma/T_nu.
when_to_use_negative:
- The analysis needs only Born-limit weak rates or does not require precision percent-level
  neutron freeze-out accuracy
- The task concerns BBN thermonuclear reaction rates or light-element abundance network
  solving rather than electroweak n<->p rate computation
- A different BBN code (PArthENoPE, AlterBBN, NUDEC_BSM) is the mandated rate provider
  and its own beyond-Born recipes must be reproduced instead
edam_operation: http://edamontology.org/operation_0004
tools:
- name: PRyMordial
  role: Host package whose dedicated module PRyM_evalnTOp.py implements the beyond-Born
    n<->p rate computation (or loads pre-stored rate tables)
  repo: github.com/vallima/PRyMordial
- name: PRIMAT
  role: 'Reference prescription for the corrections: QED radiative corrections, finite
    nucleon-mass/weak-magnetism, and finite-temperature effects (Appendix B of Ref.
    [47])'
- name: Python 3
  role: Implementation language for the Born integrand, adaptive quadrature integration,
    and CSV rate-table generation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:5053912bd81cae2f557353f736d0e2c927760c9efe15052bec187a2189853d13
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRIMAT
  - PRyMordial
  - Python 3
  edam_operation: http://edamontology.org/operation_0004
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# qed-radiative-correction-evaluation

## Summary

Computes weak neutron<->proton conversion rates beyond the Born approximation by augmenting the Born-limit charged-current integrand with QED radiative corrections, finite-nucleon-mass recoil/weak-magnetism terms, and finite-temperature effects, following PRIMAT (Appendix B of Ref. [47]) as implemented in PRyMordial's PRyM_evalnTOp.py module.

## When to use

Use when a BBN freeze-out or neutron-abundance calculation requires weak n<->p rates accurate beyond the Born approximation, or when tabulating corrected rates as a function of T_gamma/T_nu.

## When NOT to use

- The analysis needs only Born-limit weak rates or does not require precision percent-level neutron freeze-out accuracy
- The task concerns BBN thermonuclear reaction rates or light-element abundance network solving rather than electroweak n<->p rate computation
- A different BBN code (PArthENoPE, AlterBBN, NUDEC_BSM) is the mandated rate provider and its own beyond-Born recipes must be reproduced instead

## Inputs

- Born-limit n<->p rate integrand per Eq. (13) with electron and neutrino phase-space factors
- Grid of T_gamma/T_nu temperature values spanning the BBN epoch
- Normalization choice: neutron lifetime or modified Fermi constant
- Optionally, pre-stored weak-rate table files for PRyM_evalnTOp.py

## Outputs

- Corrected (beyond-Born) weak n<->p conversion rates
- Numerical table n_to_p_weak_rates.csv of corrected rates on a T_gamma/T_nu grid
- Rates consumed by PRyMordial's BBN abundance and N_eff computation

## How to apply

First code the Eq. (13) Born-limit integrand for neutron-to-proton and proton-to-neutron interconversion, including electron and neutrino phase-space factors and the standard weak matrix element, then integrate over the lepton energy variables with adaptive quadrature (SciPy; Vegas/Numba are recommended for best performance in PRyMordial). Sequentially add the three corrections: QED radiative corrections to the charged-current integrand, finite-nucleon-mass recoil and weak-magnetism terms as prescribed by PRIMAT Appendix B, and finite-temperature corrections. Evaluate the corrected rates on a grid of T_gamma/T_nu values and write them to n_to_p_weak_rates.csv, either computing ab-initio or loading pre-stored tables. Normalize the rates either via the neutron lifetime or the modified Fermi constant; this choice is the main decision point. Validation is qualitative: rates must be smooth, positive, and physically consistent across the BBN temperature range — the paper describes but does not separately benchmark this module.

## Related tools

- **PRyMordial** (Host package whose dedicated module PRyM_evalnTOp.py implements the beyond-Born n<->p rate computation (or loads pre-stored rate tables)) — github.com/vallima/PRyMordial
- **PRIMAT** (Reference prescription for the corrections: QED radiative corrections, finite nucleon-mass/weak-magnetism, and finite-temperature effects (Appendix B of Ref. [47]))
- **Python 3** (Implementation language for the Born integrand, adaptive quadrature integration, and CSV rate-table generation)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- Tabulated rates in n_to_p_weak_rates.csv are smooth (no numerical spikes) across the BBN temperature range
- All rates are positive everywhere on the T_gamma/T_nu grid
- Rates reproduce the qualitative structure of the PRIMAT Appendix B prescription (acceptance is qualitative, not threshold-based)
- With neutron-lifetime normalization, the integrated rate is consistent with the measured neutron lifetime; with modified-Fermi-constant normalization, it is consistent with that constant
- Born-limit integrand reproduced alone matches the Eq. (13) result before corrections are switched on

## Limitations

- The paper describes the module but does not separately benchmark it; no quantitative reference threshold is reported, so validation is qualitative consistency with PRIMAT Appendix B
- Rate tables may be pre-stored rather than recomputed, so users must confirm whether a stored table or the ab-initio computation is being used
- Optional dependencies (Numba, Numdifftools, Vegas) are recommended 'for the best possible usage of the package' and PyJulia/diffeqpy require installing Julia; the code can avoid Numba and Numdifftools and does not use Vegas by default

## Evidence

- [abstract] The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects.: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [intro] PRIMAT is the reference prescription for the electroweak rate corrections.: "PRIMAT [47, 48] is an user-friendly Mathematica package containing all the inputs and ingredients for an ab-initio computation"
- [readme] README identifies PRyMordial as a Python BBN package with its quick-run entry point.: "A Python package for precise computations of BBN within and beyond the Standard Model."
- [readme] README specifies the mandatory numerical dependencies used for the quadrature-based rate integrals.: "- NumPy (mandatory) – pip install numpy
- SciPy (mandatory) – pip install scipy"
