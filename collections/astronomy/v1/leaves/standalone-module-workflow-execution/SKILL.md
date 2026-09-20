---
name: standalone-module-workflow-execution
description: Use when a specific GAMBIT module (e.g. DarkBit) must be exercised on
  a single benchmark point from a yaml parameter file — e.g. computing the thermal
  freeze-out relic density — instead of launching a full statistical scan.
when_to_use_negative:
- A statistical global fit or parameter scan over many points is needed — run the
  full GAMBIT scan instead (e.g. `gambit -f yaml_files/MSSM7.yaml`)
- The target observables are direct detection, gamma-ray, or neutrino-telescope likelihoods
  handled by DDCalc, gamLike, or nulike rather than the relic density routines
- The goal is the paper's quantitative comparison between DarkSUSY and micrOMEGAs
  rather than computing one module's output at a fixed point
tools:
- name: DarkBit
  role: GAMBIT module whose stand-alone relic density example computes dark matter
    observables at a fixed benchmark point
  repo: github.com/GambitBSM/gambit_1.5
- name: GAMBIT
  role: Framework providing the standalone module examples, yaml input files, cmake
    build system, and backend machinery
  repo: github.com/GambitBSM/gambit_1.5
- name: DarkSUSY
  role: External backend whose compute routines are called by DarkBit during the freeze-out
    / Boltzmann-equation calculation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:9e7df232a62a2651e891a74abb20cc44bd8cc5e9bcea278d8a2beaa21c6f6d96
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DarkBit
  - DarkSUSY
  - GAMBIT
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# standalone-module-workflow-execution

## Summary

Executes a GAMBIT module such as DarkBit in stand-alone mode on a single benchmark model point, computing observables (e.g. dark matter relic density Ω h² via the Boltzmann equation) without running a full framework scan. Useful for quick-start validation, benchmark reproducibility, and isolating one module's compute routines.

## When to use

Use when a specific GAMBIT module (e.g. DarkBit) must be exercised on a single benchmark point from a yaml parameter file — e.g. computing the thermal freeze-out relic density — instead of launching a full statistical scan.

## When NOT to use

- A statistical global fit or parameter scan over many points is needed — run the full GAMBIT scan instead (e.g. `gambit -f yaml_files/MSSM7.yaml`)
- The target observables are direct detection, gamma-ray, or neutrino-telescope likelihoods handled by DDCalc, gamLike, or nulike rather than the relic density routines
- The goal is the paper's quantitative comparison between DarkSUSY and micrOMEGAs rather than computing one module's output at a fixed point

## Inputs

- Benchmark model-point parameter file (yaml) from the GAMBIT yaml_files folder fixing the particle spectrum and annihilation channels
- Standalone module example target shipped with GAMBIT (e.g. cond_darkbit_standalone for the DarkBit relic density example)
- Configured external backends (DarkSUSY, micrOMEGAs) downloaded via GAMBIT's backend download scripts

## Outputs

- Computed dark matter relic density (Omega*h^2) for the benchmark point, parsed from the standalone routine output
- Model-point name paired with the Omega*h^2 value, e.g. written to relic_density.csv

## How to apply

First build the framework: clone or unpack the GAMBIT repository, run `cd build; cmake ..`, then `make -jn scanners`, `make -jn gambit` and `make -jn backends` (g++/gfortran ≥5.1, CMake ≥2.8.12, Boost ≥1.41, GSL ≥2.1, Eigen ≥3.1.0, LAPACK are required), and verify with `gambit backends` / `gambit scanners`. Then launch the module's stand-alone example — for relic density this is the DarkBit stand-alone relic density example (cond_darkbit_standalone) shipped with GAMBIT — supplying the benchmark model-point yaml file from the yaml_files folder so the spectrum and annihilation channels at that single point are fixed. DarkBit evaluates the thermal freeze-out annihilation cross section, calling DarkSUSY compute routines through the backend system as needed, and integrates the Boltzmann equation to obtain the relic abundance. Parse the routine output for the computed relic density, record the model-point name together with the Ω h² value (e.g. into relic_density.csv), and confirm the value is finite and physically sensible (~0.1 order) for the specified benchmark point.

## Related tools

- **DarkBit** (GAMBIT module whose stand-alone relic density example computes dark matter observables at a fixed benchmark point) — github.com/GambitBSM/gambit_1.5
- **GAMBIT** (Framework providing the standalone module examples, yaml input files, cmake build system, and backend machinery) — github.com/GambitBSM/gambit_1.5
- **DarkSUSY** (External backend whose compute routines are called by DarkBit during the freeze-out / Boltzmann-equation calculation)

## Examples

```
cd build; cmake ..; make -j4 gambit; make -j4 backends; gambit -f yaml_files/MSSM7.yaml
```

## Evaluation signals

- The standalone run completes without errors and exits cleanly
- The recorded Omega*h^2 is a finite, physically sensible value reported by the DarkBit standalone output for the specified benchmark point (thermally produced WIMP relic densities are of order 0.1)
- `gambit backends` and `gambit scanners` list the installed backends (e.g. DarkSUSY) and scanners before running
- Re-running the same benchmark yaml reproduces the same Omega*h^2, and the standalone value is consistent with the module's output inside a full GAMBIT scan at that point

## Limitations

- Stand-alone mode fixes the spectrum and annihilation channels at a single model point — no statistical scanning or global-fit likelihoods are performed
- cmake may fail to find some dependencies without guidance on some systems; the Core paper gives further build documentation
- Full GAMBIT builds are resource-intensive (the README notes building all of GAMBIT requires substantial resources; 16 cores referenced) and backends must be downloaded and configured separately

## Evidence

- [abstract] We introduce DarkBit , an advanced software code for computing dark matter constraints on various extensions to the Standard Model of particle physics: "We introduce DarkBit , an advanced software code for computing dark matter constraints on various extensions to the Standard Model of particle physics"
- [abstract] along with interfaces to DarkSUSY , micrOMEGAs , and the neutrino telescope likelihood package nulike: "along with interfaces to DarkSUSY , micrOMEGAs , and the neutrino telescope likelihood package nulike"
- [methods] gambit -f yaml_files/MSSM7.yaml: "gambit -f yaml_files/MSSM7.yaml"
- [readme] GAMBIT is built using the CMake system. The following libraries and packages must be installed prior to configuration: "GAMBIT is built using the CMake system. The following libraries and packages must be installed prior to configuration"
