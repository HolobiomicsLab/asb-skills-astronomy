---
name: indirect-detection-experiment-configuration
description: Use when you need an indirect-detection (gamma-ray) likelihood for a
  dark matter model point under a specific current experiment, e.g. reproducing gamLike
  likelihoods inside a GAMBIT scan.
when_to_use_negative:
- The goal is direct-detection constraints — those likelihoods come from DarkBit's
  DDCalc-based routines for spin-independent/spin-dependent scattering, not gamLike
- The observable is neutrino-telescope data — that likelihood is handled by the nulike
  package, not the gamma-ray yield + gamLike path
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: GAMBIT
  role: 'Scan driver: reads the YAML input file and orchestrates the run (''gambit
    -f yaml_files/<run>.yaml''); stores results in HDF5'
  repo: github.com/GambitBSM/gambit
- name: DarkBit
  role: Produces the photon signal-yield table via its cascade-decay Monte Carlo gamma-ray
    signal-yield calculator
  repo: github.com/GambitBSM/gambit
- name: gamLike
  role: Backend likelihood calculator for current and future gamma-ray experiments;
    consumes the DarkBit signal-yield file and experiment configuration
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:46a035355b6193912d3cade7d9ea396e6b10756322207886399dcbe8ac368448
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DarkBit
  - GAMBIT
  - gamLike
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# indirect-detection-experiment-configuration

## Summary

Configure a GAMBIT run so DarkBit's gamma-ray signal-yield calculator (cascade-decay Monte Carlo) produces photon yields that are passed to the gamLike backend, yielding a log-likelihood for a chosen current gamma-ray experiment.

## When to use

Use when you need an indirect-detection (gamma-ray) likelihood for a dark matter model point under a specific current experiment, e.g. reproducing gamLike likelihoods inside a GAMBIT scan.

## When NOT to use

- The goal is direct-detection constraints — those likelihoods come from DarkBit's DDCalc-based routines for spin-independent/spin-dependent scattering, not gamLike
- The observable is neutrino-telescope data — that likelihood is handled by the nulike package, not the gamma-ray yield + gamLike path

## Inputs

- YAML input file selecting the dark matter model and target gamma-ray experiment (yaml_files/<run>.yaml)
- Dark matter model point (model parameters from the scan)
- gamLike backend with the experiment's gamma-ray data configuration
- DarkBit signal-yield file produced by the gamma-ray signal-yield calculator

## Outputs

- gamlike_loglike.dat containing the finite gamLike log-likelihood for the chosen experiment
- HDF5 scan results file recording the same experiment likelihood

## How to apply

Write a YAML input file that selects the desired dark matter model and the target gamma-ray experiment, then launch the scan with 'gambit -f yaml_files/<run>.yaml'. DarkBit's dedicated gamma-ray signal-yield calculator runs its cascade-decay Monte Carlo to produce the photon signal-yield table for the requested model point. The gamLike backend — DarkBit's dedicated likelihood calculator for current and future gamma-ray experiments — receives the signal-yield file together with the experiment configuration and returns a log-likelihood. Write that value to gamlike_loglike.dat and check it is a finite number. Validate by cross-checking the value against the likelihood for the same experiment recorded in GAMBIT's HDF5 scan output.

## Related tools

- **GAMBIT** (Scan driver: reads the YAML input file and orchestrates the run ('gambit -f yaml_files/<run>.yaml'); stores results in HDF5) — github.com/GambitBSM/gambit
- **DarkBit** (Produces the photon signal-yield table via its cascade-decay Monte Carlo gamma-ray signal-yield calculator) — github.com/GambitBSM/gambit
- **gamLike** (Backend likelihood calculator for current and future gamma-ray experiments; consumes the DarkBit signal-yield file and experiment configuration)

## Examples

```
gambit -f yaml_files/<run>.yaml
```

## Evaluation signals

- The gamLike log-likelihood written to gamlike_loglike.dat is a finite number
- The computed log-likelihood is consistent with the likelihood for the same experiment recorded in GAMBIT's HDF5 scan output
- The signal-yield table was produced by DarkBit's cascade-decay Monte Carlo for the requested model point before gamLike is invoked
- 'gambit backends' and 'gambit scanners' list the gamLike backend and the chosen scanner as installed before the run

## Limitations

- gamLike covers only current and future gamma-ray experiments; other indirect-detection channels (e.g. neutrino telescopes via nulike) need different backends
- The computed likelihood is only valid if the signal-yield table corresponds to the same model point requested in the YAML configuration

## Evidence

- [other] DarkBit computes gamma-ray observables via a dedicated signal yield calculator: [evidence span withheld — anchor status: not_found]
- [abstract] gamLike is the dedicated likelihood calculator: "a dedicated likelihood calculator for current and future experiments ( gamLike )"
- [abstract] signal yield calculator implements cascade-decay Monte Carlo: "a dedicated signal yield calculator for gamma-ray observations, which significantly extends current tools by implementing a cascade-decay Monte Carlo"
- [methods] run GAMBIT scan via yaml input file: "gambit -f yaml_files/MSSM7.yaml"
- [methods] verify installed backends and scanners: "gambit backends
  gambit scanners"
