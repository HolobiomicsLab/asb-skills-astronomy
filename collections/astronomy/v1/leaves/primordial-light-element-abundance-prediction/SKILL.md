---
name: primordial-light-element-abundance-prediction
description: Use when a task requires computing Big Bang Nucleosynthesis abundance
  predictions for helium-4, deuterium, helium-3, or lithium-7 that must be compared
  with quasar absorption or CMB-inferred baryon constraints.
when_to_use_negative:
- The analysis concerns New Physics scenarios (e.g. BSM decays or extra relativistic
  degrees of freedom) rather than reproducing Standard Model abundances — the BSM
  extensions go beyond this skill's SM-network scope
- Only the thermal/weak-rate background (n_eff, neutrino decoupling) is needed without
  nuclear abundance evolution — use the upstream thermodynamics run instead
- A different BBN code (PArthENoPE, PRIMAT, AlterBBN) is mandated by the workflow
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3450
- http://edamontology.org/topic_0152
tools:
- name: PRyMordial
  role: Core package that provides the nuclear reaction network, the coupled abundance
    ODEs integrated from O(10) MeV to O(1) keV, and the res array extraction of the
    four abundances
  repo: github.com/vallima/PRyMordial
- name: Python 3
  role: Primary runtime in which the package is written and the abundance integration
    is executed
- name: SciML kit
  role: Optional Julia differential-equation routines switched in for faster integration
    of the abundance ODEs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:fba5d9b30c9e75591e7b0065005bf96488bba74131394c49d8c98648c3ce76de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRyMordial
  - Python 3
  - SciML kit
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3450
  - http://edamontology.org/topic_0152
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# primordial-light-element-abundance-prediction

## Summary

Predict Standard Model primordial light-element abundances (Y_P, D/H, 3He/H, 7Li/H) by integrating PRyMordial's coupled nuclear-abundance ODEs over the full BBN temperature range using the largest nuclear reaction network with NACRE II key thermonuclear rates.

## When to use

Use when a task requires computing Big Bang Nucleosynthesis abundance predictions for helium-4, deuterium, helium-3, or lithium-7 that must be compared with quasar absorption or CMB-inferred baryon constraints.

## When NOT to use

- The analysis concerns New Physics scenarios (e.g. BSM decays or extra relativistic degrees of freedom) rather than reproducing Standard Model abundances — the BSM extensions go beyond this skill's SM-network scope
- Only the thermal/weak-rate background (n_eff, neutrino decoupling) is needed without nuclear abundance evolution — use the upstream thermodynamics run instead
- A different BBN code (PArthENoPE, PRIMAT, AlterBBN) is mandated by the workflow

## Inputs

- Stored thermal-background and weak-rates arrays from the upstream PRyMordial thermodynamics run (non-instantaneous decoupling treatment)
- Largest nuclear reaction network configuration with NACRE II key rates attached for 12 thermonuclear reactions

## Outputs

- Four SM light-element abundance predictions (Y_P, D/H, 3He/H, 7Li/H) collected in the res array
- bbn_abundances.csv containing the extracted primordial abundances

## How to apply

First load the stored thermal-background and weak-rates arrays from the upstream PRyMordial thermodynamics run, which uses a non-instantaneous decoupling treatment rather than assuming instantaneous neutrino decoupling. Initialize the default largest nuclear reaction network in PRyMordial and attach the NACRE II compilation for the 12 key thermonuclear rates. Integrate the coupled nuclear-abundance ODEs from O(10) MeV down to O(1) keV, running with Python 3 and optionally switching to Julia SciML kit routines for numerical speed. Extract the four SM light-element predictions into the res array and write them to bbn_abundances.csv. Validate by checking that the deuterium prediction matches quasar absorption observations while remaining consistent with the CMB-inferred baryon abundance, and that the PDG plot of primordial abundances is reproduced.

## Related tools

- **PRyMordial** (Core package that provides the nuclear reaction network, the coupled abundance ODEs integrated from O(10) MeV to O(1) keV, and the res array extraction of the four abundances) — github.com/vallima/PRyMordial
- **Python 3** (Primary runtime in which the package is written and the abundance integration is executed)
- **SciML kit** (Optional Julia differential-equation routines switched in for faster integration of the abundance ODEs)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- The four abundances written to bbn_abundances.csv reproduce the PDG plot of primordial abundances
- The predicted D/H value matches quasar absorption observations
- Predictions remain consistent with the CMB-inferred baryon abundance
- The integration spans the full BBN temperature range from O(10) MeV down to O(1) keV without truncation
- Exactly 12 NACRE II key thermonuclear rates are attached to the largest nuclear network

## Limitations

- Numerical speed depends on optional dependencies: Numba, Numdifftools, Vegas, and the Julia SciML stack (PyJulia, diffeqpy) are recommended but the code can run without them; Vegas is not used by default
- The SM prediction of 7Li/H inherits the well-known lithium problem tension with observations, so lithium agreement should not be expected as a pass criterion
- The task card documents no discussion-section limitations or reproducibility gaps; uncertainty propagation over thermonuclear rates is a package feature but was not exercised in this workflow

## Evidence

- [methods] workflow step 1: [evidence span withheld — anchor status: not_found]
- [methods] workflow step 3: [evidence span withheld — anchor status: not_found]
- [results] finding: [evidence span withheld — anchor status: not_found]
- [abstract] abstract: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [readme] readme quick example: "To run a quick example, open the terminal and once in PRyMordial folder just type: 

python runPRyM_julia.py"
