---
name: thermal-history-network-integration
description: Use when you need to compute primordial BBN abundances or N_eff from
  first-principle thermal-history calculations, e.g. to test the Standard Model in
  the Early Universe or explore New Physics active during the BBN epoch.
when_to_use_negative:
- You need a dedicated BSM code with particle-species modules such as NUDEC_BSM rather
  than a flexible network runner — PRyMordial is for general NP exploration, not a
  standalone dark-sector solver.
- The task is a highly interactive ab-initio computation better served by the Mathematica
  package PRIMAT, or a C++ BSM BBN scan better served by AlterBBN.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: PRyMordial
  role: main package integrating the thermal background, weak rates, and BBN network
  repo: github.com/vallima/PRyMordial
- name: Python 3
  role: language in which the package is written and runs entirely
- name: Julia
  role: optional backend via the SciML kit / diffeqpy for faster ODE routines
- name: NumPy
  role: mandatory dependency for numerics
- name: SciPy
  role: mandatory dependency for solver routines
- name: Numba
  role: recommended dependency for performance; the code can avoid it if needed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:9e430c78c6842477c52e0ee0028294f5246f23e4ba4aab126d16437ba232c7ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Julia
  - NumPy
  - Numba
  - PRyMordial
  - Python 3
  - SciPy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# thermal-history-network-integration

## Summary

Run the PRyMordial package to integrate the early-Universe thermal background together with weak neutron-to-proton conversion rates and the BBN thermonuclear network, producing precise light-element abundances (He-4, D, He-3, Li-7) and the effective number of relativistic degrees of freedom within and beyond the Standard Model.

## When to use

Use when you need to compute primordial BBN abundances or N_eff from first-principle thermal-history calculations, e.g. to test the Standard Model in the Early Universe or explore New Physics active during the BBN epoch.

## When NOT to use

- You need a dedicated BSM code with particle-species modules such as NUDEC_BSM rather than a flexible network runner — PRyMordial is for general NP exploration, not a standalone dark-sector solver.
- The task is a highly interactive ab-initio computation better served by the Mathematica package PRIMAT, or a C++ BSM BBN scan better served by AlterBBN.

## Inputs

- Stored thermal background evolution tables shipped with PRyMordial
- Weak neutron-to-proton conversion rate tables shipped with PRyMordial
- Eta_B / baryon-density parameter grid or BSM scenario parameters
- PRyM_Yp_DH_cosmoMC_2023.dat numerical table for cosmological likelihood analyses

## Outputs

- Final BBN mass fractions of helium-4, deuterium, helium-3, and lithium-7
- Effective number of relativistic degrees of freedom (N_eff) including non-instantaneous decoupling
- Abundance-versus-parameter tables (e.g. CSV) and plotted curves (e.g. matplotlib PNG)

## How to apply

Load the stored thermal background evolution and weak neutron-to-proton conversion rates shipped with PRyMordial, then configure the Standard Model BBN network run in Python 3 (optionally switching routines to the Julia SciML kit for faster integration). Run the solver over the temperature/redshift range of the first three minutes; PRyMordial includes non-instantaneous decoupling effects rather than assuming instantaneous neutrino decoupling. Extract final mass fractions of helium-4, deuterium, helium-3 and lithium-7 together with N_eff from the solver output; for cosmology-grade use, compare against the precomputed PRyM_Yp_DH_cosmoMC_2023.dat likelihood table. For parameter scans (e.g. a grid in eta_B), repeat the run per parameter value and tabulate abundances for plotting or MCMC interfacing. Installation should include NumPy and SciPy (mandatory) and ideally Numba and Numdifftools for best performance, per the README.

## Related tools

- **PRyMordial** (main package integrating the thermal background, weak rates, and BBN network) — github.com/vallima/PRyMordial
- **Python 3** (language in which the package is written and runs entirely)
- **Julia** (optional backend via the SciML kit / diffeqpy for faster ODE routines)
- **NumPy** (mandatory dependency for numerics)
- **SciPy** (mandatory dependency for solver routines)
- **Numba** (recommended dependency for performance; the code can avoid it if needed)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- Reproduces the PDG review behaviour of abundances versus eta_B (Fig. 2 of the paper): He-4 rising, D/He-3/Li-7 falling with baryon density.
- Deuterium prediction matches quasar absorption observations and the CMB baryon-density constraint band.
- Output N_eff consistent with precision expectations including non-instantaneous decoupling effects.
- Predicted Y_p and D/H match the precomputed values in PRyM_Yp_DH_cosmoMC_2023.dat for the same cosmological inputs.
- Run completes within the package's advertised 'fast and precise' budget — e.g. quick example via python runPRyM_julia.py terminates with converged abundances.

## Limitations

- The README lists no changelog and the paper's discussion section contains no extractable limitations or reproducibility gaps.
- Numba, Numdifftools and Vegas are only recommended (and Vegas is not used by default); skipping them may degrade performance while keeping correctness.
- Optional Julia/SciML acceleration requires installing the Julia language, PyJulia, and diffeqpy separately; the code runs correctly in pure Python 3 without them.
- Optional dependencies are avoidable by design — 'The code can easily avoid dependencies on Numba and Numdifftools and it does not make use of Vegas by default.'

## Evidence

- [abstract] The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects.: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [abstract] PRyMordial is suitable for state-of-the-art analyses in the Standard Model as well as for general investigations into New Physics active during BBN.: "PRyMordial is suitable for state-of-the-art analyses in the Standard Model as well as for general investigations into New Physics active during BBN."
- [intro] The package is written and runs entirely with Python 3: "The package is written and runs entirely with Python 3"
- [intro] the optional switch to some routines of the SciML kit, the open-source software for scientific machine learning in Julia: "the optional switch to some routines of the SciML kit [58], the open-source software for scientific machine learning in Julia"
- [readme] To run a quick example, open the terminal and once in PRyMordial folder just type: python runPRyM_julia.py: "To run a quick example, open the terminal and once in PRyMordial folder just type: 

python runPRyM_julia.py"
- [readme] A dedicated numerical table for standard cosmological analyses involving BBN likelihoods is also in the repo.: "A dedicated numerical table for standard cosmological analyses involving BBN likelihoods is also in the repo."
