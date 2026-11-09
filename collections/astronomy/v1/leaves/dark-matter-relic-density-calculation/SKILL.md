---
name: dark-matter-relic-density-calculation
description: Use when a task requires the dark matter relic density for a pinned benchmark
  model point (e.g. MSSM7), or a side-by-side DarkSUSY vs micrOMEGAs comparison on
  shared inputs.
when_to_use_negative:
- The goal is direct detection or gamma-ray likelihoods rather than relic density
  — those go through DDCalc, gamLike, or DarkBit's cascade-decay Monte Carlo signal
  yield calculator.
- The model point is not fixed (a full statistical scan is requested) — then use the
  GAMBIT scanning framework rather than a single benchmark relic density evaluation.
- Neither DarkSUSY nor micrOMEGAs backends can be built, and the DarkBit stand-alone
  routines are unavailable.
tools:
- name: DarkBit
  role: Provides the custom relic density routines that integrate the Boltzmann equation;
    runnable stand-alone or as a GAMBIT module
  repo: github.com/GambitBSM/gambit
- name: DarkSUSY
  role: External backend supplying compute routines for the annihilation cross section
    and one relic density value for the comparison
- name: micrOMEGAs
  role: External backend (4.1) evaluating the identical benchmark point to yield the
    second relic density value
- name: GAMBIT
  role: Framework whose backend system passes the same spectrum and couplings to DarkSUSY
    and micrOMEGAs; built with CMake and invoked via 'gambit -f'
  repo: github.com/GambitBSM/gambit
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:acca6f68d581609c37c2957ae80799adfa4718f352430fed4552629d08491059
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DarkBit
  - DarkSUSY
  - GAMBIT
  - micrOMEGAs
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# dark-matter-relic-density-calculation

## Summary

Compute the thermal relic density (Omega*h^2) for a dark matter model using DarkBit's custom Boltzmann-equation routines, either stand-alone or via the GAMBIT backend system that routes the same spectrum to DarkSUSY and micrOMEGAs for cross-code comparison.

## When to use

Use when a task requires the dark matter relic density for a pinned benchmark model point (e.g. MSSM7), or a side-by-side DarkSUSY vs micrOMEGAs comparison on shared inputs.

## When NOT to use

- The goal is direct detection or gamma-ray likelihoods rather than relic density — those go through DDCalc, gamLike, or DarkBit's cascade-decay Monte Carlo signal yield calculator.
- The model point is not fixed (a full statistical scan is requested) — then use the GAMBIT scanning framework rather than a single benchmark relic density evaluation.
- Neither DarkSUSY nor micrOMEGAs backends can be built, and the DarkBit stand-alone routines are unavailable.

## Inputs

- Benchmark model-point parameter YAML file (e.g. MSSM7.yaml from the yaml_files folder)
- Built GAMBIT installation with DarkBit module and compiled backends (DarkSUSY, micrOMEGAs 4.1)
- Shared MSSM7 spectrum and couplings passed via the GAMBIT backend system

## Outputs

- relic_density.csv (model point name with Omega*h^2 from the DarkBit standalone run)
- relic_density_comparison.csv (one row per backend with relic density values and the quantified relative difference)

## How to apply

Build GAMBIT from the retrieved repository with CMake ('cd build; cmake ..; make -jn gambit'), compile external backends with 'make -jn backends', and confirm installation with 'gambit backends'. For a stand-alone calculation, launch the DarkBit standalone example (cond_darkbit_standalone) with a benchmark parameter file from yaml_files so the spectrum and annihilation channels are fixed, letting DarkBit evaluate the freeze-out annihilation cross section (calling DarkSUSY compute routines as needed) and integrate the Boltzmann equation to obtain the relic abundance. For the cross-code check, run 'gambit -f yaml_files/MSSM7.yaml' with conditions routing to DarkSUSY (cond_darksusy_comparison) and micrOMEGAs 4.1 (cond_micromegas_comparison) so the identical benchmark spectrum and couplings reach both backends. Tabulate the two Omega*h^2 values and their relative difference as the quantitative consistency check, writing results to relic_density.csv / relic_density_comparison.csv.

## Related tools

- **DarkBit** (Provides the custom relic density routines that integrate the Boltzmann equation; runnable stand-alone or as a GAMBIT module) — github.com/GambitBSM/gambit
- **DarkSUSY** (External backend supplying compute routines for the annihilation cross section and one relic density value for the comparison)
- **micrOMEGAs** (External backend (4.1) evaluating the identical benchmark point to yield the second relic density value)
- **GAMBIT** (Framework whose backend system passes the same spectrum and couplings to DarkSUSY and micrOMEGAs; built with CMake and invoked via 'gambit -f') — github.com/GambitBSM/gambit

## Examples

```
cd build; cmake ..; make -j4 gambit; make -j4 backends; gambit backends; gambit -f yaml_files/MSSM7.yaml
```

## Evaluation signals

- The DarkBit standalone run completes without errors and reports a finite, physically sensible Omega*h^2 value for the specified benchmark point.
- Both DarkSUSY and micrOMEGAs backends return finite, positive relic density values for the same benchmark point.
- relic_density_comparison.csv contains one row per backend with the relative difference between the two Omega*h^2 values explicitly quantified.
- 'gambit backends' lists both dark matter backends as installed before the scan is launched.

## Limitations

- Cross-code agreement is not guaranteed a priori; the paper's purpose is precisely to quantify the comparison between the two main dark matter codes, so a nonzero relative difference is expected and must be reported rather than hidden.
- For a full GAMBIT build, substantial dependencies are compulsory (g++/gfortran 5.1+, CMake 2.8.12+, Python with yaml, Boost 1.41+, GSL 2.1+, Eigen 3.1.0+, LAPACK).
- Backend versions matter: the comparison is demonstrated with micrOMEGAs 4.1, and cmake may fail to find dependencies without guidance.

## Evidence

- [abstract] Quick-start guide and standalone relic density workflow: [evidence span withheld — anchor status: not_found]
- [methods] Scan invocation: "gambit -f yaml_files/MSSM7.yaml"
- [abstract] Quantitative DarkSUSY vs micrOMEGAs comparison: "a quantitative comparison between two of the main dark matter codes ( DarkSUSY and micrOMEGAs )"
- [intro] Backend system description: "a flexible system for interfacing external codes (the "backend" system)"
- [readme] GAMBIT build system and dependencies: "GAMBIT is built using the CMake system. The following libraries and packages must be installed prior to configuration:"
