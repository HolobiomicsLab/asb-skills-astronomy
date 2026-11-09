---
name: primordial-decoupling-physics
description: Use when the goal is to reproduce or predict the four SM BBN abundances
  (helium-4 mass fraction, D/H, 3He/H, 7Li/H) over the full BBN temperature range
  and compare them with quasar-absorption and CMB-inferred observations.
when_to_use_negative:
- When the research question targets BSM physics active during BBN (e.g. new relativistic
  degrees of freedom) rather than the SM baseline abundances
- When a first-principles thermal-bath/weak-decoupling computation itself must be
  performed — that is the upstream PRyMordial thermodynamics step, not this network-integration
  skill
- When only cosmological likelihood tables are needed — the repo ships a dedicated
  numerical table (PRyM_Yp_DH_cosmoMC_2023.dat) for standard cosmological analyses
  involving BBN likelihoods
edam_operation: http://edamontology.org/operation_2426
edam_topics: []
tools:
- name: PRyMordial
  role: Main package providing the default largest nuclear reaction network, integration
    of the coupled nuclear-abundance ODEs, and the res array with the four abundance
    predictions
  repo: github.com/vallima/PRyMordial
- name: Python 3
  role: Language in which the network integration is written and run entirely
- name: SciML kit
  role: Optional Julia routines used to speed up the ODE integration of the nuclear
    network
- name: NumPy
  role: Mandatory dependency for running PRyMordial
- name: SciPy
  role: Mandatory dependency for running PRyMordial
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:761b27da4da002ed660fe78796aab6dca3351c9a2f9f19c37b82b8069db13a51
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NumPy
  - PRyMordial
  - Python 3
  - SciML kit
  - SciPy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# primordial-decoupling-physics

## Summary

Skill for computing Standard Model primordial light-element abundances (Y_P, D/H, 3He/H, 7Li/H) by integrating the PRyMordial coupled nuclear-abundance ODEs over a 63-reaction network with NACRE II key thermonuclear rates, using pre-computed thermal-background and weak-rates arrays from the non-instantaneous decoupling thermodynamics run.

## When to use

Use when the goal is to reproduce or predict the four SM BBN abundances (helium-4 mass fraction, D/H, 3He/H, 7Li/H) over the full BBN temperature range and compare them with quasar-absorption and CMB-inferred observations.

## When NOT to use

- When the research question targets BSM physics active during BBN (e.g. new relativistic degrees of freedom) rather than the SM baseline abundances
- When a first-principles thermal-bath/weak-decoupling computation itself must be performed — that is the upstream PRyMordial thermodynamics step, not this network-integration skill
- When only cosmological likelihood tables are needed — the repo ships a dedicated numerical table (PRyM_Yp_DH_cosmoMC_2023.dat) for standard cosmological analyses involving BBN likelihoods

## Inputs

- Stored thermal-background arrays from the upstream PRyMordial thermodynamics run (non-instantaneous decoupling treatment)
- Stored weak-rates arrays (neutron-to-proton conversion) from the upstream PRyMordial run
- NACRE II compilation of the 12 key thermonuclear rates
- Default (largest, 63-reaction) PRyMordial nuclear reaction network specification

## Outputs

- res array containing the four SM light-element abundances: Y_P (helium-4 mass fraction), D/H, 3He/H, 7Li/H
- bbn_abundances.csv with the extracted abundance predictions

## How to apply

First load the stored thermal-background and weak-rates arrays produced by the upstream PRyMordial thermodynamics run, which treats neutrino decoupling as non-instantaneous rather than instantaneous. Next initialize the default (largest) nuclear reaction network in PRyMordial and attach the NACRE II compilation for the 12 key thermonuclear rates. Then integrate the coupled nuclear-abundance ODEs from O(10) MeV down to O(1) keV; the run is entirely in Python 3, with an optional speed-up via Julia SciML kit routines. Finally extract the four SM light-element abundance predictions into the res array and write them to bbn_abundances.csv. Correctness is judged by reproduction of the PDG plot of primordial abundances, agreement of the deuterium prediction with quasar absorption observations, and consistency with the CMB-inferred baryon abundance.

## Related tools

- **PRyMordial** (Main package providing the default largest nuclear reaction network, integration of the coupled nuclear-abundance ODEs, and the res array with the four abundance predictions) — github.com/vallima/PRyMordial
- **Python 3** (Language in which the network integration is written and run entirely)
- **SciML kit** (Optional Julia routines used to speed up the ODE integration of the nuclear network)
- **NumPy** (Mandatory dependency for running PRyMordial)
- **SciPy** (Mandatory dependency for running PRyMordial)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- The four abundances in res reproduce the PDG plot of primordial abundances when run over the full BBN temperature range with the largest network and NACRE II key rates
- The deuterium prediction (D/H) matches quasar absorption observations while remaining consistent with the CMB-inferred baryon abundance
- bbn_abundances.csv is produced containing Y_P, D/H, 3He/H, and 7Li/H
- Integration spans the expected temperature range, from O(10) MeV down to O(1) keV
- Output results (res array) are computed and can be checked against the demo notebooks PRyMdemoSM in the repository

## Limitations

- The thermodynamics/weak-rates stage must be run upstream first — this skill consumes its stored arrays rather than computing the decoupling physics itself
- The abundance predictions are for the Standard Model baseline; New Physics scenarios require the PRyMdemoNP extension path
- Numba, Numdifftools, and Vegas are only recommended dependencies and the code can avoid them, but best usage of the package assumes their installation; Julia/SciML acceleration requires installing Julia and diffeqpy

## Evidence

- [abstract] The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects.: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [intro] package written in Python 3 with optional Julia switch: "The package is written and runs entirely with Python 3"
- [intro] optional Julia SciML routines: "the optional switch to some routines of the SciML kit [58], the open-source software for scientific machine learning in Julia"
- [readme] README quick example command: "To run a quick example, open the terminal and once in PRyMordial folder just type: 

python runPRyM_julia.py"
- [readme] README mandatory dependencies: "- NumPy (mandatory) – pip install numpy
- SciPy (mandatory) – pip install scipy"
