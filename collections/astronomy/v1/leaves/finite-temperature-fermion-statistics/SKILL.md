---
name: finite-temperature-fermion-statistics
description: Use when computing ab-initio n<->p weak-conversion rates for BBN freeze-out
  where finite-temperature (as opposed to zero-temperature) effects on the lepton
  phase space must be included alongside QED and weak-magnetism corrections.
when_to_use_negative:
- The Born-approximation rates are sufficient for your analysis and the beyond-Born
  corrections (QED radiative, recoil/weak-magnetism, finite-T) would only add complexity
- You need a separately quantitatively benchmarked rate module — the PRyMordial paper
  describes this module but does not benchmark it against external thresholds
tools:
- name: PRyMordial
  role: Host package whose PRyM_evalnTOp.py module computes the beyond-Born n<->p
    rates including finite-temperature corrections
  repo: github.com/vallima/PRyMordial
- name: Python 3
  role: Language in which the Born-limit integrand and corrected rates are coded and
    integrated with adaptive quadrature
- name: PRIMAT
  role: Reference prescription (Appendix B of Ref. [47]) whose state-of-the-art electroweak-rate
    recipes are followed for the finite-T and other beyond-Born corrections
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:6225a0ccce2646d9a5ea139ca8df1ac16df5a868a2c8467e3f2f903cfb22ad43
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
yes_rate: 0.8333333333333334
n_claims: 12
---

# finite-temperature-fermion-statistics

## Summary

Implements the finite-temperature corrections to the weak neutron<->proton interconversion rates used in PRyMordial for neutron freeze-out during BBN, as part of the beyond-Born-approximation treatment adopted from PRIMAT (Appendix B of Ref. [47]).

## When to use

Use when computing ab-initio n<->p weak-conversion rates for BBN freeze-out where finite-temperature (as opposed to zero-temperature) effects on the lepton phase space must be included alongside QED and weak-magnetism corrections.

## When NOT to use

- The Born-approximation rates are sufficient for your analysis and the beyond-Born corrections (QED radiative, recoil/weak-magnetism, finite-T) would only add complexity
- You need a separately quantitatively benchmarked rate module — the PRyMordial paper describes this module but does not benchmark it against external thresholds

## Inputs

- Born-limit integrand for n->p and p->n interconversion (Eq. (13)) with electron and neutrino phase-space factors
- Grid of T_gamma/T_nu values spanning the BBN temperature range
- Normalization input: neutron lifetime or modified Fermi constant
- Optionally, pre-stored weak-rate files loaded instead of recomputation

## Outputs

- n_to_p_weak_rates.csv — tabulated beyond-Born n<->p weak rates on the T_gamma/T_nu grid
- Weak-rate module PRyM_evalnTOp.py implementing the QED radiative, finite-nucleon-mass/weak-magnetism, and finite-temperature corrections

## How to apply

Start from the Eq. (13) Born-limit integrand for neutron-to-proton and proton-to-neutron interconversion, including electron and neutrino phase-space factors, and numerically integrate it over the lepton energy variables using adaptive quadrature (Python 3 / SciPy, optionally Numba- or Vegas-accelerated). Then layer on the corrections prescribed by PRIMAT Appendix B: QED radiative corrections to the charged-current integrand, finite-nucleon-mass recoil and weak-magnetism terms, and finally the finite-temperature corrections to the rate integrals. Evaluate the corrected rates on a grid of T_gamma/T_nu values and write the table to n_to_p_weak_rates.csv, using PRyMordial's dedicated module PRyM_evalnTOp.py or pre-stored rate files. Normalize the computed rates either via the measured neutron lifetime or the modified Fermi constant — the choice of normalization is the main decision point.

## Related tools

- **PRyMordial** (Host package whose PRyM_evalnTOp.py module computes the beyond-Born n<->p rates including finite-temperature corrections) — github.com/vallima/PRyMordial
- **Python 3** (Language in which the Born-limit integrand and corrected rates are coded and integrated with adaptive quadrature)
- **PRIMAT** (Reference prescription (Appendix B of Ref. [47]) whose state-of-the-art electroweak-rate recipes are followed for the finite-T and other beyond-Born corrections)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- Tabulated rates in n_to_p_weak_rates.csv are smooth, positive, and physically consistent across the BBN temperature range
- Corrected rates are qualitatively consistent with the PRIMAT Appendix B prescription (no quantitative reference threshold is reported in the paper)
- Rate integrals reproduce the Eq. (13) Born-limit result when the QED radiative, recoil/weak-magnetism, and finite-temperature terms are switched off
- Normalization is self-consistent whether anchored to the neutron lifetime or the modified Fermi constant

## Limitations

- The paper describes but does not separately benchmark the PRyM_evalnTOp.py module, so acceptance is qualitative consistency with the PRIMAT Appendix B prescription rather than a quantitative threshold
- Rates may be loaded from pre-stored files rather than recomputed, which can mask dependence on the stored grid resolution in T_gamma/T_nu
- Best performance requires optional dependencies (Numba, Numdifftools, Vegas); the code can run without them but the README notes these remain recommended for the best possible usage

## Evidence

- [full_text] PRyMordial implements the refinements to the Born-level n<->p rates — QED radiative corrections, finite nucleon-mass/weak-magnetism effects, and finite-temperature effects: [evidence span withheld — anchor status: not_found]
- [full_text] Rates computed in dedicated module: [evidence span withheld — anchor status: not_found]
- [intro] PRIMAT is the ab-initio reference prescription: "PRIMAT [47, 48] is an user-friendly Mathematica package containing all the inputs and ingredients for an ab-initio computation"
- [abstract] PRyMordial offers fast and precise evaluation of BBN observables: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [readme] README quick-run instruction: "To run a quick example, open the terminal and once in PRyMordial folder just type:"
