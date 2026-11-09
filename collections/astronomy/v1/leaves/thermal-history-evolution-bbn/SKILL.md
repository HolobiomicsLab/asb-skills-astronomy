---
name: thermal-history-evolution-bbn
description: Use when a cosmology/BBN analysis needs a first-principles thermal background
  table (T_gamma, T_nu, a) or per-mil-accurate SM observables such as N_eff rather
  than an assumed instantaneous-decoupling approximation.
when_to_use_negative:
- The goal is BSM/New-Physics BBN scenarios rather than the SM thermal background
  — use the package's New-Physics workflow (e.g. PRyMdemoNP) instead of the SM thermodynamics
  run.
- A quick light-element abundance estimate is needed without a self-consistent thermal
  background — the full Boltzmann integration is unnecessary if an existing tabulated
  background suffices.
- Only post-BBN cosmological analysis is needed and the thermal history is already
  available from another code (e.g. PArthENoPE, PRIMAT, AlterBBN outputs).
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: PRyMordial
  role: 'Core package: PRyM_init.py loads the SM parameters and PRyM_thermo.py integrates
    the coupled thermodynamic background equations'
  repo: https://github.com/vallima/PRyMordial
- name: Python 3
  role: Language the package is written and runs entirely in; executes the Boltzmann
    integration and writes prym_thermo_sm.csv
- name: Julia (SciML kit, optional)
  role: Optional switch of solver routines to the SciML open-source scientific machine-learning
    kit via PyJulia/diffeqpy
- name: NumPy / SciPy
  role: Mandatory dependencies for the numerical integration of the coupled Boltzmann
    equations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:3914f90d6ca41481f63d2dc835e2bd431823afb5bcf7fad91015dfe750b8b935
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallima/PRyMordial
  tools:
  - Julia (SciML kit, optional)
  - NumPy / SciPy
  - PRyMordial
  - Python 3
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# thermal-history-evolution-bbn

## Summary

Evolve the coupled BBN-epoch cosmological thermal background — photon temperature T_gamma(t), neutrino temperature T_nu(t), and scale factor a(T_gamma) — with PRyMordial's Boltzmann-equation pipeline (PRyM_thermo.py) to compute N_eff^SM = 3.044 and Omega_nu(rel) h^2 = 5.70e-6, including non-instantaneous neutrino decoupling.

## When to use

Use when a cosmology/BBN analysis needs a first-principles thermal background table (T_gamma, T_nu, a) or per-mil-accurate SM observables such as N_eff rather than an assumed instantaneous-decoupling approximation.

## When NOT to use

- The goal is BSM/New-Physics BBN scenarios rather than the SM thermal background — use the package's New-Physics workflow (e.g. PRyMdemoNP) instead of the SM thermodynamics run.
- A quick light-element abundance estimate is needed without a self-consistent thermal background — the full Boltzmann integration is unnecessary if an existing tabulated background suffices.
- Only post-BBN cosmological analysis is needed and the thermal history is already available from another code (e.g. PArthENoPE, PRIMAT, AlterBBN outputs).

## Inputs

- Default Standard Model parameter set loaded via PRyM_init.py
- Coupled Boltzmann-equation system in PRyM_thermo.py (photon temperature, neutrino temperature, scale factor)
- Initial/early-time conditions during the BBN epoch (T_gamma of order MeV)

## Outputs

- Thermal-history table (T_gamma, T_nu, a) written to prym_thermo_sm.csv
- N_eff^SM = 3.044 (accurate to below the per-mil level)
- Omega_nu(rel) h^2 = 5.70e-6

## How to apply

Initialize PRyMordial with the default Standard Model parameter set loaded by PRyM_init.py, then integrate the coupled Boltzmann equations in PRyM_thermo.py (Python 3) to evolve T_gamma(t), T_nu(t), and a(T_gamma) through the BBN epoch. Evaluate the observables in the late-time regime T_gamma << MeV, where non-instantaneous neutrino decoupling effects are included — this is what pushes N_eff from the instantaneous-decoupling value 3.046-style idealization to the accurate 3.044. Tabulate the thermal history (T_gamma, T_nu, a) together with the computed observables and write it to prym_thermo_sm.csv. The rationale is that PRyMordial interconnects a first-principle thermal background computation with downstream neutron-to-proton (n->p) conversion and BBN abundance calculations, so a validated background table is the prerequisite for any subsequent BBN or CMB-facing analysis.

## Related tools

- **PRyMordial** (Core package: PRyM_init.py loads the SM parameters and PRyM_thermo.py integrates the coupled thermodynamic background equations) — https://github.com/vallima/PRyMordial
- **Python 3** (Language the package is written and runs entirely in; executes the Boltzmann integration and writes prym_thermo_sm.csv)
- **Julia (SciML kit, optional)** (Optional switch of solver routines to the SciML open-source scientific machine-learning kit via PyJulia/diffeqpy)
- **NumPy / SciPy** (Mandatory dependencies for the numerical integration of the coupled Boltzmann equations)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- N_eff^SM evaluates to 3.044, matching the paper's value quoted as accurate to below the per-mil level
- Omega_nu(rel) h^2 = 5.70e-6 is reproduced
- prym_thermo_sm.csv exists with the expected thermal-history columns (T_gamma, T_nu, a) in CSV format
- The tabulated (T_gamma, T_nu, a) history covers the late-time regime T_gamma << MeV where non-instantaneous neutrino decoupling is included
- A quick smoke test of the install runs without error: python runPRyM_julia.py

## Limitations

- Non-instantaneous neutrino decoupling must be included; an instantaneous-decoupling treatment will not reproduce N_eff^SM = 3.044 at per-mil accuracy.
- Optional recommended dependencies (Numba, Numdifftools, Vegas) and the Julia/SciML switch are not installed by default; the code avoids Numba/Numdifftools but performance may be suboptimal without them.
- The article's discussion section content was absent, so no explicit limitation/changelog statements on accuracy regimes beyond the stated per-mil claim were extractable.

## Evidence

- [abstract] The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects.: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [results] Solving the coupled thermodynamic background equations in the Standard Model yields N_eff^SM = 3.044, with the prediction quoted as accurate to below the per mil level.: [evidence span withheld — anchor status: not_found]
- [intro] The package is written and runs entirely with Python 3.: "The package is written and runs entirely with Python 3"
- [readme] To run a quick example, open the terminal and once in PRyMordial folder just type: python runPRyM_julia.py: "To run a quick example, open the terminal and once in PRyMordial folder just type: 

python runPRyM_julia.py"
