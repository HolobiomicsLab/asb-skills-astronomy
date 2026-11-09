---
name: runge-kutta-method-of-lines-time-integration
description: Use when a conservative hyperbolic system (e.g. 1D SRHD charges q=(D,Sx,τ))
  has been discretised in space with a finite-volume update and needs stable, higher-order-in-time
  evolution with CFL-safe step control.
when_to_use_negative:
- When the spatial discretisation has not been kept in conservative finite-volume
  form — the paper notes finite volume for the flux is required to avoid violating
  the entropy and Rankine–Hugoniot jump conditions
- When a fixed Δt would violate the CFL restriction λΔt/Δx < 1 at the finest resolution,
  e.g. Δt=0.01 with λΔt/Δx ≥ 1
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: aztekas
  role: Solver that implements the MoL-RK4 time integration of the PVRS semi-discrete
    system with HLL Riemann solver and minmod limiter for the relativistic shock-tube
    and Gaussian convergence tests
  repo: https://github.com/aztekas-code/aztekas-main
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0195494
    title: 'A direct Primitive Variable Recovery Scheme for hyperbolic conservative
      equations: The case of relativistic hydrodynamics'
  source_content_hash: sha256:ce05d85fd89775fdd884876f4761db64ea6a214caf796bec75b225f992070649
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aztekas-code/aztekas-main
  tools:
  - aztekas
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# runge-kutta-method-of-lines-time-integration

## Summary

Time-integration skill that wraps the PVRS spatial discretisation (HLL fluxes with minmod reconstruction) as a semi-discrete right-hand side L(u) of ∂u/∂t = −L(u) and advances it with a 4th-order Runge-Kutta Method of Lines (MoL-RK4), as implemented in the aztekas solver for 1D relativistic hydrodynamics.

## When to use

Use when a conservative hyperbolic system (e.g. 1D SRHD charges q=(D,Sx,τ)) has been discretised in space with a finite-volume update and needs stable, higher-order-in-time evolution with CFL-safe step control.

## When NOT to use

- When the spatial discretisation has not been kept in conservative finite-volume form — the paper notes finite volume for the flux is required to avoid violating the entropy and Rankine–Hugoniot jump conditions
- When a fixed Δt would violate the CFL restriction λΔt/Δx < 1 at the finest resolution, e.g. Δt=0.01 with λΔt/Δx ≥ 1

## Inputs

- Semi-discrete RHS L(u) built from the PVRS finite-volume update (Eq 13) with HLL fluxes (Eq 45) and minmod reconstruction (Eqs 50–57)
- Initial primitive vector u=(p,v,n) or conservative charges q=(D,Sx,τ) on a 1D grid (e.g. domain [0,1], initial discontinuity at x=0.5)
- Fixed time step Δt chosen below the CFL bound λΔt/Δx < 1 (e.g. Δt = 0.01)

## Outputs

- Time-evolved primitive variable profiles u(xi,tn+1) over the 1D grid
- L1-norm errors and per-resolution convergence orders log(error_i/error_{i−1})/log(1/2) (e.g. written to gaussian_convergence.csv)

## How to apply

First assemble the spatial operator: at each cell xi compute the HLL numerical flux f^HLL=(λR fL−λL fR+λR λL(qR−qL))/(λR−λL) at interfaces xi±1/2 from minmod-limited piecewise-linear reconstructions, and form the PVRS update ua(xi,tn+1)=ua(xi,tn)−(Δt/Δx)Aab([Fb_HLL]_{i+1/2}−[Fb_HLL]_{i−1/2}), where A=(∂q/∂u)^−1 is evaluated at the cell midpoint while all other states use the reconstructed ũ (Eq 13). Wrap this as the semi-discrete RHS L(u) so ∂u/∂t = −L(u) (Eq 14), then advance with a 4th-order Runge-Kutta Method of Lines (MoL-RK4). Enforce the CFL-type restriction λΔt/Δx < 1 (Eq 40) by choosing a fixed Δt strictly smaller than the CFL bound — e.g. Δt = 0.01 for the static Gaussian convergence test. Validate by computing the L1-norm error against exact solutions and the convergence order log(error_i/error_{i−1})/log(1/2): expect ~2 on the smooth Gaussian at fixed Δt=0.01 and first-order on the relativistic Sod shock tubes at Δx from 1/200 to 1/6400.

## Related tools

- **aztekas** (Solver that implements the MoL-RK4 time integration of the PVRS semi-discrete system with HLL Riemann solver and minmod limiter for the relativistic shock-tube and Gaussian convergence tests) — https://github.com/aztekas-code/aztekas-main

## Examples

```
cd SIMULATIONS/HD/Shock-Tube && make clean && make && ./aztekas shock.param
```

## Evaluation signals

- Static Gaussian test with fixed Δt = 0.01 yields convergence order ≈ 2 at all resolutions Δx = 1/200 … 1/6400
- Relativistic Sod blast-wave tests show monotonically decreasing L1-norm errors and first-order convergence in at least one resolution
- The chosen fixed Δt satisfies λΔt/Δx < 1 (Eq 40) at every resolution used
- RK4 advances the same semi-discrete RHS L(u) = −(Δt/Δx)Aab([F_HLL]_{i+1/2} − [F_HLL]_{i−1/2}) that produces the Eq 13 update, so a single-step consistency check on a smooth static profile passes at first order

## Limitations

- First-order convergence on shock-tube problems: discontinuities dominate the error, so ~2nd order is only attainable on smooth profiles
- Convergence order is estimated from successive resolution pairs as log(error_i/error_{i−1})/log(1/2), so a single resolution pair may not exhibit the expected order
- Relativistic schemes normally need primitive recovery via a 10th-degree polynomial per cell per step; PVRS sidesteps this but the method was demonstrated only for 1D relativistic hydrodynamics

## Evidence

- [abstract] Time integration via MoL-RK4 wrapping the PVRS update: "The time evolution is then computed using a forward finite difference scheme."
- [abstract] Rationale for keeping the spatial term in finite-volume form: "a traditional finite volume method for the flux is applied in order avoid violation of both, the entropy and "Rankine-Hugoniot" jump conditions"
- [intro] Motivation: cost of per-step primitive recovery in relativistic hydrodynamics: "at each time step, a 10th degree algebraic polynomial has to be solved for a unique given value of each component of the vector u"
- [readme] aztekas as the implementing solver for HRSC schemes: "aztekas is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes."
