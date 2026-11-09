---
name: pvrs-primitive-variable-time-integration
description: Use when solving 1D hyperbolic conservation laws (e.g. special relativistic
  hydrodynamics) where primitive recovery from conservative variables is costly or
  algebraically cumbersome, or when benchmarking a scheme's convergence order (~2)
  on smooth profiles such as a static Gaussian.
when_to_use_negative:
- The system requires analytic knowledge of u(q) that is unavailable — although PVRS
  requires no analytical knowledge of u(q), it is designed specifically for coupled
  conservative systems, not non-conservative formulations
- The target is a multidimensional problem with source terms not covered by the 1D
  special relativistic hydrodynamics setup described in the article
- A traditional finite-volume flux treatment is intentionally bypassed, which would
  risk violating entropy and Rankine-Hugoniot jump conditions across strong shocks
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: aztekas
  role: HRSC finite-volume code implementing the PVRS technique for relativistic hydrodynamics;
    used to run the Gaussian convergence and shock-tube simulations
  repo: https://github.com/aztekas-code/aztekas-main
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0195494
    title: 'A direct Primitive Variable Recovery Scheme for hyperbolic conservative
      equations: The case of relativistic hydrodynamics'
  source_content_hash: sha256:6cdac5995f6e192e16e80dcdda2e354d4f1f8ec5998a3fc6eb1ffc8d438777c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aztekas-code/aztekas-main
  tools:
  - aztekas
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# pvrs-primitive-variable-time-integration

## Summary

A numerical scheme (PVRS) that evolves hyperbolic conservative systems by recovering primitive variables directly via the chain rule applied to the time term, avoiding the per-cell algebraic primitive recovery (e.g. solving a 10th-degree polynomial in relativistic hydrodynamics). In aztekas it pairs a traditional finite-volume flux update with an HLL approximate Riemann solver, minmod-limited piecewise reconstruction, and MoL-RK4 time integration.

## When to use

Use when solving 1D hyperbolic conservation laws (e.g. special relativistic hydrodynamics) where primitive recovery from conservative variables is costly or algebraically cumbersome, or when benchmarking a scheme's convergence order (~2) on smooth profiles such as a static Gaussian.

## When NOT to use

- The system requires analytic knowledge of u(q) that is unavailable — although PVRS requires no analytical knowledge of u(q), it is designed specifically for coupled conservative systems, not non-conservative formulations
- The target is a multidimensional problem with source terms not covered by the 1D special relativistic hydrodynamics setup described in the article
- A traditional finite-volume flux treatment is intentionally bypassed, which would risk violating entropy and Rankine-Hugoniot jump conditions across strong shocks

## Inputs

- Initial condition of primitive variables on a 1D grid (e.g. static Gaussian profile on domain [0,1])
- Grid resolutions Δx ∈ {1/200, 1/400, 1/800, 1/1600, 1/3200, 1/6400} with fixed time step Δt = 0.01
- aztekas simulation parameter file (e.g. shock.param) specifying the shock-tube or Gaussian setup

## Outputs

- Numerical solution of primitive variables per time step
- Per-resolution L1-norm errors and convergence orders written to gaussian_convergence.csv
- Shock-tube solution snapshots for comparison against exact analytic solutions (S1_File.tar.gz data)

## How to apply

Evolve the primitive variables directly by applying the chain rule to the time term of the conservative equations ∂Q/∂t + ∂F/∂x = S, while keeping a traditional finite-volume flux update so entropy and Rankine-Hugoniot jump conditions are not violated. Advance time with a forward finite difference scheme; in aztekas this is realized as an HLL approximate Riemann solver for interface fluxes, minmod-limited piecewise reconstruction of primitive variables, and a 4th-order Runge-Kutta Method of Lines (MoL-RK4) integrator. For validation, initialize a static Gaussian on [0,1] as the exact reference solution and repeat runs at Δx = 1/200, 1/400, 1/800, 1/1600, 1/3200, 1/6400 with a fixed Δt = 0.01, respecting the CFL constraint λΔt/Δx < 1. Compute the L1-norm error against the exact solution at each resolution and the convergence order as log(error_i/error_{i-1})/log(1/2); expect order ≈ 2 at all resolutions, and expect errors to be dominated by discontinuities in shock-tube tests rather than the smooth region.

## Related tools

- **aztekas** (HRSC finite-volume code implementing the PVRS technique for relativistic hydrodynamics; used to run the Gaussian convergence and shock-tube simulations) — https://github.com/aztekas-code/aztekas-main

## Examples

```
cd SIMULATIONS/HD/Shock-Tube && make clean && make && ./aztekas shock.param
```

## Evaluation signals

- Computed convergence order ≈ 2 at all six tested resolutions (Δx from 1/200 to 1/6400) for the static Gaussian with fixed Δt = 0.01
- CFL constraint λΔt/Δx < 1 satisfied at every resolution despite the fixed time step
- L1-norm errors decrease by roughly a factor of 2 per halving of Δx, consistent with order log(error_i/error_{i-1})/log(1/2) ≈ 2
- Shock-tube results match exact analytic solutions within the L1-norm, with errors dominated by discontinuities rather than smooth regions
- aztekas runs compile cleanly after 'make clean; make' and execute with './aztekas shock.param' without diverging

## Limitations

- In relativistic hydrodynamics, the traditional approach requires solving a 10th-degree algebraic polynomial per component at each time step, which adds computational cost as space-time resolution increases; PVRS avoids this but is a distinct scheme whose accuracy is still dominated by discontinuities
- Validation of ~2nd-order convergence applies to smooth solutions (static Gaussian); shock-tube errors are dominated by discontinuities, so order-2 behavior should not be expected there
- aztekas as described solves non-relativistic and relativistic Euler equations on a fixed background metric (Schwarzschild, Minkowski, Kerr-Schild), not arbitrary dynamical spacetimes

## Evidence

- [abstract] This method obtains directly the primitive variables applying the chain rule to the time term of the conservative equations: "This method obtains directly the primitive variables applying the chain rule to the time term of the conservative equations"
- [abstract] a traditional finite volume method for the flux is applied in order avoid violation of both, the entropy and "Rankine-Hugoniot" jump conditions: "a traditional finite volume method for the flux is applied in order avoid violation of both, the entropy and "Rankine-Hugoniot" jump conditions"
- [intro] at each time step, a 10th degree algebraic polynomial has to be solved for a unique given value of each component of the vector u: "at each time step, a 10th degree algebraic polynomial has to be solved for a unique given value of each component of the vector u"
- [abstract] The time evolution is then computed using a forward finite difference scheme.: "The time evolution is then computed using a forward finite difference scheme."
- [readme] aztekas is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes.: "aztekas is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes."
