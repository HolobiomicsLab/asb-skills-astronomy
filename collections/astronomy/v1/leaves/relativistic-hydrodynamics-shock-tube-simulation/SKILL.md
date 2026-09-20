---
name: relativistic-hydrodynamics-shock-tube-simulation
description: 'Use when validating a conservative finite-volume solver for special
  relativistic hydrodynamics: 1D Sod shock-tube initial states must be evolved at
  multiple resolutions and L1 errors and convergence order quantified.'
when_to_use_negative:
- The problem involves non-relativistic Euler equations where standard primitive-variable
  recovery by solving an algebraic system is adequate and PVRS offers no benefit
- Exact analytical solutions are unavailable, so L1-norm errors and convergence order
  cannot be computed
- The task requires shock capture with flux updates that violate the entropy or Rankine-Hugoniot
  jump conditions (e.g., non-conservative formulations)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: aztekas
  role: Relativistic hydrodynamics solver using the PVRS technique; compiles the Shock-Tube
    simulation with HLL Riemann solver, minmod limiter, and MoL-RK4 integrator
  repo: github.com/aztekas-code/aztekas-main
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0195494
    title: 'A direct Primitive Variable Recovery Scheme for hyperbolic conservative
      equations: The case of relativistic hydrodynamics'
  source_content_hash: sha256:453c575ebde4b29316b4607564e6189d765d64f23d7f10171824ae07018202a8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - aztekas
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# relativistic-hydrodynamics-shock-tube-simulation

## Summary

Run and verify relativistic hydrodynamics shock-tube tests with the aztekas solver configured for the PVRS discretisation (HLL Riemann solver, minmod limiter, MoL-RK4), computing L1-norm errors against exact Sod solutions to demonstrate convergence.

## When to use

Use when validating a conservative finite-volume solver for special relativistic hydrodynamics: 1D Sod shock-tube initial states must be evolved at multiple resolutions and L1 errors and convergence order quantified.

## When NOT to use

- The problem involves non-relativistic Euler equations where standard primitive-variable recovery by solving an algebraic system is adequate and PVRS offers no benefit
- Exact analytical solutions are unavailable, so L1-norm errors and convergence order cannot be computed
- The task requires shock capture with flux updates that violate the entropy or Rankine-Hugoniot jump conditions (e.g., non-conservative formulations)

## Inputs

- Relativistic Sod shock-tube initial states (weak, mildly relativistic, strong blast waves) with left/right values from Table 1 of the article
- aztekas shock.param parameter file (domain [0,1], discontinuity at x=0.5, baseline N=800)
- Exact analytical solutions packaged in S1_File.tar.gz

## Outputs

- Numerical solution profiles (density, pressure, velocity) per test and per resolution (Δx1=1/200 … Δx6=1/6400)
- Per-test L1-norm error table across resolutions
- Per-test convergence orders computed as log(error_i/error_{i-1})/log(1/2)

## How to apply

Clone aztekas-main and compile in SIMULATIONS/HD/Shock-Tube with 'make clean; make'. Configure the PVRS discretisation with an HLL Riemann solver, minmod piecewise reconstruction, and 4th-order Runge-Kutta Method of Lines (MoL-RK4) time integration. Set up the three relativistic Sod tests (weak, mildly relativistic, strong blast waves) with the initial discontinuity at x = 0.5 on the domain [0, 1], N = 800 baseline cells, left/right states from Table 1. Run each test at resolutions Δx = 1/200 through 1/6400 with a fixed time step chosen sufficiently smaller than the CFL condition (Eq 40). Compute L1-norm errors against the exact analytical solutions in S1_File.tar.gz and the convergence order as log(error_i/error_{i-1})/log(1/2) for successive resolutions.

## Related tools

- **aztekas** (Relativistic hydrodynamics solver using the PVRS technique; compiles the Shock-Tube simulation with HLL Riemann solver, minmod limiter, and MoL-RK4 integrator) — github.com/aztekas-code/aztekas-main

## Examples

```
cd $AZTEKAS_PATH/SIMULATIONS/HD/Shock-Tube && make clean && make && ./aztekas shock.param
```

## Evaluation signals

- L1-norm error strictly decreases as resolution increases (Δx from 1/200 to 1/6400)
- First-order convergence attained for all three Sod tests at least at one resolution (log(error_i/error_{i-1})/log(1/2) ≈ 1)
- Numerical profiles match the exact analytical solutions from S1_File.tar.gz, including shock position, rarefaction fan, and contact discontinuity
- Fixed time step used per run remains sufficiently smaller than the CFL condition (Eq 40)
- Simulation completes without NaN/inf in conservative variables (energy, density, momentum)

## Limitations

- Recovering primitive variables in relativistic hydrodynamics requires solving a 10th-degree algebraic polynomial per component at each time step, adding computational cost as space-time resolution increases — the motivation for PVRS, but fluxes must still use traditional finite-volume updates to avoid violating entropy and Rankine-Hugoniot jump conditions
- Only 1D special relativistic hydrodynamics on a fixed background metric (Minkowski, Schwarzschild, Kerr-Schild) is covered in this solver version
- The time evolution in the basic PVRS formulation uses a forward finite difference scheme; higher-order accuracy comes from the MoL-RK4 wrapper, so first-order convergence is the demonstrated expectation

## Evidence

- [results] The L1-norm error decreases with increasing resolution: [evidence span withheld — anchor status: not_found]
- [readme] aztekas solves hyperbolic PDEs with HRSC: "aztekas is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes."
- [abstract] Convergence presented for standard shock-tube problems: "We present the convergence of the method for standard shock-tube problems of special relativistic hydrodynamics and a graphical visualisation of the errors"
- [supplementary] S1_File.tar.gz contains comparison data with exact solutions: [evidence span withheld — anchor status: not_found]
- [intro] 10th-degree polynomial recovery cost: "at each time step, a 10th degree algebraic polynomial has to be solved for a unique given value of each component of the vector u"
