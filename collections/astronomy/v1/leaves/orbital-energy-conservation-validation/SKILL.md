---
name: orbital-energy-conservation-validation
description: Use when a new or adapted symplectic integrator (isochrone splitting,
  Leapfrog) must be accepted/rejected based on total-energy drift of a test orbit,
  or when comparing energy error per unit cost between two drift-kick schemes.
when_to_use_negative:
- The potential is one whose motion is NOT well approximated by isochrone trajectories
  and the goal is to beat the baseline in that regime — the scheme only performs at
  least as well as Leapfrog there (regime A / highly elongated orbits), not orders
  of magnitude better
- You need a validated integrator for production runs rather than validating or benchmarking
  the integration scheme itself — this skill is a pedagogical demonstration workflow,
  meant as an intuitive introduction to the isochrone splitting
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: IsochroneSymplecticIntegrator_QuickExample
  role: Provides the isochrone-splitting integrator being validated; its test-particle-in-Plummer-potential
    demo is the vehicle for the energy-drift check
  repo: github.com/AstroLexandre/IsochroneSymplecticIntegrator_QuickExample
- name: gfortran
  role: Fortran compiler required to build the integrator modules before any orbit
    integration
- name: make
  role: Builds the project through the Makefile (after mkdir obj mod out) so the executable
    can be run
- name: run_IsochroneSplitting_QuickExample.x
  role: Executable invoked with param.txt and an output file path to run the integration
    and record the energy series
  repo: github.com/AstroLexandre/IsochroneSymplecticIntegrator_QuickExample
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202553886
    title: An efficient integrator for stellar dynamics in effective gravity fields
      based on the isochrone potential
  source_content_hash: sha256:bc4f3036bd3d5e5fdbce252f22ca1488b6c8c08e8a6634909586e2eb6e3e5243
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsochroneSymplecticIntegrator_QuickExample
  - gfortran
  - make
  - run_IsochroneSplitting_QuickExample.x
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Orbital energy conservation validation

## Summary

Validates a symplectic orbital integrator (e.g., the isochrone drift-kick splitting scheme) by integrating a test orbit, recording total energy at every step, and checking that the relative total-energy drift stays bounded at the level expected of a second-order symplectic scheme; also supports cost-vs-error comparison against a Leapfrog baseline.

## When to use

Use when a new or adapted symplectic integrator (isochrone splitting, Leapfrog) must be accepted/rejected based on total-energy drift of a test orbit, or when comparing energy error per unit cost between two drift-kick schemes.

## When NOT to use

- The potential is one whose motion is NOT well approximated by isochrone trajectories and the goal is to beat the baseline in that regime — the scheme only performs at least as well as Leapfrog there (regime A / highly elongated orbits), not orders of magnitude better
- You need a validated integrator for production runs rather than validating or benchmarking the integration scheme itself — this skill is a pedagogical demonstration workflow, meant as an intuitive introduction to the isochrone splitting

## Inputs

- in/param.txt with potential parameters, integration settings, isochrone splitting parameters, and initial conditions
- Initial conditions for a test orbit (e.g., in a Plummer or Hénon isochrone potential) and a configured time step
- Baseline integrator module SymplecticIntegrator.f90 (Leapfrog) for comparison runs

## Outputs

- Trajectory and energy series file (e.g., out/energy_conservation.dat) with position, velocity, and total energy per step
- Energy conservation error versus computational cost comparison table (e.g., plummer_energy_cost_comparison.csv)

## How to apply

Compile the Fortran project with gfortran via make, first creating the obj, mod, and out directories (mkdir obj mod out). Configure in/param.txt with the potential parameters, integration settings, isochrone splitting parameters, and initial conditions — either a highly elongated orbit crossing the characteristic length of the potential (regime A) or an orbit in the inner/outer region where motion is well approximated by isochrone trajectories (regime B). Run the executable with the parameter file and an output path so position, velocity, and total energy are recorded at each step, using the Kepler-equation solver machinery in KeplerEquation.f90 wherever the isochrone propagator requires it. Compute the relative total-energy drift over the integration and write the trajectory and energy series to the output file. For a baseline comparison, repeat the run with SymplecticIntegrator.f90 (standard second-order Leapfrog) and assemble an energy-error-versus-computational-cost table (e.g., wall-clock time or step count) for both integrators in both regimes.

## Related tools

- **IsochroneSymplecticIntegrator_QuickExample** (Provides the isochrone-splitting integrator being validated; its test-particle-in-Plummer-potential demo is the vehicle for the energy-drift check) — github.com/AstroLexandre/IsochroneSymplecticIntegrator_QuickExample
- **gfortran** (Fortran compiler required to build the integrator modules before any orbit integration)
- **make** (Builds the project through the Makefile (after mkdir obj mod out) so the executable can be run)
- **run_IsochroneSplitting_QuickExample.x** (Executable invoked with param.txt and an output file path to run the integration and record the energy series) — github.com/AstroLexandre/IsochroneSymplecticIntegrator_QuickExample

## Examples

```
./run_IsochroneSplitting_QuickExample.x in/param.txt out/energy_conservation.dat
```

## Evaluation signals

- The project compiles with make and the executable runs successfully on the configured param.txt
- The relative total-energy drift over the integration stays bounded at the level expected of a second-order symplectic scheme, checked by inspecting the energy column of the output file
- In the efficiency comparison, the isochrone-splitting scheme shows energy error per unit cost at least as low as Leapfrog in both regimes
- In the regime where motion is well approximated by isochrone trajectories (inner/outer Plummer regions), the gain over Leapfrog is substantially larger — up to orders of magnitude
- The output file contains position, velocity, and total energy recorded at each step of the integration

## Limitations

- Gains over the Leapfrog baseline are regime-dependent: orders of magnitude only in regions where the motion of stars is well approximated by isochrone trajectories; in highly elongated orbit regimes it performs at best equivalently
- The bundled QuickExample is a pedagogical demonstration for a test particle in a Plummer potential; adapting to more complex potentials or higher-order symplectic integrators requires code modification
- No software version numbers are specified and no changelog is provided in the repository, so exact provenance of the binary may be hard to pin down

## Evidence

- [methods] Validation of second-order energy conservation: "Before running the compilation, create the required directories if they do not exist: `mkdir obj mod out`."
- [methods] Configuration of test runs: "`param.txt` lets you configure the potential parameters, integration settings, isochrone splitting parameters, and initial conditions."
- [readme] Run command producing the energy series: "./run_IsochroneSplitting_QuickExample.x in/param.txt out/YourOutputFile.dat"
- [abstract] Regime-dependent efficiency finding: "it performs at least as well, and up to orders of magnitude better, depending on the dynamical regime of the star."
- [intro] Scope of the demo code: "The program provides a simple demonstration of how to use the isochrone propagator within a symplectic integrator, offering an intuitive and pedagogical introduction to the isochrone splitting"
