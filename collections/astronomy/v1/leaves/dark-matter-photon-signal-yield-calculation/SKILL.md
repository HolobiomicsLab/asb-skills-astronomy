---
name: dark-matter-photon-signal-yield-calculation
description: Use when a global-fit scan must score a dark matter model point against
  a current gamma-ray experiment's likelihood, and photon yields need to be generated
  from the model's annihilation/decay channels.
when_to_use_negative:
- The target observable is direct detection rather than gamma rays — use DDCalc, which
  models direct detection experiments and spin-independent/spin-dependent scattering
  likelihoods
- The target observable is neutrino telescope data — use the nulike backend instead
  of gamLike
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: DarkBit
  role: GAMBIT module whose gamma-ray signal yield calculator implements a cascade-decay
    Monte Carlo to produce photon yields
  repo: github.com/GambitBSM/gambit
- name: gamLike
  role: Backend likelihood calculator for current and future gamma-ray experiments;
    consumes DarkBit's signal-yield file and experiment configuration
  repo: github.com/GambitBSM/gambit
- name: GAMBIT
  role: Framework that runs the scan via YAML input file and provides the backend
    system, statistical framework, and HDF5 results printer
  repo: github.com/GambitBSM/gambit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:0940b74f8e52b6f02ff53bdefb2d2bb33fb5107511462d5ddf42382f324e829f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DarkBit
  - GAMBIT
  - gamLike
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# dark-matter-photon-signal-yield-calculation

## Summary

Computes gamma-ray indirect-detection likelihoods for dark matter models by combining DarkBit's gamma-ray signal yield calculator (cascade-decay Monte Carlo photon yields) with the gamLike backend likelihood calculator for a chosen experiment.

## When to use

Use when a global-fit scan must score a dark matter model point against a current gamma-ray experiment's likelihood, and photon yields need to be generated from the model's annihilation/decay channels.

## When NOT to use

- The target observable is direct detection rather than gamma rays — use DDCalc, which models direct detection experiments and spin-independent/spin-dependent scattering likelihoods
- The target observable is neutrino telescope data — use the nulike backend instead of gamLike

## Inputs

- GAMBIT YAML input file selecting a dark matter model and target gamma-ray experiment
- Dark matter model parameter point (spectrum provided by GAMBIT modules such as SpecBit)
- gamLike experiment configuration/data for the chosen gamma-ray experiment

## Outputs

- Photon signal-yield table for the model point (DarkBit signal-yield file)
- gamLike log-likelihood value for the selected experiment, written to gamlike_loglike.dat
- Per-experiment likelihood entries in the GAMBIT HDF5 scan output

## How to apply

Configure the GAMBIT run with a YAML input file that selects the desired dark matter model and the target gamma-ray experiment, launching with 'gambit -f yaml_files/<run>.yaml'. DarkBit's dedicated gamma-ray signal yield calculator implements a cascade-decay Monte Carlo to produce the photon signal-yield table for the requested model point. Pass that signal-yield file together with the experiment configuration to the gamLike backend, the dedicated likelihood calculator for current and future gamma-ray experiments. Extract the gamLike log-likelihood value returned for that experiment and write it to gamlike_loglike.dat. Validate that the log-likelihood is finite and consistent with the likelihood GAMBIT records for the same experiment in the HDF5 scan output.

## Related tools

- **DarkBit** (GAMBIT module whose gamma-ray signal yield calculator implements a cascade-decay Monte Carlo to produce photon yields) — github.com/GambitBSM/gambit
- **gamLike** (Backend likelihood calculator for current and future gamma-ray experiments; consumes DarkBit's signal-yield file and experiment configuration) — github.com/GambitBSM/gambit
- **GAMBIT** (Framework that runs the scan via YAML input file and provides the backend system, statistical framework, and HDF5 results printer) — github.com/GambitBSM/gambit

## Examples

```
gambit -f yaml_files/<run>.yaml
```

## Evaluation signals

- The computed gamLike log-likelihood is a finite number
- The log-likelihood is consistent with the likelihood for the same experiment recorded by GAMBIT in the scan output (HDF5 results file)
- The photon signal-yield table is produced for the requested model point by the cascade-decay Monte Carlo
- Backends are properly installed/verified before running: 'gambit backends' and 'gambit scanners' list gamLike

## Limitations

- Requires GAMBIT build prerequisites (g++/gfortran 5.1+, CMake 2.8.12+, Boost 1.41+, GSL 2.1+, Eigen 3.1.0+, LAPACK) and backends must be built via 'make -jn backends'
- gamLike covers current and future gamma-ray experiments only; other indirect-detection channels (neutrinos via nulike, direct detection via DDCalc) need different likelihood calculators
- HDF5 printer requires optional HDF5 and h5py dependencies; without them the HDF5-based validation path is unavailable

## Evidence

- [abstract] DarkBit computes gamma-ray observables via a dedicated signal yield calculator that implements a cascade-decay Monte Carlo: "a dedicated signal yield calculator for gamma-ray observations, which significantly extends current tools by implementing a cascade-decay Monte Carlo"
- [abstract] gamLike is the dedicated gamma-ray likelihood calculator: "a dedicated likelihood calculator for current and future experiments ( gamLike )"
- [methods] YAML-driven scan invocation pattern: "gambit -f yaml_files/MSSM7.yaml"
- [abstract] GAMBIT is the statistical fitting framework hosting DarkBit: "DarkBit is written in the framework of the Global And Modular Beyond the Standard Model Inference Tool ( GAMBIT )"
- [intro] GAMBIT includes the backend system used to interface gamLike: "a flexible system for interfacing external codes (the "backend" system)"
- [readme] HDF5 printer dependency for scan output validation: "HDF5 (for use of the hdf5 printer)"
