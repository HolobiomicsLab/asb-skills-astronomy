---
name: bbn-thermal-history-solver-configuration
description: Use when a cosmological analysis needs BBN light-element abundances and
  N_eff from a modified thermal history — e.g. scanning a new relativistic species
  X (scalar, vector, Majorana, Dirac) over a mass grid with energy exchange to the
  neutrino bath or electromagnetic plasma.
when_to_use_negative:
- The analysis targets post-BBN epochs or observables outside BBN light-element abundances
  and N_eff — PRyMordial is dedicated to the BBN era of the Early Universe.
- A non-Python/Julia ecosystem is required or a GUI-driven BBN code is preferred —
  alternatives like PArthENoPE (FORTRAN), PRIMAT (Mathematica), or AlterBBN (C++)
  exist, though PRyMordial was motivated by their limitations in precision BSM treatment
  and language/interconnection gaps.
- The scenario requires instantaneous-decoupling approximations only and does not
  need PRyMordial's non-instantaneous neutrino decoupling treatment, which is central
  to its precision.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3452
- http://edamontology.org/topic_0203
tools:
- name: PRyMordial
  role: 'Core package: evolves the coupled thermal history with its ODE solver and
    computes BBN light-element abundances and N_eff under SM and new-physics configurations'
  repo: https://github.com/vallima/PRyMordial
- name: Python 3
  role: Language in which the package is written and runs entirely; used to execute
    runPRyM_julia.py and the PRyMdemoSM / PRyMdemoNP notebooks
- name: Julia (SciML kit via PyJulia/diffeqpy)
  role: 'Optional acceleration: advanced users can switch some routines to the open-source
    SciML scientific machine learning kit'
  repo: https://sciml.ai
- name: NumPy / SciPy / Numba / Numdifftools / Vegas
  role: 'Dependencies: NumPy and SciPy are mandatory; Numba, Numdifftools, and Vegas
    are recommended for best possible usage of the package'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:04ac0ea70b852f91d079641580ea1c2a28fbbfc632ef45eb97bc0f7b6fe2133d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallima/PRyMordial
  tools:
  - Julia (SciML kit via PyJulia/diffeqpy)
  - NumPy / SciPy / Numba / Numdifftools / Vegas
  - PRyMordial
  - Python 3
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3452
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# bbn-thermal-history-solver-configuration

## Summary

Configure and run PRyMordial's coupled thermal-history and BBN abundance computation (Python 3, optional Julia SciML routines) to obtain N_eff and light-element yields, including non-instantaneous neutrino decoupling effects and new-physics species X thermalized with the plasma or neutrino bath.

## When to use

Use when a cosmological analysis needs BBN light-element abundances and N_eff from a modified thermal history — e.g. scanning a new relativistic species X (scalar, vector, Majorana, Dirac) over a mass grid with energy exchange to the neutrino bath or electromagnetic plasma.

## When NOT to use

- The analysis targets post-BBN epochs or observables outside BBN light-element abundances and N_eff — PRyMordial is dedicated to the BBN era of the Early Universe.
- A non-Python/Julia ecosystem is required or a GUI-driven BBN code is preferred — alternatives like PArthENoPE (FORTRAN), PRIMAT (Mathematica), or AlterBBN (C++) exist, though PRyMordial was motivated by their limitations in precision BSM treatment and language/interconnection gaps.
- The scenario requires instantaneous-decoupling approximations only and does not need PRyMordial's non-instantaneous neutrino decoupling treatment, which is central to its precision.

## Inputs

- PRyMordial run configuration with NP_nu_flag and NP_e_flag toggles
- Spin/quantum-structure choice for species X (real/complex scalar, massive vector, Majorana or Dirac fermion)
- Mass grid of new-physics species X thermalizing with the plasma or neutrino bath
- PRyM_Yp_DH_cosmoMC_2023.dat numerical table for standard cosmological analyses involving BBN likelihoods

## Outputs

- N_eff (effective number of relativistic degrees of freedom) per mass point, including non-instantaneous decoupling effects
- Neutron-to-proton conversion history from the coupled thermal evolution
- BBN light-element abundance yields (e.g. Yp, deuterium) per mass point
- Fig. 4 style abundance-band plot versus species mass with overlaid observational exclusion region (e.g. fig4_np_abundance_bands.png)

