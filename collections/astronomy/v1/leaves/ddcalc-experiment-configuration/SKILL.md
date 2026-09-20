---
name: ddcalc-experiment-configuration
description: Use when a dark matter benchmark point specifies both spin-independent
  and spin-dependent couplings and a direct detection likelihood (e.g. XENON1T) must
  be computed with DDCalc inside a GAMBIT scan.
when_to_use_negative:
- The analysis only involves spin-independent scattering with no spin-dependent couplings,
  so a simpler single-coupling likelihood setup suffices
- The goal is gamma-ray or neutrino telescope likelihoods (DarkBit signal yield calculator
  or nulike) rather than direct detection
- No dark matter model point or scattering couplings are specified and only a global
  parameter scan framework is needed
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3450
tools:
- name: DDCalc
  role: Direct detection package that models a large range of direct detection experiments
    and provides likelihoods for arbitrary combinations of spin-independent and spin-dependent
    scattering processes
  repo: github.com/GambitBSM/gambit
- name: DarkBit
  role: GAMBIT module whose direct detection likelihood pipeline is invoked for the
    combined SI+SD scattering configuration
  repo: github.com/GambitBSM/gambit
- name: gamLike
  role: Likelihood calculator interface through which the computed direct detection
    likelihood is extracted
  repo: github.com/GambitBSM/gambit
- name: GAMBIT
  role: Framework that runs the single-point scan via YAML input and manages DDCalc
    as an external backend
  repo: github.com/GambitBSM/gambit
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:516a90918ed21689e8af289886ff292ad91de50f1c647dc6e2a08aa3aa4c128f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DDCalc
  - DarkBit
  - GAMBIT
  - gamLike
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3450
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# ddcalc-experiment-configuration

## Summary

Configures GAMBIT's DarkBit direct detection pipeline backed by DDCalc to compute likelihoods for direct detection experiments under combined spin-independent and spin-dependent scattering, exposed via the gamLike likelihood interface.

## When to use

Use when a dark matter benchmark point specifies both spin-independent and spin-dependent couplings and a direct detection likelihood (e.g. XENON1T) must be computed with DDCalc inside a GAMBIT scan.

## When NOT to use

- The analysis only involves spin-independent scattering with no spin-dependent couplings, so a simpler single-coupling likelihood setup suffices
- The goal is gamma-ray or neutrino telescope likelihoods (DarkBit signal yield calculator or nulike) rather than direct detection
- No dark matter model point or scattering couplings are specified and only a global parameter scan framework is needed

## Inputs

- GAMBIT YAML scan input file specifying one benchmark dark matter model point with spin-independent and spin-dependent couplings
- Direct detection experiment selection (e.g. XENON1T) for DDCalc
- Built GAMBIT repository (GambitBSM/gambit_1.5) with backend download scripts configured

## Outputs

- ddcalc_likelihood.json containing a finite numeric direct detection likelihood value for the benchmark point
- Single-point GAMBIT scan output from the DarkBit direct detection pipeline

## How to apply

Write a GAMBIT YAML scan input file in the yaml_files folder specifying one benchmark dark matter model point with both spin-independent and spin-dependent couplings, and select a single direct detection likelihood (XENON1T) for DDCalc. Build the required backend with 'make -jn backends' and verify the installation with 'gambit backends' before running, since DDCalc is an external code interfaced through GAMBIT's backend system. Launch the single-point evaluation with 'gambit -f <input.yaml>' so GAMBIT invokes the DarkBit direct detection likelihood pipeline for the combined SI+SD configuration. Extract the computed likelihood through the gamLike likelihood interface and write the numeric value to ddcalc_likelihood.json. Validation requires ddcalc_likelihood.json to exist and contain a finite numeric likelihood for the benchmark point.

## Related tools

- **DDCalc** (Direct detection package that models a large range of direct detection experiments and provides likelihoods for arbitrary combinations of spin-independent and spin-dependent scattering processes) — github.com/GambitBSM/gambit
- **DarkBit** (GAMBIT module whose direct detection likelihood pipeline is invoked for the combined SI+SD scattering configuration) — github.com/GambitBSM/gambit
- **gamLike** (Likelihood calculator interface through which the computed direct detection likelihood is extracted) — github.com/GambitBSM/gambit
- **GAMBIT** (Framework that runs the single-point scan via YAML input and manages DDCalc as an external backend) — github.com/GambitBSM/gambit

## Examples

```
make -j4 backends; gambit backends; gambit -f yaml_files/ddcalc_benchmark.yaml
```

## Evaluation signals

- ddcalc_likelihood.json exists after the run
- The value in ddcalc_likelihood.json is a finite numeric likelihood for the benchmark point
- 'gambit backends' reports DDCalc as installed before the scan is launched
- The YAML input declares both spin-independent and spin-dependent couplings and selects exactly one direct detection likelihood (XENON1T)

## Limitations

- DDCalc is an external backend and must be downloaded and built separately via 'make -jn backends'; the run fails if backends are not installed and verified
- cmake may fail to find some dependencies without guidance on some systems, so backend builds may need extra configuration
- The setup targets single-point evaluation of a benchmark model; full global fits require additional modules and scanners

## Evidence

- [abstract] DDCalc models a large range of direct detection experiments and provides likelihoods for arbitrary combinations of spin-independent and spin-dependent scattering processes: "provides the corresponding likelihoods for arbitrary combinations of spin-independent and spin-dependent scattering processes"
- [methods] Verify installed backends before running: "gambit backends
  gambit scanners"
- [methods] Run GAMBIT scan via yaml input file: "gambit -f yaml_files/MSSM7.yaml"
- [intro] GAMBIT backend system interfaces external codes like DDCalc: "a flexible system for interfacing external codes (the "backend" system)"
- [readme] README lists GamLike citation tied to DarkBit paper: "**GamLike:** GAMBIT Dark Matter Workgroup: T. Bringmann, J. Conrad, et al., **DarkBit**: A GAMBIT module for computing dark matter observables and likelihoods"
