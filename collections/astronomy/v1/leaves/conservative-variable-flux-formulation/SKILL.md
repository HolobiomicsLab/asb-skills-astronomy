---
name: conservative-variable-flux-formulation
description: Use when solving hyperbolic conservative systems (e.g. 1D SRHD) with
  shock-capturing schemes, where the spatial flux term must remain in conservative
  form while variables are evolved in primitive form.
when_to_use_negative:
- The system is not hyperbolic or cannot be written in conservative form ∂q/∂t+∂f/∂x=S
  (e.g. purely non-conservative or source-dominated formulations), since the HLL jump
  structure and Rankine-Hugoniot argument depend on it
- The problem has no discontinuities and shock-capturing/conservative fluxes are unnecessary,
  e.g. a smooth static consistency check alone
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: aztekas
  role: HRSC finite-volume code implementing this conservative flux formulation for
    relativistic hydrodynamics; clone and compile the Shock-Tube simulation to run
    the scheme
  repo: https://github.com/aztekas-code/aztekas-main
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0195494
    title: 'A direct Primitive Variable Recovery Scheme for hyperbolic conservative
      equations: The case of relativistic hydrodynamics'
  source_content_hash: sha256:7f4b30237b3e28ef145f20722bc098ecff172f9f2971042bbb12d70985fae699
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aztekas-code/aztekas-main
  tools:
  - aztekas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# conservative-variable-flux-formulation

## Summary

Formulate a hyperbolic PDE system as conservative charges q=(D,Sx,τ) and fluxes f as explicit functions of primitive variables u=(p,v,n), and keep the spatial discretization in conservative finite-volume form (HLL flux) so entropy and Rankine-Hugoniot jump conditions are not violated.

## When to use

Use when solving hyperbolic conservative systems (e.g. 1D SRHD) with shock-capturing schemes, where the spatial flux term must remain in conservative form while variables are evolved in primitive form.

## When NOT to use

- The system is not hyperbolic or cannot be written in conservative form ∂q/∂t+∂f/∂x=S (e.g. purely non-conservative or source-dominated formulations), since the HLL jump structure and Rankine-Hugoniot argument depend on it
- The problem has no discontinuities and shock-capturing/conservative fluxes are unnecessary, e.g. a smooth static consistency check alone

## Inputs

- 1D hyperbolic conservative system definition (charges q and fluxes f as functions of primitives u)
- Equation-of-state closure, e.g. polytropic e=nm+p/(κ-1)
- Cell-centered grid with spacing Δx and time step Δt satisfying λΔt/Δx<1

## Outputs

- Time-evolved primitive variable fields u=(p,v,n) per cell
- Conservative HLL numerical fluxes f^HLL at cell interfaces xi±1/2
- L1-norm errors against exact solutions and estimated convergence order log(error_i/error_{i-1})/log(1/2)

## How to apply

Encode the conservative charges q=(D,Sx,τ) and fluxes f as explicit functions of the primitive variables u=(p,v,n) with c=1 units (Eqs 2-4), closed by the polytropic EOS e=nm+p/(κ-1) (Eq 5). Compute symbolically the Jacobians ∂q/∂u and ∂f/∂u and the matrix A=(∂q/∂u)^-1 (evaluated on cell midpoints xi, while reconstruction-based quantities use ũ, Eq 9). Assemble the numerical flux in conservative form via the HLL formula f^HLL=(λR fL−λL fR+λR λL(qR−qL))/(λR−λL) (Eq 45) with upwind interface state selection (Eq 46), and advance primitives as ua(xi,tn+1)=ua(xi,tn)−(Δt/Δx)Aab([Fb_HLL]_{i+1/2}−[Fb_HLL]_{i−1/2}) (Eq 13). Rationale: writing fluxes in conservative finite-volume form is what preserves entropy and Rankine-Hugoniot jump conditions across shocks, while the A-matrix times the flux difference recovers primitives directly without solving u(q) algebraically. Wrap the update as L(u) of ∂u/∂t=−L(u) (Eq 14) and integrate with MoL-RK4 under a CFL restriction λΔt/Δx<1 (Eq 40).

## Related tools

- **aztekas** (HRSC finite-volume code implementing this conservative flux formulation for relativistic hydrodynamics; clone and compile the Shock-Tube simulation to run the scheme) — https://github.com/aztekas-code/aztekas-main

## Examples

```
git clone https://github.com/aztekas-code/aztekas-main; cd aztekas-main/SIMULATIONS/HD/Shock-Tube; make; ./aztekas shock.param
```

## Evaluation signals

- Run a smooth Gaussian profile at fixed Δt=0.01 and confirm convergence order of about 2
- Verify the L1-norm error decreases monotonically as resolution doubles (order = log(error_i/error_{i-1})/log(1/2))
- Check the time step respects λΔt/Δx<1 (Eq 40) throughout the run
- Compare shock-tube results against exact analytic solutions using the L1-norm data in S1_File.tar.gz
- Confirm no violation of entropy/Rankine-Hugoniot jump conditions across discontinuities, which conservative flux form guarantees

## Limitations

- Standard FV recovery of u(q) for relativistic hydrodynamics requires solving a 10th degree algebraic polynomial per component at each time step; the PVRS/A-matrix route avoids this but requires symbolic Jacobians ∂q/∂u and ∂f/∂u
- Time step must stay below the CFL bound λΔt/Δx<1; the paper uses a fixed Δt smaller than this bound
- aztekas solves equations on a fixed background metric (Schwarzschild, Minkowski, Kerr-Schild), not dynamical spacetimes

## Evidence

- [abstract] Traditional finite volume method used for the flux to avoid violating entropy and Rankine-Hugoniot jump conditions: "a traditional finite volume method for the flux is applied in order avoid violation of both, the entropy and "Rankine-Hugoniot" jump conditions"
- [intro] Method requires no analytical knowledge of u(q): "This technique is so general that requires no analytical knowledge whatsoever of u(q). The method developed is general and valid to any set of coupled conservative equations."
- [readme] aztekas conservative finite-volume discretization: "aztekas uses a conservative finite-volume approach to obtain the dicrete form of a hyperbolic partial differential system of equations (PDE)."
- [intro] Cost of conservative-variable recovery in RHD: "at each time step, a 10th degree algebraic polynomial has to be solved for a unique given value of each component of the vector u"
