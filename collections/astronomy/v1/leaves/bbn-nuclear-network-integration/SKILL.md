---
name: bbn-nuclear-network-integration
description: Use when the task requires predicting primordial light-element abundances
  from a BBN nuclear reaction network, or validating a BBN code's predictions against
  the PDG plot, quasar deuterium absorption, and CMB-inferred baryon abundance.
when_to_use_negative:
- The task is only the thermal/weak-rate part of BBN (neutron-to-proton conversion,
  N_eff) without nuclear abundances — that is the upstream thermodynamics run, not
  the network integration.
- The problem concerns BSM physics active during BBN beyond standard rate/network
  inputs, which requires PRyMordial's New-Physics features rather than the default
  SM network run.
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: PRyMordial
  role: Provides the default largest nuclear reaction network, NACRE II key-rate attachment,
    and abundance ODE solver used to produce Y_P, D/H, 3He/H, 7Li/H
  repo: github.com/vallima/PRyMordial
- name: Python 3
  role: The language PRyMordial is written and runs entirely in; used to execute the
    network integration
  repo: github.com/vallima/PRyMordial
- name: SciML kit
  role: Optional Julia routines (via PyJulia/diffeqpy) for faster integration of the
    abundance ODEs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:56dfe78d41351a774010e4773233768ebf381d38c2844411806caa0f97aa9c09
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRyMordial
  - Python 3
  - SciML kit
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bbn-nuclear-network-integration

## Summary

Integrates PRyMordial's coupled nuclear-abundance ODEs over its largest nuclear reaction network (with NACRE II key rates) to predict the four SM light-element abundances (Y_P, D/H, 3He/H, 7Li/H) across the BBN temperature range.

## When to use

Use when the task requires predicting primordial light-element abundances from a BBN nuclear reaction network, or validating a BBN code's predictions against the PDG plot, quasar deuterium absorption, and CMB-inferred baryon abundance.

## When NOT to use

- The task is only the thermal/weak-rate part of BBN (neutron-to-proton conversion, N_eff) without nuclear abundances — that is the upstream thermodynamics run, not the network integration.
- The problem concerns BSM physics active during BBN beyond standard rate/network inputs, which requires PRyMordial's New-Physics features rather than the default SM network run.

## Inputs

- Stored thermal-background and weak-rates arrays from the upstream PRyMordial thermodynamics run
- PRyMordial default (largest) nuclear reaction network initialization
- NACRE II compilation for the 12 key thermonuclear rates

## Outputs

- res array with the four SM light-element abundances (Y_P, D/H, 3He/H, 7Li/H)
- bbn_abundances.csv

## How to apply

Load the stored thermal-background and weak-rates arrays produced by the upstream PRyMordial thermodynamics run, which treats decoupling non-instantaneously. Initialize PRyMordial's default (largest) nuclear reaction network and attach the NACRE II compilation for the 12 key thermonuclear rates. Integrate the coupled nuclear-abundance ODEs from O(10) MeV down to O(1) keV — either in pure Python 3 or, optionally, via the SciML kit routines in Julia (PyJulia/diffeqpy) for speed. Extract the four abundance predictions into the res array and write them to bbn_abundances.csv; judge correctness by comparison with the PDG plot of primordial abundances and with observed D/H from quasar absorption and the CMB-inferred baryon abundance.

## Related tools

- **PRyMordial** (Provides the default largest nuclear reaction network, NACRE II key-rate attachment, and abundance ODE solver used to produce Y_P, D/H, 3He/H, 7Li/H) — github.com/vallima/PRyMordial
- **Python 3** (The language PRyMordial is written and runs entirely in; used to execute the network integration) — github.com/vallima/PRyMordial
- **SciML kit** (Optional Julia routines (via PyJulia/diffeqpy) for faster integration of the abundance ODEs)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- The four abundances in res reproduce the PDG plot of primordial abundances when run over the full BBN temperature range with NACRE II key rates.
- The predicted deuterium (D/H) matches quasar absorption observations.
- The deuterium prediction remains consistent with the CMB-inferred baryon abundance.
- The output file bbn_abundances.csv is produced with all four abundance entries populated.
- Integration spans the intended range from O(10) MeV down to O(1) keV without premature termination.

## Limitations

- Numba and Numdifftools are optional dependencies; the code can avoid them, but their installation is recommended for the best possible usage of the package.
- Julia/SciML acceleration is optional and requires the Julia programming language plus PyJulia and diffeqpy; without them, users are limited to the pure-Python path.
- The repository README provides no dedicated discussion/limitations section for the BBN abundance workflow beyond dependency notes.

## Evidence

- [results] workflow step 3: [evidence span withheld — anchor status: not_found]
- [results] workflow step 2: [evidence span withheld — anchor status: not_found]
- [abstract] abstract: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [intro] intro language: "The package is written and runs entirely with Python 3"
- [readme] README invocation: "To run a quick example, open the terminal and once in PRyMordial folder just type: 

python runPRyM_julia.py"
