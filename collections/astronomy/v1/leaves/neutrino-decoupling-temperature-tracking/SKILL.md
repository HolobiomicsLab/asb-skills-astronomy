---
name: neutrino-decoupling-temperature-tracking
description: Use when a cosmology analysis requires a first-principles SM thermodynamic
  background (N_eff, neutrino energy density, thermal history) rather than assuming
  instantaneous neutrino decoupling or hardcoded N_eff values.
when_to_use_negative:
- When the analysis needs BBN light-element abundances (Yp, D/H) rather than the thermal
  background itself — that is a separate PRyMordial computation.
- When exploring beyond-the-SM scenarios with new decoupling physics — the default
  SM parameter set will not capture the New Physics sector.
- When a precomputed cosmoMC table suffices (e.g. PRyM_Yp_DH_cosmoMC_2023.dat) rather
  than a fresh integration of the coupled Boltzmann system.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0196
tools:
- name: PRyMordial
  role: Package providing PRyM_thermo.py for solving the coupled thermodynamic background
    equations and PRyM_init.py for loading the SM parameter set
  repo: github.com/vallima/PRyMordial
- name: Python 3
  role: Language in which PRyMordial runs entirely; executes the Boltzmann integration
    pipeline
- name: Julia
  role: Optional switch to SciML kit routines (via PyJulia/diffeqpy) for the differential
    equation solving
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:330c0918df28cdda1dbcf18f79f39eb5537a75e304b936374cac8eb282b2c51a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Julia
  - PRyMordial
  - Python 3
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# neutrino-decoupling-temperature-tracking

## Summary

Compute the Standard Model thermal background by solving the coupled Boltzmann equations for photon temperature T_gamma(t), neutrino temperature T_nu(t), and scale factor a(T_gamma) through the BBN epoch, yielding N_eff^SM = 3.044 and Omega_nu(rel) h^2 = 5.70e-6 with non-instantaneous neutrino decoupling included.

## When to use

Use when a cosmology analysis requires a first-principles SM thermodynamic background (N_eff, neutrino energy density, thermal history) rather than assuming instantaneous neutrino decoupling or hardcoded N_eff values.

## When NOT to use

- When the analysis needs BBN light-element abundances (Yp, D/H) rather than the thermal background itself — that is a separate PRyMordial computation.
- When exploring beyond-the-SM scenarios with new decoupling physics — the default SM parameter set will not capture the New Physics sector.
- When a precomputed cosmoMC table suffices (e.g. PRyM_Yp_DH_cosmoMC_2023.dat) rather than a fresh integration of the coupled Boltzmann system.

## Inputs

- Standard Model parameter set (loaded by PRyM_init.py with PRyMordial defaults)
- Boltzmann equation system for T_gamma(t), T_nu(t), a(T_gamma) in PRyM_thermo.py
- BBN-epoch temperature range (MeV scale down to T_gamma << MeV)

## Outputs

- N_eff^SM = 3.044 (accurate to below the per mil level)
- Omega_nu(rel) h^2 = 5.70e-6
- Thermal history table (T_gamma, T_nu, a) with computed observables written to prym_thermo_sm.csv

## How to apply

Initialize the PRyMordial package with the default Standard Model parameter set loaded by PRyM_init.py, then integrate the coupled Boltzmann equations in PRyM_thermo.py to evolve T_gamma(t), T_nu(t), and a(T_gamma) through the BBN epoch. Evaluate the observables N_eff and Omega_nu(rel) h^2 in the late-time regime T_gamma << MeV, where non-instantaneous neutrino decoupling effects are included, since a fixed ratio T_nu/T_gamma would miss the per-mil-level correction. Tabulate the thermal history (T_gamma, T_nu, a) together with the computed observables and write it to prym_thermo_sm.csv. The Python-only path is sufficient for precision work; Julia via PyJulia/diffeqpy and the SciML kit are optional accelerators. Validation requires reproducing the paper's quoted values, accurate to below the per mil level.

## Related tools

- **PRyMordial** (Package providing PRyM_thermo.py for solving the coupled thermodynamic background equations and PRyM_init.py for loading the SM parameter set) — github.com/vallima/PRyMordial
- **Python 3** (Language in which PRyMordial runs entirely; executes the Boltzmann integration pipeline)
- **Julia** (Optional switch to SciML kit routines (via PyJulia/diffeqpy) for the differential equation solving)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- N_eff^SM equals 3.044, matching the paper's quoted value below the per mil level
- Omega_nu(rel) h^2 equals 5.70e-6
- prym_thermo_sm.csv exists with expected thermal-history columns (T_gamma, T_nu, a)
- Late-time solution is evaluated in the regime T_gamma << MeV where non-instantaneous decoupling effects are included
- Computed observables reproduce the values reported in the PRyMordial paper within the code's stated fast-and-precise accuracy

## Limitations

- Python is the default path; the Julia/SciML option requires installing Julia, sciml.ai software, PyJulia, and diffeqpy separately
- Recommended but not mandatory dependencies include Numba, Numdifftools, and Vegas — the code can avoid them, but installation remains recommended for best usage of the package
- The paper's discussion section had no changelog content, so no reproducibility gaps or artifacts could be extracted from that source

## Evidence

- [other] PRyM_thermo.py integration of coupled Boltzmann equations: [evidence span withheld — anchor status: not_found]
- [results] N_eff SM benchmark value: [evidence span withheld — anchor status: not_found]
- [abstract] Package description including non-instantaneous decoupling: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [readme] README quick example: "To run a quick example, open the terminal and once in PRyMordial folder just type: python runPRyM_julia.py"
