---
name: shock-tube-riemann-problem-setup
description: 'Use when validating a numerical scheme for 1D special relativistic hydrodynamics
  (1DRHD) against a known exact solution: a shock tube with a discontinuity at x =
  0.5, evolved under the CFL bound lambda*dt/dx < 1 (Eq 40).'
when_to_use_negative:
- The PDE system is not hyperbolic-conservative (e.g. the non-conservative primitive-variable
  Eq 10 alone), since a Riemann-problem validation presumes conservative fluxes that
  satisfy the entropy and Rankine-Hugoniot jump conditions.
- No exact analytic Riemann solution or reference data (S1_File.tar.gz-style) is available
  for the problem's equation of state e = nm + p/(kappa-1), so L1-norm validation
  cannot be performed.
tools:
- name: aztekas
  role: Runs the conservative finite-volume shock-tube simulation; the Shock-Tube
    example lives in SIMULATIONS/HD/Shock-Tube and is driven by shock.param.
  repo: github.com/aztekas-code/aztekas-main
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0195494
    title: 'A direct Primitive Variable Recovery Scheme for hyperbolic conservative
      equations: The case of relativistic hydrodynamics'
  source_content_hash: sha256:06787eb6da4e196a1194993ee2eec79b31ddcf18502ece1b9070bf537332b028
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - aztekas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# shock-tube-riemann-problem-setup

## Summary

Sets up the relativistic Sod shock-tube (Riemann) problem as a discontinuous initial-value problem on [0,1] with left/right states from Table 1, so that conservative schemes can be validated against the exact Riemann solution via L1-norm errors.

## When to use

Use when validating a numerical scheme for 1D special relativistic hydrodynamics (1DRHD) against a known exact solution: a shock tube with a discontinuity at x = 0.5, evolved under the CFL bound lambda*dt/dx < 1 (Eq 40).

## When NOT to use

- The PDE system is not hyperbolic-conservative (e.g. the non-conservative primitive-variable Eq 10 alone), since a Riemann-problem validation presumes conservative fluxes that satisfy the entropy and Rankine-Hugoniot jump conditions.
- No exact analytic Riemann solution or reference data (S1_File.tar.gz-style) is available for the problem's equation of state e = nm + p/(kappa-1), so L1-norm validation cannot be performed.

## Inputs

- Relativistic Sod shock-tube initial left/right states (Table 1 of the paper)
- Parameter file shock.param (domain, cell count N, final time, time step)
- Exact Riemann solution reference data from S1_File.tar.gz (supplementary)

## Outputs

- Numerical density (and other primitive) profiles vs the exact Riemann solution, e.g. fdm_sod_profile.png
- Per-resolution L1-norm errors and convergence orders, e.g. fdm_l1_errors.csv

## How to apply

Initialise the relativistic Sod shock tube on the domain [0,1] with the discontinuity at x = 0.5, using the initial left/right states of Table 1, N = 800 cells, and a fixed time step satisfying the CFL bound of Eq (40) (lambda*dt/dx < 1). Evolve the conservative charges D, Sx, tau (Eqs 2-4) with a conservative finite-volume flux treatment for the spatial derivative, because a non-conservative discretisation violates the entropy and Rankine-Hugoniot jump conditions at the shock. At the chosen final time, interpolate the numerical solution onto the exact Riemann solution data shipped in S1_File.tar.gz and compute the L1-norm error plus the convergence order log(error_i/error_{i-1})/log(1/2) per resolution (e.g. dx = 1/200 ... 1/6400). With aztekas, the shock-tube case is prepared under SIMULATIONS/HD/Shock-Tube, compiled with make, and launched with ./aztekas shock.param.

## Related tools

- **aztekas** (Runs the conservative finite-volume shock-tube simulation; the Shock-Tube example lives in SIMULATIONS/HD/Shock-Tube and is driven by shock.param.) — github.com/aztekas-code/aztekas-main

## Examples

```
cd SIMULATIONS/HD/Shock-Tube && make clean && make && ./aztekas shock.param
```

## Evaluation signals

- The CFL constraint lambda*dt/dx < 1 (Eq 40) must hold at every resolution run.
- L1-norm error against the exact Riemann solution must decrease monotonically with resolution for a conservative scheme; a near-zero convergence order at the shock flags a non-conservative discretisation.
- Convergence order per resolution must match log(error_i/error_{i-1})/log(1/2) computed on the interpolated profiles.
- The initial discontinuity must sit at x = 0.5 on [0,1] with the Table 1 states reproduced exactly at t = 0.
- Conservative charges D, Sx, tau (Eqs 2-4) must remain consistent throughout the run (no drift), since the flux must be discretised conservatively.

## Limitations

- The convergence check presupposes an exact analytic Riemann solution; errors are visualised via fluctuations of numerical values relative to that exact solution.
- A finite-difference-only discretisation of the primitive-variable equation (Eq 10) is stable but not consistent, so it converges to a different weak solution — Riemann-problem setup alone does not guarantee correctness without a conservative flux.
- aztekas as shipped solves hydrodynamics (Euler equations) for a perfect fluid, optionally on fixed background metrics (Schwarzschild, Minkowski, Kerr-Schild); other systems need the setup script and new simulation files.

## Evidence

- [abstract] Conservative FV flux preserves jump conditions: "a traditional finite volume method for the flux is applied in order avoid violation of both, the entropy and "Rankine-Hugoniot" jump conditions"
- [intro] 1DRHD convergence tests are the target case: "We also show how this method can be applied in the particular case of 1D special relativistic hydrodynamics (1DRHD). For this particular case, we construct convergence tests."
- [supplementary] Exact Riemann reference data location: "In the file “S1_File.tar.gz”, we attached the relevant data of the comparison between numerical simulations and exact solutions used to obtain the L1—norm."
- [readme] aztekas is a conservative finite-volume HRSC solver: "_aztekas_ uses a conservative finite-volume approach to obtain the dicrete form of a hyperbolic partial differential system of equations (PDE)."
