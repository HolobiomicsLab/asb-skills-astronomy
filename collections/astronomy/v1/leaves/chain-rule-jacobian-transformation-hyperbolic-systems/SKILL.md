---
name: chain-rule-jacobian-transformation-hyperbolic-systems
description: Use when evolving a coupled hyperbolic conservative system ∂q/∂t + ∂f/∂x
  = 0 where inverting q→u analytically (or via a polynomial root-find per cell per
  step) is expensive, and a direct primitive-variable update is acceptable.
when_to_use_negative:
- When the analytic or cheap algebraic recovery u(q) is available at negligible cost,
  so the Jacobian/A=(∂q/∂u)^−1 machinery is unnecessary
- When the conservative-to-primitive Jacobian ∂q/∂u is not available symbolically
  or numerically for the chosen EOS/closure, since the whole update hinges on A=(∂q/∂u)^−1
- When enforcing strict conservation of the primitive variables themselves is required
  — the scheme conserves via the finite-volume flux term, not the primitive update
tools:
- name: aztekas
  role: Open-source code implementing the PVRS technique to solve relativistic and
    non-relativistic hydrodynamic Euler equations with HRSC finite-volume schemes;
    used to run the shock-tube simulations
  repo: github.com/aztekas-code/aztekas-main
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0195494
    title: 'A direct Primitive Variable Recovery Scheme for hyperbolic conservative
      equations: The case of relativistic hydrodynamics'
  source_content_hash: sha256:abfdbcf798da5f69f803bb16c35c3105889bfd9507bec72dbc975185b43d485b
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
yes_rate: 0.6
n_claims: 5
---

# chain-rule-jacobian-transformation-hyperbolic-systems

## Summary

A Primitive Variable Recovery Scheme (PVRS) that transforms conservative charges q into primitive variables u directly via the chain rule on the time term, using the Jacobian matrix A=(∂q/∂u)^−1 evaluated symbolically once, combined with a conservative finite-volume (HLL) spatial update. It avoids solving a costly algebraic recovery system (e.g. a 10th-degree polynomial in relativistic hydrodynamics) at every cell and time step.

## When to use

Use when evolving a coupled hyperbolic conservative system ∂q/∂t + ∂f/∂x = 0 where inverting q→u analytically (or via a polynomial root-find per cell per step) is expensive, and a direct primitive-variable update is acceptable.

## When NOT to use

- When the analytic or cheap algebraic recovery u(q) is available at negligible cost, so the Jacobian/A=(∂q/∂u)^−1 machinery is unnecessary
- When the conservative-to-primitive Jacobian ∂q/∂u is not available symbolically or numerically for the chosen EOS/closure, since the whole update hinges on A=(∂q/∂u)^−1
- When enforcing strict conservation of the primitive variables themselves is required — the scheme conserves via the finite-volume flux term, not the primitive update

## Inputs

- 1D system of hyperbolic conservative equations with charges q=(D,Sx,τ) and fluxes f written explicitly in terms of primitive variables u=(p,v,n), c=1 units
- Equation-of-state closure, e.g. polytropic e=nm+p/(κ−1) (Eq 5)
- Grid geometry (cell midpoints xi, interfaces xi±1/2), time step Δt satisfying λΔt/Δx<1 (Eq 40)

## Outputs

- Evolved primitive-variable vector u=(p,v,n) at each cell and time step (Eq 13 update)
- Semi-discrete right-hand side L(u) for MoL-RK4 integration (Eq 14)
- L1-norm errors against exact solutions and convergence order log(error_i/error_{i−1})/log(1/2)

## How to apply

Write the conservative charges q and fluxes f as explicit functions of the primitive variables u (e.g. SRHD q=(D,Sx,τ) in terms of u=(p,v,n) with c=1 and polytropic EOS e=nm+p/(κ−1)), then compute the Jacobians ∂q/∂u and ∂f/∂u once, symbolically with a CAS, and invert to obtain A=(∂q/∂u)^−1. Reconstruct the primitive variables piecewise-linearly at interfaces xi±1/2 with a minmod limiter on the cell slopes, and compute the HLL numerical flux f^HLL=(λR fL−λL fR+λR λL(qR−qL))/(λR−λL) with the interface upwind-state selection. Evaluate A on the cell midpoint xi while all reconstructed matrices/vectors (Mab, Aab, fa, qa) use the interface reconstruction ũ. Assemble the PVRS update ua(xi,tn+1)=ua(xi,tn)−(Δt/Δx)Aab([F_HLL]_{i+1/2}^n−[F_HLL]_{i−1/2}^n), keeping the spatial term in conservative finite-volume form so entropy and Rankine–Hugoniot jump conditions are not violated. Wrap the update as L(u) of ∂u/∂t=−L(u), integrate with 4th-order Runge–Kutta Method of Lines, and enforce a CFL-type restriction λΔt/Δx<1 with a fixed Δt below that bound.

## Related tools

- **aztekas** (Open-source code implementing the PVRS technique to solve relativistic and non-relativistic hydrodynamic Euler equations with HRSC finite-volume schemes; used to run the shock-tube simulations) — github.com/aztekas-code/aztekas-main

## Examples

```
git clone https://github.com/aztekas-code/aztekas-main; cd SIMULATIONS/HD/Shock-Tube; make; ./aztekas shock.param
```

## Evaluation signals

- First-order consistency check on a smooth static profile: update must be first-order accurate, with a smooth Gaussian test reaching convergence order ≈2 at fixed Δt=0.01
- L1-norm error against exact analytic solutions must decrease as resolution doubles, with order estimated as log(error_i/error_{i−1})/log(1/2)
- Convergence demonstrated on standard 1D special-relativistic shock-tube problems against exact solutions
- Time step must satisfy the CFL-type bound λΔt/Δx<1 throughout; violations signal instability or a wrong characteristic-speed estimate

## Limitations

- In relativistic hydrodynamics the standard recovery route requires solving a 10th-degree algebraic polynomial per component per time step, which is what PVRS avoids — but the alternative requires symbolic availability of A=(∂q/∂u)^−1
- The spatial term must remain in conservative finite-volume (HLL) form; applying the chain-rule update non-conservatively risks violating entropy and Rankine–Hugoniot jump conditions
- Δt must be fixed below the CFL bound λΔt/Δx<1; the method is demonstrated in 1D (1DRHD) and requires no analytic knowledge of u(q), but the primitive update itself is not conservative

## Evidence

- [abstract] This method obtains directly the primitive variables applying the chain rule to the time term of the conservative equations: "This method obtains directly the primitive variables applying the chain rule to the time term of the conservative equations"
- [abstract] This numerical technique evades the recovery of the primitive vector by solving an algebraic system of equations as it is often used: "This numerical technique evades the recovery of the primitive vector by solving an algebraic system of equations as it is often used and so, it generalises standard techniques to solve these kind of"
- [intro] at each time step, a 10th degree algebraic polynomial has to be solved for a unique given value of each component of the vector u: "at each time step, a 10th degree algebraic polynomial has to be solved for a unique given value of each component of the vector u"
- [readme] _aztekas_ is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes.: "_aztekas_ is a program that solves hyperbolic partial differential equations in conservative form using High Resolution Shock-Capturing (HRSC) schemes."
