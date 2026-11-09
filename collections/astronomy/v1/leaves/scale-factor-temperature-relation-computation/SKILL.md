---
name: scale-factor-temperature-relation-computation
description: Use when the analysis requires a first-principles thermal history (T_gamma,
  T_nu, scale factor a) through the BBN epoch, or a benchmark SM value of the effective
  number of relativistic degrees of freedom N_eff and Omega_nu h^2 including non-instantaneous
  neutrino decoupling.
when_to_use_negative:
- The goal is ab-initio neutron-to-proton conversion and light-element abundance (Yp,
  D/H) computation rather than the pure thermal background — that is a separate PRyMordial
  pipeline stage.
- The scenario requires a BBN code with a graphical user interface or Mathematica-based
  workflow (e.g. PArthENoPE or PRIMAT) rather than a Python-3 scriptable thermal-bath
  computation.
- Only instantaneous neutrino decoupling is adequate (N_eff = 3 suffices); the non-instantaneous
  treatment is the point of this skill.
tools:
- name: PRyMordial
  role: Package providing the coupled Boltzmann-equation thermodynamics module (PRyM_thermo.py)
    and SM parameter initialization (PRyM_init.py) used to compute T_gamma(t), T_nu(t),
    a(T_gamma), N_eff, and Omega_nu h^2.
  repo: github.com/vallima/PRyMordial
- name: Python 3
  role: The language in which PRyMordial is written and runs entirely; the thermodynamics
    pipeline is executed as a Python 3 script.
- name: Julia (SciML kit via PyJulia/diffeqpy)
  role: 'Optional backend: advanced users may switch some routines to the SciML kit
    for scientific machine learning in Julia, e.g. differential-equation solvers.'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:999b1f2f8559f80fe638fa60ed226b3069a6b12aca91cdb9e47fdd76d2b606ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Julia (SciML kit via PyJulia/diffeqpy)
  - PRyMordial
  - Python 3
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scale-factor-temperature-relation-computation

## Summary

Computes the Standard Model thermal background during BBN by integrating the coupled Boltzmann equations for photon temperature T_gamma(t), neutrino temperature T_nu(t), and scale factor a(T_gamma) in PRyMordial's PRyM_thermo.py, yielding non-instantaneous neutrino decoupling effects such as N_eff^SM = 3.044 and Omega_nu(rel) h^2 = 5.70e-6.

## When to use

Use when the analysis requires a first-principles thermal history (T_gamma, T_nu, scale factor a) through the BBN epoch, or a benchmark SM value of the effective number of relativistic degrees of freedom N_eff and Omega_nu h^2 including non-instantaneous neutrino decoupling.

## When NOT to use

- The goal is ab-initio neutron-to-proton conversion and light-element abundance (Yp, D/H) computation rather than the pure thermal background — that is a separate PRyMordial pipeline stage.
- The scenario requires a BBN code with a graphical user interface or Mathematica-based workflow (e.g. PArthENoPE or PRIMAT) rather than a Python-3 scriptable thermal-bath computation.
- Only instantaneous neutrino decoupling is adequate (N_eff = 3 suffices); the non-instantaneous treatment is the point of this skill.

## Inputs

- SM parameter set loaded by PRyM_init.py (default Standard Model parameters)
- PRyM_thermo.py Boltzmann-equation module (Python 3)
- Temperature range spanning the BBN epoch through the late-time regime T_gamma << MeV

## Outputs

- Thermal history table (T_gamma, T_nu, a) written to prym_thermo_sm.csv (CSV format)
- N_eff^SM = 3.044 (effective number of relativistic degrees of freedom, per-mil accuracy)
- Omega_nu(rel) h^2 = 5.70e-6

## How to apply

Initialize the PRyMordial package with the default Standard Model parameter set loaded by PRyM_init.py. Integrate the coupled Boltzmann equations in PRyM_thermo.py (Python 3) to evolve the photon temperature T_gamma(t), the neutrino temperature T_nu(t), and the scale factor a(T_gamma) through the BBN epoch; Numba and Numdifftools are recommended dependencies for best performance. Evaluate the observables in the late-time regime T_gamma << MeV, where non-instantaneous neutrino decoupling effects are included — this is what shifts N_eff from the instantaneous-decoupling value 3 toward the precision value 3.044. Tabulate the thermal history (T_gamma, T_nu, a) together with the computed observables and write the table to a CSV file (e.g. prym_thermo_sm.csv). Validate by checking that N_eff^SM = 3.044 is reproduced to below the per-mil level, consistent with the code's stated fast-and-precise accuracy.

## Related tools

- **PRyMordial** (Package providing the coupled Boltzmann-equation thermodynamics module (PRyM_thermo.py) and SM parameter initialization (PRyM_init.py) used to compute T_gamma(t), T_nu(t), a(T_gamma), N_eff, and Omega_nu h^2.) — github.com/vallima/PRyMordial
- **Python 3** (The language in which PRyMordial is written and runs entirely; the thermodynamics pipeline is executed as a Python 3 script.)
- **Julia (SciML kit via PyJulia/diffeqpy)** (Optional backend: advanced users may switch some routines to the SciML kit for scientific machine learning in Julia, e.g. differential-equation solvers.)

## Examples

```
cd PRyMordial && python runPRyM_julia.py
```

## Evaluation signals

- N_eff^SM evaluates to 3.044 within the code's stated fast-and-precise accuracy (below the per-mil level).
- Omega_nu(rel) h^2 evaluates to 5.70e-6.
- Output table prym_thermo_sm.csv exists in CSV format with the expected thermal-history columns (T_gamma, T_nu, a).
- The late-time evaluation is taken in the regime T_gamma << MeV where non-instantaneous decoupling effects are included.
- Dependencies declared as mandatory in the README (NumPy, SciPy) are installed so the integration runs as intended.

## Limitations

- Optional dependencies (Numba, Numdifftools, Vegas, PyJulia/diffeqpy) require Julia (julialang.org) and SciML (sciml.ai); the code can run without Numba/Numdifftools but their installation is recommended for best possible usage.
- Existing public BBN codes compared in the paper (PArthENoPE, PRIMAT, AlterBBN) suffer from limitations in accuracy, language, or interconnection with first-principle thermal background computations — this skill should be used with PRyMordial's own first-principles pipeline rather than expecting cross-code output to match exactly.
- The validation target is the Standard Model benchmark; the article notes no changelog/discussion limitations section was extractable, so BSM-variant thermal histories are out of scope for this SM skill.

## Evidence

- [abstract] The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects.: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [intro] The package is written and runs entirely with Python 3: "The package is written and runs entirely with Python 3"
- [intro] tools that allow for numerically efficient, yet precise computations that test the SM in the Early Universe: "it is therefore very timely to have tools at our disposal that allow for numerically efficient, yet precise computations that test the SM in the Early Universe"
- [readme] README quick-run command: "To run a quick example, open the terminal and once in PRyMordial folder just type: python runPRyM_julia.py"
- [readme] README mandatory dependencies: "Dependencies:
-------------
- NumPy (mandatory) – pip install numpy
- SciPy (mandatory) – pip install scipy"
