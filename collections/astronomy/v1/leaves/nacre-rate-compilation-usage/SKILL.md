---
name: nacre-rate-compilation-usage
description: Use when predicting primordial light-element abundances from a BBN nuclear
  network, when comparing abundance predictions to quasar absorption or CMB baryon-density
  constraints, or when rate choices must be specified for the 63-reaction network.
when_to_use_negative:
- The task concerns only the thermal-bath or weak-rate thermodynamics stage (n_p conversion,
  N_eff) rather than the nuclear-abundance network stage.
- The analysis targets New Physics scenarios that would invalidate assuming the SM
  light-element predictions over the standard NACRE II rates.
- A different, smaller reaction network or non-NACRE rate compilation is required
  for sensitivity studies of individual rates.
edam_operation: http://edamontology.org/operation_0244
tools:
- name: PRyMordial
  role: Hosts the largest nuclear reaction network into which the NACRE II key rates
    are attached and which integrates the coupled nuclear-abundance ODEs
  repo: github.com/vallima/PRyMordial
- name: Python 3
  role: Language in which the network integration runs entirely, per the package requirement
- name: SciML kit
  role: Optional Julia routines used to speed up the ODE integration of the nuclear
    network
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:d573cc3426033cf01e5457adbbc7d45655f4369da86790ad228b3600c9e27eba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRyMordial
  - Python 3
  - SciML kit
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# nacre-rate-compilation-usage

## Summary

Attaches the NACRE II compilation of key thermonuclear rates to PRyMordial's largest nuclear reaction network so the coupled nuclear-abundance ODEs reproduce the Standard Model light-element primordial abundances (Y_P, D/H, 3He/H, 7Li/H) over the BBN temperature range.

## When to use

Use when predicting primordial light-element abundances from a BBN nuclear network, when comparing abundance predictions to quasar absorption or CMB baryon-density constraints, or when rate choices must be specified for the 63-reaction network.

## When NOT to use

- The task concerns only the thermal-bath or weak-rate thermodynamics stage (n_p conversion, N_eff) rather than the nuclear-abundance network stage.
- The analysis targets New Physics scenarios that would invalidate assuming the SM light-element predictions over the standard NACRE II rates.
- A different, smaller reaction network or non-NACRE rate compilation is required for sensitivity studies of individual rates.

## Inputs

- Stored thermal-background and weak-rates arrays from the upstream PRyMordial thermodynamics run (non-instantaneous decoupling treatment)
- NACRE II compilation of 12 key thermonuclear rates
- Default (largest) nuclear reaction network with 63 reactions
- Baryon abundance consistent with CMB constraints

## Outputs

- Four SM light-element abundances (helium-4 mass fraction Y_P, D/H, 3He/H, 7Li/H) in the res array
- bbn_abundances.csv
- Primordial-abundance values comparable to the PDG plot and quasar absorption observations

## How to apply

First load the stored thermal-background and weak-rates arrays from the upstream PRyMordial thermodynamics run, which uses a non-instantaneous decoupling treatment. Initialize the default (largest) nuclear reaction network in PRyMordial and attach the NACRE II compilation for the 12 key thermonuclear rates rather than ad hoc rate choices. Integrate the coupled nuclear-abundance ODEs from O(10) MeV down to O(1) keV using Python 3, optionally switching to Julia SciML kit routines for speed. Extract the four SM light-element abundance predictions into the res array and write them to bbn_abundances.csv. Correct application is judged by reproducing the PDG plot of primordial abundances, with the deuterium prediction matching quasar absorption observations while remaining consistent with the CMB-inferred baryon abundance.

## Related tools

- **PRyMordial** (Hosts the largest nuclear reaction network into which the NACRE II key rates are attached and which integrates the coupled nuclear-abundance ODEs) — github.com/vallima/PRyMordial
- **Python 3** (Language in which the network integration runs entirely, per the package requirement)
- **SciML kit** (Optional Julia routines used to speed up the ODE integration of the nuclear network)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- Running over the full BBN temperature range with the largest nuclear network and NACRE II key rates reproduces the PDG plot of primordial abundances.
- The deuterium prediction matches quasar absorption observations.
- The abundance prediction remains consistent with the CMB-inferred baryon abundance.
- The four SM light-element abundances are successfully written into the res array and exported to bbn_abundances.csv.
- The integration covers the full temperature span from O(10) MeV down to O(1) keV.

## Limitations

- The workflow depends on upstream stored thermal-background and weak-rates arrays; it cannot run standalone without the PRyMordial thermodynamics run.
- Only 12 key thermonuclear rates are taken from NACRE II within the default (largest) network; rate-uncertainty exploration beyond these is not part of this skill.
- The Julia SciML speed-up is optional and requires additional installation (PyJulia, diffeqpy); the pure-Python path may be slower.

## Evidence

- [methods] Initialize the default (largest) nuclear reaction network in PRyMordial and attach the NACRE II compilation for the 12 key thermonuclear rates.: [evidence span withheld — anchor status: not_found]
- [abstract] The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects.: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom"
- [intro] the optional switch to some routines of the SciML kit [58], the open-source software for scientific machine learning in Julia: "the optional switch to some routines of the SciML kit [58], the open-source software for scientific machine learning in Julia"
- [readme] A Python package for precise computations of BBN within and beyond the Standard Model.: "A Python package for precise computations of BBN within and beyond the Standard Model."
