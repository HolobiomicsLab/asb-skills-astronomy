---
name: spin-independent-spin-dependent-scattering-combination
description: Use when a task requires a direct detection likelihood for arbitrary
  combinations of spin-independent and spin-dependent WIMP scattering processes at
  one or more benchmark dark matter model points.
when_to_use_negative:
- The analysis only needs indirect detection (gamma-ray or neutrino telescope) likelihoods,
  which DarkBit handles via its cascade-decay Monte Carlo signal yield calculator
  or the nulike package instead of DDCalc.
- The dark matter model point has only spin-independent or only spin-dependent couplings,
  so a combined SI+SD likelihood configuration is unnecessary.
- No compiled GAMBIT installation or DDCalc backend is available and the task cannot
  run 'make -jn backends' (e.g. missing compilers, Boost, GSL, Eigen, or LAPACK prerequisites).
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: GAMBIT
  role: Framework that runs the YAML-driven scan and invokes the DarkBit direct detection
    likelihood pipeline
  repo: github.com/GambitBSM/gambit
- name: DarkBit
  role: GAMBIT module supplying direct detection likelihoods for arbitrary combinations
    of spin-independent and spin-dependent scattering processes
  repo: github.com/GambitBSM/gambit
- name: DDCalc
  role: Direct detection backend that models direct detection experiments (e.g. XENON1T)
    and computes the combined SI+SD likelihood
- name: gamLike
  role: Dedicated likelihood calculator interface through which the DDCalc direct
    detection likelihood is extracted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:80fb14223a79a0266dfedf90600f8bfdd04364302621c8a6bff576630b5a75fe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DDCalc
  - DarkBit
  - GAMBIT
  - gamLike
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# spin-independent-spin-dependent-scattering-combination

## Summary

Computes a direct detection likelihood for a dark matter benchmark point with both spin-independent (SI) and spin-dependent (SD) couplings by running GAMBIT's DarkBit module backed by the DDCalc direct detection package and extracting the value via gamLike.

## When to use

Use when a task requires a direct detection likelihood for arbitrary combinations of spin-independent and spin-dependent WIMP scattering processes at one or more benchmark dark matter model points.

## When NOT to use

- The analysis only needs indirect detection (gamma-ray or neutrino telescope) likelihoods, which DarkBit handles via its cascade-decay Monte Carlo signal yield calculator or the nulike package instead of DDCalc.
- The dark matter model point has only spin-independent or only spin-dependent couplings, so a combined SI+SD likelihood configuration is unnecessary.
- No compiled GAMBIT installation or DDCalc backend is available and the task cannot run 'make -jn backends' (e.g. missing compilers, Boost, GSL, Eigen, or LAPACK prerequisites).

## Inputs

- GAMBIT YAML scan input file specifying one benchmark dark matter model point with both spin-independent and spin-dependent couplings
- Direct detection experiment selection (e.g. XENON1T) configured for DDCalc
- Compiled DDCalc backend installed via the GAMBIT backend system

## Outputs

- ddcalc_likelihood.json containing a finite numeric direct detection likelihood value for the benchmark point

## How to apply

Create a GAMBIT YAML scan input file in the yaml_files folder that specifies a single benchmark dark matter model point with both SI and SD couplings, selecting a single direct detection likelihood (e.g. XENON1T) for DDCalc. Build the external codes with 'make -jn backends' (GAMBIT uses a flexible 'backend' system for interfacing external codes) and verify the installation with 'gambit backends'. Launch the single-point evaluation with 'gambit -f <input.yaml>' so GAMBIT invokes the DarkBit direct detection likelihood pipeline, which DDCalc backs by modeling the direct detection experiment. Extract the computed likelihood through the gamLike likelihood interface and write the numeric value to ddcalc_likelihood.json. DDCalc is appropriate here because it models a large range of direct detection experiments and supplies likelihoods for arbitrary SI+SD combinations, whereas narrower likelihood tools would not.

## Related tools

- **GAMBIT** (Framework that runs the YAML-driven scan and invokes the DarkBit direct detection likelihood pipeline) — github.com/GambitBSM/gambit
- **DarkBit** (GAMBIT module supplying direct detection likelihoods for arbitrary combinations of spin-independent and spin-dependent scattering processes) — github.com/GambitBSM/gambit
- **DDCalc** (Direct detection backend that models direct detection experiments (e.g. XENON1T) and computes the combined SI+SD likelihood)
- **gamLike** (Dedicated likelihood calculator interface through which the DDCalc direct detection likelihood is extracted)

## Examples

```
gambit -f yaml_files/DM_benchmark_SI_SD.yaml
```

## Evaluation signals

- ddcalc_likelihood.json exists after the run.
- The value in ddcalc_likelihood.json is a finite numeric likelihood for the benchmark point (not NaN or an error string).
- 'gambit backends' reports DDCalc as an installed backend before the scan is launched.
- The YAML input specifies exactly one benchmark point and a single direct detection likelihood (XENON1T), so the output corresponds to one model point rather than a scan grid.
- The likelihood was obtained through the gamLike interface backed by DDCalc, confirming the combined SI+SD configuration was actually used.

## Limitations

- DDCalc covers direct detection experiments only; neutrino telescope likelihoods require nulike and gamma-ray yields require DarkBit's cascade-decay Monte Carlo calculator.
- The workflow requires building backends first ('make -jn backends'); running GAMBIT without verifying backend installation with 'gambit backends' will fail at likelihood evaluation.
- GAMBIT build requires specific prerequisites (g++/gfortran 5.1+, CMake 2.8.12+, Boost 1.41+, GSL 2.1+, Eigen 3.1.0+, LAPACK), and cmake may fail to find some dependencies without guidance.

## Evidence

- [abstract] DarkBit's DDCalc package models a large range of direct detection experiments and provides likelihoods for arbitrary combinations of spin-independent and spin-dependent scattering processes.: "a direct detection package that models a large range of direct detection experiments ( DDCalc )"
- [abstract] DarkBit provides likelihoods for arbitrary combinations of spin-independent and spin-dependent scattering processes.: "provides the corresponding likelihoods for arbitrary combinations of spin-independent and spin-dependent scattering processes"
- [methods] GAMBIT's backend system interfaces external codes like DDCalc, built via 'make -jn backends' and verified with 'gambit backends'.: [evidence span withheld — anchor status: not_found]
- [methods] GAMBIT scans are launched from YAML input files with 'gambit -f', as in the example 'gambit -f yaml_files/MSSM7.yaml'.: [evidence span withheld — anchor status: not_found]
- [abstract] gamLike is the dedicated likelihood calculator through which experiment likelihoods are accessed.: "a dedicated likelihood calculator for current and future experiments ( gamLike )"
