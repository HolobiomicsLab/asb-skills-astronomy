---
name: freeze-out-annihilation-cross-section-interpretation
description: Use when a single fixed benchmark dark matter model point needs its thermal
  freeze-out annihilation cross section evaluated and its relic density (Omega*h^2)
  computed via DarkBit's Boltzmann-equation integration, rather than a full GAMBIT
  global scan.
when_to_use_negative:
- A full global parameter scan with statistical likelihoods over many model points
  is required — use the full GAMBIT scanning framework instead of the DarkBit stand-alone
  example
- The quantity of interest is a direct-detection or gamma-ray likelihood rather than
  the relic density — use DDCalc, gamLike or DarkBit's detection routines instead
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3450
tools:
- name: DarkBit
  role: Provides the relic density routines that evaluate the thermal freeze-out annihilation
    cross section and integrate the Boltzmann equation; run here as a stand-alone
    tool
  repo: github.com/GambitBSM/gambit
- name: DarkSUSY
  role: Backend whose compute routines are called by DarkBit to evaluate the annihilation
    cross section and related dark matter properties
- name: micrOMEGAs
  role: Alternative external dark matter code interfaced by DarkBit, usable for cross-checking
    relic density results
- name: GAMBIT
  role: Framework that ships the cond_darkbit_standalone standalone module example
    and provides yaml_files model-point parameter inputs
  repo: github.com/GambitBSM/gambit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:03ea4a53a17abb6b3b8c7ff348804043583e16abaebe062d4ddd8c205817aaf9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DarkBit
  - DarkSUSY
  - GAMBIT
  - micrOMEGAs
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3450
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# freeze-out-annihilation-cross-section-interpretation

## Summary

Interprets thermal freeze-out annihilation cross sections and relic abundance computations produced by DarkBit, which can run as a stand-alone tool or as a GAMBIT module, and which calls DarkSUSY compute routines as needed. It covers evaluating the annihilation cross section at a fixed benchmark model point and integrating the Boltzmann equation to obtain Omega*h^2.

## When to use

Use when a single fixed benchmark dark matter model point needs its thermal freeze-out annihilation cross section evaluated and its relic density (Omega*h^2) computed via DarkBit's Boltzmann-equation integration, rather than a full GAMBIT global scan.

## When NOT to use

- A full global parameter scan with statistical likelihoods over many model points is required — use the full GAMBIT scanning framework instead of the DarkBit stand-alone example
- The quantity of interest is a direct-detection or gamma-ray likelihood rather than the relic density — use DDCalc, gamLike or DarkBit's detection routines instead

## Inputs

- Benchmark model-point parameter file from GAMBIT's yaml_files folder (spectrum and annihilation channels fixed at a single point)
- DarkBit stand-alone relic density example configuration (cond_darkbit_standalone)
- Built GAMBIT/DarkBit installation with backends available

## Outputs

- Computed thermal freeze-out annihilation cross section for the benchmark point
- Dark matter relic density Omega*h^2 for the benchmark point
- relic_density.csv mapping the model point name to its Omega*h^2 value

## How to apply

Launch the DarkBit stand-alone relic density example (cond_darkbit_standalone), one of the standalone module examples shipped with GAMBIT, so dark matter observables are computed without running a full framework scan. Supply the benchmark model-point parameter file from the yaml_files folder as input to fix the spectrum and annihilation channels at that single point. Let DarkBit's routines evaluate the thermal freeze-out annihilation cross section — calling DarkSUSY compute routines as backend where needed — and integrate the Boltzmann equation to obtain the relic abundance for the benchmark point. Parse the routine output for the computed relic density and record the model point name together with its Omega*h^2 value (e.g. to relic_density.csv). Validate that the run completed without errors and that the reported Omega*h^2 is finite and physically sensible. Note that DarkBit also interfaces to external codes such as micrOMEGAs, so cross-code comparisons of the relic density are possible when more than one backend is available.

## Related tools

- **DarkBit** (Provides the relic density routines that evaluate the thermal freeze-out annihilation cross section and integrate the Boltzmann equation; run here as a stand-alone tool) — github.com/GambitBSM/gambit
- **DarkSUSY** (Backend whose compute routines are called by DarkBit to evaluate the annihilation cross section and related dark matter properties)
- **micrOMEGAs** (Alternative external dark matter code interfaced by DarkBit, usable for cross-checking relic density results)
- **GAMBIT** (Framework that ships the cond_darkbit_standalone standalone module example and provides yaml_files model-point parameter inputs) — github.com/GambitBSM/gambit

## Examples

```
gambit -f yaml_files/MSSM7.yaml
```

## Evaluation signals

- The stand-alone run completes without errors
- The recorded Omega*h^2 is finite and physically sensible (relic density of order the observed dark matter abundance)
- The output relic_density.csv pairs the specified benchmark model point name with the Omega*h^2 value reported by the DarkBit standalone output
- If multiple backends are enabled, relic densities from DarkSUSY and micrOMEGAs interfaces agree within their documented numerical differences

## Limitations

- The stand-alone example fixes a single benchmark model point; it does not explore parameter space the way a full GAMBIT scan does
- Results depend on the external backend (DarkSUSY or micrOMEGAs) being correctly downloaded, built and registered via GAMBIT's backend system
- Backends and their dependencies must be verified as installed (e.g. via 'gambit backends') before the relic density run can succeed

## Evidence

- [abstract] DarkBit includes custom relic density routines in addition to interfaces to external codes: "along with interfaces to DarkSUSY , micrOMEGAs , and the neutrino telescope likelihood package nulike"
- [other] DarkBit runs both stand-alone and as a GAMBIT module: [evidence span withheld — anchor status: not_found]
- [methods] Annihilation cross section evaluation calls DarkSUSY routines: [evidence span withheld — anchor status: not_found]
- [methods] GAMBIT build and backend verification steps precede the run: "gambit backends
  gambit scanners"
- [other] Benchmark point input comes from yaml_files: [evidence span withheld — anchor status: not_found]
