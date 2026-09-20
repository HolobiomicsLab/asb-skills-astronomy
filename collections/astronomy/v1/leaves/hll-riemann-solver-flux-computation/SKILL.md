---
name: hll-riemann-solver-flux-computation
description: Use when a hyperbolic conservative system (e.g. 1D SRHD with charges
  q=(D,Sx,τ)) needs cell-interface numerical fluxes without solving an algebraic primitive-recovery
  polynomial per cell and time step.
when_to_use_negative:
- The system is not in hyperbolic conservative form (the PVRS+HLL machinery derives
  the update from ∂q/∂t + ∂f/∂x = 0 with A=(∂q/∂u)^−1)
- You need an exact (analytic) Riemann solution or a solver with resolved contact
  waves — HLL uses only two signal speeds λL, λR and smears contact discontinuities
- The spatial term is to be treated by finite differences instead of the conservative
  finite-volume flux, which the paper warns violates entropy and Rankine-Hugoniot
  jump conditions
edam_operation: http://edamontology.org/operation_2426
tools:
- name: aztekas
  role: Solver implementing the PVRS discretisation with the HLL approximate Riemann
    solver, minmod limiter and MoL-RK4 integrator; the Shock-Tube simulation directory
    is compiled and run to exercise the flux computation
  repo: https://github.com/aztekas-code/aztekas-main
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0195494
    title: 'A direct Primitive Variable Recovery Scheme for hyperbolic conservative
      equations: The case of relativistic hydrodynamics'
  source_content_hash: sha256:e9125b1b26ee63de96895470d1386de66e50fd9a0b9ea1fe1e6415807f58a950
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aztekas-code/aztekas-main
  tools:
  - aztekas
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hll-riemann-solver-flux-computation

## Summary

Compute interface numerical fluxes with a two-wave HLL approximate Riemann solver (Eq 45), the spatial-discretisation core of the PVRS scheme implemented in aztekas for 1D special relativistic hydrodynamics. The HLL flux keeps the spatial term in conservative finite-volume form so entropy and Rankine-Hugoniot jump conditions are preserved while the primitive variables u=(p,v,n) are evolved directly.

## When to use

Use when a hyperbolic conservative system (e.g. 1D SRHD with charges q=(D,Sx,τ)) needs cell-interface numerical fluxes without solving an algebraic primitive-recovery polynomial per cell and time step.

## When NOT to use

- The system is not in hyperbolic conservative form (the PVRS+HLL machinery derives the update from ∂q/∂t + ∂f/∂x = 0 with A=(∂q/∂u)^−1)
- You need an exact (analytic) Riemann solution or a solver with resolved contact waves — HLL uses only two signal speeds λL, λR and smears contact discontinuities
- The spatial term is to be treated by finite differences instead of the conservative finite-volume flux, which the paper warns violates entropy and Rankine-Hugoniot jump conditions

## Inputs

- Primitive variable vector u=(p,v,n) per cell of a 1D grid on domain [0,1] (e.g. relativistic Sod left/right states from Table 1 with discontinuity at x=0.5, or a static Gaussian profile)
- Conservative charges q=(D,Sx,τ) and fluxes f expressed as functions of u for the polytropic EOS e=nm+p/(κ−1)
- Interface signal speeds λL, λR and left/right reconstructed states ũ (minmod-limited, Eqs 50-57)
- Grid resolution Δx ∈ {1/200, 1/400, 1/800, 1/1600, 1/3200, 1/6400} and fixed time step Δt (e.g. 0.01) respecting λΔt/Δx<1

## Outputs

- HLL numerical flux vector F_HLL at each cell interface xi±1/2
- Updated primitive variables ua(xi,tn+1) via the PVRS Eq 13 update integrated with MoL-RK4
- Per-resolution L1-norm errors and convergence orders (e.g. written to gaussian_convergence.csv for the static Gaussian test)

## How to apply

Encode the conservative charges q=(D,Sx,τ) and fluxes f as explicit functions of the primitive variables u=(p,v,n) (c=1, polytropic EOS e=nm+p/(κ−1), Eqs 2-5), then obtain the cell-interface left/right states by minmod-limited piecewise-linear reconstruction of ũ (Eqs 50-57). At each interface xi±1/2 compute the HLL numerical flux f^HLL=(λR fL−λL fR+λR λL(qR−qL))/(λR−λL) (Eq 45), selecting the upwind state via Eq 46 when the signal speeds λL, λR have the same sign. The signal speeds must satisfy the CFL restriction λΔt/Δx<1 (Eq 40) with a fixed Δt chosen sufficiently below the bound. Feed the flux difference [F_HLL]_{i+1/2}−[F_HLL]_{i−1/2} into the PVRS update ua(xi,tn+1)=ua−(Δt/Δx)Aab[Fb_HLL] with A=(∂q/∂u)^−1 evaluated at the cell midpoint (Eq 13), and wrap it as the right-hand side L(u) of ∂u/∂t=−L(u) for MoL-RK4 time integration. Keep the spatial term strictly in conservative finite-volume form — replacing it with a finite-difference stencil would violate entropy and jump conditions at shocks.

## Related tools

- **aztekas** (Solver implementing the PVRS discretisation with the HLL approximate Riemann solver, minmod limiter and MoL-RK4 integrator; the Shock-Tube simulation directory is compiled and run to exercise the flux computation) — https://github.com/aztekas-code/aztekas-main

## Examples

```
git clone https://github.com/aztekas-code/aztekas-main && cd aztekas-main/SIMULATIONS/HD/Shock-Tube && make && ./aztekas shock.param
```

## Evaluation signals

- Static Gaussian experiment with fixed Δt=0.01 reaches convergence order ≈2 (log(error_i/error_{i−1})/log(1/2)) at all six resolutions Δx=1/200 … 1/6400
- Relativistic Sod tests (weak, mildly relativistic, strong) show monotonically decreasing L1-norm errors with resolution and first-order convergence at least at one resolution per test
- CFL constraint λΔt/Δx<1 (Eq 40) holds at every step for the chosen fixed Δt
- Discontinuities remain sharply captured, confirming the spatial term stayed in conservative finite-volume (HLL flux-difference) form rather than finite differences
- HLL flux matches Eq 45 including the Eq 46 upwind-state selection when λL, λR share the same sign

## Limitations

- Shock-tube errors are dominated by discontinuities: only first-order convergence is attained at shocks, second order only on smooth profiles
- The fixed Δt=0.01 used for the Gaussian convergence study must still satisfy λΔt/Δx<1, which bounds the usable resolutions for a given wave speed
- The matrix A=(∂q/∂u)^−1 must be derived (e.g. symbolically) and evaluated at the cell midpoint, while reconstruction uses ũ; inconsistencies between these evaluation points degrade accuracy

## Evidence

- [methods] PVRS HLL flux definition: [evidence span withheld — anchor status: not_found]
- [abstract] conservative form rationale: "a traditional finite volume method for the flux is applied in order avoid violation of both, the entropy and "Rankine-Hugoniot" jump conditions"
- [intro] finite-difference/finite-volume combination: "we show how it is possible to construct a general numerical iteration method, using a combination of finite differences and finite volume integration techniques"
- [readme] aztekas HRSC scope: "aztekas is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes."
