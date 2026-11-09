---
name: boltzmann-equation-integration-early-universe
description: Use when a cosmology or BBN analysis requires a first-principles thermal
  background (N_eff, Omega_nu(rel) h^2, T_gamma/T_nu evolution) with non-instantaneous
  neutrino decoupling, rather than an assumed instantaneous-decoupling snapshot.
when_to_use_negative:
- The analysis only needs BBN light-element abundances computed from a pre-computed
  background, without evolving the coupled thermodynamic equations itself
- An instantaneous-decoupling approximation (fixed N_eff) is acceptable and per-mil
  precision on the neutrino thermal history is not required
edam_operation: http://edamontology.org/operation_2426
tools:
- name: PRyMordial
  role: Package whose PRyM_init.py and PRyM_thermo.py modules implement the coupled
    Boltzmann-equation integration of the early-universe thermal background
  repo: github.com/vallima/PRyMordial
- name: Python 3
  role: Language the package is written and runs entirely in; executes the thermodynamics
    pipeline
- name: Julia / SciML kit
  role: Optional switch to SciML differential-equation routines (via PyJulia and diffeqpy)
    for faster ODE integration
- name: NumPy / SciPy
  role: Mandatory numerical dependencies for the Boltzmann-equation solver
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:a69a6a8e5cdb1440c173f657069b65bbac8e24d6efd18787efc81e863446a1df
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Julia / SciML kit
  - NumPy / SciPy
  - PRyMordial
  - Python 3
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# boltzmann-equation-integration-early-universe

## Summary

Integrate the coupled Boltzmann equations in PRyMordial's PRyM_thermo.py (Python 3) to evolve the photon temperature T_gamma(t), neutrino temperature T_nu(t), and scale factor a(T_gamma) through the BBN epoch, computing N_eff and the relativistic neutrino density Omega_nu h^2 for the Standard Model or a BSM thermal history.

## When to use

Use when a cosmology or BBN analysis requires a first-principles thermal background (N_eff, Omega_nu(rel) h^2, T_gamma/T_nu evolution) with non-instantaneous neutrino decoupling, rather than an assumed instantaneous-decoupling snapshot.

## When NOT to use

- The analysis only needs BBN light-element abundances computed from a pre-computed background, without evolving the coupled thermodynamic equations itself
- An instantaneous-decoupling approximation (fixed N_eff) is acceptable and per-mil precision on the neutrino thermal history is not required

## Inputs

- Standard Model parameter set loaded by PRyM_init.py
- Coupled Boltzmann-equation specification in PRyM_thermo.py (state variables T_gamma(t), T_nu(t), a(T_gamma))
- Optional BSM thermal-bath input parameters for New Physics during BBN

## Outputs

- Thermal-history table prym_thermo_sm.csv with columns for T_gamma, T_nu, and a
- SM observables: N_eff^SM = 3.044 (per-mil accuracy) and Omega_nu(rel) h^2 = 5.70e-6
- Numerical table for BBN likelihoods (e.g. PRyM_Yp_DH_cosmoMC_2023.dat for standard cosmological analyses)

## How to apply

Initialize PRyMordial with the default Standard Model parameter set loaded by PRyM_init.py, then integrate the coupled Boltzmann equations in PRyM_thermo.py (Python 3) to evolve T_gamma(t), T_nu(t), and a(T_gamma) through the BBN epoch. Evaluate the observables N_eff^SM and Omega_nu(rel) h^2 in the late-time regime T_gamma << MeV, where non-instantaneous neutrino decoupling effects are retained rather than assuming instantaneous decoupling. Tabulate the thermal history (T_gamma, T_nu, a) together with the computed observables and write the table to prym_thermo_sm.csv. Validate by checking the CSV exists with the expected thermal-history columns and that N_eff^SM = 3.044 (accurate to below the per mil level) and Omega_nu(rel) h^2 = 5.70e-6 are reproduced; optional Julia/SciML routines via PyJulia and diffeqpy can accelerate the ODE solves.

## Related tools

- **PRyMordial** (Package whose PRyM_init.py and PRyM_thermo.py modules implement the coupled Boltzmann-equation integration of the early-universe thermal background) — github.com/vallima/PRyMordial
- **Python 3** (Language the package is written and runs entirely in; executes the thermodynamics pipeline)
- **Julia / SciML kit** (Optional switch to SciML differential-equation routines (via PyJulia and diffeqpy) for faster ODE integration)
- **NumPy / SciPy** (Mandatory numerical dependencies for the Boltzmann-equation solver)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- prym_thermo_sm.csv exists in CSV format with the expected thermal-history columns (T_gamma, T_nu, a)
- N_eff^SM = 3.044 is reproduced to below the per mil level, matching the paper's stated fast-and-precise accuracy
- Omega_nu(rel) h^2 = 5.70e-6 is reproduced in the late-time regime T_gamma << MeV
- The evolved T_nu/T_gamma ratio reflects non-instantaneous neutrino decoupling (heating of T_nu relative to instantaneous decoupling) rather than a fixed 0.716... snapshot
- The integration runs end-to-end via the documented entry point python runPRyM_julia.py without missing-dependency errors (NumPy and SciPy mandatory; Numba/Numdifftools/Vegas recommended)

## Limitations

- Optional dependencies (PyJulia, diffeqpy, SciML) require installation of the Julia language and sciml.ai software; the code avoids Numba/Numdifftools/Vegas by default but these are recommended for best usage
- The benchmark observables (N_eff^SM = 3.044, Omega_nu(rel) h^2 = 5.70e-6) apply to the Standard Model; BSM scenarios require modified inputs and different validation targets
- Existing public BBN codes (PArthENoPE, PRIMAT, AlterBBN) were identified as having limitations and missing features, so cross-code comparisons may not be one-to-one

## Evidence

- [abstract] The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects.: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [abstract] PRyMordial: A package dedicated to efficient computations of observables in the Early Universe with the focus on the cosmological era of Big Bang Nucleosynthesis (BBN).: "In this work we present PRyMordial: A package dedicated to efficient computations of observables in the Early Universe with the focus on the cosmological era of Big Bang Nucleosynthesis (BBN)."
- [intro] The package is written and runs entirely with Python 3: "The package is written and runs entirely with Python 3"
- [readme] To run a quick example, open the terminal and once in PRyMordial folder just type: python runPRyM_julia.py: "To run a quick example, open the terminal and once in PRyMordial folder just type: python runPRyM_julia.py"
- [readme] NumPy (mandatory) - pip install numpy; SciPy (mandatory) - pip install scipy: "- NumPy (mandatory) – pip install numpy
- SciPy (mandatory) – pip install scipy"
