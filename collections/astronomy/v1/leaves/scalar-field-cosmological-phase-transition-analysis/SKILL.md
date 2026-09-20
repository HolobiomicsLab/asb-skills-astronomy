---
name: scalar-field-cosmological-phase-transition-analysis
description: Use when a scalar-field potential (1D/2D test models, singlet, R2HDM/C2HDM/N2HDM)
  must be analyzed for cosmological phase transitions and critical temperatures.
when_to_use_negative:
- The task is scanning parameter points for a singlet model rather than tracing phases
  and transitions of a single given potential (use ./bin/scan_Z2_scalar_singlet_model
  instead)
- The goal is only cross-checking BSMPT transition results; BSMPT integration is not
  necessary for PhaseTracer users unless they wish to use potentials from BSMPT
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: PhaseTracer
  role: Core C++14 package that traces the minima of the effective potential, calculates
    critical temperatures, and generates phase/potential plots via the example binaries
  repo: https://github.com/PhaseTracer/PhaseTracer
- name: alglib
  role: Dependency installed for the build (libalglib-dev); if not found, built from
    source via alglib-cmake
  repo: https://github.com/S-Dafarra/alglib-cmake
- name: NLopt
  role: Dependency (libnlopt-cxx-dev) installed via package manager before building
    PhaseTracer
- name: Eigen3
  role: Dependency (libeigen3-dev) installed via package manager before building PhaseTracer
- name: Boost
  role: Dependency (libboost-filesystem-dev, libboost-log-dev) installed via package
    manager before building PhaseTracer
- name: GSL
  role: Dependency (libgsl-dev) installed via package manager before building PhaseTracer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1140/epjc/s10052-020-8035-2
    title: 'PhaseTracer: tracing cosmological phases and calculating transition properties'
  source_content_hash: sha256:488acf9c04b2295f9a3859eb90c60708834b3f0a5eea4b515fae0103d5e5b342
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PhaseTracer/PhaseTracer
  tools:
  - Boost
  - C++14
  - Eigen3
  - GSL
  - NLopt
  - PhaseTracer
  - alglib
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4074074074074074
n_claims: 27
---

# scalar-field-cosmological-phase-transition-analysis

## Summary

Run PhaseTracer example binaries (e.g. ./bin/run_2D_test_model with the -d flag) to trace minima of the effective potential as temperature changes, identify critical temperatures where minima are degenerate, and produce phase/potential plots and a phase-transition table.

## When to use

Use when a scalar-field potential (1D/2D test models, singlet, R2HDM/C2HDM/N2HDM) must be analyzed for cosmological phase transitions and critical temperatures.

## When NOT to use

- The task is scanning parameter points for a singlet model rather than tracing phases and transitions of a single given potential (use ./bin/scan_Z2_scalar_singlet_model instead)
- The goal is only cross-checking BSMPT transition results; BSMPT integration is not necessary for PhaseTracer users unless they wish to use potentials from BSMPT

## Inputs

- A scalar-field effective potential for a Standard Model extension (any number of scalar fields), e.g. the shipped 1D/2D test models
- Optionally potentials supplied by other packages (FlexibleSUSY, BSMPT) via build-time integration flags

## Outputs

- Traced minima of the effective potential as a function of temperature (phase trace from debugging output)
- Critical temperatures at which the traced minima are degenerate
- phase_plot.png phase-plot figure and phase_transition_table.tsv table

## How to apply

Install the C++14 dependencies (libalglib-dev, libnlopt-cxx-dev, libeigen3-dev, libboost-filesystem-dev, libboost-log-dev, libgsl-dev on Ubuntu/Debian), then clone github.com/PhaseTracer/PhaseTracer and build with mkdir build && cd build && cmake .. && make. From the repository root, run the compiled example binary — for the 2D test model ./bin/run_2D_test_model — adding the -d flag to enable debugging information and plots of the phases and potential. Parse the debugging output to collect the minima of the effective potential traced as a function of temperature, then extract the critical temperatures at which the traced minima are degenerate. Render the phase-plot figure (phase_plot.png) and write the phase/transition table (phase_transition_table.tsv). For bounce-action work, optionally rebuild with cmake -D BUILD_WITH_BP=ON to integrate BubbleProfiler, or use BUILD_WITH_FS=ON (FlexibleSUSY) / BUILD_WITH_BSMPT=ON (BSMPT potentials).

## Related tools

- **PhaseTracer** (Core C++14 package that traces the minima of the effective potential, calculates critical temperatures, and generates phase/potential plots via the example binaries) — https://github.com/PhaseTracer/PhaseTracer
- **alglib** (Dependency installed for the build (libalglib-dev); if not found, built from source via alglib-cmake) — https://github.com/S-Dafarra/alglib-cmake
- **NLopt** (Dependency (libnlopt-cxx-dev) installed via package manager before building PhaseTracer)
- **Eigen3** (Dependency (libeigen3-dev) installed via package manager before building PhaseTracer)
- **Boost** (Dependency (libboost-filesystem-dev, libboost-log-dev) installed via package manager before building PhaseTracer)
- **GSL** (Dependency (libgsl-dev) installed via package manager before building PhaseTracer)

## Examples

```
git clone https://github.com/PhaseTracer/PhaseTracer; cd PhaseTracer; mkdir build; cd build; cmake ..; make; cd ..; ./bin/run_2D_test_model -d
```

## Evaluation signals

- The run completes with traced phases and identified critical temperatures at which the minima are degenerate
- The phase-plot figure (phase_plot.png) is generated without errors when running with the -d flag
- The phase/transition table (phase_transition_table.tsv) is written with the extracted critical temperatures
- The build succeeds with a C++14 compliant compiler after the dependency apt/dnf/brew install, so the ./bin examples and ./bin/unit_tests run
- If BSMPT/FlexibleSUSY/BubbleProfiler integrations are enabled, their example binaries (run_R2HDM, run_THDMIISNMSSMBCsimple, run_BP_2d) execute consistently

## Limitations

- Plots of the phases and potential with the -d flag are only available for the first two examples (run_1D_test_model, run_2D_test_model)
- Unsuccessful compilation of BSMPT will not affect other examples; BSMPT is not necessary unless using BSMPT potentials
- FlexibleSUSY has additional dependencies and will report errors if these are not present
- alglib may not be found by cmake; use the alglib-cmake standalone build and set ALGLIB_DIR/CMAKE_PREFIX_PATH
- No artifact availability statement (e.g. DOI, Zenodo archive) beyond the GitHub source reference

## Evidence

- [abstract] Traces minima of the effective potential as temperature changes: "traces the minima of effective potential as the temperature changes, and then calculates the critical temperatures, at which the minima are degenerate"
- [methods] Build and run 2D example: "git clone https://github.com/PhaseTracer/PhaseTracer
cd PhaseTracer
mkdir build
cd build
cmake ..
make"
- [methods] Enable debugging and plots with -d flag: "If you want to see debugging information or obtain plots of the phases and potential for the first two examples above you can add the -d flag"
- [readme] Dependencies via package manager: "sudo apt install libalglib-dev libnlopt-cxx-dev libeigen3-dev libboost-filesystem-dev libboost-log-dev libgsl-dev"
- [readme] Run 2D example with -d flag: "./bin/run_1D_test_model -d 
    ./bin/run_2D_test_model -d"
