---
name: saturation-state-evaluation
description: 'Use when building or extending an ExoLyn reaction network: new condensation
  reactions must be added to parameters.txt and their Gibbs formation energies sourced
  so equilibrium vapor concentrations, saturation states, and condensation rates can
  be evaluated.'
when_to_use_negative:
- The goal is opacity or spectral output rather than cloud thermodynamics — combine
  ExoLyn with optool or petitRADTRANS instead of editing the reaction network.
- Condensation rates are already known or prescribed externally — ExoLyn's value lies
  in computing them self-consistently from thermodynamics.
- A detailed kinetic microphysical cloud model is required — ExoLyn is a 'golden mean'
  balance of physical consistency and computational efficiency, not a replacement
  for more complex cloud formation models.
tools:
- name: ExoLyn
  role: 1D cloud model whose transport equation consumes the self-consistently computed
    condensation rates
  repo: https://github.com/helonghuangastro/exolyn
- name: read.py
  role: Ingests reactions.txt and gibbs.txt chemical data
  repo: https://github.com/helonghuangastro/exolyn
- name: init.py
  role: Initializes domain size and initial equilibrium concentrations
  repo: https://github.com/helonghuangastro/exolyn
- name: functions.py
  role: Contains the equilibrium functions to be solved
  repo: https://github.com/helonghuangastro/exolyn
- name: sol.f90 (Fortran)
  role: Custom Fortran linear-system solver, much faster than the numpy solver
  repo: https://github.com/helonghuangastro/exolyn
- name: numpy
  role: Fallback linear-system solver; slower than sol.f90
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202451112
    title: 'ExoLyn: A golden mean approach to multispecies cloud modeling in atmospheric
      retrieval'
  source_content_hash: sha256:f4c72f7943939b19db9b1c37fecdcf1807cd25bac2b6947759e4ac66abe2278f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/helonghuangastro/exolyn
  tools:
  - ExoLyn
  - Python
  - bash
  - functions.py
  - init.py
  - numpy
  - read.py
  - sol.f90 (Fortran)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# saturation-state-evaluation

## Summary

Computes cloud condensation rates self-consistently from thermodynamics (Gibbs formation energies and saturation states) rather than prescribing them, for ExoLyn's 1D coupled transport of cloud particles and vapor.

## When to use

Use when building or extending an ExoLyn reaction network: new condensation reactions must be added to parameters.txt and their Gibbs formation energies sourced so equilibrium vapor concentrations, saturation states, and condensation rates can be evaluated.

## When NOT to use

- The goal is opacity or spectral output rather than cloud thermodynamics — combine ExoLyn with optool or petitRADTRANS instead of editing the reaction network.
- Condensation rates are already known or prescribed externally — ExoLyn's value lies in computing them self-consistently from thermodynamics.
- A detailed kinetic microphysical cloud model is required — ExoLyn is a 'golden mean' balance of physical consistency and computational efficiency, not a replacement for more complex cloud formation models.

## Inputs

- parameters.txt (python-like config: reactions, gas and solid species list; # = comments; expressions may reference previously defined parameters)
- reactions.txt (chemical reactions read by read.py)
- gibbs.txt (Gibbs formation energies for all reaction molecules, e.g. from janaf.nist.gov)
- Chosen T–P point and elemental composition for the initial equilibrium state

## Outputs

- Equilibrium vapor concentrations for all gas species
- Saturation states for condensing species
- Derived cloud condensation rates (e.g. written to condensation_rates.csv) feeding ExoLyn's transport equation

## How to apply

Edit the reactions in parameters.txt (# marks comments), then supply Gibbs formation energy data for every molecule involved in the reaction in gibbs.txt, sourcing values from janaf.nist.gov; finally confirm that all gas and solid species involved are declared in parameters.txt (declaration order does not matter). Read.py ingests reactions.txt and gibbs.txt, and init.py computes the initial equilibrium concentrations at the chosen T–P point and elemental composition. Evaluate the equilibrium functions defined in functions.py and solve the resulting linear systems with the compiled Fortran program sol.f90 (run `bash build.sh` first) because it is much faster than the numpy solver. The result feeds ExoLyn's transport equation: condensation rates are derived self-consistently from thermodynamics instead of being externally prescribed, so any new species must be thermodynamically complete before the coupled solution is meaningful.

## Related tools

- **ExoLyn** (1D cloud model whose transport equation consumes the self-consistently computed condensation rates) — https://github.com/helonghuangastro/exolyn
- **read.py** (Ingests reactions.txt and gibbs.txt chemical data) — https://github.com/helonghuangastro/exolyn
- **init.py** (Initializes domain size and initial equilibrium concentrations) — https://github.com/helonghuangastro/exolyn
- **functions.py** (Contains the equilibrium functions to be solved) — https://github.com/helonghuangastro/exolyn
- **sol.f90 (Fortran)** (Custom Fortran linear-system solver, much faster than the numpy solver) — https://github.com/helonghuangastro/exolyn
- **numpy** (Fallback linear-system solver; slower than sol.f90)

## Examples

```
bash build.sh && python3 relaxation.py
```

## Evaluation signals

- Every molecule in each reaction in parameters.txt has a corresponding Gibbs formation energy entry in gibbs.txt (read.py runs without missing-data errors).
- All gas and solid species involved are declared in parameters.txt, regardless of order.
- sol.f90 is compiled (bash build.sh) and used in place of the numpy solver for the linear systems.
- Initial equilibrium concentrations from init.py are consistent with the chosen T–P point and elemental composition.
- Resulting condensation rates and saturation states enter the transport equation solution without external prescription of rates.

## Limitations

- Gibbs formation energy data must be manually supplied for all reaction molecules (source: janaf.nist.gov); a missing entry breaks the equilibrium evaluation.
- The reaction set is user-defined in parameters.txt, so thermodynamic completeness of the network is the user's responsibility.
- Numpy solving is much slower than the compiled Fortran sol.f90, so uncompiled setups degrade performance.
- ExoLyn results are consistent with more complex cloud formation models but trade away microphysical detail for speed (results in only a few seconds).

## Evidence

- [abstract] ExoLyn solves the transport equation of cloud particles and vapor under self-consistently calculated cloud condensation rates: "ExoLyn solves the transport equation of cloud particles and vapor under cloud condensation rates that are self-consistently calculated from thermodynamics"
- [readme] Gibbs data source and gibbs.txt requirement: "Please also implement gibbs formation energy data for all the molecules involved in the reaction in `gibbs.txt`. You may find these data on janaf.nist.gov."
- [readme] Species declaration rule: "The last step is to make sure all the gas and solid species are contained in parameters.txt. The order does not matter."
- [methods] Fortran solver preference: "**sol.f90** is a custom fortran program to numerically solve linear systems. It's much faster than the numpy solver."
- [methods] Initialization role: "**init.py** initializes the code, finding appropriate domain size and then calculating the initial equilibrium concentrations."
