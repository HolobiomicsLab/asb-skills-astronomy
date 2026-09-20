---
name: dark-matter-observable-computation
description: Use when the analysis goal is to compute dark matter observables or likelihoods
  (relic abundance, gamma-ray yields, spin-independent/spin-dependent direct-detection
  limits) for a BSM model via a GAMBIT module-mode scan.
when_to_use_negative:
- The goal is collider or flavour observables rather than dark matter ones — those
  belong to ColliderBit or FlavBit modules, not DarkBit
- Only a stand-alone DarkBit calculation is needed and no GAMBIT statistical/parameter-scanning
  framework (backend system, printer, scanner) is required
- The GAMBIT installation lacks the compulsory dependencies (Boost, GSL 2.1+, Eigen,
  LAPACK) or the dark matter backends have not been downloaded and configured
tools:
- name: DarkBit
  role: GAMBIT module that computes dark matter observables and likelihoods, including
    the gamma-ray cascade-decay Monte Carlo signal yield calculator
  repo: github.com/GambitBSM/gambit
- name: GAMBIT
  role: Framework providing the module system, backend system, statistical/parameter
    scanning framework, and printer through which DarkBit is run and its output captured
  repo: github.com/GambitBSM/gambit
- name: DarkSUSY
  role: External backend interfaced by DarkBit for computing dark matter properties
    (e.g. relic density, detection rates)
- name: micrOMEGAs
  role: External backend interfaced by DarkBit; DarkBit supports quantitative comparison
    between DarkSUSY and micrOMEGAs results
- name: DDCalc
  role: Direct detection package used by DarkBit to model direct detection experiments
    and provide likelihoods for spin-independent and spin-dependent scattering
- name: nulike
  role: Neutrino telescope likelihood package interfaced by DarkBit
- name: gamLike
  role: Dedicated gamma-ray likelihood calculator for current and future experiments,
    interfaced by DarkBit
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:1c3afd75dbc6216bbf637e74470aa7e72f4f099cfc4afcccd9a59414696e19f2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DDCalc
  - DarkBit
  - DarkSUSY
  - GAMBIT
  - gamLike
  - micrOMEGAs
  - nulike
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# dark-matter-observable-computation

## Summary

Computes dark matter observables and likelihoods (e.g. relic density, gamma-ray signal yields, direct-detection constraints) by running DarkBit as a GAMBIT module on an example YAML input, using the DarkSUSY, micrOMEGAs, DDCalc, nulike and gamLike backends.

## When to use

Use when the analysis goal is to compute dark matter observables or likelihoods (relic abundance, gamma-ray yields, spin-independent/spin-dependent direct-detection limits) for a BSM model via a GAMBIT module-mode scan.

## When NOT to use

- The goal is collider or flavour observables rather than dark matter ones — those belong to ColliderBit or FlavBit modules, not DarkBit
- Only a stand-alone DarkBit calculation is needed and no GAMBIT statistical/parameter-scanning framework (backend system, printer, scanner) is required
- The GAMBIT installation lacks the compulsory dependencies (Boost, GSL 2.1+, Eigen, LAPACK) or the dark matter backends have not been downloaded and configured

## Inputs

- GAMBIT input YAML file (e.g. yaml_files/MSSM7.yaml or other example YAML files in the yaml_files folder)
- Built GAMBIT installation with DarkBit module and configured backends (DarkSUSY, micrOMEGAs, DDCalc, nulike, gamLike)
- Scanner and backend configurations verified via `gambit backends` and `gambit scanners`

## Outputs

- Dark matter observable and likelihood results emitted through the GAMBIT printer
- Complete module-mode run log (darkbit_scan_output.txt) containing the DarkBit module header

## How to apply

First verify the installed compute components with `gambit backends` and `gambit scanners` to confirm the external codes DarkBit relies on (DarkSUSY, micrOMEGAs, DDCalc, nulike, gamLike) are configured through GAMBIT's backend system. Then launch the scan in module mode with `gambit -f yaml_files/MSSM7.yaml`, noting that other example YAML files are provided in the yaml_files folder of the gambit_1.5 distribution. Capture the complete module-mode output through the GAMBIT printer, including the dark matter observable and likelihood results, and write the run log to darkbit_scan_output.txt. Validation is that the gambit invocation completes without fatal errors, the log file exists, and the log contains the DarkBit module header plus observable/likelihood results (substring check on 'DarkBit').

## Related tools

- **DarkBit** (GAMBIT module that computes dark matter observables and likelihoods, including the gamma-ray cascade-decay Monte Carlo signal yield calculator) — github.com/GambitBSM/gambit
- **GAMBIT** (Framework providing the module system, backend system, statistical/parameter scanning framework, and printer through which DarkBit is run and its output captured) — github.com/GambitBSM/gambit
- **DarkSUSY** (External backend interfaced by DarkBit for computing dark matter properties (e.g. relic density, detection rates))
- **micrOMEGAs** (External backend interfaced by DarkBit; DarkBit supports quantitative comparison between DarkSUSY and micrOMEGAs results)
- **DDCalc** (Direct detection package used by DarkBit to model direct detection experiments and provide likelihoods for spin-independent and spin-dependent scattering)
- **nulike** (Neutrino telescope likelihood package interfaced by DarkBit)
- **gamLike** (Dedicated gamma-ray likelihood calculator for current and future experiments, interfaced by DarkBit)

## Examples

```
gambit backends && gambit scanners && gambit -f yaml_files/MSSM7.yaml > darkbit_scan_output.txt
```

## Evaluation signals

- The `gambit -f yaml_files/MSSM7.yaml` invocation completes without fatal errors (script_runs check)
- The output log file darkbit_scan_output.txt exists
- The log contains the DarkBit module header (contains_substring on 'DarkBit')
- The log contains the dark matter observable and likelihood results emitted through the GAMBIT printer
- `gambit backends` and `gambit scanners` report that the DarkBit-dependent backends are configured before the run

## Limitations

- cmake can fail to find some dependencies on some systems without guidance; compulsory dependencies include g++/gfortran 5.1+, CMake 2.8.12+, Python 2.7+, Boost 1.41+, GSL 2.1+, Eigen 3.1.0+, LAPACK and pkg-config
- DarkBit's dark matter codes are external backends that must be downloaded and configured (e.g. via GAMBIT's backend download scripts); the run fails if they are not configured
- HDF5/h5py, MPI and other printer/parallel-sampling features are optional and absent installs restrict printer and parallel sampling options

## Evidence

- [abstract] DarkBit as an advanced dark matter code: "We introduce DarkBit , an advanced software code for computing dark matter constraints on various extensions to the Standard Model of particle physics"
- [abstract] gamma-ray signal yield calculator: "a dedicated signal yield calculator for gamma-ray observations, which significantly extends current tools by implementing a cascade-decay Monte Carlo"
- [intro] backend system rationale: "a flexible system for interfacing external codes (the "backend" system)"
- [methods] verify components: "gambit backends
  gambit scanners"
- [methods] launch module-mode scan: "gambit -f yaml_files/MSSM7.yaml"
- [readme] README module citation: "DarkBit: A GAMBIT module for computing dark matter observables and likelihoods, Eur. Phys. J. C 77 (2017) 831, arXiv:1705.07920"