## How to apply

Install PRyMordial (Python 3 with mandatory NumPy/SciPy; Numba, Numdifftools, and Vegas recommended; PyJulia and diffeqpy optional for the SciML Julia routines). Enable the NP flags in the run configuration — NP_nu_flag for energy exchange with the neutrino bath and NP_e_flag for exchange with the electromagnetic plasma — and select the spin/quantum structure of species X plus the mass grid over which it thermalizes. Evolve the coupled Boltzmann/ODE thermal history with PRyMordial's solver, retaining non-instantaneous decoupling effects, to obtain N_eff and the neutron-to-proton conversion history at each mass point. Then run the BBN abundance computation per point to assemble predicted abundance bands versus species mass, which can be overlaid with observational exclusion regions as in the paper's Fig. 4. Validate by checking the reproduced bands and exclusion boundary against the corresponding published panel.

## Related tools

- **PRyMordial** (Core package: evolves the coupled thermal history with its ODE solver and computes BBN light-element abundances and N_eff under SM and new-physics configurations) — https://github.com/vallima/PRyMordial
- **Python 3** (Language in which the package is written and runs entirely; used to execute runPRyM_julia.py and the PRyMdemoSM / PRyMdemoNP notebooks)
- **Julia (SciML kit via PyJulia/diffeqpy)** (Optional acceleration: advanced users can switch some routines to the open-source SciML scientific machine learning kit) — https://sciml.ai
- **NumPy / SciPy / Numba / Numdifftools / Vegas** (Dependencies: NumPy and SciPy are mandatory; Numba, Numdifftools, and Vegas are recommended for best possible usage of the package)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- Reproduced abundance bands versus species mass visually agree with the corresponding Fig. 4 panel of the published paper (band positions and exclusion boundary).
- N_eff values include non-instantaneous decoupling effects, matching the code's advertised capability of 'fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom'.
- The Standard-Model run (PRyMdemoSM) reproduces the dedicated numerical table PRyM_Yp_DH_cosmoMC_2023.dat shipped in the repo for BBN-likelihood cosmological analyses.
- Toggling NP_nu_flag and NP_e_flag changes the neutrino-to-photon temperature ratio and the time-temperature relation in the expected directions for energy exchange with the neutrino bath versus the electromagnetic plasma.
- The saved figure (e.g. fig4_np_abundance_bands.png) contains both the predicted abundance bands and the overlaid observational exclusion region on the same mass axis.

## Limitations

- Vegas is not used by default and the code can avoid Numba and Numdifftools dependencies, but installing the recommended libraries remains advised for the best possible usage of the package.
- Optional Julia-based acceleration (PyJulia, diffeqpy) requires separate installation of the Julia programming language and SciML; without it, some performance may be forfeited.
- The enriched article index reports no discussion-section content, so no limitations, artifacts, or reproducibility gaps could be extracted from the discussion text.

## Evidence

- [abstract] PRyMordial offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects.: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [intro] Python 3 with optional Julia SciML routines as the implementation language for precision Early-Universe studies.: "The package is written and runs entirely with Python 3"
- [intro] PRyMordial fills gaps left by PArthENoPE, PRIMAT, and AlterBBN for BSM-capable precision BBN analyses.: "While powerful and flexible, these public codes nevertheless suffer from a few limitations and/or missing features."
- [readme] README quick-start invocation and dependency list for running PRyMordial.: "To run a quick example, open the terminal and once in PRyMordial folder just type: 

python runPRyM_julia.py"
- [readme] README identifies NumPy and SciPy as mandatory dependencies, with Numba, Numdifftools, Vegas recommended.: "- NumPy (mandatory) – pip install numpy
- SciPy (mandatory) – pip install scipy
- Numba (recommended) – pip install numba"
- [readme] README points to PRyMdemoSM and PRyMdemoNP notebooks and the BBN-likelihood numerical table.: "A dedicated numerical table for standard cosmological analyses involving BBN likelihoods is also in the repo."
