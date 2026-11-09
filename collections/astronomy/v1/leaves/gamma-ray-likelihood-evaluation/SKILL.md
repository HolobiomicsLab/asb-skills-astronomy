---
name: gamma-ray-likelihood-evaluation
description: Use when a global-fit or single-point analysis needs a gamma-ray experiment
  log-likelihood for a dark matter model, and photon yields from annihilation or decay
  cascades must be produced and compared to experiment data.
when_to_use_negative:
- The desired observable is direct detection — use DDCalc instead, which models a
  large range of direct detection experiments
- The desired observable is neutrino telescope data — use the nulike likelihood package
  instead
- No gamma-ray experiment likelihood is needed and only dark matter observables like
  relic density are required
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: DarkBit
  role: GAMBIT module whose gamma-ray signal yield calculator (with cascade-decay
    Monte Carlo) produces the photon yields fed to gamLike
  repo: github.com/GambitBSM/gambit
- name: gamLike
  role: Dedicated backend likelihood calculator for current and future gamma-ray experiments;
    consumes the DarkBit signal-yield file and experiment configuration and returns
    the log-likelihood
  repo: github.com/GambitBSM/gambit
- name: GAMBIT
  role: 'Framework orchestrating the run: YAML-driven scan, backend system loading
    gamLike, and HDF5 printer for validation'
  repo: github.com/GambitBSM/gambit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:11bb974f11f55856da2015d702d66eccdb25560c27929ec2606a7dd2eba41e49
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DarkBit
  - GAMBIT
  - gamLike
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# gamma-ray-likelihood-evaluation

## Summary

Compute gamma-ray indirect-detection likelihoods for a dark matter model point by combining DarkBit's gamma-ray signal yield calculator (with its cascade-decay Monte Carlo) with the gamLike backend's likelihood calculators for current and future gamma-ray experiments.

## When to use

Use when a global-fit or single-point analysis needs a gamma-ray experiment log-likelihood for a dark matter model, and photon yields from annihilation or decay cascades must be produced and compared to experiment data.

## When NOT to use

- The desired observable is direct detection — use DDCalc instead, which models a large range of direct detection experiments
- The desired observable is neutrino telescope data — use the nulike likelihood package instead
- No gamma-ray experiment likelihood is needed and only dark matter observables like relic density are required

## Inputs

- YAML input file selecting the dark matter model and target gamma-ray experiment (yaml_files/<run>.yaml)
- Dark matter model point (parameters supplied through the GAMBIT dependency tree)
- Experiment configuration for the chosen gamma-ray experiment (used by gamLike)

## Outputs

- Photon signal-yield table produced by DarkBit's gamma-ray signal yield calculator
- gamLike log-likelihood value for the selected experiment
- gamlike_loglike.dat file containing the extracted log-likelihood
- HDF5 scan results file recording the likelihood for the same experiment

## How to apply

Configure a GAMBIT run via a YAML input file that selects the desired dark matter model and the target gamma-ray experiment, then launch with 'gambit -f yaml_files/<run>.yaml'. DarkBit's dedicated gamma-ray signal yield calculator produces the photon signal-yield table for the requested model point, using its cascade-decay Monte Carlo treatment of the annihilation/decay channels. Pass the resulting signal-yield file together with the experiment configuration to the gamLike backend, which is the dedicated likelihood calculator for current and future gamma-ray experiments. Extract the gamLike log-likelihood value returned for that experiment and write it to gamlike_loglike.dat. Validate that the computed log-likelihood is finite and consistent with the likelihood recorded for the same experiment in GAMBIT's HDF5 scan output.

## Related tools

- **DarkBit** (GAMBIT module whose gamma-ray signal yield calculator (with cascade-decay Monte Carlo) produces the photon yields fed to gamLike) — github.com/GambitBSM/gambit
- **gamLike** (Dedicated backend likelihood calculator for current and future gamma-ray experiments; consumes the DarkBit signal-yield file and experiment configuration and returns the log-likelihood) — github.com/GambitBSM/gambit
- **GAMBIT** (Framework orchestrating the run: YAML-driven scan, backend system loading gamLike, and HDF5 printer for validation) — github.com/GambitBSM/gambit

## Examples

```
gambit -f yaml_files/<run>.yaml  # run YAML config selecting DM model + gamma-ray experiment; DarkBit signal yields feed gamLike; log-likelihood written to gamlike_loglike.dat
```

## Evaluation signals

- The computed gamLike log-likelihood is a finite number
- The log-likelihood is consistent with the likelihood for the same experiment recorded by GAMBIT in the HDF5 scan output
- The value is written successfully to gamlike_loglike.dat
- The signal-yield table was produced by DarkBit's cascade-decay Monte Carlo for the requested model point before gamLike was invoked
- 'gambit backends' and 'gambit scanners' verify that the gamLike backend was installed and built correctly

## Limitations

- gamLike covers current and future gamma-ray experiments only; other indirect-detection channels (e.g. neutrino telescopes) require different likelihood packages such as nulike
- The photon yield depends on DarkBit's cascade-decay Monte Carlo treatment, which significantly extends but differs from other existing tools
- HDF5 printer support requires optional HDF5 and the h5py Python module, needed for the validation cross-check
- Building GAMBIT and its backends requires specific dependencies (Boost 1.41+, GSL 2.1+, Eigen 3.1.0+, LAPACK, cmake 2.8.12+); for the full build 16 GB RAM may be needed

## Evidence

- [abstract] DarkBit computes gamma-ray observables via a dedicated signal yield calculator that implements a cascade-decay Monte Carlo: "a dedicated signal yield calculator for gamma-ray observations, which significantly extends current tools by implementing a cascade-decay Monte Carlo"
- [abstract] gamLike is the dedicated likelihood calculator for gamma-ray experiments: "a dedicated likelihood calculator for current and future experiments ( gamLike )"
- [methods] GAMBIT runs are launched with YAML input files selecting model and experiment: "gambit -f yaml_files/MSSM7.yaml"
- [methods] Backends must be built and verified before invoking gamLike: "gambit backends
  gambit scanners"
- [intro] GAMBIT's backend system is the mechanism used to interface gamLike: "a flexible system for interfacing external codes (the "backend" system)"
- [readme] HDF5 output is used for the validation cross-check: " - HDF5 (for use of the hdf5 printer)
 - the h5py python module (for use of the hdf5 printer)"
